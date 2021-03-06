<properties 
	pageTitle="SQL Database の XEvent リング バッファー コード | Microsoft Azure" 
	description="Azure SQL Database で、リング バッファー ターゲットの使用によって簡素化された TRANSACT-SQL のコード サンプルを提供します。" 
	services="sql-database" 
	documentationCenter="" 
	authors="MightyPen" 
	manager="jeffreyg" 
	editor="" 
	tags=""/>


<tags 
	ms.service="sql-database" 
	ms.workload="data-management" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="12/30/2015" 
	ms.author="genemi"/>


# SQL Database での拡張イベント向けリング バッファー ターゲット コード


テスト中の拡張イベントに関する情報を、最も簡単な方法で取得およびレポートするための完全なコード サンプルが必要です。拡張イベントのデータの最も簡単なターゲットは、[リング バッファー ターゲット](http://msdn.microsoft.com/library/ff878182.aspx)です。


このトピックでは、以下を実行する TRANSACT-SQL コードのサンプルについて説明します。


1. 表示させるデータでテーブルを作成する。

2. 既存の拡張イベントのセッション (つまり **sqlserver.sql\_statement\_starting**) を作成する。
 - イベントは、次の特定の Update 文字列を含む SQL ステートメントに限定される: **statement LIKE ’%UPDATE tabEmployee%’**。
 - イベントの出力をリング バッファー タイプのターゲット (つまり **package0.ring\_buffer**) に送信するよう選択できる。

3. イベント セッションを開始する。

4. 単純な SQL UPDATE ステートメントをいくつか発行する。

5. リング バッファーからイベント出力を取得する SQL SELECT を発行する。
 - **sys.dm\_xe\_database\_session\_targets** と他の動的管理ビュー (DMV) を結合する。

6. イベント セッションを停止する。

7. リング バッファー ターゲットを削除して、そのリソースを解放する。

8. イベント セッションとデモ テーブルを削除する。


## 前提条件


- Azure アカウントとサブスクリプション。[無料試用版](https://azure.microsoft.com/pricing/free-trial/)にサインアップできます。


- テーブルを作成できるデータベース。
 - 必要に応じて、数分で [**AdventureWorksLT** デモ データベースを作成できる](sql-database-get-started.md)。


- SQL Server Management Studio (ssms.exe)、2015 年 8 月のプレビューまたはそれ以降のバージョン。最新の ssms.exe をダウンロードすることができる。
 - [トピック内のリンク。](http://msdn.microsoft.com/library/mt238290.aspx)
 - [ダウンロードへの直接リンク。](http://go.microsoft.com/fwlink/?linkid=616025)
 - ssms.exe を定期的に (できれば毎月) 更新することをお勧めします。


## サンプル コード


わずかな変更を加えると、以下のリング バッファーのコード サンプルを、Azure SQL Database または Microsoft SQL Server のいずれかで実行できます。異なる点は、手順 5. の FROM 句に含まれるいくつかの動的管理ビュー (DMV) の名前の中に「\_database」というノード名があることです。次に例を示します。

- sys.dm\_xe**\_database**\_session\_targets
- sys.dm\_xe\_session\_targets


&nbsp;


```
GO
----  Transact-SQL.
---- Step set 1.

SET NOCOUNT ON;
GO


IF EXISTS
	(SELECT * FROM sys.objects
		WHERE type = 'U' and name = 'tabEmployee')
BEGIN
	DROP TABLE tabEmployee;
END
GO


CREATE TABLE tabEmployee
(
	EmployeeGuid         uniqueIdentifier   not null  default newid()  primary key,
	EmployeeId           int                not null  identity(1,1),
	EmployeeKudosCount   int                not null  default 0,
	EmployeeDescr        nvarchar(256)          null
);
GO


INSERT INTO tabEmployee ( EmployeeDescr )
	VALUES ( 'Jane Doe' );
GO

---- Step set 2.


IF EXISTS
	(SELECT * from sys.database_event_sessions
		WHERE name = 'eventsession_gm_azuresqldb51')
BEGIN
	DROP EVENT SESSION eventsession_gm_azuresqldb51
		ON DATABASE;
END
GO


CREATE
	EVENT SESSION eventsession_gm_azuresqldb51
	ON DATABASE
	ADD EVENT
		sqlserver.sql_statement_starting
			(
			ACTION (sqlserver.sql_text)
			WHERE statement LIKE '%UPDATE tabEmployee%'
			)
	ADD TARGET
		package0.ring_buffer
			(SET
				max_memory = 500   -- Units of KB.
			);
GO

---- Step set 3.


ALTER EVENT SESSION eventsession_gm_azuresqldb51
	ON DATABASE
	STATE = START;
GO

---- Step set 4.


SELECT 'BEFORE_Updates', EmployeeKudosCount, * FROM tabEmployee;

UPDATE tabEmployee
	SET EmployeeKudosCount = EmployeeKudosCount + 102;

UPDATE tabEmployee
	SET EmployeeKudosCount = EmployeeKudosCount + 1015;

SELECT 'AFTER__Updates', EmployeeKudosCount, * FROM tabEmployee;
GO

---- Step set 5.


SELECT
    se.name                      AS [session-name],
    ev.event_name,
    ac.action_name,
    st.target_name,
    se.session_source,
    st.target_data,
    CAST(st.target_data AS XML)  AS [target_data_XML]
FROM
               sys.dm_xe_database_session_event_actions  AS ac

    INNER JOIN sys.dm_xe_database_session_events         AS ev  ON ev.event_name = ac.event_name
        AND CAST(ev.event_session_address AS BINARY(8)) = CAST(ac.event_session_address AS BINARY(8))

    INNER JOIN sys.dm_xe_database_session_object_columns AS oc
         ON CAST(oc.event_session_address AS BINARY(8)) = CAST(ac.event_session_address AS BINARY(8))

    INNER JOIN sys.dm_xe_database_session_targets        AS st
         ON CAST(st.event_session_address AS BINARY(8)) = CAST(ac.event_session_address AS BINARY(8))

    INNER JOIN sys.dm_xe_database_sessions               AS se
         ON CAST(ac.event_session_address AS BINARY(8)) = CAST(se.address AS BINARY(8))
WHERE
        oc.column_name = 'occurrence_number'
    AND
        se.name        = 'eventsession_gm_azuresqldb51'
    AND
        ac.action_name = 'sql_text'
ORDER BY
    se.name,
    ev.event_name,
    ac.action_name,
    st.target_name,
    se.session_source
;
GO

---- Step set 6.


ALTER EVENT SESSION eventsession_gm_azuresqldb51
	ON DATABASE
	STATE = STOP;
GO

---- Step set 7.


ALTER EVENT SESSION eventsession_gm_azuresqldb51
	ON DATABASE
	DROP TARGET package0.ring_buffer;
GO

---- Step set 8.


DROP EVENT SESSION eventsession_gm_azuresqldb51
	ON DATABASE;
GO

DROP TABLE tabEmployee;
GO
```


&nbsp;


## リング バッファーの内容


ここでは、ssms.exe を使用してコード サンプルを実行しました。


結果を表示するために、列ヘッダー **target\_data\_XML** の下のセルをクリックしました。

結果ウィンドウで、列ヘッダー **target\_data\_XML** の下のセルをクリックしました。これにより、ssms.exe にもう 1 つのファイル タブが作成され、結果のセルの内容が XML として表示されました。


出力は、次のブロックに示されています。ここに含まれるのは、2 つの**<event>**要素です。


&nbsp;


```
<RingBufferTarget truncated="0" processingTime="0" totalEventsProcessed="2" eventCount="2" droppedCount="0" memoryUsed="1728">
  <event name="sql_statement_starting" package="sqlserver" timestamp="2015-09-22T15:29:31.317Z">
    <data name="state">
      <type name="statement_starting_state" package="sqlserver" />
      <value>0</value>
      <text>Normal</text>
    </data>
    <data name="line_number">
      <type name="int32" package="package0" />
      <value>7</value>
    </data>
    <data name="offset">
      <type name="int32" package="package0" />
      <value>184</value>
    </data>
    <data name="offset_end">
      <type name="int32" package="package0" />
      <value>328</value>
    </data>
    <data name="statement">
      <type name="unicode_string" package="package0" />
      <value>UPDATE tabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 102</value>
    </data>
    <action name="sql_text" package="sqlserver">
      <type name="unicode_string" package="package0" />
      <value>
---- Step set 4.


SELECT 'BEFORE_Updates', EmployeeKudosCount, * FROM tabEmployee;

UPDATE tabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 102;

UPDATE tabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 1015;

SELECT 'AFTER__Updates', EmployeeKudosCount, * FROM tabEmployee;
</value>
    </action>
  </event>
  <event name="sql_statement_starting" package="sqlserver" timestamp="2015-09-22T15:29:31.327Z">
    <data name="state">
      <type name="statement_starting_state" package="sqlserver" />
      <value>0</value>
      <text>Normal</text>
    </data>
    <data name="line_number">
      <type name="int32" package="package0" />
      <value>10</value>
    </data>
    <data name="offset">
      <type name="int32" package="package0" />
      <value>340</value>
    </data>
    <data name="offset_end">
      <type name="int32" package="package0" />
      <value>486</value>
    </data>
    <data name="statement">
      <type name="unicode_string" package="package0" />
      <value>UPDATE tabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 1015</value>
    </data>
    <action name="sql_text" package="sqlserver">
      <type name="unicode_string" package="package0" />
      <value>
---- Step set 4.


SELECT 'BEFORE_Updates', EmployeeKudosCount, * FROM tabEmployee;

UPDATE tabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 102;

UPDATE tabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 1015;

SELECT 'AFTER__Updates', EmployeeKudosCount, * FROM tabEmployee;
</value>
    </action>
  </event>
</RingBufferTarget>
```


#### リング バッファーが保持するリソースの解放


リング バッファーでの作業が終わったら、リング バッファーを削除して、次のように **ALTER** リソースを発行することでリソースを解放できます。


```
ALTER EVENT SESSION eventsession_gm_azuresqldb51
	ON DATABASE
	DROP TARGET package0.ring_buffer;
GO
```


イベント セッションの定義は更新されますが、削除はされません。後で、イベント セッションにリング バッファーの別のインスタンスを追加できます。


```
ALTER EVENT SESSION eventsession_gm_azuresqldb51
	ON DATABASE
	ADD TARGET
		package0.ring_buffer
			(SET
				max_memory = 500   -- Units of KB.
			);
```


## 詳細情報


Azure SQL Database での拡張イベントに関する主なトピックは次のとおりです。


- [Extended event considerations in SQL Database (SQL Database での拡張イベントに関する考慮事項)](sql-database-xevent-db-diff-from-svr.md)。この記事では、Microsoft SQL Server と Azure SQL Database の間で異なる拡張イベントの一部の機能を比較します。


拡張イベントのその他のコード サンプルのトピックは、次のリンクから参照できます。ただし、対象が Azure SQL Database または Microsoft SQL Server のどちらかを確認するために、サンプルを定期的にチェックする必要があります。これにより、変更がサンプル実行に十分であるかを判断できます。


- Azure SQL Database のコード サンプル: [Event File target code for extended events in SQL Database (SQL Database での拡張イベントのイベント ファイル ターゲット コード)](sql-database-xevent-code-event-file.md)


<!--
('lock_acquired' event.)

- Code sample for SQL Server: [Determine Which Queries Are Holding Locks](http://msdn.microsoft.com/library/bb677357.aspx)
- Code sample for SQL Server: [Find the Objects That Have the Most Locks Taken on Them](http://msdn.microsoft.com/library/bb630355.aspx)
-->

<!---HONumber=AcomDC_0128_2016-->