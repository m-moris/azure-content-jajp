<properties
   pageTitle="アプリケーション用の Web フロントエンドの作成 | Microsoft Azure"
   description="ASP.NET 5 Web API プロジェクト、および ServiceProxy を介したサービス間通信を使用して、Web に Service Fabric アプリケーションを公開します。"
   services="service-fabric"
   documentationCenter=".net"
   authors="seanmck"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.devlang="dotNet"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA"
   ms.date="01/19/2016"
   ms.author="seanmck"/>


# アプリケーション用の Web サービス フロントエンドの構築

既定では、Azure Service Fabric サービスには、Web に対するパブリック インターフェイスがありません。HTTP クライアントにアプリケーションの機能を公開するには、エントリ ポイントとして機能する Web プロジェクトを作成し、そこから個々のサービスと通信する必要があります。

このチュートリアルでは、ステートフル サービス プロジェクト テンプレートに基づく信頼性の高いサービスが既に含まれるアプリケーションに ASP.NET 5 Web API フロントエンドを追加する方法を説明します。「[Creating your first application in Visual Studio (Visual Studio での初めてのアプリケーションの作成)](service-fabric-create-your-first-application-in-visual-studio.md)」をまだ行っていない場合は、このチュートリアルを始める前に行うことをお勧めします。


## アプリケーションへの ASP.NET 5 サービスの追加

ASP.NET 5 は軽量のクロスプラットフォーム Web 開発フレームワークであり、これを使用すると、最新の Web UI と Web API を作成できます。ASP.NET Web API プロジェクトを既存のアプリケーションに追加してみましょう。

1. ソリューション エクスプローラーで、アプリケーション プロジェクトの **[サービス]** を右クリックして、**[ファブリック サービスの追加]** を選択します。

	![既存アプリケーションへの新しいサービスの追加][vs-add-new-service]

2. **[サービスの作成]** ページで、**[ASP.NET 5]** を選択し、名前を付けます。

	![Choosing ASP.NET web service in the new service dialog][vs-new-service-dialog]

3. 次のページには、一連の ASP.NET 5 プロジェクト テンプレートが表示されます。これらは、Service Fabric アプリケーションの外部で ASP.NET 5 プロジェクトを作成した場合に表示されるテンプレートと同じです。このチュートリアルでは、**[Web API]** を選択します。ただし、完全な Web アプリケーションの作成にも同じ概念を適用できます。

	![ASP.NET プロジェクトの種類の選択][vs-new-aspnet-project-dialog]

    作成した Web API プロジェクトでは、2 つのサービスがアプリケーションに含まれます。アプリケーションの作成を続けながら、まったく同じ方法でさらにサービスを追加することができます。それぞれを個別にバージョン管理およびアップグレードできます。

>[AZURE.NOTE] Service Fabric の 11 月のパブリック プレビュー リリースでは、ASP.NET プロジェクトを処理する際にパスが長いと既知の問題が生じることがわかりました。このようなプロジェクトを作成する際は、問題を回避するために、コードと構成パッケージ名だけでなく、アプリケーションとサービスの種類にも短い名前を選ぶことをお勧めします。

## アプリケーションの実行

手順を把握できるように、新しいアプリケーションをデプロイし、ASP.NET 5 Web API テンプレートによって提供される既定の動作を見てみます。

1. Visual Studio で F5 キーを押してアプリをデバッグします。

2. デプロイが完了すると、Visual Studio はブラウザーを起動して ASP.NET Web API サービスのルート (例: http://localhost:33003) を開きます。ポート番号はランダムに割り当てられるため、お使いのコンピューターでは異なる場合があります。ASP.NET 5 Web API テンプレートではルートの既定の動作が指定されないため、ブラウザーでエラーが発生します。

3. ブラウザーの場所に `/api/values` を追加します。こうすることで、Web API テンプレート内の ValuesController に対して `Get` メソッドが呼び出されます。テンプレートで指定される既定の応答として、2 つの文字列を格納する JSON 配列が返されます。

    ![ASP.NET 5 Web API テンプレートから返される既定値][browser-aspnet-template-values]

    チュートリアルの最後では、これらの既定値をステートフル サービスからの最新のカウンター値で置き換えます。


## サービスへの接続

Service Fabric は、Reliable Services との通信方法において完全な柔軟性を提供します。1 つのアプリケーション内には、TCP 経由でアクセスできるサービスもあれば、HTTP REST API 経由でアクセスできるサービスもあります。また、Web ソケットを使用してアクセスできるサービスもまだあります。使用可能なオプションおよびトレードオフについては、「[サービスとの通信](service-fabric-connect-and-communicate-with-services.md)」を参照してください。このチュートリアルでは簡単な手法の 1 つに従い、SDK で提供される `ServiceProxy`/`ServiceRemotingListener` クラスを使用します。

(リモート プロシージャ コールつまり RPC でモデル化された) `ServiceProxy` アプローチで、サービスのパブリック コントラクトとして機能するインターフェイスを定義します。次に、そのインターフェイスを使用して、サービスと対話するためのプロキシ クラスを生成します。


### インターフェイスの作成

最初に、ステートフル サービスとそのクライアントの間のコントラクトとして機能する、ASP.NET 5 プロジェクトを含むインターフェイスを作成します。

1. ソリューション エクスプローラーでソリューションを右クリックし、**[追加]**、**[新しいプロジェクト]** の順に選択します。

2. 左側のナビゲーション ウィンドウで **[Visual C#]** エントリを選択し、**[クラス ライブラリ]** テンプレートを選択します。.NET Framework のバージョンが **4.5.1** に設定されていることを確認します。

    ![ステートフル サービス用のインターフェイス プロジェクトの作成][vs-add-class-library-project]

3. インターフェイスを `ServiceProxy` で使用可能にするには、そのインターフェイスが IService インターフェイスから派生している必要があります。このインターフェイスは、Service Fabric NuGet パッケージの 1 つに含まれます。パッケージを追加するには、新しいクラス ライブラリ プロジェクトを右クリックして、**[NuGet パッケージの管理]** を選択します。

4. **[プレリリースを含める]** チェック ボックスがオンになっていることを確認した後、**Microsoft.ServiceFabric.Services** パッケージを検索してインストールします。

    ![Services NuGet パッケージの追加][vs-services-nuget-package]

5. クラス ライブラリで、1 つのメソッド `GetCountAsync` を含むインターフェイスを作成し、IService からインターフェイスを拡張します。

    ```c#
    namespace MyStatefulService.Interfaces
    {
        using Microsoft.ServiceFabric.Services.Remoting;

        public interface ICounter: IService
        {
            Task<long> GetCountAsync();
        }
    }
    ```


### ステートフル サービスでのインターフェイスの実装

インターフェイスを定義したので、次にステートフル サービスでそれを実装する必要があります。

1. ステートフル サービスでは、インターフェイスを含むクラス ライブラリ プロジェクトへの参照を追加します。

    ![ステートフル サービスのクラス ライブラリ プロジェクトへの参照の追加][vs-add-class-library-reference]

2. `StatefulService` を継承するクラスを探し (`MyStatefulService` など)、それを拡張して `ICounter` インターフェイスを実装します。

    ```c#
    using MyStatefulService.Interfaces;

    ...

    public class MyStatefulService : StatefulService, ICounter
    {        
          // ...
    }
    ```

3. 次に、`ICounter` インターフェイスで定義されている単一のメソッド `GetCountAsync` を実装します。

    ```c#
    public async Task<long> GetCountAsync()
    {
      var myDictionary =
        await this.StateManager.GetOrAddAsync<IReliableDictionary<string, long>>("myDictionary");

        using (var tx = this.StateManager.CreateTransaction())
        {          
            var result = await myDictionary.TryGetValueAsync(tx, "Counter-1");
            return result.HasValue ? result.Value : 0;
        }
    }
    ```


### ServiceRemotingListener を使用したステートフル サービスの公開

`ICounter` インターフェイスを実装した後、ステートフル サービスを他のサービスから呼び出すことができるようにする最後の手順は、通信チャネルを開くことです。ステートフル サービスのために、Service Fabric には、`CreateServiceReplicaListeners` と呼ばれるオーバーライド可能なメソッドが用意されています。このメソッドを使用すると、サービスに対して有効にする通信の種類に基づき、1 つ以上の通信リスナーを指定できます。

>[AZURE.NOTE] ステートレス サービスへの通信チャネルを開く同等のメソッドは、`CreateServiceInstanceListeners` という名前です。

この場合、既存の `CreateServiceReplicaListeners` メソッドを置換し、`ServiceRemotingListener` のインスタンスを指定します。これにより、`ServiceProxy` を介してクライアントから呼び出すことができる RPC エンドポイントが作成されます。

```c#
using Microsoft.ServiceFabric.Services.Remoting.Runtime;

...

protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
{
    return new List<ServiceReplicaListener>()
    {
        new ServiceReplicaListener(
            (initParams) =>
                new ServiceRemotingListener<ICounter>(initParams, this))
    };
}
```


### ServiceProxy クラスを使用したサービスとの対話

ステートフル サービスでは、他のサービスからトラフィックを受信する準備ができました。残りの手順は、サービスと通信するためのコードを ASP.NET Web サービスから追加するだけです。

1. ASP.NET プロジェクトで、`ICounter` インターフェイスを含むクラス ライブラリへの参照を追加します。

2. 前にクラス ライブラリ プロジェクトで行ったのと同様に、ASP.NET プロジェクトに Microsoft.ServiceFabric.Services パッケージを追加します。これにより、`ServiceProxy` クラスが提供されます。

3. **Controllers** フォルダーで `ValuesController` クラスを開きます。現時点では、`Get` メソッドはハードコーディングされた文字配列 "value1" と "value2" を返すだけであることに注意してください。これらは、先にブラウザーで見たものと一致します。この実装を次のコードに置き換えます。

    ```c#
    using MyStatefulService.Interfaces;
    using Microsoft.ServiceFabric.Services.Remoting.Client;

    ...

    public async Task<IEnumerable<string>> Get()
    {
        ICounter counter =
            ServiceProxy.Create<ICounter>(0, new Uri("fabric:/MyApp/MyStatefulService"));

        long count = await counter.GetCountAsync();

        return new string[] { count.ToString() };
    }
    ```

    コードの最初の行が重要です。ステートフル サービスへの ICounter プロキシを作成するには、パーティション ID とサービス名の 2 つの情報を提供する必要があります。

    パーティション分割を使用すると、顧客 ID や郵便番号などの定義したキーに基づいて異なるバケットにステートを分割することにより、ステートフル サービスを拡張できます。この単純なアプリケーションではステートフル サービスはパーティションを 1 つしか持たないので、キーは重要ではありません。どのキーを指定しても同じパーティションになります。サービスのパーティション分割の詳細については、「[Service Fabric Reliable Services をパーティション分割する方法](service-fabric-concepts-partitioning.md)」を参照してください。

    サービス名は、fabric:/&lt;アプリケーション名&gt;/&lt;サービス名&gt; の形式の URI です。

    これら 2 つの情報により、Service Fabric は要求送信先のコンピューターを一意に識別できます。また、`ServiceProxy` クラスは、ステートフル サービス パーティションをホストするコンピューターで障害が発生し、別のコンピューターを昇格させて引き継ぐ必要がある状況を、シームレスに処理します。この抽象化により、他のサービスを処理するクライアント コードの作成が大幅に簡略化されます。

    プロキシを作成した後は、`GetCountAsync` メソッドを呼び出して結果を返すだけです。

4. もう一度 F5 キーを押して、変更したアプリケーションを実行します。前と同じように、Visual Studio はブラウザーを自動的に起動して Web プロジェクトのルートを表示します。"api/values" パスを追加すると、現在のカウンター値が返されることがわかります。

    ![ブラウザーに表示されたステートフル カウンター値][browser-aspnet-counter-value]

    定期的にブラウザーを更新して、カウンターの値が更新されるのを確認します。


## アクターについて

このチュートリアルではステートフル サービスと通信する Web フロントエンドの追加を取り上げました。ただし、非常によく似たモデルに従ってアクターと対話することもできます。実際にはもう少し簡単です。

アクター プロジェクトを作成すると、Visual Studio によってインターフェイス プロジェクトが自動的に生成されます。そのインターフェイスを使用して Web プロジェクトでアクター プロキシを生成し、アクターと通信できます。通信チャネルは自動的に指定されます。そのため、このチュートリアルでステートフル サービスに関して行ったような `ServiceRemotingListener` の確立に相当するようなことは何も行う必要がありません。

## ローカル クラスター上の Web サービスのしくみ

一般に、ローカル クラスターにデプロイした複数コンピューター クラスターにまったく同じ Service Fabric アプリケーションをデプロイすると、高い確率で意図したとおりに動作させることができます。これは、ローカル クラスターが 5 ノードの構成を 1 つのコンピューターにまとめただけであるためです。

ただし、Web サービスに関しては 1 つの重要な違いがあります。クラスターが Azure でのようにロード バランサーの背後に配置されている場合、ロード バランサーはコンピューター間にトラフィックをラウンドロビンするだけなので、Web サービスをすべてのコンピューターにデプロイする必要があります。これは、サービスの `InstanceCount` に特別な値 "-1" を設定することによって実現できます。

これに対して、Web サービスをローカルで実行するときは、必ずサービスのインスタンスが 1 つだけ実行されているようにする必要があります。そうしないと、同じパスとポートでリッスンしている複数のプロセスから競合が発生します。そのため、ローカル デプロイの場合は Web サービスのインスタンス数を 1 に設定する必要があります。

異なる環境で異なる値を構成する方法については、「[複数の環境のアプリケーション パラメーターを管理する](service-fabric-manage-multiple-environment-app-configuration.md)」を参照してください。

## 次のステップ

- [アプリケーションをクラウドにデプロイするために Azure にクラスターを作成します](service-fabric-cluster-creation-via-portal.md)
- [サービスとの通信について学習します](service-fabric-connect-and-communicate-with-services.md)
- [ステートフル サービスのパーティション分割について学習します](service-fabric-concepts-partitioning.md)

<!-- Image References -->

[vs-add-new-service]: ./media/service-fabric-add-a-web-frontend/vs-add-new-service.png
[vs-new-service-dialog]: ./media/service-fabric-add-a-web-frontend/vs-new-service-dialog.png
[vs-new-aspnet-project-dialog]: ./media/service-fabric-add-a-web-frontend/vs-new-aspnet-project-dialog.png
[browser-aspnet-template-values]: ./media/service-fabric-add-a-web-frontend/browser-aspnet-template-values.png
[vs-add-class-library-project]: ./media/service-fabric-add-a-web-frontend/vs-add-class-library-project.png
[vs-add-class-library-reference]: ./media/service-fabric-add-a-web-frontend/vs-add-class-library-reference.png
[vs-services-nuget-package]: ./media/service-fabric-add-a-web-frontend/vs-services-nuget-package.png
[browser-aspnet-counter-value]: ./media/service-fabric-add-a-web-frontend/browser-aspnet-counter-value.png

<!---HONumber=AcomDC_0128_2016-->