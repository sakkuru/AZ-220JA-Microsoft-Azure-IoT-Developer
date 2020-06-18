---
lab:
    title: 'ラボ 04: IoT デバイスを Azure に接続する'
    module: 'モジュール 2：デバイスとデバイス通信'
---

# IoT デバイスを Azure に接続する

## ラボシナリオ

Contoso は、高品質のチーズを生産することで知られています。同社の人気と販売の両方が急成長しているため、彼らは顧客の期待に応える高品質なチーズを確実に維持するための方策を講じたいと考えています。

昔は、各勤務シフト中に工場の現場担当者が温度と湿度のデータを収集していました。同社は、新しい施設が稼働し始めるにつれて、工場の拡張により監視の強化が必要になることと、データを収集する手動プロセスでは調整できなくなることを懸念しています。

Contoso は温度と湿度を監視するために、IoT デバイスを使用する自動化システムを立ち上げることにしました。テレメトリ データの通信速度は調整可能で、チーズのバッチが環境に敏感なプロセスを進める際に製造プロセスが確実に制御されるようにします。

この資産監視ソリューションをフルスケールの実装前に評価するには、IoT デバイス (温度センサーと湿度センサーを含む) を IoT Hub に接続します。このラボでは、.NET Core コンソール アプリケーションを使用して実際の IoT デバイスをシミュレートします。

次のリソースが作成されます。

!「ラボ 4 アーキテクチャ」(media/LAB_AK_04-architecture.png)

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
| リソース グループ | AZ-220-RG |
| IoT Hub | AZ-220-HUB-_{YOUR-ID}_ |

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

    * すべてのファイル
      * ラボ
          * 04 - IoT デバイスを Azure に接続する
            * セットアップ

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

    RGName="AZ-220-RG"
    IoTHubName="AZ-220-HUB-{YOUR-ID}"

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

`iot` Azure CLI モジュールには、`az iot hub device-identity` コマンド グループの下にある Azure IoT Hub 内の IoT デバイスを管理するためのいくつかのコマンドが含まれています。これらのコマンドは、スクリプト内の IoT デバイスを管理したり、コマンドライン/ターミナルから直接 IoT デバイスを管理するために使用できます。

#### タスク 1: サブスクリプションの管理

1 つのアカウントに関連付けられた複数のサブスクリプションを保持できるため、サブスクリプションを一覧表示する方法、現在アクティブなサブスクリプションを選択する方法、および既定のサブスクリプションを変更する方法を理解することが重要です。

1. 必要に応じて、Azure アカウントの認証情報を使用して Azure portal にログインします。

    複数の Azure アカウントをお持ちの場合は、このコースで使用するサブスクリプションに関連付けられているアカウントでログインしていることを確認してください。

1. Azure Portal の上部にある 「**Cloud Shell**」 アイコンをクリックし、Azure Portal 内で **Azure Cloud Shell** を開きます。

1. Cloud Shell のストレージの設定に関するメッセージが表示された場合は、デフォルトをそのまま使用します。

1. ウィンドウが開いたら、Cloud Shell 内で **Bash** ターミナルのオプションを選択します。

1. コマンド プロンプトで、使用可能なサブスクリプションを一覧表示するには、次のコマンドを入力します。

    ```bash
    az account list --output table

    Name                      CloudName    SubscriptionId                        State    IsDefault
    ------------------------  -----------  ------------------------------------  -------  -----------
    Subscription1             AzureCloud   aa1122bb-4bd0-462b-8449-a1002aa2233a  Enabled  True
    Subscription2             AzureCloud   aa1122bb-4bd0-462b-8449-a1002aa2233b  Enabled  False
    Azure Pass - Sponsorship  AzureCloud   aa1122bb-4bd0-462b-8449-a1002aa2233c  Enabled  False
    ```

    ご覧のとおり、コースで使用されている **Azure Pass - スポンサーシップ** は一覧表示されますが、既定のサブスクリプションには設定されていません。

1. 現在アクティブなサブスクリプションを表示するには、次のコマンドを入力します。

    ```bash
    az account show -o table

    EnvironmentName    IsDefault    Name           State    TenantId
    -----------------  -----------  -------------  -------  ------------------------------------
    AzureCloud         True         Subscription1  Enabled  aa1122bb-4bd0-462b-8449-a1002aa2233a
    ```

1. 現在のセッションの既定のサブスクリプションを **Azure Pass - スポンサーシップ** に変更するには、次のコマンドを入力します。

    ```bash
    az account set --subscription "Azure Pass - Sponsorship"
    ```

    > **注意**: **--subscription** 引数を使用して、サブスクリプション**名**または**サブスクリプション ID** のいずれかを使用できます。2 つのサブスクリプションに同じ名前を使用している場合は、**サブスクリプション ID** を使用する**必要があります**。

1. 変更を確認するには、次のコマンドを入力します。

    ```bash
    az account show -o table

    EnvironmentName    IsDefault    Name                      State    TenantId
    -----------------  -----------  ------------------------  -------  ------------------------------------
    AzureCloud         True         Azure Pass - Sponsorship  Enabled  aa1122bb-4bd0-462b-8449-a1002aa2233c
    ```

リソースなどを作成するときにはいつでも、このサブスクリプションが現在のセッションで使用されます。

#### タスク 2: IoT ハブ Device ID を作成する

1. Cloud Shell 内で、Cloud Shell に IoT 拡張機能がインストールされていることを確認するには、次のコマンドを実行します。

    ``` sh
    az extension add --name azure-cli-iot-ext
    ```

1. 依然として Cloud Shell 内で、シミュレートされたデバイスに使用される Azure IoT Hub に**デバイス ID** を作成するために、次の Azure CLI コマンドを実行します。

    ```sh
    az iot hub device-identity create --hub-name {IoTHubName} --device-id SimulatedDevice1
    ```

    > **注意**:  _{IoTHubName}_ プレースホルダーを Azure IoT Hub の名前に必ず置き換えてください。IoT ハブ名を忘れた場合は、次のコマンドを入力できます。
    >
    >```sh
    >az iot hub list -o table
    >```

#### タスク 3: デバイス接続文字列を取得する

1. Cloud Shell 内で、次の Azure CLI コマンドを実行して、IoT Hub に追加されたばかりのデバイス ID の_デバイス接続文字列_を取得します。この接続文字列は、シミュレートされたデバイスを Azure IoT Hub に接続するために使用されます。

    ```cmd/sh
    az iot hub device-identity show-connection-string --hub-name {IoTHUbName} --device-id SimulatedDevice1 --output table
    ```

1. 前のコマンドから出力された**デバイス接続文字列**をメモします。後で使用するために、これを保存する必要があります。

    接続文字列は次の形式になります。

    ```text
    HostName={IoTHubName}.azure-devices.net;DeviceId=SimulatedDevice1;SharedAccessKey={SharedAccessKey}
    ```

### エクササイズ 3: シミュレートされたデバイス (C#) の構成とテスト

この演習では、前の演習で作成したデバイス ID と共有アクセス キーを使用して Azure IoT Hub に接続するために、C# で記述されているシミュレートされたデバイスを構成します。次に、デバイスをテストし、IoT Hub がデバイスから想定どおりに製品利用統計情報を受信していることを確認します。

#### タスク 1: ラボ 4 スターター コード プロジェクトを開く

1. Visual Studio Code の新しいインスタンスを開きます。

1. 左側のメニューで 「**Explorer**」 をクリックします。

    「Explorer」 ウィンドウに、ファイル/フォルダー階層が一覧表示されます。Visual Studio Code の新しいインスタンスには、開いているフォルダーはありません。

1. 「ファイル」 メニューで、「**フォルダーを開く**」 をクリックします。

1. 「フォルダーを開く」 ダイアログで、スタート コード プロジェクトが含まれているラボ 4 フォルダーに移動します。

    ラボ 4 スターター プロジェクト フォルダーのローカル パスは、次のようになります。

    * AZ-220-Microsoft-Azure-IoT-Developer-master
      * すべてのファイル
        * ラボ
          * 04 - IoT デバイスを Azure に接続する
            * スターター

    > **注意**: ラボ 3 で開発環境をセットアップするときに GitHub プロジェクトを複製しました。リソース ファイルを見つけるのに必要な場合は、コースの講師に確認してください。

1. フォルダーを開くには、「**スターター**」 をクリックし、「**フォルダーの選択**」 をクリックします。

    Visual Studio Code の 「Explorer」 ペインに、2 つの C# プロジェクト ファイルが表示されます。

    * SimulatedDevice.cs
    * SimulatedDevice.csproj

#### タスク 2: デバイス接続文字列を更新する

1. 「Visual Studio Code Explorer」 ペインで、「SimulatedDevice.cs」 ファイルを開き、「**SimulatedDevice.cs**」 をクリックします。

1. エディター ビューで、変数 `s_connectionString` を含むコード行を探します。

    ```C#
    private readonly static string s_connectionString = "{Your device connection string here}";
    ```

1. 値のプレースホルダー `{Your device connection string here}` を、以前にコピーしたデバイス接続文字列に置き換えます。

    これにより、シミュレートされたデバイスは、Azure IoT Hub との認証、接続、通信を行うことができます。

    構成すると、変数は次のようになります (特定の接続情報が含まれています)。

    ```csharp
    private readonly static string s_connectionString = "HostName={IoTHubName}.azure-devices.net;DeviceId=SimulatedDevice1;SharedAccessKey={SharedAccessKey}";
    ```

1. **表示**メニューで、**ターミナル** をクリック します。   

    選択したターミナル シェルが Windows コマンド プロンプトであることを確認します。

1. ターミナル ビューのコマンド プロンプトで、次のコマンドを入力します。

    ```cmd/sh
    dotnet run
    ```

    このコマンドは、シミュレートされたデバイス アプリケーションをビルドして実行します。ターミナルの場所が `SimulatedDevice.cs` ファイルを持つディレクトリに設定されていることを確認します。

    > **注意**:  コマンドが `Malformed Token` などのエラー メッセージを出力する場合は、 **デバイス接続文字列**が `s_connectionString` 変数の値として正しく構成されていることを確認します。

1. シミュレートされたデバイス アプリケーションが実行されると、`temperature` と `humidity` の値を含むイベント メッセージが Azure IoT Hub に送信されます。

    ターミナル出力は次のようになります。

    ```text
    IoT Hub C# Simulated Device. Ctrl-C to exit.

    10/25/2019 6:10:12 PM > Sending message: {"temperature":27.714212817472504,"humidity":63.88147743599558}
    10/25/2019 6:10:13 PM > Sending message: {"temperature":20.017463779085066,"humidity":64.53511070671263}
    10/25/2019 6:10:14 PM > Sending message: {"temperature":20.723927165718717,"humidity":74.07808918230147}
    10/25/2019 6:10:15 PM > Sending message: {"temperature":20.48506045736608,"humidity":71.47250854944461}
    10/25/2019 6:10:16 PM > Sending message: {"temperature":25.027703996760632,"humidity":69.21247714628115}
    10/25/2019 6:10:17 PM > Sending message: {"temperature":29.867399432634656,"humidity":78.19206098010395}
    10/25/2019 6:10:18 PM > Sending message: {"temperature":33.29597232085465,"humidity":62.8990878830194}
    10/25/2019 6:10:19 PM > Sending message: {"temperature":25.77350195766124,"humidity":67.27347029711747}
    ```

    > **注意**: ひとまず、シミュレートされたデバイス アプリを実行したままにします。次のタスクは、IoT ハブがテレメトリ メッセージを受信していることを確認することです。

#### タスク 3: Azure IoT Hub に送信されるテレメトリ ストリームを確認する

このタスクでは、Azure CLI を使用して、シミュレートされたデバイスから送信されたテレメトリが Azure IoT Hub によって受信されていることを確認します。

1. ブラウザーを使用して [Azure Cloud Shell](https://shell.azure.com/) を開き、このコースで使用している Azure サブスクリプションでログインします。

1. Azure Cloud Shell で、次のコマンドを入力します。

    ```cmd/sh
    az iot hub monitor-events --hub-name {IoTHubName} --device-id SimulatedDevice1
    ```

    _必ず、**IoTHubName** プレースホルダーを Azure IoT Hub の名前に置き換えてください。_

    > **注意**:  Azure CLI コマンドを実行しているときに _「IoT 拡張機能のバージョンに必要な依存関係の更新が必要です」_ というメッセージが表示された場合は、`y` を押して更新を受け入れ、`Enter` キーを押します。  これにより、期待通りにコマンドは続行されます。

    `--device-id` パラメーターは省略可能ですが、単一のデバイスのイベントを監視できるようになります。パラメーターを省略すると、コマンドにより指定された Azure IoT Hub に送信されるすべてのイベントが監視されます。

    `az iot hub` Azure CLI モジュール内の `monitor-events` コマンドにより、コマンド ライン/ターミナル内から Azure IoT Hub に送信されるデバイス テレメトリとメッセージを監視する機能が提供されます。

1. `az iot hub monitor-events` Azure CLI コマンドにより、指定された Azure IoT Hub に到達するイベントの JSON 表現が出力されることに注意してください。 

    このコマンドを使用すると、IoT Hub に送信されるイベントを監視できます。また、デバイスが IoT ハブに接続して通信できることを確認します。

    次のように表示されたメッセージを確認するはずです。

    ```cmd/sh
    Starting event monitor, filtering on device: SimulatedDevice1, use ctrl-c to stop...
    {
        "event": {
            "origin": "SimulatedDevice1",
            "payload": "{\"temperature\":25.058683971901743,\"humidity\":67.54816981383979}"
        }
    }
    {
        "event": {
            "origin": "SimulatedDevice1",
            "payload": "{\"temperature\":29.202181296051563,\"humidity\":69.13840303623043}"
        }
    }
    ```

1. IoT ハブがテレメトリを受信していることを確認したら、Azure Cloud Shell および Visual Studio Code ウィンドウで **Ctrl+C** キーを押します。

    「Ctrl-C」 は実行中のアプリを停止するために使用されます。常に不要なアプリやジョブをシャットダウンすることを忘れないでください。
