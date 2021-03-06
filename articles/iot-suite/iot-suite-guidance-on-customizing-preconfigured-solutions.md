<properties
	pageTitle="構成済みソリューションのカスタマイズ | Microsoft Azure"
	description="Azure IoT Suite の構成済みソリューションのカスタマイズ方法に関するガイダンスを提供します。"
	services=""
    suite="iot-suite"
	documentationCenter=".net"
	authors="stevehob"
	manager="timlt"
	editor=""/>

<tags
     ms.service="iot-suite"
     ms.devlang="dotnet"
     ms.topic="article"
     ms.tgt_pltfrm="na"
     ms.workload="na"
     ms.date="09/29/2015"
     ms.author="stevehob"/>

# 構成済みソリューションのカスタマイズ

Azure IoT Suite で提供される構成済みソリューションを利用すれば、スイート内のサービスを連動させ、エンド ツー エンド ソリューションを提供できます。ここから出発し、いくつかのポイントで特定のシナリオのためにソリューションを拡張したり、カスタマイズしたりできます。次のセクションでは、これらの一般的なカスタマイズ ポイントの概要を示します。

## ソース コードの検索

構成済みソリューションのソース コードは、次のリポジトリの Github で入手できます。

- リモート監視: [https://www.github.com/Azure/azure-iot-remote-monitoring](https://github.com/Azure/azure-iot-remote-monitoring)

このソースは、Azure IoT Suite を使用して、リモート監視のコア機能を実装するためのパターンを示すために提供されます。

## 構成済みルールの変更

リモート監視ソリューションには、ダッシュボードに表示されるテレメトリおよびアラーム ロジックを実装するための 2 つの [Azure Stream Analytics](https://azure.microsoft.com/services/stream-analytics/) ジョブが含まれます。

最初のジョブでは、テレメトリの受信ストリームからのすべてのデータを選択し、2 つの異なる出力を作成します。このジョブには「**[ソリューション名]-テレメトリ**」という名前が付けられます。

- 最初の出力は `SELECT *` ですべてのデータを取得し、それを BLOB に出力します。この BLOB ストレージは、ダッシュボードがそのグラフを作成するために未処理の値を読み取る場所です。
- 2 番目の出力では、5 分間のスライディング ウィンドウで `AVG()`、`MIN()`、および `MAX()` の計算を行います。このデータはダッシュボードのダイヤルに表示されます。

Stream Analytics ユーザー インターフェイスを使用すれば、これらのジョブを直接編集し、ロジックを変更したり、シナリオに固有のロジックを追加したりすることができます。

2 番目のジョブでは、ソリューションの **[ルール]** ページで作成した [デバイスからしきい値] の値を操作します。このジョブは、参照データとして、デバイスごとに設定されるしきい値の値を使用します。そのしきい値の値を比較し、実際の値より大きい (`>`) かどうかを確認します。このジョブを修正し、たとえば、比較演算子を変更できます。

> [AZURE.NOTE] リモート監視ダッシュボードは特定のデータによって異なるため、ジョブを変更するとダッシュボードで障害が発生する可能性があります。

## 独自のルールの追加

構成済みの Azure Stream Analytics ジョブの変更だけでなく、Azure ポータルを使用して、新しいジョブを追加したり、新しいクエリを既存のジョブに追加したりできます。

## デバイスのカスタマイズ

最も一般的な拡張アクティビティの 1 つは、シナリオに固有のデバイスの操作です。デバイスを操作するためのいくつかの方法があります。これらの方法には、シナリオに合わせたシミュレーション対象デバイスの変更や、[IoT デバイス SDK][] を使用したソリューションへの物理デバイスの接続が含まれます。

デバイスをリモート監視の構成済みソリューションに追加する手順については、[IoT Suite とデバイスの接続](iot-suite-connecting-devices.md)に関するドキュメントを参照してください

### 独自のシミュレーション対象デバイスの作成

リモート監視ソリューション ソース コード (上記を参照) には .NET シミュレーターが含まれます。このシミュレーターはソリューションの一部としてプロビジョニングされるものであり、さまざまなメタデータやテレメトリを送信したり、さまざまなコマンドに応答したりするために変更できます。

また、Azure IoT には、リモート監視の構成済みソリューションと連動するように設計されている [C SDK サンプル](https://github.com/Azure/azure-iot-sdks/c/serializer/samples/remote_monitoring)があります。

### 独自の (物理) デバイスの構築と使用

[Azure IoT SDK](https://github.com/Azure/azure-iot-sdks) では、IoT ソリューションにさまざまな種類のデバイス (言語およびオペレーティング システム) を接続するためのライブラリが提供されます。

## 次のステップ

IoT デバイスの詳細については、[Azure IoT 開発者向けサイト](https://azure.microsoft.com/develop/iot/)でリンクとドキュメントを検索してください。

[IoT デバイス SDK]: https://azure.microsoft.com/documentation/articles/iot-hub-sdks-summary/

<!---HONumber=AcomDC_0128_2016-->