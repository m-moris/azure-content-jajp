<properties
   pageTitle="Azure SQL Data Warehouse の概要 | Microsoft Azure"
   description="エンタープライズ クラスの分散データベースであり、ペタバイト単位の量までリレーショナル データと非リレーショナル データを処理できます。業界初のクラウド データ ウェアハウスであり、数秒で拡大、縮小、および一時停止できます。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="lodipalm"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.devlang="NA"
   ms.topic="get-started-article"
   ms.tgt_pltfrm="NA"
   ms.workload="data-services"
   ms.date="03/03/2016"
   ms.author="lodipalm;barbkess;mausher;jrj;sonyama;"/>


# Azure SQL Data Warehouse の概要

Azure SQL Data Warehouse はエンタープライズ クラスの分散データベースであり、大規模なリレーショナル データと非リレーショナル データを処理できます。実証済みの SQL の機能と、数秒で拡大、縮小、および一時停止できる機能を組み合わせた業界初のクラウド データ ウェアハウスです。また、SQL Data Warehouse は Azure に深く根差しており、数秒で簡単にデプロイできます。さらに、サービスは完全に管理されているため、ソフトウェア修正プログラムの適用、保守、およびバックアップに関する面倒な問題を取り除くことができます。SQL Data Warehouse の組み込みの自動バックアップでは、フォールト トレランスとセルフ サービス復元をサポートしています。

SQL Data Warehouse の作成時には、Azure をフル活用して、どんな企業のワークロードにも対応可能なデータ ウェアハウスを作成できるように、いくつかの鍵となる属性に重点を置きました。

## 最適化

### データ ウェアハウスのアーキテクチャ

根本的に、SQL Data Warehouse では、最大規模のオンプレミス エンタープライズ データ ウェアハウスを実行することを本来の目的として設計された Microsoft の大量並列処理 (MPP) アーキテクチャを使用しています。このアーキテクチャでは、組み込みのデータ ウェアハウスのパフォーマンスの向上を利用し、SQL Data Warehouse で複雑な SQL クエリのコンピューティングを容易にスケールアウトおよび並列化できるようにしています。さらに、SQL Data Warehouse のアーキテクチャは、Azure でそのプレゼンスを生かせるように設計されています。この 2 つの特徴を組み合わせると、アーキテクチャは次の 4 つの主なコンポーネントに分割されます。

![SQL Data Warehouse のアーキテクチャ][1]

- **制御ノード:** SQL Data Warehouse を任意の開発ツール、読み込みツール、またはビジネス インテリジェンス ツールと一緒に使用する場合は、制御ノードに接続します。SQL Data Warehouse の制御ノードは SQL Database です。接続時は、標準の SQL Database のような外観と操作性になります。ただし、システム内で発生するデータの移動とコンピューティングはすべてバックグラウンドで調整されます。コマンドが制御ノードに発行されると、コマンドは一連のクエリに分解され、サービスのコンピューティング ノードに渡されます。

- **コンピューティング ノード:** 制御ノードの場合と同様に、SQL Data Warehouse のコンピューティング ノードは、SQL Database を利用します。コンピューティング ノードの仕事は、サービスのコンピューティング機能としての役割を果たすことです。バックグラウンドでは、データが SQL Data Warehouse に読み込まれると常に、サービスの当該ノード間に分散されます。制御ノードはコマンドを受信すると常に、各コンピューティング ノード用に分割し、コンピューティング ノードは対応するデータを処理します。各コンピューティング ノードは計算を完了すると、担当部分の結果を制御ノードに渡します。制御ノードは結果を集計してから答えを返します。

- **ストレージ:** SQL Data Warehouse のストレージはすべて、標準の Azure Storage BLOB です。すなわち、コンピューティング ノードはデータ操作時に、BLOB との間で書き込みと読み取りを直接行います。透過的に、ほぼ無制限に拡張可能な Azure Storage の機能を使用すると、ストレージを自動的に拡大縮小することができます。これはコンピューティングとは別に行うことができます。また、Azure Storage では、スケーリングまたは一時停止の間もストレージを保持し、バックアップおよび復元のプロセスを合理化し、安全でよりフォールト トレラントなストレージを使用することができます。

- **データ移動サービス:** SQL Data Warehouse ですべてを結び付ける最後のピースは、データ移動サービスです。データ移動サービスにより、制御ノードは、すべてのコンピューティング ノードと通信してデータを渡すことができます。また、このデータ移動サービスにより、コンピューティング ノードは相互にデータを渡すことができ (他のコンピューティング ノードのデータにアクセス可能)、さらに、結合と集計を完了するのに必要なデータを取得することができます。

### エンジンの最適化

この MPP アプローチにより、SQL Data Warehouse は、大量データに関する問題を解決する際に上記のような分割および整理の手法を利用することができます。SQL Data Warehouse のデータは分割され、サービスのコンピューティング ノードに分散されるので、各コンピューティング ノードはデータの担当部分を並列に処理することができます。最後に、結果は制御ノードに渡され、そこで集計されてからユーザーに渡されます。この方法はまた、データ ウェアハウスに固有の複数のパフォーマンスの最適化によって支援されています。

- SQL Data Warehouse では、サービス上のすべてのデータにわたって高度なクエリ オプティマイザーと複雑な統計セットを使用して、クエリ プランを作成します。データ サイズと分散に関する情報を使用するサービスは、特定のクエリ操作のコストの評価結果に基づいて分散クエリを最適化することができます。

- 最適なクエリ プランを作成することに加えて、SQL Data Warehouse では必要に応じてコンピューティング リソース間でデータを効率的に移動し、クエリを実行する高度なアルゴリズムや手法もあります。これらの操作はデータ ウェアハウスのデータ移動サービスに組み込まれているので、最適化は自動的に行われます。

- クラスター化された列ストア インデックスを SQL Data Warehouse に含めることも、クエリ パフォーマンスの高速化を実現する上で重要です。列ベースのストレージを使用することで、SQL Data Warehouse では従来の行指向型ストレージと比較して最大で 5 倍の圧縮率を得ることができ、最大で 10 倍のクエリ パフォーマンスを得ることができます。データ ウェアハウスのクエリでは、テーブル全体をスキャンしたり、テーブルのすべてのパーティションをスキャンしたりすることがよくあり、クエリ段階でのデータ移動の影響を最小限に抑えるため、列ストア インデックスがうまく機能します。

## 拡張性

SQL Data Warehouse のアーキテクチャでは、それぞれを個別にスケーリングできる、ストレージとコンピューティングの分離を導入しています。SQL Database の迅速で簡単なデプロイメント構造により、追加のコンピューティングは一瞬のうちに有効になります。これを補完するには、Azure Storage BLOB を使用します。BLOB は、レプリケートされたストレージを安定して提供するだけでなく、低コストで簡単に展開を実現するためのインフラストラクチャも提供します。クラウド規模のストレージと Azure のコンピューティングを組み合わせて使用する SQL Data Warehouse では、必要なときに必要とするクエリのパフォーマンス ストレージに対して支払いを行うことができます。コンピューティング量の変更は、Azure クラシック ポータルでスライダーを左または右に移動するようにいとも簡単に行うことができますが、T-SQL および PowerShell を使用してコンピューティングをスケジュールしたりワークロードに追加したりすることもできます。

SQL Data Warehouse では、ストレージとは無関係にコンピューティングの量を完全に制御できるほか、データ ウェアハウスを完全に停止することができます。ストレージを適切に維持したまま、コンピューティングを Azure の主プールに解放することで、コストをすぐに節約できます。必要になったときに、コンピューティングを再開するだけで、ワークロードでデータとコンピューティングを利用できます。

SQL Data Warehouse 内のコンピューティングの使用状況は、SQL Data Warehouse ユニット (DWU) を使用して測定されます。DWU は、データ ウェアハウスが備えている基となる機能の尺度であり、標準的なパフォーマンスが任意の時点でウェアハウスに確実に関連付けられるように設計されています。具体的には、DWU を使用して次のことを保証しています。

- 基になるハードウェアまたはソフトウェアを気にせず、データ ウェアハウスを効率的にスケーリングすることができます。

- データ ウェアハウスのサイズを変更する前に、DWU レベルで表示されるパフォーマンスを把握することができます。

- インスタンスの基になるハードウェアとソフトウェアは、ワークロードのパフォーマンスに影響を与えることなく変更または移動することができます。

- ワークロードのパフォーマンスに影響を与えることなく、サービスの基になるアーキテクチャの調整を行うことができます。

- SQL Data Warehouse のパフォーマンスを迅速に向上させる場合、スケーラブルに、かつシステムに均等に影響が及ぶようにそれを行うことを保証できます。

### Data Warehouse ユニット

具体的には、データ ウェアハウスのワークロードのパフォーマンスとの関連性が高いことがわかっている 3 つの正確なメトリックの尺度として Data Warehouse ユニットを説明します。一般的な可用性について、このような重要なワークロード メトリックが、データ ウェアハウス用に選択した DWU に対して直線的にスケーリングすることを目標にしています。

**スキャン/集計:** このワークロード メトリックでは、大量の行をスキャンして複雑な集計を実行する標準的なデータ ウェアハウス クエリを使用します。これは IO と CPU を集中的に使用する操作です。

**読み込み:** このメトリックは、サービスにデータを取り込む機能を測定します。読み込みは、Azure Storage BLOB から代表的なデータセットを読み込む PolyBase を使用して実行されます。このメトリックは、サービスのネットワークおよび CPU 要素にストレスをかけるように設計されています。

**CREATE TABLE AS SELECT (CTAS):** CTAS はテーブルのコピーを作成する機能を測定します。　　この測定には、ストレージからのデータの読み取り、アプライアンスのノード間でのデータ分散、ストレージへのデータの再書き込みが必要です。CPU とネットワークを集中的に使用する操作です。

### スケーリングするタイミング

全体的に見て、DWU は単純である必要があります。より短時間で結果を得る必要がある場合は DWU を増やし、増加された DWU に対して支払いを行います。それほど高いコンピューティング能力が必要ではない場合は、DWU を減らし、小さな DWU に対して支払いを行います。DWU 数の変更について検討する可能性があるケースは次のとおりです。

- 夜間や週末などクエリを実行する必要がない場合は、コンピューティング リソースを一時停止して実行中のすべてのクエリをキャンセルし、データ ウェアハウスに割り当てられているすべての DWU を削除します。

- 大量データの読み込みまたは変換操作を実行する場合は、データが短時間で使用可能になるようにスケール アップすることが必要になる可能性があります。

- 理想的な DWU 値を理解するには、データ読み込みの後で、拡大/縮小を行い、いくつかのクエリを実行してください。スケーリングは簡単に行えるので、1 時間以上コミットしなくても、さまざまなレベルのパフォーマンスを試すことができます。

> [AZURE.NOTE] データ ボリュームが小さくなると、アーキテクチャまたは SQL Data Warehouse のせいで、予想通りのパフォーマンス スケーリングを達成できない場合があるので注意してください。正確なパフォーマンス テスト結果を取得するためには、1 TB 以上のデータ ボリュームで始めることをお勧めします。

## 統合

SQL Data Warehouse は SQL Server の実績のあるリレーショナル データベース エンジンに基づいており、エンタープライズ データ ウェアハウスに期待する多くの機能を備えています。Transact-SQL を既に知っている場合、その知識を SQL Data Warehouse に当てはめて理解するのは簡単です。上級者か初心者かに関係なく、ドキュメントの中で紹介している例は導入の際に役立ちます。全体的に、SQL Data Warehouse の言語要素の構成方法について、次のように考えることができます。

- SQL Data Warehouse は、多くの操作で SQL Server の TRANSACT-SQL (TSQL) 構文を使用し、従来の広範な SQL コンストラクト セット (ストアド プロシージャ、ユーザー定義関数、テーブルのパーティション分割、インデックス、および照合順序など) をサポートしています。

- SQL Data Warehouse には、クラスター化列ストア インデックス、PolyBase 統合、データ監査 (脅威の評価による) など、最先端の SQL Server 機能も複数搭載されています。

- SQL Data Warehouse はまだ開発中であるため、TSQL 言語要素の中でもデータ ウェアハウスのワークロードであまり一般的でないもの、または SQL Server にとって新しいものは、この時点で使用できない場合があります。これに関連する詳細については、移行に関するドキュメントを参照してください。

Transact-SQL と、SQL Server、SQL Data Warehouse、SQL Database、および Analytics Platform System 間の共通する機能を使用して、データのニーズに合ったソリューションを作成できます。パフォーマンス、セキュリティ、およびスケール要件に基づいて、データを保持する場所を決定し、必要に応じてデータをさまざまなシステム間で転送することができます。

SQL Data Warehouse は、SQL Server の TSQL サーフェイス領域を採用したことに加えて、SQL Server のユーザーが使い慣れている多くのツールと統合されています。具体的には、次のようないくつかのカテゴリのツールを SQL Data Warehouse と統合することに力を注ぎました。

**従来の SQL Server ツール:** SQL Server Analysis Services、Integration Services、Reporting services との完全な統合が、SQL Data Warehouse で使用できます。

**クラウド ベースのツール:** SQL Data Warehouse は、Azure の複数の新しいツールと共に使用できるほか、Azure's Data Factory、Stream Analytics、Machine Learning、および Power BI と緊密に統合されています。

**サードパーティ製のツール:** 多数のサード パーティ ツール プロバイダーが自社のツールと SQL Data Warehouse との統合を認定しています。完全な一覧を参照してください。

## ハイブリッド

SQL Data Warehouse と PolyBase を併用することで、ユーザーが従来とは異なる方法でエコシステム全体でデータを移動できるようにしました。これにより、非リレーショナルおよびオンプレミス データ ソースを使用して高度なハイブリッド シナリオをセットアップする機能はロック解除されます。

Polybase は簡単に使用でき、使い慣れている同じ T-SQL コマンドを使用してさまざまなソースのデータを活用できます。Polybase を使用すると、Azure BLOB Storage に保持されている非リレーショナル データに対して、通常のテーブルのようにクエリを実行することができます。Polybase を使用して非リレーショナル データをクエリしたり、非リレーショナル データを SQL Data Warehouse にインポートしたりします。

- PolyBase は外部テーブルを使用して非リレーショナル データにアクセスします。テーブルの定義は SQL Data Warehouse に格納され、通常のリレーショナル データにアクセスする場合と同様に SQL およびツールでアクセスすることができます。

- Polybase は統合時に依存性がありません。Polybase がサポートしているすべてのソースに同じ機能を公開します。Polybase によって読み取られるデータは、区切り記号で区切られたファイルや ORC ファイルなど、さまざまな形式を使用できます。

- PolyBase を使用すると、HD Insight クラスターのストレージとしても使用されている BLOB Storage にアクセスできます。これにより、リレーショナル ツールおよび非リレーショナル ツールを使用して同じデータに従来とは異なる方法でアクセスできます。


## 次のステップ

SQL Data Warehouse の概要について学習しましたので、次は、使用を開始するために必要な [データ ウェアハウスのワークロード]、[プロビジョニング]、および[サンプル データ]の読み込みについて理解してください。

>[AZURE.NOTE] 記事の改善にご協力ください。「この記事は役に立ちましたか?」という質問に「いいえ」を選択した場合は、不足しているものやどうすれば記事を改善できるかについてのご提案をお願いいたします。ご協力ありがとうございます。

<!--Image references-->
[1]: ./media/sql-data-warehouse-overview-what-is/dwarchitecture.png

<!--Article references-->
[データ ウェアハウスのワークロード]: ./sql-data-warehouse-overview-workload.md
[サンプル データ]: ./sql-data-warehouse-get-started-load-samples.md
[プロビジョニング]: ./sql-data-warehouse-get-started-provision.md

<!--MSDN references-->

<!--Other Web references-->

<!---HONumber=AcomDC_0309_2016-->