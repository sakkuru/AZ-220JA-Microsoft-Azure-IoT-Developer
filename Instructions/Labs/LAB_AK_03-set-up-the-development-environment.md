---
lab:
    title: '課題 03: 開発環境の設定'
    module: 'モジュール 2： デバイスとデバイス通信'
---

# 開発環境の設定

## ラボシナリオ

Contoso の開発者の 1 人として、Azure IoT ソリューションの構築を開始する前に、開発環境をセットアップすることが重要なステップであることをご存じでしょう。Microsoft  には、IoT ソリューションの開発とサポートに使用できるツールが多数用意されており、チームがどのツールを使用するかについて決定する必要があることをご存じでしょう。チームが IoT ソリューションの開発に使用できる作業環境を、Azure クラウド側とローカルの作業環境の両方で準備します。

議論した後に、チームは開発環境について次の大まかな決定を下しました。

* オペレーティング システム:  Windows 10 は OS として使用されます。Windows はほとんどのチームで使用されているので、論理的な選択でした。Azure IoT サービスは他のオペレーティング システム (Mac OS や Linux など) をサポートしており、Microsoft では、これらの代替案のいずれかを選択したチームのメンバー向けにサポート ドキュメントが用意されています。
* 一般的なコーディング ツール:  Visual Studio Code と Azure CLI は、主要なコーディング ツールとして使用されます。これらのツールでは両方とも、Azure IoT SDK を活用する IoT の拡張機能がサポートされます。
* IoT Edge ツール: Docker Desktop コミュニティと Python は、カスタムの IoT Edge モジュールの開発をサポートするために使用されます。

これらの決定を支持して、次の環境を設定します。

* Windows 10 64 ビット: Pro、Enterprise、または Education (ビルド 15063 以降)。下記が含まれます。
  * 4 GB – 8 GB システム RAM (Docker の場合は高い方が良い)
  * Windows の Hyper-V およびコンテナの機能を有効にする必要があります。
  * BIOS レベルのハードウェア仮想化サポートは、BIOS 設定で有効にする必要があります。

  > **注意**: 仮想マシンに開発環境を設定する場合、VM 環境は入れ子になった仮想化 [入れ子になった仮想化](https://docs.microsoft.com/ja-jp/virtualization/hyper-v-on-windows/user-guide/nested-virtualization) をサポートする必要があります。

* Azure CLI (現在/最新)
* .NET Core 3.1.200 (またはそれ以降) の SDK
* VS Code (最新)
* Python 3.7 (3.8 ではない)
* Linux コンテナーに設定された Docker Desktop Community 2.1.0.5 以降
* Power BI Desktop (データ視覚化用)
* VS Code と Azure CLI の IoT 拡張機能
* node.js (latest)

> **注意**: 仮想マシンは、このコースに向けて作成されました。ここでは、上記で指定されているツールの大部分を提供しています。以下の手順では、準備済み VM の使用、または Windows PC をローカルで使用する環境の設定をサポートしています。

## このラボで

この課題では、次の内容を学習します。

* このコースの課題で使用する基本ツールおよび製品をインストールします。
* Azure CLI および Visual Studio Code 向けの Azure IoT 拡張機能をインストールします。
* 開発環境のセットアップを確認する

## ラボの手順

### 演習 1: 開発者ツールと製品のインストール

> **注意**: この演習に関連付けられているツールと製品は、このコース用に作成された仮想マシンに事前インストールされています。続行する前に、ホストされたラボの VM 環境を使用してラボを完了するのか、またはお使いの PC にローカルで開発環境を設定するのかについて、コース担当講師に確認してください。

#### タスク 1: .NET Core のインストール

.NET Core は、Web サイト、サービス、およびコンソール アプリを構築するためのクロスプラットフォーム バージョンの .NET です。

1. .NET Core ダウンロード ページを開くには、次のリンクを使用します。[.NET のダウンロード](https://dotnet.microsoft.com/download)

1. .NET ダウンロード ページの 「.NET Core」 で、「**.NET Core SDK のダウンロード**」 をクリックします。 

    .NET Core SDK は、.NET Core アプリのビルドに使用されます。このコースのラボでは、コード ファイルをビルドしたり、編集したりするために、is を使用します。

    Windows コンピューターで .NET Core を使用するアプリを実行するだけで済む場合は、.NET Core ランタイムをインストールできます。また、プレビュー版と従来のバージョンの .NET Core をインストールするには、「すべての .NET Core のダウンロード..」 のリンクを使用します。

1. ポップアップ メニューの **「実行」** をクリックし、画面の指示に従ってインストールを完了します。 

    インストールは、1 分未満で完了します。次のコンポーネントがインストールされます。

    * .NET Core SDK 3.1.100 以降
    * .NET Core Runtime 3.1.100 以降
    * ASP.NET Core Runtime 3.1.100 以降
    * .NET Core Windows Desktop Runtime 3.1.0 以降

    詳細については、以下のリソースを参照することができます。

    * [.NET Core に関するドキュメント](https://aka.ms/dotnet-docs-jpn)
    * [.NET Core の依存関係と要件](https://docs.microsoft.com/ja-jp/dotnet/core/install/dependencies?tabs=netcore31&pivots=os-windows)
    * [SDK に関するドキュメント](https://aka.ms/dotnet-sdk-docs-jpn)
    * [リリース ノート](https://aka.ms/netcore3releasenotes)
    * [チュートリアル:](https://aka.ms/dotnet-tutorials-jpn)

#### タスク 2: Visual Studio Code のインストール

Visual Studio Code は、デスクトップ上で実行され、Windows、macOS、および Linux で使用できる、軽量で強力なソース コード エディターです。JavaScript、TypeScript、Node.js 向けの組み込みのサポートが付属しており、他の言語(C++、C#、Java、Python、PHP、Go など) や実行時間 (.NET、Unity など) の拡張機能の豊富なエコシステムを備えています。

1. 「Visual Studio Code のダウンロード」 ページを開くには、次のリンクをクリックします。[Visual Studio Code のダウンロード](https://code.visualstudio.com/Download)

    Mac OS X および Linux に Visual Studio Code をインストールする手順については、[こちら](https://code.visualstudio.com/docs/setup/setup-overview) の Visual Studio Code セットアップ ガイドを参照してください。このページには、Windows のインストールに関する詳細な手順とヒントも記載されています。

1. 「Visual Studio Code のダウンロード」 ページで、**Windows** をクリックします。

    ダウンロードを開始すると、2 つのことが起こります。ポップアップ ダイアログが開き、開始ガイダンスが一部表示されます。

1. ポップアップ ダイアログでセットアップ プロセスを開始するには、**「実行」** をクリックし、画面の指示に従います。 

    インストーラをダウンロード フォルダに保存する場合は、フォルダを開いて、VSCodeSetup 実行可能ファイルをダブルクリックして、インストールを完了できます。

    既定では、Visual Studio Code は "C:\Program Files (x86)\Microsoft VS Code" フォルダーの場所 (64 ビット コンピューターの場合) にインストールされます。このセットアップ プロセスは、約 1 分間かかるのみです。

    > **注意**:  .NET Framework 4.5 は、Windows にインストールするときに、Visual Studio Code が必要です。Windows 7 を使用している場合は、[.NET Framework 4.5](https://www.microsoft.com/ja-jp/download/details.aspx?id=30653) がインストールされていることを確認してください。

    Visual Studio Code をインストールする詳細な手順については、Microsoft Visual Studio Code インストール手順ガイドを参照してください。[https://code.visualstudio.com/Docs/editor/setup](https://code.visualstudio.com/Docs/editor/setup)

#### タスク 3: Azure CLI のインストール

Azure CLI 2.2 は、Azure 関連のスクリプト作成タスクを簡単に実行できるように設計されたコマンドライン ツールです。また、柔軟にデータをクエリすることができ、非ブロッキング プロセスとして長時間実行される操作をサポートします。

1. ブラウザーを開き、「Azure CLI 2.2 ツールのダウンロード」 ページに移動します。[Azure CLI 2.2 のインストール](https://docs.microsoft.com/ja-jp/cli/azure/install-azure-cli?view=azure-cli-latest "Azure CLI 2.2 Install")

    Azure CLI ツールの最新バージョンをインストールする必要があります。バージョン 2.2 が、この 「"azure-cli-latest" ダウンロード」 ページに記載されている最新バージョンでない場合は、より新しいバージョンをインストールしてください。

1. 「Azure CLI  2.2 のインストール」 ページで、OS のインストール オプションを選択し、画面の指示に従って Azure CLI ツールをインストールします。

    このコースのラボで、Azure CLI 2.2 ツールを使用する手順の詳細について説明しますが、さらなる詳細については、[Azure CLI 2.2 の使用を開始する](https://docs.microsoft.com/ja-jp/cli/azure/get-started-with-azure-cli?view=azure-cli-latest)を参照してください

#### タスク 4: Python 3.7 のインストール

IoT Edge と Docker をサポートするために Python 3.7 を使用します。 

1. Web ブラウザで、[https://www.python.org/downloads/](https://www.python.org/downloads/) に移動します

1. 「特定のリリースを探す」 の下にある Python 3.7.6 の右側にある **「ダウンロード」** をクリックします。

1. Python 3.7.6 ページで、ページの 「ファイル」 セクションまでスクロールします。

1. 「ファイル」 で、お使いのオペレーティング システムに適したインストーラー ファイルを選択します。

1. プロンプトが表示されたら、インストーラを実行するオプションを選択します。

1. 「Python 3.7.6 のインストール」 ダイアログで、**「Python 3.7 を PATH に追加」**  をクリックします。 

1. **「今すぐインストール」** をクリックします。

1. 「セットアップが成功しました」 ページが表示されたら、**「パスの長さの制限を無効にする」** をクリックします。 

1. インストール プロセスを完了するには、**「閉じる」** をクリックします。 

#### タスク 5:  Docker Desktop のインストール

IoT Edge モジュールの展開を扱う課題では、Docker Desktop コミュニティ 2.1.0.5 以降を Linux コンテナに設定して使用します。

1. Web ブラウザで、[https://docs.docker.com/docker-for-windows/install/](https://docs.docker.com/docker-for-windows/install/) に移動します:

    左側のナビゲーション メニューから、追加のオペレーティング システムのインストールにアクセスできます。

1. Windows PC がシステム要件を満たしていることを確認します。

    Windows の設定を使用して、「Windows の機能」ダイアログ ボックスを開き、Hyper-V とコンテナが有効になっていることを確認できます。

1. **「Docker ハブからダウンロード」** をクリックします

1. 「Windows 用の Docker Desktop」 で、**「Windows 用 Docker Desktop を取得する (安定版)」** をクリックします。 

1. インストールを開始するには、**「実行」** をクリックします。

    Docker Desktop のインストール ダイアログを表示するには、少し時間がかかることがあります。

1. インストールに成功しましたというメッセージが表示されたら、 **「閉じる」** をクリックします。 

    インストール後、Docker Desktop は自動的に起動しません。Docker Desktop を起動するには、"Docker" を検索し、検索結果で 「Docker Desktop」 を選択します。ステータス バーのホエール アイコンが安定している場合、Docker Desktop は稼働中で、ターミナル ウィンドウからアクセスできます。
    
#### タスク6 - node.jsのインストール

一部のサンプルWebアプリケーションは、node.jsを使用してローカルで実行されます。以下の手順では、node.jsがインストールされ、最新のバージョンが実行されていることを確認します。

1. ブラウザを使用して、[node.js ダウンロードページ](https://nodejs.org/en/#home-downloadhead)を開きます。

1. 最新のLTS（Long Term Support）バージョン（記事執筆時点では14.16.0）をダウンロードする。

1. プロンプトが表示されたら、インストーラーを実行するオプションを選択する。

1. インストーラーを実行します。

   * **End-User License Agreement** - 規約に同意して **Next** をクリックします。
   * **Destination Folder** - デフォルトを受け入れて（必要であれば変更して）、**Next**をクリックします。
   * **カスタムセットアップ** - デフォルトを受け入れ、**次へ**をクリックします。
   * **ネイティブモジュール用のツール** - **自動的にインストールする**にチェックを入れ、**次へ**をクリックします。
   * **Ready to install Node.js** - **Install**をクリックします。
     * UACダイアログで**Yes**をクリックします。

1. インストールが完了するのを待って、「**Finish**」をクリックします。

1. **Install Additional Tools for Node.js**コマンドウィンドウで、プロンプトが表示されたら、**Enter**を押して続行します。

1. UACダイアログで、**Yes**をクリックします。

    複数のパッケージのダウンロードとインストールが行われます。この作業には時間がかかります。

1. インストールが完了したら、新しい**コマンドシェルを開き、以下のコマンドを入力してください。

    ```powershell
    node --version
    ```

    nodeのインストールが成功すると，インストールされたバージョンが表示されます。

### エクササイズ 2: 開発ツール拡張機能のインストール

Visual Studio Code と Azure CLI ツールはどちらも、開発者がソリューションをより効率的に作成するのに役立つ拡張機能をサポートします。Microsoft によって開発された IoT の拡張機能は、IoT SDK を活用し、開発期間を短縮します。

#### タスク 1: Visual Studio Code 拡張機能のインストール

1. Visual Studio Code を起動します。

1. 「Visual Studio Code」 ウィンドウの左側で、**「拡張機能」** をクリックします。

    「拡張機能」 ボタンは、上から 6 番目です。ボタンの上にマウス ポインターを置くと、ボタンのタイトルを表示できます。

1. Visual Studio Code の拡張機能マネージャーで、次の拡張機能を検索して、インストールします。

    * Microsoft による [Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools) (`vsciot-vscode.azure-iot-tools`)
    * Microsoft による [C# for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-vscode.csharp) (`ms-vscode.csharp`)

#### タスク 2: Azure CLI 拡張機能をインストールする

1. 新しいコマンドライン/ターミナル ウィンドウを開きます。

1. コマンド プロンプトで、IoT 用の Azure CLI 拡張機能をインストールするには、次のコマンドを入力します。

    ```bash
    az extension add --name azure-cli-iot-ext
    ```

#### タスク 3: 開発環境の設定を確認する

開発環境が正常に設定されたことを確認する必要があります。これが完了すると、IoT ソリューションの構築を開始する準備が整います。

1. 新しいコマンドライン/ターミナル ウィンドウを開きます。

1. 現在インストールされているバージョンの Azure CLI のバージョン情報を出力する次のコマンドを実行して、**Azure CLI** のインストールを検証します。 

    ```cmd/sh
    az --version
    ```

    `az --version` コマンドは、インストールした Azure CLI のバージョン情報 (`azure-cli` バージョン番号) を出力します。このコマンドにより、IoT 拡張機能を含む、インストールされているすべての Azure CLI モジュールのバージョン番号も出力されます。次のような出力が表示されます。

    ```cmd/sh
    azure-cli                           2.20.0
    core                                2.20.0
    telemetry                           1.0.6
    Extensions:
    azure-iot                           0.10.9
    ```

1. 次のコマンドを実行して、現在インストールされているバージョンの .NET Core SDK のバージョン番号を出力して、**NET Core 3.x SDK** のインストールを検証します。

    ```cmd/sh
    dotnet --version
    ```

1. `dotnet --version` コマンドは、現在インストールされている .NET Core SDK のバージョンを出力します。これは .NET Core 3.1 以上である必要があります。

開発環境を設定する必要があります。

### エクササイズ 3: コースの課題ファイルと代替ツールの設定

このコースの多くのラボでは、課題活動の開始点として使用できるコード プロジェクトなど、あらかじめ構築されたリソースを使用しています。GitHub プロジェクトを使用して、これらのラボ リソースへのアクセスを提供します。コース ラボ (GitHub プロジェクトに含まれるリソース) を直接サポートするリソースに加えて、実際のコース以外の学習機会をサポートするために使用できるツールもあります。以下の手順は、これらのリソースの種類の両方の構成を示しています。

#### タスク 1: コース課題ファイルをダウンロードする

Microsoft は、ラボ リソース ファイルへのアクセスを提供する GitHub リポジトリを作成しました。これらのファイルを開発環境でローカルに保有することが、一部の場合では必要であり、その他の多くの場合では便利です。このタスクでは、開発環境内でリポジトリの内容をダウンロードして抽出します。

1. Web ブラウザーで、次の場所に移動します。[https://github.com/MicrosoftLearning/AZ-220-Microsoft-Azure-IoT-Developer](https://github.com/MicrosoftLearning/AZ-220-Microsoft-Azure-IoT-Developer)

1. ページの右側で、**「複製またはダウンロード」** をクリックし、**「ZIP のダウンロード」** をクリックします。   

1. ZIP ファイルを開発環境に保存するには、**「保存」** をクリックします。 

1. ファイルを保存したら、**「フォルダを開く」** をクリックします。

1. 保存した ZIP ファイルを右クリックし、「**すべて抽出**」 をクリックします

1. 「**参照**」 をクリックし、アクセスに便利なフォルダーの場所に移動します。 

1. ファイルを抽出するには、「**抽出**」 をクリックします。 

    ファイルの場所をメモしておきます。

#### タスク 2: Azure PowerShell モジュールのインストール

> **注意**: このコースの課題アクティビティでは、PowerShell を使用しませんが、PowerShell を使用するリファレンス ドキュメントにサンプル コードが記載されている場合があります。PowerShell コードを実行する場合は、次の手順を使用してインストール手順を完了できます。

Azure PowerShell は、PowerShell コマンド ラインから直接 Azure リソースを管理するためのコマンドレットのセットです。Azure PowerShell は、簡単に学習し、使用を開始できるように設計されていながら、自動化のための強力な機能を提供しています。.NET 標準で記述された Azure PowerShell は、Windows 上では PowerShell 5.1、すべてのプラットフォームでは PowerShell 6.x 以降で動作します。

> **警告**:  Windows 用 PowerShell 5.1 用の AzureRM モジュールと Az モジュールの両方を、同時にインストールできません。システムで AzureRM を使用可能な状態に保つ必要がある場合は、PowerShell Core 6.x 以降の Az モジュールをインストールします。これを行うには、PowerShell Core 6.x 以降をインストールした後、PowerShell Core ターミナルでこれらの手順に従います。

1. Azure PowerShell モジュールを、現在のユーザーのみに対してインストールするか (推奨されるアプローチ)、すべてのユーザーに対してインストールするかを決定します。

1. 選択した PowerShell ターミナルを起動する - すべてのユーザーに対してインストールする場合、「**管理者として実行**」 を選択するか、macOS または Linux で **sudo** コマンドを使用することで、管理者特権で PowerShell セッションを起動する必要があります。

1. 現在のユーザーに対してのみインストールするには、次のコマンドを入力します。

    ```powershell
    Install-Module -Name Az -AllowClobber -Scope CurrentUser
    ```

    または、システム上のすべてのユーザーに対してインストールするには、次のコマンドを入力します。

    ```powershell
    Install-Module -Name Az -AllowClobber -Scope AllUsers
    ```

1. 既定では、PowerShell ギャラリーは PowerShellGet の信頼されたリポジトリとして構成されていません。PowerShell ギャラリーを初めて使用すると、次のプロンプトが表示されます。

    ```output
    信頼されていないリポジトリ

    信頼されていないリポジトリからモジュールをインストールしようとしています。このリポジトリを信頼している場合は、
    Set-PSRepository コマンドレットを実行して、InstallationPolicy の値を変更してください。

    「PSCallery」からモジュールをインストールしますか。
    「Y」 はい 「A」 すべてにはい 「N」 いいえ 「L」 すべてにいいえ 「S」 中断する 「?」ヘルプ (既定は "N"):
    ```

1. インストールを続行するには、「**はい**」 または 「**すべてにはい**」 と答えます。

    AZ モジュールは、Azure PowerShell コマンドレットのロールアップ モジュールです。これをインストールすると、使用可能な Azure Resource Manager モジュールがすべてダウンロードされ、そのコマンドレットを利用できるようになります。

> **注意**: **AZ** モジュールが既にインストールされている場合は、次の方法で最新バージョンに更新できます。
> 
> ```powershell
> Update-Module -Name Az
> ```
