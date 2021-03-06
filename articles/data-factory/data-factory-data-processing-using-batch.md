<properties
    pageTitle="Azure Batch と Data Factory を使用した HPC とデータのオーケストレーション"
    description="Azure Batch の並列処理機能を使用して、Azure Data Factory パイプラインで膨大な量のデータを処理する方法について説明します。"
    services="data-factory"
    documentationCenter=""
    authors="spelluru"
    manager="jhubbard"
    editor="monicar"/>

<tags
    ms.service="data-factory"
    ms.workload="data-services"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/20/2016"
    ms.author="spelluru"/>
# Azure Batch と Data Factory を使用した HPC とデータのオーケストレーション

ハイ パフォーマンス コンピューティング (HPC) はこれまでオンプレミスのデータ センターのドメイン (データを処理するスーパー コンピューター) でしたが、使用可能な物理マシンの数によって制限されます。Azure Batch サービスは、HPC をサービスとして提供することでこれを変革します。マシンを必要な数だけ構成することができます。また Batch は、作業のスケジューリングと調整を処理して、実行するアルゴリズムに集中できるようにします。Azure Data Factory は Batch を完全に補完するもので、データ移動のオーケストレーションを簡素化します。Data Factory を使用すると、ETL のデータの定期的な移動を指定し、データを処理した後、結果を永続記憶域に移動することができます。たとえば、センサーから収集されたデータは、(Data Factory によって) 一時的な場所に移動されます。ここで Batch が (Data Factory の制御下で) データを処理して新しい結果セットを生成します。その後、Data Factory によって結果が最終的なリポジトリに移動されます。これら 2 つのサービスを連動させることで、HPC を効率的に使用して定期的なスケジュールで大量のデータを処理することができます。

ここでは、大規模なデータセットを自動的に移動して処理するエンド ツー エンドのソリューション例を提供します。このアーキテクチャは、金融サービスによるリスク モデリング、画像処理とレンダリング、およびゲノムの分析などの多くのシナリオに関係します。設計者および IT 意思決定者は、ダイアグラムと基本的な手順から概要を取得します。開発者は、独自の実装の出発点として、コードを使用できます。この記事には、ソリューション全体が含まれています。

これらのサービスに精通していない場合は、以下のサンプル ソリューションの前に、「[Azure Batch](../batch/batch-api-basics.md)」と「[Data Factory](data-factory-introduction.md)」を参照してください。

## アーキテクチャ ダイアグラム

このダイアグラムでは、1) Data Factory がデータの移動と処理を調整する方法、および 2) Azure Batch が並列的にデータを処理する方法を示します。簡単に参照できるように、図をダウンロードして印刷 (11 x 17 インチまたは A3 サイズ): [Azure Batch と Data Factory を使用した HPC とデータのオーケストレーション](http://go.microsoft.com/fwlink/?LinkId=717686)

![サービスとしての HPC 図](./media/data-factory-data-processing-using-batch/image1.png)

処理の基本的な手順を次に示します。ソリューションには、エンド ツー エンド ソリューションをビルドするためのコードと説明が含まれています。

1.  コンピューティング ノード (VM) のプールで Azure Batch を構成します。各ノードのノード数とサイズを指定することができます。

2.  Azure Blob Storage、Azure Batch のコンピューティング サービス、入出力データ、およびデータを移動および変換するアクティビティを持つワークフローまたはパイプラインを表すエンティティで構成される、Azure Data Factory のインスタンスを作成します。

3.  Data Factory のパイプラインには、ノードの Azure Batch プール上で実行するように構成されるカスタム .NET アクティビティがあります。

4.  Azure ストレージに BLOB として大量の入力データを格納します。データは、論理スライス (通常は時間) に分割されます。

5.  Data Factory は、データをコピーします。このデータは、2 次拠点と並行して処理されます。

6.  Data Factory は、Batch によって割り当てられるプールを使用して、カスタム アクティビティを実行します。Data Factory は、アクティビティを同時に実行できます。各アクティビティは、データのスライスを処理します。結果は Azure Storage に格納されます。

7.  すべての結果を取得した後、Data Factory は、アプリを介した配布、または他のツールによる追加の処理のために、3 次拠点にその結果を移動させます。

## アーキテクチャ ソリューション

ソリューションでは、タイム シリーズで編成された入力ファイルで、検索語句 ("Microsoft") の出現数をカウントします。出力ファイルにその数を出力します。

**時間**: Azure、Data Factory、および Batch を理解し、前提条件を完了した場合、このソリューションを完了するために、1 ～ 2 時間かかると推定しています。

### 前提条件

1.  **Azure サブスクリプション**。Azure サブスクリプションがない場合は、無料試用版のアカウントを数分で作成することができます。「[無料試用版](https://azure.microsoft.com/pricing/free-trial/)」を参照してください。

2.  **Azure ストレージ アカウント**。このチュートリアルのデータを格納するには、Azure ストレージ アカウントを使用します。Azure ストレージ アカウントがない場合は、「[ストレージ アカウントの作成](../storage/storage-create-storage-account.md#create-a-storage-account)」を参照してください。サンプル ソリューションでは、Blob Storage を使用します。

3.  [Azure ポータル](http://manage.windowsazure.com/)を使用して、**Azure Batch アカウント**を作成します。「[Azure Batch アカウントの作成と管理](../batch/batch-account-create-portal.md)」を参照してください。Azure Batch のアカウント名とアカウント キーをメモしておきます。また、[New-AzureRmBatchAccount](https://msdn.microsoft.com/library/mt603749.aspx) コマンドレットを使用して、Azure Batch アカウントを作成することもできます。このコマンドレットの使用に関する詳細な手順については、「[Azure Batch PowerShell コマンドレットの概要](../batch/batch-powershell-cmdlets-get-started.md)」を参照してください。

    サンプル ソリューションでは、(Azure Data Factory パイプラインを通じて間接的に) Azure Batch を使用して、仮想マシンの管理コレクションであるコンピューティング ノードのプールで、同じ方法でデータを処理します。

4.  2 個以上のコンピューティング ノードで **Azure Batch プール** を作成します。

	 [Azure Batch Explorer ツール](https://github.com/Azure/azure-batch-samples/tree/master/CSharp/BatchExplorer)のソース コードをダウンロードし、コンパイルして使用し、プールを作成することもできますが (**このサンプル ソリューションには強くお勧めします**)、[.NET 向け Azure Batch ライブラリ](../batch/batch-dotnet-get-started.md)を使用して、プールを作成することもできます。Azure Batch Explorer の使用手順については、[Azure Batch Explorer サンプル チュートリアル](http://blogs.technet.com/b/windowshpc/archive/2015/01/20/azure-batch-explorer-sample-walkthrough.aspx)のページを参照してください。[New-AzureRmBatchPool](https://msdn.microsoft.com/library/mt628690.aspx) コマンドレットを使用して、Azure Batch プールを作成することもできます。

	 Batch Explorer を使用して、次の設定でプールを作成します。

	-   プールの ID を入力します (**プール ID**)。**プールの ID** は、Data Factory ソリューションを作成するときに必要になります。

	-   **オペレーティング システム ファミリ**設定には、**Windows Server 2012 R2** を指定します。

	-   **コンピューティング ノードごとの最大タスク**の設定値には、**2** を指定します。

	-   **ターゲットの専用数**の設定値には、**2** を指定します。

	 ![](./media/data-factory-data-processing-using-batch/image2.png)

5.  [Azure Storage エクスプローラー 6 (ツール)](https://azurestorageexplorer.codeplex.com/) または [CloudXplorer](http://clumsyleaf.com/products/cloudxplorer) (ClumsyLeaf ソフトウェアから)。これらは、クラウドでホストされるアプリケーションのログを含む、Azure Storage プロジェクト内のデータを調べたり、変更したりするための GUI ツールです。

    1.  プライベートなアクセス (匿名アクセスなし) で **mycontainer** という名前のコンテナーを作成します

    2.  **CloudXplorer** を使用している場合、次の構造でフォルダーおよびサブフォルダーを作成します。

 		![](./media/data-factory-data-processing-using-batch/image3.png)

		 **Inputfolder** と **outputfolder** は、**mycontainer** のトップ レベル フォルダーにあり、**inputfolder** には日付タイム スタンプ (YYYY-MM-DD-HH) を含むサブフォルダーがあります。

		 **Azure Storage エクスプローラー**使用している場合は、次の手順で、以下の名前を持つファイルをアップロードする必要があります。inputfolder/2015-11-16-00/file.txt、inputfolder/2015-11-16-01/file.txt など。この操作では、フォルダーを自動的に作成します。

	3.  テキスト ファイル **file.txt** を、キーワード **Microsoft** を含めた内容でコンピューターに作成します。例: “test custom activity Microsoft test custom activity Microsoft”。

	4.  ファイルを Azure Blob Storage 内の次の入力フォルダーにアップロードします。

		![](./media/data-factory-data-processing-using-batch/image4.png)

	 	**Azure Storage エクスプローラー** を使用している場合、ファイル **file.txt** を **mycontainer** にアップロードします。ツールバーで **[コピー]** をクリックして、BLOB のコピーを作成します。**[BLOB のコピー]** ダイアログ ボックスで、**[宛先 BLOB 名]** を「**inputfolder/2015-11-16-00/file.txt**」に変更します。 この手順を繰り返して、inputfolder/2015-11-16-01/file.txt、inputfolder/2015-11-16-02/file.txt、inputfolder/2015-11-16-03/file.txt、inputfolder/2015-11-16-04/file.txt などを作成します。この操作では、フォルダーを自動的に作成します。

	3.  **customactivitycontainer** という名前の別のコンテナーを作成します。カスタム アクティビティの zip ファイルを、このコンテナーにアップロードします。

6.  **Microsoft Visual Studio 2012 以降** (Data Factory ソリューションで使用するカスタム Batch アクティビティを作成する場合)。

### ソリューションを作成する手順の概要

1.  Data Factory ソリューションで使用するカスタム アクティビティを作成します。このカスタム アクティビティには、データ処理ロジックが含まれています。

    1.  Visual Studio (または任意のコード エディター) で .NET Class Library プロジェクトを作成し、入力データを処理するコードを追加してプロジェクトをコンパイルします。

    2.  出力フォルダーにあるすべてのバイナリ ファイルと PDB (省略可能) ファイルを zip 形式で圧縮します。

    3.  zip ファイルを Azure Blob Storage にアップロードします。

	詳細な手順については、「[カスタム アクティビティの作成](#_Coding_the_custom)」セクションを参照してください。

2.  カスタム アクティビティを使用する Azure Data Factory を作成します。

    1.  Azure Data Factory を作成します。

    2.  リンクされたサービスを作成します。

        1.  StorageLinkedService: BLOB にアクセスするためのストレージの資格情報を指定します。

        2.  AzureBatchLinkedService: コンピューティングとして Azure Batch を指定します。

    3.  データセットを作成します。

        1.  InputDataset: 入力 BLOB のストレージ コンテナーとフォルダーを指定します。

        2.  OuputDataset: 出力 BLOB のストレージ コンテナーとフォルダーを指定します。

    4.  カスタム アクティビティを使用するパイプラインを作成します。

    5.  パラメーターを実行してテストします。

    6.  パイプラインをデバッグします。

 	詳細な手順については、「[Data Factory の作成](#create-the-data-factory)」セクションを参照してください。

## カスタム アクティビティの作成

Data Factory のカスタム アクティビティは、このサンプル ソリューションの核となる機能です。サンプル ソリューションでは、Azure Batch を使用して、カスタム アクティビティを実行します。カスタム アクティビティを開発し、Azure Data Factory パイプラインで使用する方法の基本情報は、「[Azure Data Factory パイプラインでカスタム アクティビティを使用する](data-factory-use-custom-activities.md)」を参照してください。

Azure Data Factory パイプラインで使用できる .NET カスタム アクティビティを作成するには、**IDotNetActivity** インターフェイスを実装したクラスを含む **.NET Class Library** プロジェクトを作成する必要があります。このインターフェイスには、**Execute** という 1 つのメソッドのみが含まれます。このメソッドのシグネチャを次に示します。

	public IDictionary<string, string> Execute(
	            IEnumerable<LinkedService> linkedServices,
	            IEnumerable<Dataset> datasets,
	            Activity activity,
	            IActivityLogger logger)

このメソッドには、理解しておく必要がある重要な要素がいくつかあります。

-   このメソッドには次の 4 つのパラメーターがあります。

    1.  **linkedServices**。入力/出力データ ソースを Data Factory にリンクする、リンクされたサービスの列挙可能な一覧です。このサンプルでは、入力と出力の両方に使用される Azure Storage 型のリンクされたサービスは 1 つのみです。

    2.  **datasets**。データセットの列挙可能な一覧です。このパラメーターを使用すると、入力と出力のデータセットに定義されている場所とスキーマを取得できます。

    3.  **activity**。このパラメーターは、現在のコンピューティング エンティティ (この場合は Azure Batch サービス) を表します。

    4.  **logger**。logger を使用すると、パイプラインの “User” ログとして表示されるデバッグ コメントが出力されます。

-   メソッドから、カスタム アクティビティの連結に使用できるディクショナリが返されます。このサンプル ソリューションでは、この機能を使用しません。

### 手順: カスタム アクティビティの作成

1.  Visual Studio で .NET クラス ライブラリ プロジェクトを作成します。

    1.  **Visual Studio 2012**/**2013/2015** を起動します。

    2.  **[ファイル]** をクリックし、**[新規作成]** をポイントして、**[プロジェクト]** をクリックします。

    3.  **[テンプレート]** を展開し、**[Visual C#]** を選択します。このチュートリアルでは C# を使用しますが、カスタム アクティビティの開発には、どの .NET 言語でも使用できます。

    4.  右側にあるプロジェクトの種類の一覧から **[クラス ライブラリ]** を選択します。

    5.  **[プロジェクト名]** に「**MyDotNetActivity**」と入力します。

    6.  **[場所]** で **[C:\\ADF]** を選択します。**ADF** フォルダーがない場合には、フォルダーを作成します。

    7.  **[OK]** をクリックしてプロジェクトを作成します。

2.  **[ツール]** をクリックし、**[NuGet パッケージ マネージャー]** をポイントして、**[パッケージ マネージャー コンソール]** をクリックします。

3.  **[パッケージ マネージャー コンソール]** で、次のコマンドを実行して、**Microsoft.Azure.Management.DataFactories** をインポートします。

			Install-Package Microsoft.Azure.Management.DataFactories

4.  **Azure Storage** NuGet パッケージをプロジェクトにインポートします。Blob Storage API を使用するため、このパッケージが必要です。

		Install-Package Azure.Storage

5.  次の **using** ディレクティブをプロジェクト内のソース ファイルに追加します。

		using System.IO;
		using System.Globalization;
		using System.Diagnostics;
		using System.Linq;

		using Microsoft.Azure.Management.DataFactories.Models;
		using Microsoft.Azure.Management.DataFactories.Runtime;

		using Microsoft.WindowsAzure.Storage;
		using Microsoft.WindowsAzure.Storage.Blob;

6.  **namespace** の名前を **MyDotNetActivityNS** に変更します。

		namespace MyDotNetActivityNS

7.  クラス名を **MyDotNetActivity** に変更し、次に示すように **IDotNetActivity** インターフェイスから派生させます。

		public class MyDotNetActivity : IDotNetActivity

8.  **IDotNetActivity** インターフェイスの **Execute** メソッドを **MyDotNetActivity** クラスに実装 (追加) し、次のサンプル コードをメソッドにコピーします。このメソッドで使用されるロジックのための説明については、「[メソッドの実行](#execute-method)」セクションを参照してください。

		/// <summary>
        /// Execute method is the only method of IDotNetActivity interface you must implement.
        /// In this sample, the method invokes the Calculate method to perform the core logic.  
		/// </summary>
        public IDictionary<string, string> Execute(
            IEnumerable<LinkedService> linkedServices,
            IEnumerable<Dataset> datasets,
            Activity activity,
            IActivityLogger logger)
        {

            // declare types for input and output data stores
            AzureStorageLinkedService inputLinkedService;

            // declare dataset types
            CustomDataset inputLocation;
            AzureBlobDataset outputLocation;

            Dataset inputDataset = datasets.Single(dataset => dataset.Name == activity.Inputs.Single().Name);
            inputLocation = inputDataset.Properties.TypeProperties as CustomDataset;

            foreach (LinkedService ls in linkedServices)
                logger.Write("linkedService.Name {0}", ls.Name);

            // using First method instead of Single since we are using the same
            // Azure Storage linked service for input and output.
            inputLinkedService = linkedServices.First(
                linkedService =>
                linkedService.Name ==
                inputDataset.Properties.LinkedServiceName).Properties.TypeProperties
                as AzureStorageLinkedService;

            string connectionString = inputLinkedService.ConnectionString; // To create an input storage client.
            string folderPath = GetFolderPath(inputDataset);
            string output = string.Empty; // for use later.

            // create storage client for input. Pass the connection string.
            CloudStorageAccount inputStorageAccount = CloudStorageAccount.Parse(connectionString);
            CloudBlobClient inputClient = inputStorageAccount.CreateCloudBlobClient();

            // initialize the continuation token before using it in the do-while loop.
            BlobContinuationToken continuationToken = null;
            do
            {   // get the list of input blobs from the input storage client object.
                BlobResultSegment blobList = inputClient.ListBlobsSegmented(folderPath,
                                         true,
                                         BlobListingDetails.Metadata,
                                         null,
                                         continuationToken,
                                         null,
                                         null);

                // Calculate method returns the number of occurrences of
                // the search term (“Microsoft”) in each blob associated
        		// with the data slice.
        		//
        	    // definition of the method is shown in the next step.
                output = Calculate(blobList, logger, folderPath, ref continuationToken, "Microsoft");

            } while (continuationToken != null);

            // get the output dataset using the name of the dataset matched to a name in the Activity output collection.
            Dataset outputDataset = datasets.Single(dataset => dataset.Name == activity.Outputs.Single().Name);
            // convert to blob location object.
            outputLocation = outputDataset.Properties.TypeProperties as AzureBlobDataset;

            folderPath = GetFolderPath(outputDataset);

            logger.Write("Writing blob to the folder: {0}", folderPath);

            // create a storage object for the output blob.
            CloudStorageAccount outputStorageAccount = CloudStorageAccount.Parse(connectionString);
            // write the name of the file.
            Uri outputBlobUri = new Uri(outputStorageAccount.BlobEndpoint, folderPath + "/" + GetFileName(outputDataset));

            logger.Write("output blob URI: {0}", outputBlobUri.ToString());
            // create a new blob and upload the output text.
            CloudBlockBlob outputBlob = new CloudBlockBlob(outputBlobUri, outputStorageAccount.Credentials);
            logger.Write("Writing {0} to the output blob", output);
            outputBlob.UploadText(output);

            // return a new Dictionary object (unused in this code).
            return new Dictionary<string, string>();
        }


9.  次のヘルパー メソッドをクラスに追加します。これらのメソッドは、**Execute** メソッドによって呼び出されます。最も重要な点は、**Calculate** メソッドは、各 BLOB で反復処理されるコードを分離することです。

        /// <summary>
        /// Gets the folderPath value from the input/output dataset.
		/// </summary>
		private static string GetFolderPath(Dataset dataArtifact)
		{
		    if (dataArtifact == null || dataArtifact.Properties == null)
		    {
		        return null;
		    }

		    AzureBlobDataset blobDataset = dataArtifact.Properties.TypeProperties as AzureBlobDataset;
		    if (blobDataset == null)
		    {
		        return null;
		    }

		    return blobDataset.FolderPath;
		}

		/// <summary>
		/// Gets the fileName value from the input/output dataset.
		/// </summary>

		private static string GetFileName(Dataset dataArtifact)
		{
		    if (dataArtifact == null || dataArtifact.Properties == null)
		    {
		        return null;
		    }

		    AzureBlobDataset blobDataset = dataArtifact.Properties.TypeProperties as AzureBlobDataset;
		    if (blobDataset == null)
		    {
		        return null;
		    }

		    return blobDataset.FileName;
		}

		/// <summary>
		/// Iterates through each blob (file) in the folder, counts the number of instances of search term in the file,
		/// and prepares the output text that will be written to the output blob.
		/// </summary>

		public static string Calculate(BlobResultSegment Bresult, IActivityLogger logger, string folderPath, ref BlobContinuationToken token, string searchTerm)
		{
		    string output = string.Empty;
		    logger.Write("number of blobs found: {0}", Bresult.Results.Count<IListBlobItem>());
		    foreach (IListBlobItem listBlobItem in Bresult.Results)
		    {
		        CloudBlockBlob inputBlob = listBlobItem as CloudBlockBlob;
		        if ((inputBlob != null) && (inputBlob.Name.IndexOf("$$$.$$$") == -1))
		        {
		            string blobText = inputBlob.DownloadText(Encoding.ASCII, null, null, null);
		            logger.Write("input blob text: {0}", blobText);
		            string[] source = blobText.Split(new char[] { '.', '?', '!', ' ', ';', ':', ',' }, StringSplitOptions.RemoveEmptyEntries);
		            var matchQuery = from word in source
		                             where word.ToLowerInvariant() == searchTerm.ToLowerInvariant()
		                             select word;
		            int wordCount = matchQuery.Count();
		            output += string.Format("{0} occurrences(s) of the search term "{1}" were found in the file {2}.\r\n", wordCount, searchTerm, inputBlob.Name);
		        }
		    }
		    return output;
		}


	**GetFolderPath** メソッドは、データセットが指すフォルダーのパスを返し、**GetFileName** メソッドは、データセットが指す BLOB/ファイルの名前を返します。

	    "name": "InputDataset",
	    "properties": {
	        "type": "AzureBlob",
	        "linkedServiceName": "StorageLinkedService",
	        "typeProperties": {
	            "fileName": "file.txt",
	            "folderPath": "mycontainer/inputfolder/{Year}-{Month}-{Day}-{Hour}",


	**Calculate** メソッドは、入力ファイル (フォルダー内の BLOB) 内のキーワード **Microsoft** のインスタンス数を計算します。検索語句 ("Microsoft") は、コードにハード コーディングされています。

10.  プロジェクトをコンパイルします。メニューの **[ビルド]** をクリックし、**[ソリューションのビルド]** をクリックします。

11.  **Windows エクスプローラー**を起動し、ビルドの種類に応じて、**bin\\debug** フォルダーまたは **bin\\release** フォルダーに移動します。

12.  **\\bin\\Debug** フォルダー内のすべてのバイナリを含む zip ファイル、**MyDotNetActivity.zip** を作成します。エラー発生時の問題の原因となったソース コードの行番号など、追加情報を取得するために、MyDotNetActivity.**pdb** ファイルを含めることもできます。

	![](./media/data-factory-data-processing-using-batch/image5.png)

13.  **MyDotNetActivity.zip** を BLOB として **customactvitycontainer** にアップロードします。この BLOB コンテナーは、**ADFTutorialDataFactory** 内の **StorageLinkedService** リンク サービスが使用する Azure BLOB ストレージ内にあります。BLOB コンテナー **customactivitycontainer** が既に存在していなければ、作成します。

### Execute メソッド

ここでは、Execute メソッドのコードの詳細と注意事項について説明します。

1.	入力コレクションを反復処理するメンバーは、[Microsoft.WindowsAzure.Storage.Blob](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.storage.blob.aspx) 名前空間にあります。BLOB コレクションの反復処理では、**BlobContinuationToken** クラスを使用する必要があります。基本的に、既存のループのメカニズムとして、トークンに do-while ループを使用する必要があります。詳細については、「[.NET から BLOB ストレージを使用する方法](../storage/storage-dotnet-how-to-use-blobs.md)」を参照してください。基本的なループを次に示します。

		// Initialize the continuation token.
		BlobContinuationToken continuationToken = null;
		do
		{
		// Get the list of input blobs from the input storage client object.
		BlobResultSegment blobList = inputClient.ListBlobsSegmented(folderPath,
		    					true,
		                                  BlobListingDetails.Metadata,
		                                  null,
		                                  continuationToken,
		                                  null,
		                                  null);
		// Return a string derived from parsing each blob.
		    output = Calculate(blobList, logger, folderPath, ref continuationToken, "Microsoft");

		} while (continuationToken != null);

	詳細については、[ListBlobsSegmented](https://msdn.microsoft.com/library/jj717596.aspx) メソッドのドキュメントを参照してください。

2.	論理的には、BLOB セットを操作するコードを do-while ループ内に配置します。**Execute** メソッドの場合、do-while ループは BLOB の一覧を **Calculate** というメソッドに渡します。Calculate メソッドは、セグメント内のすべての BLOB を反復処理した結果である **output** という文字列変数を返します。

	また、**Calculate** メソッドに渡された BLOB 内の検索語句 (**Microsoft**) の出現回数を返します。

		output += string.Format("{0} occurrences of the search term "{1}" were found in the file {2}.\r\n", wordCount, searchTerm, inputBlob.Name);

3.	**Calculate** メソッドの処理が完了すると、新しい BLOB に出力されます。そのため、処理対象の BLOB セットごとに、結果を新しい BLOB に出力できます。新しい BLOB に出力するには、まず出力データセットを検索します。

		// Get the output dataset using the name of the dataset matched to a name in the Activity output collection.
		Dataset outputDataset = datasets.Single(dataset => dataset.Name == activity.Outputs.Single().Name);

		// Convert to blob location object.
		outputLocation = outputDataset.Properties.TypeProperties as AzureBlobDataset;

4.	また、このコードは、**GetFolderPath** というヘルパー メソッドも呼び出して、フォルダー パス (ストレージ コンテナー名) を取得します。

		folderPath = GetFolderPath(outputDataset);

	**GetFolderPath** は、FolderPath というプロパティがある AzureBlobDataSet に DataSet オブジェクトをキャストします。

		AzureBlobDataset blobDataset = dataArtifact.Properties.TypeProperties as AzureBlobDataset;

		return blobDataset.FolderPath;

5.	コードで、ファイル名 (BLOB 名) を取得する **GetFileName** メソッドを呼び出します。このコードは、フォルダー パスを取得する上記のコードと似ています。

		AzureBlobDataset blobDataset = dataArtifact.Properties.TypeProperties as AzureBlobDataset;

		return blobDataset.FileName;

6.	新しい URI オブジェクトを作成してファイル名が出力されます。URI コンストラクターは **BlobEndpoint** プロパティを使用して、コンテナー名を返します。フォルダー パスとファイル名が追加され、出力 BLOB URI が構築されます。

		// Write the name of the file.
		Uri outputBlobUri = new Uri(outputStorageAccount.BlobEndpoint, folderPath + "/" + GetFileName(outputDataset));

7.	ファイル名が出力されるので、**Calculate** メソッドから出力文字列を新しい BLOB に出力できるようになります。

		// Create a new blob and upload the output text.
		CloudBlockBlob outputBlob = new CloudBlockBlob(outputBlobUri, outputStorageAccount.Credentials);
		logger.Write("Writing {0} to the output blob", output);
		outputBlob.UploadText(output);


## Data Factory の作成

「[カスタム アクティビティの作成](#create-the-custom-activity)」セクションでは、カスタム アクティビティを作成し、バイナリと PDB ファイルを含む zip ファイルを Azure Blob コンテナーにアップロードしました。ここでは、**カスタム アクティビティ**を使用した**パイプライン**で Azure **Data Factory** を作成します。

カスタム アクティビティの入力データセットは、Blob Storage の入力フォルダー (mycontainer\\inputfolder) 内にある BLOB (ファイル) を表します。アクティビティの出力データセットは、Blob Storage の出力フォルダー (mycontainer\\outputfolder) 内にある出力 BLOB を表します。

次のように、入力フォルダーに 1 つ以上のファイルをドロップします。

	mycontainer -> inputfolder
		2015-11-16-00
		2015-11-16-01
		2015-11-16-02
		2015-11-16-03
		2015-11-16-04

たとえば、次の内容を含む 1 つのファイル (file.txt) を各フォルダーにドロップします。

	test custom activity Microsoft test custom activity Microsoft

入力フォルダーに複数のファイルが含まれている場合でも、各入力フォルダーは Azure Data Factory のスライスに相当します。パイプラインで各スライスが処理されるときに、カスタム アクティビティでは、そのスライスの入力フォルダー内にあるすべての BLOB が反復処理されます。

同じ内容の 5 つの出力ファイルが表示されます。たとえば、2015-11-16-00 フォルダーのファイルを処理して出力されるファイルには、次の内容が含まれます。

	2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-00/file.txt.

同じ内容の複数のファイル (file.txt、file2.txt、file3.txt) を入力フォルダーにドロップした場合、出力ファイルに次のコンテンツが表示されます。各フォルダー (2015-11-16-00 など) は、フォルダーに複数の入力ファイルがある場合でも、このサンプルのスライスに対応します。

	2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-00/file.txt.
	2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-00/file2.txt.
	2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-00/file3.txt.


出力ファイルに 3 行あることに注意してください。スライス (2015-11-16-00) に関連付けられているフォルダー内の入力ファイル (BLOB) ごとに 1 行あります。

アクティビティの実行ごとに、1 つのタスクが作成されます。このサンプルでは、パイプラインにアクティビティが 1 つだけあります。スライスがパイプラインによって処理されると、カスタム アクティビティはスライスを処理する Azure Batch 上で実行されます。スライスが 5 個あるため (スライスごとに複数の BLOB またはファイルを持つことができます)、Azure Batch でタスクが 5 個作成されます。タスクが Batch 上で実行されると、タスクは実際に実行されているカスタム アクティビティです。

次のチュートリアルでは、追加情報を示します。

### 手順 1: Data Factory を作成する

1.  [Azure ポータル](https://portal.azure.com/)にログインした後、次の操作を行います。

    1.  左側のメニューの **[新規]** をクリックします。

    2.  **[新規]** ブレードの **[データ + 分析]** をクリックします。

    3.  **[データ分析]** ブレードの **[Data Factory]** をクリックします。

2.  **[新しい Data Factory]** ブレードで、[名前] フィールドに「**CustomActivityFactory**」と入力します。Azure Data Factory の名前はグローバルに一意にする必要があります。**""CustomActivityFactory" という名前の Data Factory は使用できません"** というエラーが発生した場合は、Data Factory の名前を変更して (**yournameCustomActivityFactory** など) 作成し直してください。

3.  **[リソース グループ名]** をクリックし、既存のリソース グループを選択するか、新しいリソース グループを作成します。

4.  作成する Data Factory に適したサブスクリプションとリージョンを使用していることを確認します。

5.  **[新しいデータ ファクトリ]** ブレードで **[作成]** をクリックします。

6.  作成した Data Factory は、Azure ポータルの**ダッシュボード**に表示されます。

7.  データ ファクトリが正常に作成されると、データ ファクトリの内容を表示するデータ ファクトリ ページが表示されます。

 ![](./media/data-factory-data-processing-using-batch/image6.png)

### 手順 2. リンク サービスを作成する

リンクされたサービスは、データ ストアまたはコンピューティング サービスを Azure Data Factory にリンクします。この手順では、**Azure Storage** アカウントと **Azure Batch** クラスターを Data Factory にリンクさせます。

#### Azure Storage のリンクされたサービスを作成する

1.  **[CustomActivityFactory]** の **[Data Factory]** ブレードで、**[作成およびデプロイ]** をクリックします。Data Factory エディタが起動します。

2.  コマンド バーの **[新しいデータ ストア]** をクリックし、**[Azure Storage]** を選択します。 Azure Storage のリンクされたサービスを作成するための JSON スクリプトがエディターに表示されます。

    ![](./media/data-factory-data-processing-using-batch/image7.png)

3.  **accountname** は Azure ストレージ アカウントの名前に、**accountkey** は Azure ストレージ アカウントのアクセス キーに置き換えます。ストレージ アクセス キーを取得する方法については、「[ストレージ アクセス キーの表示、コピーおよび再生成](../storage/storage-create-storage-account.md#view-copy-and-regenerate-storage-access-keys)」を参照してください。

4.  コマンド バーの **[デプロイ]** をクリックして、リンク サービスをデプロイします。

    ![](./media/data-factory-data-processing-using-batch/image8.png)

#### Azure Batch のリンクされたサービスの作成

この手順では、Data Factory のカスタム アクティビティの実行に使用される、**Azure Batch** アカウント用にリンクされたサービスを作成します。

1.  コマンド バーの **[新規計算]** をクリックし、**[Azure Batch]** を選択します。 Azure Batch のリンク サービスを作成するための JSON スクリプトが、エディターに表示されます。

2.  JSON スクリプト内では:

    1.  **account name** を Azure Batch アカウントの名前に置き換えます。

    2.  **access key** を Azure Batch アカウントのアクセス キーに置き換えます。

    3.  **poolName** プロパティにプールの ID を入力しています**。**このプロパティでは、プール名またはプール ID のいずれかを指定できます。

    4.  **batchUri** JSON プロパティにバッチ URI を入力します。**[Azure Batch アカウント] ブレード**の **URL** は、次の形式です: <accountname>.<region>.batch.azure.com。JSON の **batchUri** プロパティでは、URL から **"accountname." を削除**する必要があります。例: "batchUri": "https://eastus.batch.azure.com"。

        ![](./media/data-factory-data-processing-using-batch/image9.png)

    5.  **linkedServiceName** プロパティに、**StorageLinkedService** を指定します。このリンクされたサービスは、前の手順で作成しています。このストレージは、ファイルおよびログのステージング領域として使用されます。

3.  コマンド バーの **[デプロイ]** をクリックして、リンク サービスをデプロイします。

### 手順 3: データセットを作成する

この手順では、入力データと出力データを示すデータセットを作成します。

#### 入力データセットの作成

1.  Data Factory **エディター**のツール バーで **[新しいデータセット]** ボタンをクリックし、ドロップダウン メニューから **[Azure BLOB ストレージ]** をクリックします。

2.  右側のウィンドウの JSON を、次の JSON スニペットに置き換えます。

		{
		    "name": "InputDataset",
		    "properties": {
		        "type": "AzureBlob",
		        "linkedServiceName": "StorageLinkedService",
		        "typeProperties": {
		            "folderPath": "mycontainer/inputfolder/{Year}-{Month}-{Day}-{Hour}",
		            "format": {
		                "type": "TextFormat"
		            },
		            "partitionedBy": [
		                {
		                    "name": "Year",
		                    "value": {
		                        "type": "DateTime",
		                        "date": "SliceStart",
		                        "format": "yyyy"
		                    }
		                },
		                {
		                    "name": "Month",
		                    "value": {
		                        "type": "DateTime",
		                        "date": "SliceStart",
		                        "format": "MM"
		                    }
		                },
		                {
		                    "name": "Day",
		                    "value": {
		                        "type": "DateTime",
		                        "date": "SliceStart",
		                        "format": "dd"
		                    }
		                },
		                {
		                    "name": "Hour",
		                    "value": {
		                        "type": "DateTime",
		                        "date": "SliceStart",
		                        "format": "HH"
		                    }
		                }
		            ]
		        },
		        "availability": {
		            "frequency": "Hour",
		            "interval": 1
		        },
		        "external": true,
		        "policy": {}
		    }
		}


	 このチュートリアルで後半では、開始時刻 2015-11-16T00:00:00Z、終了時刻 2015-11-16T05:00:00Z のパイプラインを作成します。データを**毎時**生成するようにスケジュールしているので、5 個の入出力スライスがあります (**00**:00:00 ～ **05**:00:00 の間)。

	 入力データセットの **frequency** と **interval** はそれぞれ **Hour** と **1** に設定されています。つまり、1 時間に 1 つの入力スライスを利用できます。

	 次に各スライスの開始時刻を示します。この開始時刻は上記の JSON スニペットの **SliceStart** システム変数で表されます。

	| **スライス** | **開始時刻** |
	|-----------|-------------------------|
	| 1 | 2015-11-16T**00**:00:00 |
	| 2 | 2015-11-16T**01**:00:00 |
	| 3 | 2015-11-16T**02**:00:00 |
	| 4 | 2015-11-16T**03**:00:00 |
	| 5 | 2015-11-16T**04**:00:00 |

	 **folderPath** は、スライス開始時間 (**SliceStart**) の年、月、日、時間の部分を使用して計算されます。そのため、入力フォルダーをスライスにマップする方法を示します。

	| **スライス** | **開始時刻** | **入力フォルダー** |
	|-----------|-------------------------|-------------------|
	| 1 | 2015-11-16T**00**:00:00 | 2015-11-16-**00** |
	| 2 | 2015-11-16T**01**:00:00 | 2015-11-16-**01** |
	| 3 | 2015-11-16T**02**:00:00 | 2015-11-16-**02** |
	| 4 | 2015-11-16T**03**:00:00 | 2015-11-16-**03** |
	| 5 | 2015-11-16T**04**:00:00 | 2015-11-16-**04** |

3.  ツール バーの **[デプロイ]** をクリックし、**InputDataset** テーブルを作成してデプロイします。エディターのタイトル バーに **"テーブルが正常に作成されました"** というメッセージが表示されていることを確認します。

#### 出力データセットの作成

この手順では、出力データを表す AzureBlob 型の別のデータセットを作成します。

1.  Data Factory **エディター**のツール バーで **[新しいデータセット]** ボタンをクリックし、ドロップダウン メニューから **[Azure BLOB ストレージ]** をクリックします。

2.  右側のウィンドウの JSON を、次の JSON スニペットに置き換えます。

		{
		    "name": "OutputDataset",
		    "properties": {
		        "type": "AzureBlob",
		        "linkedServiceName": "StorageLinkedService",
		        "typeProperties": {
		            "fileName": "{slice}.txt",
		            "folderPath": "mycontainer/outputfolder",
		            "partitionedBy": [
		                {
		                    "name": "slice",
		                    "value": {
		                        "type": "DateTime",
		                        "date": "SliceStart",
		                        "format": "yyyy-MM-dd-HH"
		                    }
		                }
		            ]
		        },
		        "availability": {
		            "frequency": "Hour",
		            "interval": 1
		        }
		    }
		}


 	各入力スライスの出力 BLOB/ファイルが生成されます。次に、各スライスの出力ファイルの命名方法を示します。すべての出力ファイルは、**mycontainer\\outputfolder** という 1 つの出力フォルダーに生成されます。


	| **スライス** | **開始時刻** | **出力ファイル** |
	|-----------|-------------------------|-----------------------|
	| 1 | 2015-11-16T**00**:00:00 | 2015-11-16-**00.txt** |
	| 2 | 2015-11-16T**01**:00:00 | 2015-11-16-**01.txt** |
	| 3 | 2015-11-16T**02**:00:00 | 2015-11-16-**02.txt** |
	| 4 | 2015-11-16T**03**:00:00 | 2015-11-16-**03.txt** |
	| 5 | 2015-11-16T**04**:00:00 | 2015-11-16-**04.txt** |

	 入力フォルダー (例: 2015-11-16-00) 内のすべてのファイルは、開始時刻が 2015-11-16-00 であるスライスの一部です。このスライスを処理すると、カスタム アクティビティは各ファイルをスキャンし、出力ファイルに、検索語句 (“Microsoft”) の出現回数が記述された 1 行を生成します。2015-11-16-00 フォルダーに 3 つのファイルがある場合は、出力ファイル (2015-11-16-00.txt) に 3 行が出力されます。

3.  ツール バーの **[デプロイ]** をクリックし、**OutputDataset** を作成してデプロイします。

### 手順 4: カスタム アクティビティを使用して、パイプラインを作成して実行する

この手順では、以前に作成したカスタム アクティビティである、1 つのアクティビティでパイプラインを作成します。

> [AZURE.IMPORTANT] BLOB コンテナー内の入力フォルダーに **file.txt** をまだアップロードしていない場合は、パイプラインを作成する前に行ってください。**isPaused** プロパティがパイプラインの JSON で false に設定されているため、パイプラインは **start** の日を過ぎると、すぐに実行されます。

1.  Data Factory エディターで、コマンド バーの **[新しいパイプライン]** をクリックします。このコマンドが表示されない場合は、**[...] \(省略記号)** をクリックすると表示されます。

2.  右側のウィンドウの JSON を、次の JSON スクリプトに置き換えます。

		{
			"name": "PipelineCustom",
			"properties": {
				"description": "Use custom activity",
				"activities": [
					{
						"type": "DotNetActivity",
						"typeProperties": {
							"assemblyName": "MyDotNetActivity.dll",
							"entryPoint": "MyDotNetActivityNS.MyDotNetActivity",
							"packageLinkedService": "StorageLinkedService",
							"packageFile": "customactivitycontainer/MyDotNetActivity.zip"
						},
						"inputs": [
							{
								"name": "InputDataset"
							}
						],
						"outputs": [
							{
								"name": "OutputDataset"
							}
						],
						"policy": {
							"timeout": "00:30:00",
							"concurrency": 5,
							"retry": 3
						},
						"scheduler": {
							"frequency": "Hour",
							"interval": 1
						},
						"name": "MyDotNetActivity",
						"linkedServiceName": "AzureBatchLinkedService"
					}
				],
				"start": "2015-11-16T00:00:00Z",
				"end": "2015-11-16T05:00:00Z",
				"isPaused": false
		   }
		}

	以下の点に注意してください。

	-   パイプラインには、**DotNetActivity** 型のアクティビティが 1 つだけあります。

	-   **AssemblyName** を DLL の名前 (**MyDotNetActivity.dll**) に設定します。

	-   **EntryPoint** を **MyDotNetActivityNS.MyDotNetActivity** に設定します。基本的に、コード内の \<namespace\>.\<classname\> です。

	-   **PackageLinkedService** は **StorageLinkedService** に設定されます。これは、カスタム アクティビティの zip ファイルを含む Blob Storage を示します。入力/出力ファイルとカスタム アクティビティ zip ファイルに別の Azure Storage アカウントを使用している場合、Azure Storage のリンクされたサービスを別に作成する必要があります。この記事では、同じ Azure Storage アカウントを使用している前提で説明します。

	-   **PackageFile** を **customactivitycontainer/MyDotNetActivity.zip** に設定します。形式は \<containerforthezip\>/\<nameofthezip.zip\> です。

	-   カスタム アクティビティは入力として **InputDataset**、出力として **OutputDataset** を使用します。

	-   カスタム アクティビティの **linkedServiceName** プロパティは、**AzureBatchLinkedService** を示します。これによって、Azure Batch でカスタム アクティビティが実行する必要がある Azure Data Factory がわかります。

	-   **concurrency** の設定は重要です。既定値を使用する場合は、Azure Batch プールにコンピューティング ノードが複数ある場合でも、スライスは 1 つずつ処理されます。そのため、Azure Batch の並行処理機能を利用することはありません。**concurrency** をより大きな値に設定した場合、2 とすると、つまり 2 つのスライス (Azure Batch 内の 2 つのタスクに対応する) が、同時に処理できます。この場合、Azure Batch プール内の VM が両方利用されます。そのため、concurrency プロパティを適切に設定します。

	-   既定では、1 つのみのタスク (スライス) が、VM 上で任意のポイントで実行されます。これは、既定では、**VM ごとの最大タスク**が、Azure Batch プールに対して 1 に設定されているためです。前提条件の一部として、このプロパティを 2 に設定したプールを作成しているため、2 つの Data Factory のスライスは、同時に VM 上で実行される場合があります。


	-   既定では、**isPaused** プロパティは、false に設定されています。この例では、スライスが過去に開始されているので、パイプラインは即時に実行されます。このプロパティを true に設定すると、パイプラインを一時停止できます。また false に設定し直すと再開されます。

	-   **start** 時刻と **end** 時刻は 5 時間離れており、スライスは毎時生成されるので、パイプラインでは 5 個のスライスが生成されます。

3.  コマンド バーの **[デプロイ]** をクリックして、パイプラインをデプロイします。

### 手順 5: パイプラインをテストする

この手順では、ファイルを入力フォルダーにドロップして、パイプラインをテストします。入力フォルダーごとに 1 つのファイルを使用して、パイプラインをテストしましょう。

1.  Azure ポータルの [Data Factory] ブレードで、**[ダイアグラム]** をクリックします。

    ![](./media/data-factory-data-processing-using-batch/image10.png)

2.  ダイアグラム ビューで、**[InputDataset]** という入力データセットをダブルクリックします。

    ![](./media/data-factory-data-processing-using-batch/image11.png)

3.  5 個のスライスがすべて準備完了の **[InputDataset]** ブレードが表示されます。各スライスの **[スライス開始時間]** と **[スライス終了時間]** に注意してください。

    ![](./media/data-factory-data-processing-using-batch/image12.png)

4.  **[ダイアグラム] ビュー**で、**[OutputDataset]** をクリックします。

5.  スライスが生成されると、準備完了状態の出力スライスが 5 個表示されます。

    ![](./media/data-factory-data-processing-using-batch/image13.png)

6.  [Azure Batch エクスプローラー](http://blogs.technet.com/b/windowshpc/archive/2015/01/20/azure-batch-explorer-sample-walkthrough.aspx)を使用して、**スライス**に関連付けられている**タスク**を表示し、各スライスが実行された VM を表示します。ジョブが **adf-<poolname>** という名前で作成されていることを確認します。このジョブには、スライスごとにタスクがあります。この例では、スライスが 5 個あるため、Azure Batch には 5 個のタスクがあります。Azure Data Factory のパイプラインの JSON の **concurrency** を **5** に、VM を **2** 個持つ Azure Batch プールの **[VM ごとの最大タスク]** を **2** に設定すると、タスクが高速で実行されます (**作成** 時刻を参照)。

    ![](./media/data-factory-data-processing-using-batch/image14.png)

7.  Azure Blob Storage の **mycontainer** の **outputfolder** に、出力ファイルが表示されます。

    ![](./media/data-factory-data-processing-using-batch/image15.png)

    出力ファイルが 5 個 (入力スライスごとに 1 個) 表示されます。出力ファイルごとに、次のようなコンテンツがあります。

    	2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-00/file.txt.

    次のダイアグラムでは、Data Factory のスライスを Azure Batch のタスクにマップする方法を示します。この例では、スライスごとに 1 つだけを実行します。

    ![](./media/data-factory-data-processing-using-batch/image16.png)

8.  フォルダー内の複数のファイルを試してみましょう。フォルダー **2015-11-06-01** にある file.txt と同じ内容で、　**file2.txt**、**file3.txt**、**file4.txt**、および **file5.txt** のファイルを作成します。

9.  出力フォルダーで、**2015-11-16-01.txt** という出力ファイルを**削除**します。

10. **[OutputDataset]** ブレードで、**[スライス開始時間]** を **11/16/2015 01:00:00 AM** に設定したスライスを右クリックして、**[実行]** をクリックし、スライスを再実行または再度処理します。スライスには、1 個のファイルではなく、5 個のファイルがあります。

    ![](./media/data-factory-data-processing-using-batch/image17.png)

11. スライスを実行して、その状態が **[準備完了]** になると、Blob Storage の **mycontainer** の **outputfolder** で、このスライスの出力ファイル (**2015-11-16-01.txt**) の内容を確認します。スライスのファイルごとに 1 行あります。

		2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-01/file.txt.
		2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-01/file2.txt.
		2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-01/file3.txt.
		2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-01/file4.txt.
		2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-01/file5.txt.


    **注:** 5 つの入力ファイルを試す前に、出力ファイル 2015-11-16-01.txt を削除していない場合は、以前に実行したスライスから 1 行と、現在実行しているスライスから 5 行表示されます。既定では、コンテンツが既に存在している場合、出力ファイルに追加されます。

### パイプラインのデバッグ

デバッグには、いくつかの基本的な技術があります。

1.  入力スライスが **[準備完了]** に設定されていない場合、入力フォルダー構造が正しく、入力フォルダーに file.txt が存在することを確認します。

    ![](./media/data-factory-data-processing-using-batch/image3.png)

2.  問題の解決に役立つ情報をログに記録するには、カスタム アクティビティの **Execute** メソッドで、**IActivityLogger** オブジェクトを使用します。ログに記録されたメッセージは user\_0.log ファイルに表示されます。

    **[OutputDataset]** ブレードで、スライスをクリックすると、そのスライスの **[データ スライス]** ブレードが表示されます。そのスライスの**アクティビティの実行**が表示されます。このスライスには 1 回のアクティビティの実行が表示されます。コマンド バーの **[実行]** をクリックする場合、同じスライスの別のアクティビティの実行を開始できます。

    アクティビティの実行をクリックすると、**[アクティビティの実行の詳細]** ブレードにログ ファイルの一覧が表示されます。**user\_0.log** ファイルにログに記録されたメッセージが表示されます。エラーが発生すると、パイプライン/アクティビティ JSON の再試行回数が 3 に設定されるので、3 つのアクティビティの実行が表示されます。アクティビティの実行をクリックすると、ログ ファイルが表示されます。このファイルを確認すると、エラーを解決することができます。

    ![](./media/data-factory-data-processing-using-batch/image18.png)

    ログ ファイルの一覧で、**user-0.log** をクリックします。右側のパネルには、**IActivityLogger.Write** メソッドの使用結果が表示されます。

    ![](./media/data-factory-data-processing-using-batch/image19.png)

    また、system-0.log では、システム エラー メッセージと例外を確認することもできます。

		Trace\_T\_D\_12/6/2015 1:43:35 AM\_T\_D\_\_T\_D\_Verbose\_T\_D\_0\_T\_D\_Loading assembly file MyDotNetActivity...

		Trace\_T\_D\_12/6/2015 1:43:35 AM\_T\_D\_\_T\_D\_Verbose\_T\_D\_0\_T\_D\_Creating an instance of MyDotNetActivityNS.MyDotNetActivity from assembly file MyDotNetActivity...

		Trace\_T\_D\_12/6/2015 1:43:35 AM\_T\_D\_\_T\_D\_Verbose\_T\_D\_0\_T\_D\_Executing Module

		Trace\_T\_D\_12/6/2015 1:43:38 AM\_T\_D\_\_T\_D\_Information\_T\_D\_0\_T\_D\_Activity e3817da0-d843-4c5c-85c6-40ba7424dce2 finished successfully

3.  zip ファイルに **PDB** を含めると、エラーの詳細にエラー発生時の**コール スタック**などの情報も記録されます。

4.  カスタム アクティビティの zip ファイルのすべてのファイルは、サブフォルダーがない**トップ レベル**に置く必要があります。

    ![](./media/data-factory-data-processing-using-batch/image20.png)

5.  **assemblyName** (MyDotNetActivity.dll)、**entryPoint** (MyDotNetActivityNS.MyDotNetActivity)、**packageFile** (customactivitycontainer/MyDotNetActivity.zip)、および **packageLinkedService** (zip ファイルを含む Azure Blob Storage を示す必要があります) が正しい値に設定されていることを確認します。

6.  エラーを修正し、スライスを再処理する場合は、**[OutputDataset]** ブレードのスライスを右クリックし、**[実行]** をクリックします。

    ![](./media/data-factory-data-processing-using-batch/image21.png)

    **注:** **adfjobs** という名前の Azure Blob Storage に、**コンテナー**が表示されます。このコンテナーは自動的に削除されませんが、ソリューションのテストを完了した後に、安全に削除することができます。同様に、Data Factory ソリューションでは、**adf-<pool ID/name>:job-0000000001** という名前の Azure Batch **ジョブ**を作成します。必要な場合は、ソリューションのテストを実行した後、このジョブを削除することができます。

### サンプルの拡張

Azure Data Factory および Azure Batch の機能の詳細については、このサンプルを拡張することができます。たとえば、異なる時間範囲でスライスを処理するには、次の操作を行います。

1.  **inputfolder** に2015-11-16-05、2015-11-16-06、201-11-16-07、2011-11-16-08、2015-11-16-09 のサブフォルダーを追加し、これらのフォルダーに入力ファイルを配置します。パイプラインの終了時刻を 2015-11-16T05:00:00Z から 2015-11-16T10:00:00Z に変更します。**[ダイアグラム] ビュー**で、**[InputDataset]** をダブルクリックして、入力スライスが準備完了であることを確認します。**[OuptutDataset]** をダブルクリックして、出力スライスの状態を表示します。準備完了の状態である場合、出力ファイルの outputfolder を確認します。

2.  特に Azure Batch 上で発生する処理のように、どのようにソリューションのパフォーマンスに影響するかを理解するには、**concurrency** の設定を増加または減少させます。(**concurrency** の設定の詳細については、「手順 4: パイプラインを作成して実行する」を参照してください。)

3.  **[VM ごとの最大タスク]** を高くまたは低くして、プールを作成します。Data Factory ソリューションで、Azure Batch のリンクされたサービスを更新して、新しく作成したプールを使用します。(**VM ごとの最大タスク**の設定の詳細については、「手順 4: パイプラインを作成して実行する」を参照してください。)

4.  **[自動スケール]** 機能で、Azure Batch プールを作成します。Azure Batch プール内のコンピューティング ノードの自動スケールは、アプリケーションによって使用される処理能力を動的に調整します。「[Azure Batch プール内のコンピューティング ノードの自動スケール](../batch/batch-automatic-scaling.md)」を参照してください。

    サンプル ソリューションで、**Execute** メソッドは、出力データ スライスを生成するために入力データ スライスを処理する、**Calculate** メソッドを呼び出します。入力データを処理し、Execute メソッドで呼び出される Calculate メソッドを、メソッドの呼び出しに置換する独自のメソッドを記述することができます。

## 次の手順: データの処理

データを処理した後、**Microsoft Power BI** などのオンライン ツールで使用することができます。Power BI や Azure で使用する方法を理解するために役立つリンクを次に示します。

-   [Power BI でのデータセットの参照](https://support.powerbi.com/knowledgebase/articles/475159)

-   [Power BI Desktop の概要](https://support.powerbi.com/knowledgebase/articles/471664)

-   [Power BI でのデータの更新](https://support.powerbi.com/knowledgebase/articles/474669)

-   [Azure と Power BI - 基本的な概要](https://support.powerbi.com/knowledgebase/articles/568614)

## 参照

-   [Azure Data Factory](https://azure.microsoft.com/documentation/services/data-factory/)

    -   [Azure Data Factory サービスの概要](data-factory-introduction.md)

    -   [Azure Data Factory を使ってみる](data-factory-build-your-first-pipeline.md)

    -   [Azure Data Factory パイプラインでカスタム アクティビティを使用する](data-factory-use-custom-activities.md)

-   [Azure Batch](https://azure.microsoft.com/documentation/services/batch/)

    -   [Azure Batch の基礎](../batch/batch-technical-overview.md)

    -   [Azure Batch 機能の概要](../batch/batch-api-basics.md)

    -   [Azure ポータルで Azure Batch アカウントを作成して管理する](../batch/batch-account-create-portal.md)

    -   [Azure Batch ライブラリ .NET の概要](../batch/batch-dotnet-get-started.md)

<!----HONumber=AcomDC_0128_2016-->