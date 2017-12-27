---
title:   Evaluate a model in an Azure WebApi using CNTK Library Managed API
author:    chrisbasoglu
ms.author:   cbasoglu
ms.date:  07/31/2017
ms.custom:   cognitive-toolkit
ms.topic:   conceptual
ms.service:  Cognitive-services
ms.devlang:   dotnet
---

# CNTK Library Managed API を使って Azure WebApi 上でモデルを評価する

## Azure Machine Learning コマンドラインでのデプロイ

Azure に CNTK のモデルをデプロイし Web API を通して実行する方法の一つに、Azure Machine Learning のコマンドラインインターフェースがあります。詳細については、[こちら](https://docs.microsoft.com/ja-jp/azure/machine-learning/preview/scenario-image-classification-using-cntk)を参照してください。

## ASP.NET でデプロイ

Azure に CNTK のモデルをデプロイし、Azure のエンドポイントにリクエストを送信してデプロイされたモデルでデータを評価する手順を説明しましょう。この機能を、Azure の基本的な機能にフォーカスして WebApi のフォームでビルドします。パラメーターの渡し方といった詳細な機能については、　Azure のドキュメントを参照してください。


## 前提条件

現在、VS2015 で CNTK をビルドしているため、このバージョンにフォーカスしています。

#### Visual Studio の Web 開発機能

Visual StudioでWeb開発機能を有効にする必要があります。 この機能が有効かどうかは、VS installer を実行して確認できます(`コントロールパネル -> プログラムと機能 -> Microsoft Visual Studio 201x`, 右クリックし `変更` をクリックして VS installer を実行)。

#### Azure SDK

開発するマシンには he Azure SDK for .NET のインストールが必要です。ダウンロードページはこちらです: [Azure SDK Download](https://azure.microsoft.com/downloads/)

#### Azure Account

Azure に CNTK モデルをホストするため、Azure アカウントが必要です。MSDN か Visual Studio subscription をお持ちの場合、このチュートリアルでのモデルのホストが可能です。CNTK をホストするのには、64ビットの仮想マシンが必要のため、無料の Azure アカウントではできません。

最初に WebApi をローカルで開発し、それから Azure のインスタンスにアップロードします。 そのため、Azure にアクセスしなくてもほとんどの手順を行うことができます。

### やってみまよう

既に **[CNTKAzureTutorial01](https://github.com/Microsoft/CNTK/tree/master/Examples/Evaluation/CNTKAzureTutorial01)** というプロジェクトの青写真があります。このプロジェクトは CNTK の GitHub リポジトリの一部であり、 `Examples \ Evaluation \ CNTKAzureTutorial01` フォルダにあります。

**サンプルプロジェクトは CNTK Library Managed API を使用しています。EvalDll APIを使用するには、モデルを評価するために、[CNTK EvalDLL API](./evaldll-evaluation-on-windows.md) を使用してください。EvalDll の使用に関するチュートリアルは、[Evaluate a model in Azure WebApi using EvalDll](./evaluate-a-model-in-an-azure-webapi-using-evaldll.md) にあります。**

必要なコードをすべて追加してから、このソリューションを開始することをお勧めします。ここでは、チュートリアルのプロジェクトを作成する操作の一覧は以下です。

- Visual Studio で `ファイル -> 新規作成 -> プロジェクト -> Visual C# -> Web -> ASP.NET ウェブアプリケーション ` から新しいプロジェクト/ソリューション CNTKAzureTutorial01 を作成します。Azure API App テンプレートを選択し、'Web API'への参照を追加し、ローカルにホストされることをを確認します（まだクラウドにホストされていません）。

- 次にチュートリアルプロジェクトをビルドするために、以下のコード変更を行います：
    - ValueController.cs で必要なディレクティブをいくつか追加します。
    - `public async Task<IEnumerable<string>> Get()` のコードを、実際に CNTK の評価関数が呼ばれるように変更します。
    - `public async Task<string[]> EvaluateCustomDNN(string imageUrl)` 関数を追加する。. この関数は、CNTK 評価サンプル（ CNTK リポジトリの Example\Evaluation\CSEvalClient\Program.cs ファイルの EvaluateImageClassificationModel メソッド）から採用しました。
    - ビットマップのリサイズ機能を追加するために、`CNTKImageProcessing.cs` ファイルを追加しました。これは、CNTK リポジトリの Examples\Evaluation\ImageExtension\CNTKImageProcessing.cs の名前空間とクラス名です。
    - ソリューションで作成されたバイナリーのディレクトリは、アプリケーションの環境変数に追加する必要があります。プロジェクトにはネイティブの DLL が含まれており、標準の検索パスで到達可能な場合にのみ読み込まれますので、この追加が必要です。`global.asax` の `Application_Start()` メソッドに次のコードを追加します。
   
             string pathValue = Environment.GetEnvironmentVariable("PATH");
             string domainBaseDir = AppDomain.CurrentDomain.BaseDirectory;
             string cntkPath = domainBaseDir + @"bin\";
             pathValue += ";" + cntkPath;
             Environment.SetEnvironmentVariable("PATH", pathValue);
   
           
### ローカルの WebApi にホストする

これまで変更を行ってきました。プロジェクトに含まれている CNTK の評価機能を取得し、評価する必要があります。

プロジェクトに CNTK 評価機能を追加しましょう。ここでは、NuGet パッケージを使います。Visual Studio で、`ツール -> NuGet パッケージ マネージャー -> ソリューションの Nuget パッケージの管理` を選択し、オンラインのソースで `nuget.org` を選択し `CNTK` を検索します。そして、最新バージョン（`CNTK.GPU` または `CNTK.CPUOnly`）をプロジェクトにインストールします。

![NuGet](./pictures/EvaluateWebApiCntkLibrary/nuget_manager.png)

次は、モデルの評価です。[ResNet20_CIFAR10_CNTK.model](https://www.cntk.ai/Models/CNTK_Pretrained/ResNet20_CIFAR10_CNTK.model) をダウンロードし、プロジェクトフォルダーの `CNTK\Models` ディレクトリに保存します。

CNTK は、64ビットの実行環境が必要です。構成マネージャーで、プロジェクトが x64 プラットフォーム用にコンパイルされていることを確認してください。さらに、作った WebApi を、64ビットのIISのインスタンスでホストする必要があります。`ツール -> オプション -> プロジェクトおよびソリューション -> Webプロジェクト` で、"Web サイトおよびプロジェクト用のIIS Express の 64ビット バージョンを使用する" が選択して強制することができます。

![Project](./pictures/EvaluateWebApiCntkLibrary/setting_64_bits_in_vs.png)

これでローカルのマシンでモデルを評価するのに必要なセットアップが全てできました。Visual Studio で　`F5` を押してプロジェクトを実行しましょう。デフォルトのウェブサイトがインターネットのブラウザで開き、エラーメッセージが表示されます。これは、ウェブサイトではなく WebApi を作成したので想定通りです。実装したWebApiは、ブラウザのアドレスを次のように変更することで簡単に呼び出せます：

`http://localhost:<portnumber>/api/values`

ValuesController クラスの `Get()` メソッドが呼ばれ、`EvaluateCustomDNN()` メソッドが呼ばれてブラウザに結果が返ってきます。

![local](./pictures/EvaluateWebApiCntkLibrary/local_webapi_evaluation.png)

### Azure上の WebApi にホストする

これで1爪のミッションを達成しました。次は、Azure上にこの機能をホストします。プロジェクトのビルドメニューで、`発行` コマンドを選択します。Pick 発行のターゲットは、`Microsoft Azure App Service` を選択します。
 
![Azure](./pictures/EvaluateWebApiCntkLibrary/publishing_webapp.png)

AppServiceダイアログでアカウントにログインし、適切なサブスクリプションとリソースグループを選択する必要があります。64ビットの仮想マシンをサポートしているリソースグループを選択してください（このためには、'Free' のリソースグループでは不十分です）。 発行の最後のステップでは、設定メニューで x64 の構成を選択する必要があります。これは、Azure に CNTK のバイナリーコンポーネントを発行するために必要です。

![AzureSettings](./pictures/EvaluateWebApiCntkLibrary/publishing_step.png)

モデルを発行してブラウザーでWebApiを呼ぶと、エラーメッセージが表示されます。Azure ポータルを開き、WebApi が 64 ビットプラットフォーム上で動作していることを確認します（必要に応じて設定を変更し、'保存' します。これにより Azure の仮想マシンのインスタンスも再起動します）。

![Azure64Settings](./pictures/EvaluateWebApiCntkLibrary/setting_64_bits_in_portal.png)

これらの変更を実行すると、次のURL で WebApi を呼び出すことができます。
`http://<yourwebapp>.azurewebsites.net/api/values`
 
![AzureSettings](./pictures/EvaluateWebApiCntkLibrary/remote_webapi_evaluation.png)

このプロジェクトで、Azure の WebApi で CNTK Library Managed API を使用して CNTK 評価機能を統合し、Azure で CNTK 評価バイナリを実行する設定方法を示しました。次のステップでは、新しいAPIを追加して、評価機能にデータを動的に提供したり、モデルの新しいバージョンをアップロードします。これらは WebApi / Azure の開発タスクです。これについては Azure のドキュメントを参照してください。

