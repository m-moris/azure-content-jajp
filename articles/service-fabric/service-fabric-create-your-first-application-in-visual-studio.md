<properties
   pageTitle="Visual Studio で最初の Service Fabric アプリケーションを作成する | Microsoft Azure"
   description="Visual Studio を使用して Service Fabric アプリケーションを作成、デプロイ、デバッグする"
   services="service-fabric"
   documentationCenter=".net"
   authors="seanmck"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.devlang="dotNet"
   ms.topic="hero-article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA"
   ms.date="01/11/2016"
   ms.author="seanmck"/>

# Visual Studio で最初の Azure Service Fabric アプリケーションを作成する

Service Fabric SDK には、Service Fabric アプリケーションの作成、デプロイ、およびデバッグのためのテンプレートとツールを提供する Visual Studio 用アドインが含まれます。このトピックでは、Visual Studio で初めてのアプリケーションを作成するプロセスについて説明します。

## 前提条件

作業を開始する前に、[開発環境がセットアップ](service-fabric-get-started.md)されていることを確認してください。

## ビデオ チュートリアル

次のビデオでは、このチュートリアルの手順について説明します。

>[AZURE.VIDEO creating-your-first-service-fabric-application-in-visual-studio]

## アプリケーションを作成する

Service Fabric のアプリケーションには、アプリケーションの機能を提供する際にそれぞれ特定の役割を果たすサービスを 1 つ以上含めることができます。新しいプロジェクト ウィザードを使用すれば、アプリケーション プロジェクトと共に最初のサービス プロジェクトを作成できます。サービスは後で追加することができます。

1. Visual Studio を管理者として起動します。

2. **[ファイル]、[新しいプロジェクト]、[クラウド]、[Service Fabric アプリケーション]** の順にクリックします。

3. アプリケーションに名前を付けて、**[OK]** をクリックします。

	![Visual Studio の [新しいプロジェクト] ダイアログ][1]

4. 次のページでは、アプリケーションに含める最初のサービスの種類を選択するように求められます。このチュートリアルでは、**[ステートフル]** を選択します。それに名前を付けて、**[OK]** をクリックします。

	![Visual Studio の [新しいサービス] ダイアログ][2]

	>[AZURE.NOTE] オプションの詳細については、「[フレームワークの選択](service-fabric-choose-framework.md)」を参照してください。

	Visual Studio によって、アプリケーション プロジェクトとステートフル サービス プロジェクトが作成され、ソリューション エクスプ ローラーに表示されます。

	![アプリケーションとステートフル サービスの作成後に表示されるソリューション エクスプ ローラー][3]

	アプリケーション プロジェクトでは、コードは直接含まれません。代わりに、一連のサービス プロジェクトが参照されます。さらに、アプリケーション プロジェクトには、他に次の 3 つの種類のコンテンツが含まれます。

	- **発行プロファイル**: さまざまな環境に合わせてツールの基本設定を管理するために使用します。

	- **スクリプト**: アプリケーションをデプロイ/アップグレードするための PowerShell スクリプトが含まれます。このスクリプトは、Visual Studio ではバックグラウンドで使用されます。コマンド ラインで直接呼び出すことができます。

	- **アプリケーション定義**: アプリケーション マニフェストおよび関連するアプリケーション パラメーターのファイルが含まれます。これにより、アプリケーションが定義され、特定の環境専用にアプリケーションを構成することができます。

    サービス プロジェクトのコンテンツの概要については、「[Reliable Services の概要](service-fabric-reliable-services-quick-start.md)」を参照してください。

## アプリケーションをデプロイしデバッグする

これでアプリケーションの用意ができたので、実行してみることができます。

1. Visual Studio で F5 キーを押して、デバッグ用にアプリケーションをデプロイします。

	>[AZURE.NOTE] 完了までにしばらく時間がかかります。初めての場合、Visual Studio によって開発用のローカル クラスターが作成されるためです。ローカル クラスターでは、複数のコンピューターからなるクラスターで使用するのと同じプラットフォーム コードを 1 台のコンピューター上でのみ実行します。Visual Studio の出力ウィンドウにクラスターの作成状態が表示されます。

	クラスターの準備が整うと、SDK に含まれているローカル クラスター システム トレイ マネージャー アプリケーションから通知を受け取ります。

	![ローカル クラスター システム トレイ通知][4]

2. アプリケーションが起動すると、Visual Studio によって診断イベント ビューアーが自動的に呼び出されます。ここでは、サービスからトレース出力を確認することができます。

	![診断イベント ビューアー][5]

	ステートフル サービス テンプレートの場合、表示されるメッセージに含まれるのは、MyStatefulService.cs の `RunAsync` メソッド内でインクリメントされているカウンター値のみです。

3. コードが実行されているノードなど、詳細を確認するには、イベントのいずれかを展開します。次の場合、ノード 2 になっていますが、使用するコンピューターによって異なる可能性があります。

	![診断イベント ビューアーの詳細][6]

	ローカル クラスターには、1 台のコンピューターでホストされている 5 つのノードが含まれています。ノードがそれぞれ異なるコンピューター上にある 5 ノード クラスターを模倣します。ローカル クラスター上のノードのいずれかをダウンさせて、コンピューターの損失をシミュレートし、これと同時に Visual Studio デバッガーを実行します。

    >[AZURE.NOTE] プロジェクト テンプレートによって生成されたアプリケーション診断イベントでは、付属の `ServiceEventSource` クラスを使用します。詳細については、「[ローカルでのサービスの監視方法と診断方法](service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally.md)」を参照してください。

4. StatefulService から派生したサービス プロジェクト内のクラス (たとえば、MyStatefulService) を見つけ、`RunAsync` メソッドの最初の行にブレークポイントを設定します。

	![ステートフル サービスの RunAsync メソッド内のブレークポイント][7]

5. ローカル クラスター マネージャーのシステム トレイ アプリを右クリックし、**[ローカル クラスターの管理]** を選択して Service Fabric Explorer を起動します。

    ![ローカル クラスター マネージャーから Service Fabric Explorer を起動する][systray-launch-sfx]

    Service Fabric Explorer ではクラスターを視覚的に表現します。これには、クラスターにデプロイされた一連のアプリケーションや、クラスターを構成する一連の物理的なノードなども含まれます。Service Fabric Explorer の詳細については、「[クラスターの視覚化](service-fabric-visualizing-your-cluster.md)」を参照してください。

6. 左側のウィンドウで、**[クラスター]、[ノード]** の順に展開し、コードが実行されているノードを見つけます。

7. **[アクション]、[非アクティブ化 (再起動)]** の順にクリックし、コンピューターの再起動をシミュレートします。

	![Service Fabric Explorer でノードを停止する][sfx-stop-node]

	すぐに、Visual Studio にブレークポイント ヒットが表示されます。これは、1 つのノードで行っていた計算が別のノードにシームレスにフェールオーバーするためです。

8. 診断イベント ビューアーに戻り、メッセージを確認します。イベントが実際には別のノードから取得されている場合でも、カウンターが継続的にインクリメントされていることに注意してください。

    ![フェールオーバー後の診断イベント ビューアー][diagnostic-events-viewer-detail-post-failover]

### クリーンアップしています

  まとめに入る前に、ローカル クラスターが非常に現実的であることを覚えておくことが重要です。デバッガーを停止し、Visual Studio を閉じた後でも、アプリケーションはバック グラウンドで実行し続けます。アプリケーションの性質によっては、このバックグラウンド アクティビティはコンピューター上の大量のリソースを占有する場合があります。この問題は、いくつかのオプションで管理することができます。

  1. 個々のアプリケーションとそのデータのすべてを削除するには、Service Fabric Explorer で **[アプリケーションの削除]** を使用します。

  2. クラスターをシャット ダウンするが、アプリケーションのデータとトレースは保持するという場合は、システム トレイ アプリで **[クラスターの停止]** をクリックします。

  3. クラスターを完全に削除するには、システム トレイ アプリで **[クラスターの削除]** をクリックします。このオプションを使用すると、次回 Visual Studio で F5 キーを押したときに、デプロイメントがさらに遅くなることに注意してください。しばらくの間、ローカル クラスターを使用しない場合や、リソースを解放する必要がある場合にのみこれを使用してください。



## 次のステップ

- [Web サービス フロント エンドを使用してインターネットにサービスを公開する方法を参照](service-fabric-add-a-web-frontend.md)
- [Azure でクラスターを作成する方法の説明](service-fabric-cluster-creation-via-portal.md)
- [Reliable Services の詳細](service-fabric-reliable-services-quick-start.md)
- [Reliable Actors プログラミング モデルを使用してサービスを作成してみる](service-fabric-reliable-actors-get-started.md)

<!-- Image References -->

[1]: ./media/service-fabric-create-your-first-application-in-visual-studio/new-project-dialog.png
[2]: ./media/service-fabric-create-your-first-application-in-visual-studio/new-project-dialog-2.png
[3]: ./media/service-fabric-create-your-first-application-in-visual-studio/solution-explorer-stateful-service-template.png
[4]: ./media/service-fabric-create-your-first-application-in-visual-studio/local-cluster-manager-notification.png
[5]: ./media/service-fabric-create-your-first-application-in-visual-studio/diagnostic-events-viewer.png
[6]: ./media/service-fabric-create-your-first-application-in-visual-studio/diagnostic-events-viewer-detail.png
[7]: ./media/service-fabric-create-your-first-application-in-visual-studio/runasync-breakpoint.png
[sfx-stop-node]: ./media/service-fabric-create-your-first-application-in-visual-studio/sfe-deactivate-node.png
[systray-launch-sfx]: ./media/service-fabric-create-your-first-application-in-visual-studio/launch-sfx.png
[diagnostic-events-viewer-detail-post-failover]: ./media/service-fabric-create-your-first-application-in-visual-studio/diagnostic-events-viewer-detail-post-failover.png

<!---HONumber=AcomDC_0218_2016-->