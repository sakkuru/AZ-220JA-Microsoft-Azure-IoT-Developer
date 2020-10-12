---
lab:
    title: 'ラボ 04: IoT デバイスを Azure に接続する'
    module: 'モジュール 2: デバイスとデバイス通信'
---

# IoT デバイスを Azure に接続する

## ラボシナリオ

Contoso は、高品質のチーズを生産することで知られています。同社の人気と販売の両方が急成長しているため、彼らは顧客の期待に応える高品質なチーズを確実に維持するための方策を講じたいと考えています。

昔は、各勤務シフト中に工場の現場担当者が温度と湿度のデータを収集していました。同社は、新しい施設が稼働し始めるにつれて、工場の拡張により監視の強化が必要になることと、データを収集する手動プロセスでは調整できなくなることを懸念しています。

Contoso は温度と湿度を監視するために、IoT デバイスを使用する自動化システムを立ち上げることにしました。テレメトリ データの通信速度は調整可能で、チーズのバッチが環境に敏感なプロセスを進める際に製造プロセスが確実に制御されるようにします。

この資産監視ソリューションをフルスケールの実装前に評価するには、IoT デバイス (温度センサーと湿度センサーを含む) を IoT Hub に接続します。このラボでは、.NET Core コンソール アプリケーションを使用して実際の IoT デバイスをシミュレートします。

次のリソースが作成されます。

![ラボ 4 アーキテクチャ](media/LAB_AK_04-architecture.png)

## このラボで

このラボでは、次のタスクを正常に達成します。

* ラボの前提条件が満たされていることを確認する (必要な Azure リソースがあること)
* Azure CLI を使用して Azure IoT Hub でデバイス ID を登録する
* シミュレートされた IoT デバイス (事前に構築され、C# で記述) を構成して Azure IoT Hub に接続する
* シミュレートされたデバイスを実行して、device-to-cloud へのテレメトリ メッセージを Azure IoT Hub に送信します
* Azure CLI を使用してデバイスのテレメトリが Azure IoT Hub によって受信されていることを確認する

## ラボの手順

### 演習 1: ラボの前提条件を確認する

このラボでは、次の Azure リソースが使用可能であることを前提としています。

| リソースの種類:  | リソース名 |
| :-- | :-- |
| リソース グループ | rg-az220 |
| IoT Hub | iot-az220-training-{your-id} |

これらのリソースが利用できない場合は、演習 2 に進む前に、以下の指示に従って **lab04-setup.azcli** スクリプトを実行する必要があります。スクリプト ファイルは、開発環境構成 (ラボ 3) の一部としてローカルに複製した GitHub リポジトリに含まれています。

> **注意**:  **lab04-setup.azcli** スクリプトは、**Bash** シェル環境で実行するために記述されています。Azure Cloud Shell でこれを実行するのが、最も簡単な方法です。 

1. ブラウザーを使用して [Azure Cloud Shell](https://shell.azure.com/) を開き、このコースで使用している Azure サブスクリプションでログインします。

1. Cloud Shell のストレージの設定に関するメッセージが表示された場合は、デフォルトをそのまま使用します。

1. Azure シェルが **Bash** を使用していることを確認します。

    「Azure Cloud Shell」 ページの左上隅にあるドロップダウンは、環境を選択するために使用されます。選択されたドロップダウンの値が **Bash**であることを確認します。 

1. Azure Shell ツール バーで、「**ファイルのアップロード/ダウンロード**」 をクリックします(右から 4番目のボタン)。

1. ドロップダウンで、「**アップロード**」 をクリックします。

1. ファイル選択ダイアログで、開発環境を構成したときにダウンロードした GitHub ラボ ファイルのフォルダーの場所に移動します。

    このコースのラボ 3 にあたる「開発環境のセットアップ」では、ZIP ファイルをダウンロードしてコンテンツをローカルに抽出することで、ラボ リソースを含む GitHub リポジトリを複製しました。抽出されたフォルダー構造には、次のフォルダー パスが含まれます。
    
    * Allfiles
      * Labs
          * 04-Connect an IoT Device to Azure
            * Setup

    lab04-setup.azcli スクリプト ファイルは、ラボ 4 の設定フォルダー内にあります。

1. **lab04-setup.azcli** ファイルを選択し、「**開く**」 をクリックします。   

    ファイルのアップロードが完了すると、通知が表示されます。

1. 正しいファイルがアップロードされたことを確認するには、次のコマンドを入力します。

    ```bash
    ls
    ```

    `ls` コマンドを使用して、現在のディレクトリの内容を表示します。一覧にある lab04-setup.azcli ファイルを確認できるはずです。

1. セットアップ スクリプトを含むこのラボのディレクトリを作成し、そのディレクトリに移動するには、次の Bash コマンドを入力します。

    ```bash
    mkdir lab4
    mv lab04-setup.azcli lab4
    cd lab4
    ```

    これらのコマンドは、このラボのディレクトリを作成し、**lab04-setup.azcli** ファイルをそのディレクトリに移動させ、新しいディレクトリを現在の作業ディレクトリにするための変更を行います。

1. **lab04-setup.azcli** に実行権限があることを確認するには、次のコマンドを入力します。 

    ```bash
    chmod +x lab04-setup.azcli
    ```

1. Cloud Shell ツール バーで **lab04-setup.azcli** ファイルを編集するには、「**エディターを開く**」 (右から 2 番目のボタン - **{ }**) をクリックします。   

1. 「**ファイル**」 の一覧で lab4 フォルダーを展開し、「**lab4**」 をクリックし、「**lab04-setup.azcli**」 をクリックします。

    エディタは **lab04-setup.azcli** ファイルの内容を表示します。

1. エディターで、`{YOUR-ID}` と `{YOUR-LOCATION}` 変数の値を更新します。

    例として以下のサンプルを参照し、このコースの開始時に作成した一意の ID、つまり **CAH191211** に `{YOUR-ID}` を設定し、リソース グループと一致する場所に `{YOUR-LOCATION}` を設定する必要があります。

    ```bash
    #!/bin/bash

    RGName="rg-az220"
    IoTHubName="iot-az220-training-{your-id}"

    Location="{YOUR-LOCATION}"
    ```

    > **注意**:  `{YOUR-LOCATION}` 変数は、すべてのリソースをデプロイするリージョンの短い名前に設定する必要があります。次のコマンドを入力すると、使用可能な場所と短い名前 (「**名前**」 の列) の一覧を表示できます。
    >
    > ```bash
    > az account list-locations -o Table
    >
    > 表示名 		　緯度　		経度　		名前
    > --------------------  ----------  -----------  ------------------
    > East Asia             22.267      114.188      eastasia
    > Southeast Asia        1.283       103.833      southeastasia
    > Central US            41.5908     -93.6208     centralus
    > East US               37.3719     -79.8164     eastus
    > East US 2             36.6681     -78.3889     eastus2
    > ```

1. エディター画面の右上で、ファイルに加えた変更を保存してエディタを閉じるには、「..」 をクリックし、「**エディタを閉じる**」 をクリックします。 

    保存を求められたら、「**保存**」 をクリックすると、エディタが閉じます。 

    > **注意**:  **CTRL+S**を使っていつでも保存でき、 **CTRL+Q**を押してエディターを閉じます。

1. このラボに必要なリソースを作成するには、次のコマンドを入力します。

    ```bash
    ./lab04-setup.azcli
    ```

    これは、実行するのに数分かかります。各ステップが完了すると、JSON 出力が表示されます。

スクリプトが完了したら、ラボを続行することができます。

### 演習 2: Azure CLI を使用して Azure IoT Hub デバイス ID を作成する

このコースでは、IoT Hubの機能を使用して、Contoso向けのスケーラブルでフル機能のIoTソリューションを構築しますが、このラボでは、IoT Hubを使用して、IoTHubとIoTデバイス間の信頼性の高い安全な双方向通信を確立することに焦点を当てます。

この演習では、AzureポータルでIoT Hubを開き、新しいIoTデバイスをデバイスレジストリに追加してから、IoT Hubがデバイス用に作成した接続文字列のコピーを取得します（後からデバイスコードで使用します）。

#### タスク 1: デバイスの作成

1. 必要に応じて、Azure アカウントの認証情報を使用して Azure portal にログインします。

    複数の Azure アカウントをお持ちの場合は、このコースで使用するサブスクリプションに関連付けられているアカウントでログインしていることを確認してください。

1. AZ-220ダッシュボードが表示されていることを確認します。

1. rg-az220リソースグループのタイルで、iot-az220-training-{your-id}をクリックします。

1. IoT Hubブレードの左側のメニューの[エクスプローラー]で、[IoTデバイス]をクリックします。

1. IoTデバイスペインの上部にある[+新規作成]をクリックします。

1. [デバイスID]フィールドに、**sensor-th-0001**と入力します

    デバイスID（デバイスID）は、デバイス認証とアクセス制御に使用されます。

    デバイスIDの命名規則を確立すると便利です。これにはいくつかの理由があります。たとえば、デバイスIDは、IoTHubがデバイスを表すために使用する値です。したがって、あるデバイスを別のデバイスと簡潔かつ有益に区別するデバイスIDを持つことは役に立ちます。

    上記の _sensor-th-0001_ は、この装置はセンサを持ち、温度や湿度の値をレポートするデバイスであり、シリーズの最初のデバイスであることがわかります。Contosoには、これらのデバイスが200個または5000個インストールされており、工場のフロアから環境条件を報告している場合があります。デバイスIDは、デバイスを識別する方法の1つです。

1. [認証の種類]で、[対称キー]が選択されていることを確認します。

    使用可能な認証には3つのタイプがあることに注意してください。このラボでは、3つの最も単純な対称キーを活用します。X.509証明書とその認証への使用については、後のラボで説明します。

1. 主キーとセカンダリキーのフィールドが無効になっていることに注意してください。

1. [自動生成キー]で、チェックボックスが選択されていることを確認します。

    自動生成キーを選択すると、主キーとセカンダリキーフィールドは無効になり、レコードが保存された後に挿入されます。自動生成キーの選択を解除すると、これらのフィールドが有効になり、値を直接入力できるようになります。

1. [このデバイスをIoTハブに接続する]で、[有効化]が選択されていることを確認します。

    ロールアウトの前にデバイスエントリを作成する場合は、デバイスの最初の作成時にここで[無効化]オプションを選択できます。デバイスレコードを保持したいが、関連付けられたデバイスがIoTハブに接続しないようにする場合は、将来この値を無効に設定することもできます。

1. **親デバイス**では、**親デバイスがありません**のままにします。

    IoTデバイスは、IoT Edgeデバイスなどの他のデバイスの親となる場合があります。コースの後半で、親子デバイスの関係を実装する機会が得られます。

1. このデバイスレコードをIoT Hubに追加するには、[保存]をクリックします。

    しばらくすると、IoTデバイスペインが更新され、新しいデバイスが一覧表示されます。

#### タスク 2: デバイス接続文字列を取得する

デバイスがIoTハブに接続するには、接続を確立する必要があります。このラボでは、接続文字列を使用してデバイスをIoT Hubに直接接続します（これは認証のために対称鍵認証と呼ばれることがよくあります）。

対称鍵認証を使用する場合、2つの接続文字列を使用できます。1つは主キーを使用し、もう1つはセカンダリキーを使用します。上記のように、主キーとセカンダリキーは、デバイスレコードが保存された後にのみ生成されます。したがって、接続文字列の1つを取得するには、最初にレコードを保存し（上記のタスクで行ったように）、次にデバイスレコードを再度開く必要があります（これを今から実行しようとしています）。

1. [IoTデバイス]ペインの、**sensor-th-0001**をクリックします。

1. 少し時間を取って、**sensor-th-0001**デバイス詳細ブレードの内容を確認してください。

    デバイスのプロパティに加えて、デバイス詳細ブレードは、ブレードの上部に沿って、デバイスに関連するいくつかの機能（ダイレクトメソッドやデバイスツインなど）へのアクセスを提供することに注意してください。

1. キーと接続文字列の値が入力されていることに注意してください。

    値はデフォルトで難読化されていますが、各フィールドの右側にある「目」アイコンをクリックして、値の表示と非表示を切り替えることができます。

1. [プライマリ接続文字列]フィールドの右側にある[コピー]をクリックします。

    コピーボタンは右端にあります。

    > 注: ラボの後半でプライマリ接続文字列値を使用する必要があるため、アクセス可能な場所に保存することをお勧めします（おそらく、値をNotePadなどのテキストエディターに貼り付けます）。

    接続文字列は次の形式になります。

    ```
    HostName={IoTHubName}.azure-devices.net;DeviceId=sensor-th-0001;SharedAccessKey={SharedAccessKey}
    ```


### 演習 3: シミュレートされたデバイスの作成とテスト(C#)

Azure IoT Device SDKを使用すると、デバイスクライアントを使用してIoTデバイスで実行されるアプリを構築できます。SDKのツールは、安全な接続を確立し、メッセージをパッケージ化し、IoTハブとの通信を実装するのに役立ちます。デバイスSDKは、IoTハブからメッセージ、ジョブ、メソッド、またはデバイスツインの更新を受信するのにも役立ちます。

この演習では、Visual Studio CodeとAzureIoT Device SDKを使用してシミュレートされたデバイスアプリケーションを作成します。前の演習で作成したデバイスIDと共有アクセスキー（プライマリ接続文字列）を使用して、デバイスをAzure IoT Hubに接続します。次に、セキュリティで保護されたデバイスの接続と通信をテストして、IoT Hubがデバイスからシミュレートされた温度と湿度の値を期待どおりに受信していることを確認します。

> 注: シミュレートされたデバイスコードはC#プログラミング言語を使用して記述しますが、他のプログラミング言語に慣れている場合や、プログラミングに慣れていない場合でも、手順は簡単に実行できます。重要なことは、IoT Device SDKがコードでどのように実装されているかを認識することです。

#### タスク 1: 最初のプロジェクトを作成する

1. 新しいコマンドライン/ターミナルウィンドウを開きます。

    たとえば、Windows**コマンドプロンプト**コマンドラインアプリケーションを使用できます。

1. シミュレートされたデバイスアプリケーションを作成するフォルダーの場所に移動します。

    ルートフォルダの場所は重要ではありませんが、短いフォルダパスで簡単に見つけられるものが役立ちます。

1. コマンドプロンプトで、「CaveDevice」という名前のディレクトリを作成し、現在のディレクトリをそのディレクトリに変更するには、次のコマンドを入力します。

    ```
    mkdir CaveDevice
    cd CaveDevice
    ```
1. 新しい.NETコンソールアプリケーションを作成するには、次のコマンドを入力します。

    ```
    dotnet new console
    ```

    このコマンドは、プロジェクトファイルとともにProgram.csファイルをフォルダーに作成します。

1. シミュレートされたデバイスアプリに必要なAzure IoT Device SDKとコードライブラリをインストールするには、次のコマンドを入力します。

    ```
    dotnet add package Microsoft.Azure.Devices.Client
    ```

    > 注: Microsoft.Azure.Devices.Clientパッケージには、Azure IoT Device SDK for .NETが含まれており、依存関係としてNewtonsoft.Jsonパッケージが含まれています。Newtonsoft.Jsonのパッケージには、そのJSONの作成と操作を支援するAPIが含まれています。

    次のタスクでは、シミュレートされたデバイスアプリをビルドしてテストします。

1. すべてのアプリケーションの依存関係がダウンロードされるようにするには、次のコマンドを入力します

    ```
    dotnet restore
    ```

1. Visual Studio Codeを開きます。

1. ファイルメニューでOpen folderクリックします。

1. CaveDeviceのディレクトリ作成した場所に移動します。

1. フォルダのリストで、[CaveDevice]をクリックし、[フォルダの選択]をクリックします。

    Visual Studio CodeのEXPLORERペインに、2つのC#プロジェクトファイルが一覧表示されます。

    * CaveDevice.csproj
    * Program.cs
    > 注:「Required assets to build and debug are missing from CaveDevice. Add them?」というメッセージが表示された場合、[はい]をクリックして続行できます。

#### タスク2: アプリケーションを探索する

アプリケーションは現在2つのファイルを含んでいます。

* CaveDevice.csproj
* Program.cs

このタスクでは、Visual Studio Codeを使用して、2つのアプリケーションファイルの内容と目的を確認します。

1. EXPLORERメニューで、CaveDevice.csprojをクリックします。

    これで、CaveDevice.csprojファイルがコードエディターペインで開かれるはずです。

1. CaveDevice.csprojファイルの内容を確認してください。

    ファイルの内容は次のようになります。

    ```xml
    <Project Sdk="Microsoft.NET.Sdk">

        <PropertyGroup>
            <OutputType>Exe</OutputType>
            <TargetFramework>netcoreapp3.1</TargetFramework>
        </PropertyGroup>

        <ItemGroup>
            <PackageReference Include="Microsoft.Azure.Devices.Client" Version="1.31.2" />
        </ItemGroup>

    </Project>
    ```

    > 注: ファイル内のパッケージのバージョン番号は、上記の番号と異なる場合がありますが、問題あはりません。

    プロジェクトファイル（.csproj）は、作業中のプロジェクトのタイプを指定するXMLドキュメントです。この場合、プロジェクトはSdkスタイルのプロジェクトです。

    ご覧のとおり、プロジェクト定義には、PropertyGroupとItemGroupの2つのセクションが含まれています。

    PropertyGroupは、このプロジェクトをビルドすると、生成されることを出力のタイプを定義します。この場合、.NET Core3.1を対象とする実行可能ファイルを作成します。

    ItemGroupは、アプリケーションに必要な任意の外部ライブラリを指定します。これらの参照はNuGetパッケージ用であり、各パッケージ参照はパッケージ名とバージョンを指定します。`dotnet add package`コマンドはプロジェクトファイルにこれらの参照を追加し、`dotnet restore`コマンドは、依存関係のすべてがダウンロードされたことを確実にしました。

    > 情報: NuGetの詳細については、[こちら](https://docs.microsoft.com/en-us/nuget/what-is-nuget)をご覧ください。

1. 続いてProgram.csを開きます。

1. Program.csファイルの内容を確認してください。

    ファイルの内容は次のようになります。

    ```csharp
    using System;

    namespace CaveSensor
    {
        class Program
        {
            static void Main(string[] args)
            {
                Console.WriteLine("Hello World!");
            }
        }
    }
    ```

    このプログラムは単に「Hello World！」と書くだけです。コマンドラインウィンドウに移動します。ここには多くのコードはありませんが、注目に値することがいくつかあります。

    * `using`エリア - コードで使用する名前空間をリストします（これは通常、ファイルの先頭で行われます）。この例では、コードはを`System`を使用していることを示しています。つまり、コードでSystem名前空間に含まれているコンポーネントを使用する場合、そのコード行にSystemという単語を明示的に記述する必要はありません。たとえば、上記のコードでは、Consoleクラスは「Hello World！」を記述するために使用されます。ConsoleクラスはのSystem名前空間一部ですが、`Console`を使用するときに`System`を含める必要はありませんでした。これの利点は、一部の名前空間が非常に深くネストされていることを考慮すると、より明らかになります（5つ以上のレベルが一般的です）。`using System`を指定しなかった場合は、コンソール行を次のように記述する必要があります。

        ```
        System.Console.WriteLine("Hello World!");
        ```
    * namespaceエリア - これは、名前空間に続く`{}`に含まれるクラスが、その名前空間の一部であることを指定します。 したがって、**Console**が**System**名前空間の一部であるのと同様に、上記の例では、**Program**クラスは**CaveSensor**名前空間の一部であり、そのフルネームは**CaveSensor.Program**です。


    * classエリア - これは**Program**クラスの内容を定義します。1つのソースファイル内に複数のクラスを含めることができます

    > 注: 開発者は通常、特に大規模なプロジェクトでは、クラスを独自のソースファイル（ソースファイルごとに1つのクラス）に分割します。ただし、このコースのラボでは、ファイルごとに複数のクラスを含めることになります。これは、ラボの手順を簡素化するのに役立ち、ベストプラクティスを意味するものではありません。

1. Visual Studio Code の**表示**メニューで、**ターミナル**をクリックします。

    これにより、Visual Studio Codeウィンドウの下部に統合ターミナルが開きます。ターミナルウィンドウを使用して、コンソールアプリケーションをコンパイルして実行します。

1. **ターミナル**ペインで、現在のディレクトリパスがCaveDeviceフォルダに設定されていることを確認します。

    ターミナルコマンドプロンプトには、現在のディレクトリパスが含まれています。入力したコマンドは現在の場所で実行されるため、CaveDeviceフォルダー内にいることを確認してください。

1. CaveDeviceプロジェクトをビルドして実行するために、次のコマンドを入力します。

    ```
    dotnet run
    ```

1. **Hello World！** が表示されます。

    しばらくすると、 **Hello World!** が`dotnet run`と入力したコマンドのすぐ下の行に表示されます。

    シミュレートされたデバイスアプリケーションで同じ`Console.WriteLine`アプローチを使用して情報をローカルに表示します。これにより、IoT Hubに送信されている情報を確認し、デバイスによって完了されているプロセスを追跡できます。

    このHello Worldアプリはいくつかの基本的な概念を示していますが、明らかにシミュレートされたデバイスではありません。次のタスクでは、このコードをシミュレートされたデバイスのコードに置き換えます。

#### タスク3: シミュレートされたデバイスコードを実装する

このタスクでは、Visual Studio Codeを使用して、Azure IoT DeviceSDKを利用してIoTHubリソースに接続するコードを入力します。

1.  Program.csを開きます。

1. 既存のコードをすべて選択してから削除します。

1. コードエディタで、シミュレートされたデバイスアプリケーションの基本構造を作成するには、次のコードを入力します。

    ```csharp
    // INSERT using statements below here

    namespace CaveDevice
    {
        class Program
        {
            // INSERT variables below here

            // INSERT Main method below here

            // INSERT SendDeviceToCloudMessagesAsync method below here

            // INSERT CreateMessageString method below here

        }

        // INSERT EnvironmentSensor class below here

    }
    ```

    > 注: ご覧のとおり、名前空間とクラスは保持されていますが、他の項目はプレースホルダーコメントです。次の手順では、特定のコメントの下のファイルにコードを挿入します。

1. `// INSERT using statements below here` コメントがある場所に移動します。

1. アプリケーションコードが使用する名前空間を指定するために、次のコードを入力します。

    ```csharp
    using System;
    using System.Text;
    using System.Threading.Tasks;
    using Microsoft.Azure.Devices.Client;
    using Newtonsoft.Json;
    ```

    > ヒント: コードを挿入するとき、コードレイアウトは理想的ではない場合があります。コードエディターペインを右クリックし、[ドキュメントのフォーマット]をクリックすると、Visual Studio Codeでドキュメントを書式設定できます。タスクペインを開いて（F1キーを押す）、[Format Document]と入力して、Enterキーを押すと、同じ結果を得ることができます。また、Windowsでは、このタスクのショートカットは**SHIFT + ALT + F**です。

1. `// INSERT variables below here` コメントがある場所に移動します。

1. プログラムが使用している変数を指定するため、次のコードを入力します。

    ```csharp
    // Contains methods that a device can use to send messages to and receive from an IoT Hub.
    private static DeviceClient deviceClient;

    // The device connection string to authenticate the device with your IoT hub.
    // Note: in real-world applications you would not "hard-code" the connection string
    // It could be stored within an environment variable, passed in via the command-line or
    // stored securely within a TPM module.
    private readonly static string connectionString = "{Your device connection string here}";
    ```

1. 入力したコード（およびコードコメント）を確認してください。

    **deviceClient**の変数は、**DeviceClient**のインスタンスの格納するために使用されます。このクラスは、AzureののIoT Device SDKから来て、デバイスがメッセージを送信し、IoT Hubから受信するために使用できるメソッドが含まれています。

    **ConnectionString**変数は、我々が以前に作成したデバイスの接続文字列が含まれています。この値は、**DeviceClient**がIoT Hubに接続するために使用します。

    重要: このコース全体を通して、このラボと他のラボで、接続文字列、パスワード、およびその他の構成情報がアプリケーションにハードコードされている例を確認できます。これはラボを簡素化するためだけに行われるものであり、推奨される方法ではありません。

    コードコメントに記載されているように、接続文字列および同様の構成値は、環境変数、コマンドラインパラメーターなどの代替手段を介して提供するか、より適切には、トラステッドプラットフォームモジュール（TPM）などのセキュリティで保護されたハードウェアに格納する必要があります。

1. 入力したコードで、IoT Hubからコピーしたプライマリ接続文字列を使用して**connectionString**の値を更新します。

    更新されると、**connectionString***は次のようになります。

    ```
    private readonly static string connectionString = "HostName=iot-az220-training-dm200420.azure-devices.net;DeviceId=sensor-th-0001;SharedAccessKey=hfavUmFgoCPA9feWjyfTx23SUHr+dqG9X193ctdEd90=";
    ```

1. `// INSERT Main method below here`コメントを見つけます。

1. シミュレートされたデバイスアプリケーションの**Main**メソッドを作成するために、次のコードを入力します。

    ```csharp
    private static void Main(string[] args)
    {
        Console.WriteLine("IoT Hub C# Simulated Cave Device. Ctrl-C to exit.\n");

        // Connect to the IoT hub using the MQTT protocol
        deviceClient = DeviceClient.CreateFromConnectionString(connectionString, TransportType.Mqtt);
        SendDeviceToCloudMessagesAsync();
        Console.ReadLine();
    }
    ```

    **Main**メソッドは、アプリが開始されると実行されるアプリケーションの最初の部分です。

1. 入力したコード（およびコードコメント）を確認してください。

    シンプルなデバイスアプリの基本構造は次のとおりです。

    * IoTハブに接続する
    * アプリにテレメトリを送信する（デバイスからクラウドへのメッセージ）

    **deviceClient**変数は、**DeviceClient**静的メソッド**CreateFromConnectionString**の結果で初期化されることに注意してください。このメソッドは、前に指定した接続文字列を使用し、デバイスがテレメトリの送信に使用するプロトコル（この場合はMQTT）を選択します。

    > 注: 実稼働アプリケーションでは、**CreateFromConnectionString**メソッド呼び出しは、接続の問題を適切に処理するために例外処理コードでラップされます。このラボコードやその他のラボコードは、重要なポイントを強調するために可能な限りシンプルに保たれているため、簡潔にするためにほとんどのエラー処理は省略されています。

    一度接続されると、**SendDeviceToCloudMessagesAsync**メソッドが呼び出されます。メソッド名に「赤い波線」の下線が引かれていることに気付くかもしれません。これは、Visual Studio Codeが**SendDeviceToCloudMessagesAsync**がまだ実装されていないことに気付いたためです。メソッドをこの後追加します。

    最後に、アプリケーションはユーザー入力を待ちます。

    > 情報: **DeviceClient**クラスは[ここ](https://docs.microsoft.com/en-us/dotnet/api/microsoft.azure.devices.client.deviceclient?view=azure-dotnet)に記載されています。

    > 情報: **CreateFromConnectionString**メソッドは[ここ](https://docs.microsoft.com/en-us/dotnet/api/microsoft.azure.devices.client.deviceclient.createfromconnectionstring?view=azure-dotnet#Microsoft_Azure_Devices_Client_DeviceClient_CreateFromConnectionString_System_String_Microsoft_Azure_Devices_Client_TransportType_)に記載されています。

    > 情報: サポートされているトランスポートプロトコルは、[ここ](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-devguide-protocols)に記載されています。

1. `// INSERT - SendDeviceToCloudMessagesAsync below here` コメントを見つけます。

1. SendDeviceToCloudMessagesAsyncメソッドを作成するため、次のコードを入力します。

    ```csharp
    private static async void SendDeviceToCloudMessagesAsync()
    {
        // Create an instance of our sensor
        var sensor = new EnvironmentSensor();

        while (true)
        {
            // read data from the sensor
            var currentTemperature = sensor.ReadTemperature();
            var currentHumidity = sensor.ReadHumidity();

            var messageString = CreateMessageString(currentTemperature, currentHumidity);

            // create a byte array from the message string using ASCII encoding
            var message = new Message(Encoding.ASCII.GetBytes(messageString));

            // Add a custom application property to the message.
            // An IoT hub can filter on these properties without access to the message body.
            message.Properties.Add("temperatureAlert", (currentTemperature > 30) ? "true" : "false");

            // Send the telemetry message
            await deviceClient.SendEventAsync(message);
            Console.WriteLine("{0} > Sending message: {1}", DateTime.Now, messageString);

            await Task.Delay(1000);
        }
    }
    ```

    **SendDeviceToCloudMessagesAsync** メソッドの宣言に`async`キーワードが含まれていることに注意してください。これは、メソッドに`await`キーワードを使用する非同期コードが含まれていることを指定し、コールバックを処理するようコンパイラーに指示します。

1. 入力したコード（およびコードコメント）を確認してください。

    このメソッドは、典型的なメッセージループを実装します。

    * 1つ以上のセンサーから読み取る
    * 送信するメッセージを作成する
    * メッセージを送信する
    * しばらく待つか、イベントが発生するなどを待ちます。
    * ループを繰り返す
    
    次にメソッドコードについて詳しく説明します。

    * コードで最初に行うことは、**EnvironmentSensor**クラスのインスタンスを作成することです。これはループの外側で行われ、ループ内のセンサーデータのシミュレーションをサポートするために使用されます。間もなく**EnvironmentSensor**クラスを追加します。

    * 次に、無限ループ`while(true) {}`を開始します。 ユーザーがCTRL + Cを押すまで繰り返されます。

    * ループ内で最初に行うことは、センサーから温度と湿度を読み取り、それらの値を使用してメッセージ文字列を作成することです。**CreateMessageString**のコードもすぐに追加します。

    * 次に、IoTHubに送信される実際のメッセージを作成します。これを行うには、Azure IoT Device SDKから**Message**クラスのインスタンスを作成します。これは、IoT Hubとのやり取りに使用されるメッセージを表すデータ構造です（IoT Hubは特定のメッセージ形式を想定しています）。**Message**クラスに使用するコンストラクターでは、メッセージ文字列をバイト配列としてエンコードする必要があります。

    * 次に、追加のプロパティをメッセージに追加-ここでは、たとえば**temperatureAlert**を、**currentTemperature**が30以上の場合は`true`、そうでなければ`false`に設定します。

    * 次に、`await deviceClient.SendEventAsync(message);`を介してテレメトリメッセージを送信します。この行には`await`キーワードが含まれていることに注意してください。これは、次のコードが非同期であり、将来的に完了することをコンパイラーに指示します。完了すると、このメソッドは次の行で実行を継続します。

    * 最後に、ローカルコンソールウィンドウにメッセージ文字列を書き込んで、テレメトリがIoT Hubに送信されたことを示し、1000ミリ秒（1秒）待ってからループを繰り返します。

    > 情報: `async`、`await`やC#での非同期プログラミングの詳細については[こちら](https://docs.microsoft.com/en-us/dotnet/csharp/async)をご覧ください。

    > 情報: **Message**クラスのドキュメントは[こちら](https://docs.microsoft.com/en-us/dotnet/api/microsoft.azure.devices.client.message?view=azure-dotnet)

1. `// INSERT CreateMessageString method below here`コメントを見つけます。

1. **CreateMessageString**のセンサーの測定値からJSON文字列を作成するため、次のコードを入力します。

    ```csharp
    private static string CreateMessageString(double temperature, double humidity)
    {
        // Create an anonymous object that matches the data structure we wish to send
        var telemetryDataPoint = new
        {
            temperature = temperature,
            humidity = humidity
        };

        // Create a JSON string from the anonymous object
        return JsonConvert.SerializeObject(telemetryDataPoint);
    }
    ```

    このメソッドは、温度と湿度のプロパティを持つ匿名オブジェクトを作成し、それを **telemetryDataPoint** に割り当てます。

    **telemetryDataPoint** は、その後 **JsonConvert** クラス経由でJSON文字列に変換されます。JSON文字列値が返され、メッセージのペイロードとして使用されます。

1. `// INSERT EnvironmentSensor class below here` コメントを見つけます。

1. **EnvironmentSensor**クラスを作成するには、次のコードを入力します。

    ```csharp
    /// <summary>
    /// This class represents a sensor 
    /// real-world sensors would contain code to initialize
    /// the device or devices and maintain internal state
    /// a real-world example can be found here: https://bit.ly/IoT-BME280
    /// </summary>
    internal class EnvironmentSensor
    {
        // Initial telemetry values
        double minTemperature = 20;
        double minHumidity = 60;
        Random rand = new Random();

        internal EnvironmentSensor()
        {
            // device initialization could occur here
        }

        internal double ReadTemperature()
        {
            return minTemperature + rand.NextDouble() * 15;
        }

        internal double ReadHumidity()
        {
            return minHumidity + rand.NextDouble() * 20;
        }
    }
    ```

    これは、乱数を使用して温度と湿度を表す値を返す非常に単純なクラスです。実際には、センサーとの対話は、特に低レベルでセンサーと通信して測定値を導出する必要がある場合（適切な単位で直接読み取るのではなく）、はるかに複雑になることがよくあります。

    > 情報: [ここ](https://bit.ly/IoT-BME280)で、単純な温度、湿度、および圧力センサーと相互作用するコードのより代表的な例を表示できます。

1. ファイルメニューの、保存を選択します。

1. 完成したアプリケーションを確認するのに少し時間を取ってください。

    完成したアプリケーションは、単純なシミュレートされたデバイスを表しています。デバイスをIoT Hubに接続し、デバイスからクラウドへのメッセージを送信する方法を示します。

    これで、アプリケーションをテストする準備ができました

#### タスク4: アプリケーションをテストする

1. Visual Studio Codeメニューで、[ターミナル]をクリックします。

    選択したターミナルシェルがWindowsコマンドプロンプトであることを確認します。

1. ターミナルビューのコマンドプロンプトで、次のコマンドを入力します。

    ```bash
    dotnet run
    ```

    このコマンドは、シミュレートされたデバイスアプリケーションをビルドして実行します。端末の場所が**CaveDevice.cs**ファイルのあるディレクトリに設定されていることを確認してください。

    > 注: コマンドが`Malformed Token`エラーメッセージまたはその他のエラーメッセージを出力する場合は、プライマリ接続文字列の値が**connectionString**変数の値として正しく構成されていることを確認してください。

    追加のエラーメッセージが表示された場合は、このラボのFinalフォルダーで参照できる完成したソリューションコードを参照することで、コードが正しく作成されたことを確認できます。この最終フォルダーは、ラボ3で開発環境をセットアップするときにダウンロードしたラボリソースファイルに含まれています。フォルダーパスは次のとおりです。

    * Allfiles
        * Labs
            * LAB_AK_04-connect-iot-device-to-azure
                * Final

1. ターミナルに表示されるメッセージ文字列の出力を確認します。

    シミュレートされたデバイスアプリケーションが実行されるtemperatureと、humidity値を含むイベントメッセージがAzure IoT Hubに送信され、コンソールにメッセージ文字列出力が表示されます。

    出力は次のようになります。

    ```bash
    10/25/2019 6:10:12 PM > Sending message: {"temperature":27.714212817472504,"humidity":63.88147743599558}
    10/25/2019 6:10:13 PM > Sending message: {"temperature":20.017463779085066,"humidity":64.53511070671263}
    10/25/2019 6:10:14 PM > Sending message: {"temperature":20.723927165718717,"humidity":74.07808918230147}
    10/25/2019 6:10:15 PM > Sending message: {"temperature":20.48506045736608,"humidity":71.47250854944461}
    10/25/2019 6:10:16 PM > Sending message: {"temperature":25.027703996760632,"humidity":69.21247714628115}
    10/25/2019 6:10:17 PM > Sending message: {"temperature":29.867399432634656,"humidity":78.19206098010395}
    10/25/2019 6:10:18 PM > Sending message: {"temperature":33.29597232085465,"humidity":62.8990878830194}
    10/25/2019 6:10:19 PM > Sending message: {"temperature":25.77350195766124,"humidity":67.27347029711747}
    ```

    > 注: 今のところ、シミュレートされたデバイスアプリは実行したままにしておきます。次のタスクは、IoTハブがテレメトリメッセージを受信して​​いることを確認することです。

#### タスク5: Azure IoT Hubに送信されたテレメトリストリームを確認する

このタスクでは、Azure CLIを使用して、シミュレートされたデバイスによって送信されたテレメトリがAzure IoTHubによって受信されていることを確認します。

1. ブラウザーを使用して、Azure Cloud Shellを開き、このコースで使用しているAzureサブスクリプションでログインします。

1. Azure Cloud Shellで、IoTハブによって受信されているイベントメッセージを監視するには、次のコマンドを入力します。

    ```bash
    az iot hub monitor-events --hub-name {IoTHubName} --device-id sensor-th-0001
    ```

    _{IoTHubName}_ プレースホルダーをAzure IoT Hubの名前に置き換えてください。

    > 注: Azure CLIコマンドの実行時に、「Dependency update required for IoT extension version」というメッセージが表示された場合は、`y`を入力して更新を受け入れEnterを押します。これにより、コマンドを期待どおりに続行できます。

    この`monitor-events`コマンド（`az iot hub`Azure CLIモジュール内）は、Azure IoT Hubに送信されるデバイステレメトリおよびその他のメッセージタイプを監視する機能を提供します。これは、コード開発中に非常に便利なツールになる可能性があり、コマンドラインインターフェイスの利便性も優れています。

    この`--device-id`パラメーターはオプションであり、単一のデバイスからイベントを監視できます。パラメーターを省略した場合、コマンドは指定されたAzure IoT Hubに送信されたすべてのイベントを監視します。

1. `az iot hub monitor-events`Azure CLIコマンドは、指定されたAzure IoT Hubに到着するイベントのJSON表現を出力することに注意してください。

    このコマンドを使用すると、IoTハブに送信されるイベントを監視できます。また、デバイスがIoTハブに接続して通信できることも確認しています。

    次のようなメッセージが表示されます。

    ```bash
    Starting event monitor, filtering on device: sensor-th-0001, use ctrl-c to stop...
    {
        "event": {
            "origin": "sensor-th-0001",
            "payload": "{\"temperature\":25.058683971901743,\"humidity\":67.54816981383979}"
        }
    }
    {
        "event": {
            "origin": "sensor-th-0001",
            "payload": "{\"temperature\":29.202181296051563,\"humidity\":69.13840303623043}"
        }
    }
    ```

1. IoT Hubがテレメトリを受信して​​いることを確認したら、Azure CloudShellウィンドウとVisual Studio CodeウィンドウでCtrl-Cを押します。

    Ctrl-Cは、実行中のアプリを停止するために使用されます。不要なアプリやジョブをシャットダウンすることを常に忘れないでください。