<properties
    pageTitle="スケールアウトされたクラウド データベース全体をレポートする | Microsoft Azure"
    description="行方向のパーティション分割でエラスティック クエリを設定する方法"    
    services="sql-database"
    documentationCenter=""  
    manager="jhubbard"
    authors="torsteng"/>

<tags
    ms.service="sql-database"
    ms.workload="sql-database"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="05/27/2016"
    ms.author="torsteng" />

# スケールアウトされたクラウド データベース全体をレポートする (プレビュー)

![シャード間のクエリ][1]

シャード化されたデータベースは、スケールアウトされたデータ層の全体に行を分散させます。スキーマは、すべての参加データベース上で同じで、行方向のパーティション分割とも呼ばれます。エラスティック クエリを使用すると、シャード化されたデータベース内のすべてのデータベースにまたがるレポートを作成できます。

クイック スタートについては、「[スケールアウトされたクラウド データベース全体をレポートする](sql-database-elastic-query-getting-started.md)」を参照してください。

シャード化されていないデータベースについては、「[Query across cloud databases with different schemas (スキーマが異なるクラウド データベース間のクエリ)](sql-database-elastic-query-vertical-partitioning.md)」をご覧ください。

 
## 前提条件

* エラスティック データベース クライアント ライブラリを使用して、シャード マップを作成します。「[シャード マップの管理](sql-database-elastic-scale-shard-map-management.md)」を参照してください。または、「[エラスティック データベース ツールの概要](sql-database-elastic-scale-get-started.md)」のサンプル アプリを使用します。
* あるいは、「[既存のデータベースをスケールアウトされたデータベースに移行する](sql-database-elastic-convert-to-use-elastic-tools.md)」を参照してください。
* ユーザーは、ALTER ANY EXTERNAL DATA SOURCE アクセス許可を所有している必要があります。このアクセス許可は、ALTER DATABASE アクセス許可に含まれています。
* ALTER ANY EXTERNAL DATA SOURCE アクセス許可は、基になるデータ ソースを参照するために必要です。

## 概要

これらのステートメントを使うと、エラスティック クエリ データベース内のシャーディングされたデータのメタデータ表現を作成できます。


1. [CREATE MASTER KEY](https://msdn.microsoft.com/library/ms174382.aspx)
2. [CREATE DATABASE SCOPED CREDENTIAL](https://msdn.microsoft.com/library/mt270260.aspx)
3. [CREATE EXTERNAL DATA SOURCE](https://msdn.microsoft.com/library/dn935022.aspx)
4. [CREATE EXTERNAL TABLE](https://msdn.microsoft.com/library/dn935021.aspx) 

## 1\.1 データベース スコープのマスター キーと資格情報の作成 

この資格情報は、リモート データベースに接続するために、エラスティック クエリによって使用されます。

    CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'password';
    CREATE DATABASE SCOPED CREDENTIAL <credential_name>  WITH IDENTITY = '<username>',  
    SECRET = '<password>'
    [;]
 
**注**: *"\<username\>"* にサフィックス *"@servername"* が含まれていないことを確認してください。

## 1\.2 外部データ ソースの作成

構文:

	<External_Data_Source> ::=    
	CREATE EXTERNAL DATA SOURCE <data_source_name> WITH                               	           
			(TYPE = SHARD_MAP_MANAGER,
                   	LOCATION = '<fully_qualified_server_name>',
			DATABASE_NAME = ‘<shardmap_database_name>',
			CREDENTIAL = <credential_name>, 
			SHARD_MAP_NAME = ‘<shardmapname>’ 
                   ) [;] 

### 例 

	CREATE EXTERNAL DATA SOURCE MyExtSrc 
	WITH 
	( 
		TYPE=SHARD_MAP_MANAGER,
		LOCATION='myserver.database.windows.net', 
		DATABASE_NAME='ShardMapDatabase', 
		CREDENTIAL= SMMUser, 
		SHARD_MAP_NAME='ShardMap' 
	);
 
現在の外部データ ソースの一覧を取得します。

	select * from sys.external_data_sources; 

外部データ ソースは、シャード マップを参照します。エラスティック クエリは、外部データ ソースと基になるシャード マップを使用して、データ層にあるデータベースを列挙します。シャード マップを読み取る場合と、エラスティック クエリの処理中にシャード上のデータにアクセスする場合は、同じ資格情報が使用されます。

## 1\.3 外部テーブルの作成 
 
構文:

	CREATE EXTERNAL TABLE [ database_name . [ schema_name ] . | schema_name. ] table_name  
        ( { <column_definition> } [ ,...n ])     
	    { WITH ( <sharded_external_table_options> ) }
	) [;]  
	
	<sharded_external_table_options> ::= 
      DATA_SOURCE = <External_Data_Source>,       
	  [ SCHEMA_NAME = N'nonescaped_schema_name',] 
      [ OBJECT_NAME = N'nonescaped_object_name',] 
      DISTRIBUTION = SHARDED(<sharding_column_name>) | REPLICATED |ROUND_ROBIN

**例**

	CREATE EXTERNAL TABLE [dbo].[order_line]( 
		 [ol_o_id] int NOT NULL, 
		 [ol_d_id] tinyint NOT NULL,
		 [ol_w_id] int NOT NULL, 
		 [ol_number] tinyint NOT NULL, 
		 [ol_i_id] int NOT NULL, 
		 [ol_delivery_d] datetime NOT NULL, 
		 [ol_amount] smallmoney NOT NULL, 
		 [ol_supply_w_id] int NOT NULL, 
		 [ol_quantity] smallint NOT NULL, 
		 [ol_dist_info] char(24) NOT NULL 
	) 
	
	WITH 
	( 
		DATA_SOURCE = MyExtSrc, 
	 	SCHEMA_NAME = 'orders', 
	 	OBJECT_NAME = 'order_details', 
		DISTRIBUTION=SHARDED(ol_w_id)
	); 

外部テーブルの一覧を現在のデータベースから取得します。

	SELECT * from sys.external_tables; 

外部テーブルを削除するには

	DROP EXTERNAL TABLE [ database_name . [ schema_name ] . | schema_name. ] table_name[;]

### 解説

DATA\_SOURCE 句は、外部テーブルに使用される外部データ ソース (シャード マップ) を定義します。

SCHEMA\_NAME 句と OBJECT\_NAME 句は、外部テーブルの定義を別のスキーマ内のテーブルにマップします。これらを省略した場合、リモート オブジェクトのスキーマは "dbo" と見なされ、その名前は定義されている外部テーブルの名前と同一であると見なされます。これは、リモート テーブルの名前が、外部テーブルを作成するデータベースで既に取得されている場合に便利です。たとえば、スケールアウトされたデータ層のカタログ ビューまたは DMV の集計ビューを取得する外部テーブルを定義する場合が挙げられます。カタログ ビューと DMV は既にローカルに存在するため、外部テーブルの定義にその名前を使うことはできません。代わりに、別の名前を使用して、カタログ ビューまたは DMV の名前を SCHEMA\_NAME 句または OBJECT\_NAME 句で使用します (次の例を参照してください)。

DISTRIBUTION 句は、このテーブルに使用するデータ分散を指定します。クエリ プロセッサは、DISTRIBUTION 句で提供される情報を使用して、最も効率的なクエリ プランを作成します。

1. **SHARDED** は、データがデータベース間で行方向にパーティション分割されることを意味します。データ分散のパーティション分割キーは、**<sharding_column_name>** パラメーターです。
2. **REPLICATED** は、テーブルの同一のコピーが各データベースに存在することを意味します。データベース間でレプリカが同じであることを自分で確認する必要があります。
3. **ROUND\_ROBIN** は、テーブルがアプリケーションに依存する分散方法を使用して、行方向にパーティション分割されることを意味します。 

**データ層参照**: 外部テーブル DDL は、外部データ ソースを参照します。外部データ ソースは、データ層のすべてのデータベースを見つけるために必要な情報を外部テーブルに提供するシャード マップを指定します。


### セキュリティに関する考慮事項 

外部テーブルへのアクセス権を持つユーザーは、外部データ ソース定義に指定された資格情報の下で、基になるリモート テーブルへのアクセス権を自動的に取得します。外部データ ソースの資格情報による不要な特権の昇格を防ぎます。外部テーブルに対して、通常のテーブルであるかのように GRANT または REVOKE を使用します。

外部データ ソースと外部テーブルを定義すると、外部テーブルに対して完全に T-SQL を使用できるようになります。

## 例: 行方向にパーティション分割されたデータベースのクエリ 

次のクエリでは、倉庫、注文、および注文明細行の間で 3 方向結合を実行し、いくつかの集計と選択的フィルターを使用します。ここでは、(1) 行方向のパーティション分割 (シャーディング) のほか、(2) 倉庫、注文、および注文明細行が倉庫の ID 列でシャード化されること、エラスティック クエリがシャード上の結合を併置できること、クエリの負荷の高い部分をシャード上で並列に処理できることを想定しています。

	select  
		 w_id as warehouse,
		 o_c_id as customer,
		 count(*) as cnt_orderline,
		 max(ol_quantity) as max_quantity,
		 avg(ol_amount) as avg_amount, 
		 min(ol_delivery_d) as min_deliv_date
	from warehouse 
	join orders 
	on w_id = o_w_id
	join order_line 
	on o_id = ol_o_id and o_w_id = ol_w_id 
	where w_id > 100 and w_id < 200 
	group by w_id, o_c_id 
 
## T-SQL リモート実行のストアド プロシージャ: sp\_execute\_remote

エラスティック クエリには、シャードへの直接アクセスを提供するストアド プロシージャも導入されています。このストアド プロシージャは [sp\_execute\_remote](https://msdn.microsoft.com/library/mt703714) と呼ばれ、リモート データベースでリモート ストアド プロシージャまたは T-SQL コードを実行するときに使用できます。使用できるパラメーターは次のとおりです。

* データ ソース名 (nvarchar): RDBMS 型の外部データ ソースの名前。 
* クエリ (nvarchar): 各シャードで実行する T-SQL クエリ。 
* パラメーター宣言 (nvarchar) (省略可能): (sp\_executesql などの) クエリ パラメーターで使用される、パラメーターのデータ型定義を含む文字列。 
* パラメーター値のリスト (省略可能): (sp\_executesql などの) パラメーター値のコンマ区切りリスト。

sp\_execute\_remote では、起動パラメーターで指定された外部データ ソースを使用して、指定された T-SQL ステートメントをリモート データベースで実行します。shardmap マネージャー データベースとリモート データベースへの接続には、外部データ ソースの資格情報を使用します。

例:

	EXEC sp_execute_remote
		N'MyExtSrc',
		N'select count(w_id) as foo from warehouse' 

## ツールの接続性  

通常の SQL Server 接続文字列を使用して、アプリケーション、BI、およびデータ統合ツールを、外部テーブル定義を持つデータベースに接続できます。使用しているツールのデータ ソースとして SQL Server がサポートされていることを確認してください。次に、ツールに接続される他の SQL Server データベースと同様にエラスティック クエリ データベースを参照して、外部テーブルをローカル テーブルであるかのようにツールまたはアプリケーションから使用します。

## ベスト プラクティス 

* エラスティック クエリ エンドポイント データベースに、SQL データベース ファイアウォール経由でのシャード マップ データベースとすべてのシャードへのアクセスが許可されていることを確認します。  

* 外部テーブルで定義されたデータ分散を検証または適用しません。実際のデータ分散がテーブル定義に指定されたデータ分散と異なる場合、クエリが予期しない結果を生成する場合があります。

* シャーディング キーによる述語で特定のシャードを処理から安全に除外できる場合、エラスティック クエリでは、現在のところ、シャードの除去を実行しません。

* エラスティック クエリは、計算の大部分をシャード上で実行できるクエリに最適です。通常、最適なクエリ パフォーマンスが得られるのは、シャード上で評価可能な選択的なフィルター述語を使用した場合、またはすべてのシャード上でパーティション分割方法により実行可能な、パーティション分割キーによる結合を使用した場合となります。その他のクエリ パターンでは、シャードからヘッド ノードに大量のデータを読み込むことが必要になる場合があり、パフォーマンスが低下する可能性があります。

[AZURE.INCLUDE [elastic-scale-include](../../includes/elastic-scale-include.md)]

<!--Image references-->
[1]: ./media/sql-database-elastic-query-horizontal-partitioning/horizontalpartitioning.png
<!--anchors-->

<!---HONumber=AcomDC_0601_2016-->
