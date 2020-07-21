---
lab:
    title: 'ラボ 11: Azure IoT Edge の概要'
    module: 'モジュール 6: Azure IoT Edge のデプロイプロセス'
---

# Azure IoT Edge の概要

## ラボ シナリオ

グローバル市場で地元の消費者を奨励するために、Contoso は地元の職人と提携し、世界中の新しい地域でチーズを生産しています。 

各地では、地元のチーズを作成するために使用される混合および処理機械が装備されている複数の生産ラインをサポートしています。現在、施設にはそれぞれの機械に IoT デバイスが接続されています。これらのデバイスはセンサー データを Azure にストリーミングし、すべてのデータがクラウドで処理されます。 

大量のデータが収集され、一部のマシンで即時の時間応答が必要なため、Contoso は IoT Edge ゲートウェイ デバイスを使用して、インテリジェンスの一部を即時処理に使用したいと考えています。データの一部は引き続きクラウドに送信されます。また、IoT Edge にデータ インテリジェンスを導入することで、ローカル ネットワークが貧弱な場合でも、データを処理して迅速に対応できるようになります。

Azure IoT Edge ソリューションのプロトタイプ作成を任されました。まず、温度を監視する IoT Edge デバイス (チーズ加工処理マシンの 1 台に接続されているデバイスをシミュレートします) を設定します。次に、そのデバイスを使用して平均温度を計算し、プロセス制御値を超えた場合にアラート通知を生成するデバイスにStream Analytics モジュールをデプロイします。

次のリソースが作成されます。

![ラボ 11 アーキテクチャ](media/LAB_AK_11-architecture.png)

## このラボでは

このラボでは、次のタスクを完了します。

* ラボの前提条件を確認する
* Azure IoT Edge が有効な Linux VM をデプロイする
* Azure CLI を使用して IoT ハブで IoT Edge デバイス ID を作成する
* IoT Edge デバイスを IoT ハブに接続する
* Edge デバイスに Edge モジュールを追加する
* IoT Edge モジュールとして Azure Stream Analytics をデプロイする

## ラボの手順

### 演習 1: ラボの前提条件を確認する

このラボでは、次の Azure リソースが使用可能であることを前提としています。

| リソースの種類:  | リソース名 |
| :-- | :-- |
| リソース グループ | rg-az220 |
| IoT Hub | iot-az220-training-{your-id} |

これらのリソースが利用できない場合は、演習 2 に進む前に、以下の手順に従って **lab11-setup.azcli** スクリプトを実行する必要があります。スクリプト ファイルは、開発環境構成 (ラボ 3) の一部としてローカルに複製した GitHub リポジトリに含まれています。

> **注意**:   **lab11-setup.azcli** スクリプトは **Bash** シェル環境で実行するように記述されています。これは Azure Cloud Shell で実行する最も簡単な方法です。

1. ブラウザーを使用して [Azure Cloud Shell](https://shell.azure.com/) を開き、このコースで使用している Azure サブスクリプションでログインします。

1. Cloud Shell のストレージの設定に関するメッセージが表示された場合は、デフォルトをそのまま使用します。

1. Azure シェルが **Bash** を使用していることを確認します。

    「Azure Cloud Shell」 ページの左上隅にあるドロップダウンは、環境を選択するために使用されます。選択されたドロップダウンの値が **Bash**であることを確認します。 

1. Azure Shell ツール バーで、「**ファイルのアップロード/ダウンロード**」 をクリックします (右から 4 番目のボタン)。

1. ドロップダウンで、「**アップロード**」 をクリックします。

1. ファイル選択ダイアログで、開発環境を構成したときにダウンロードした GitHub ラボ ファイルのフォルダーの場所に移動します。

    このコースのラボ 3「開発環境の設定」では、ZIP ファイルをダウンロードしてコンテンツをローカルに抽出することで、ラボ リソースを含む GitHub リポジトリを複製しました。抽出されたフォルダー構造には、次のフォルダー パスが含まれます。

    * Allfiles
      * Labs
          * 11-Introduction to Azure IoT Edge
            * Setup

    lab11-setup.azcli スクリプト ファイルは、ラボ 11 の設定フォルダーにあります。

1. **lab11-setup.azcli** ファイルを選択し、「**開く**」 をクリックします。   

    ファイルのアップロードが完了すると、通知が表示されます。

1. 正しいファイルがアップロードされたことを確認するには、次のコマンドを入力します。

    ```bash
    ls
    ```

    `ls` コマンドを実行すると、現在のディレクトリの内容が一覧表示されます。lab11-setup.azcli ファイルが一覧表示されます。

1. セットアップ スクリプトを含むこのラボのディレクトリを作成し、そのディレクトリに移動するには、次の Bash コマンドを入力します。

    ```bash
    mkdir lab11
    mv lab11-setup.azcli lab11
    cd lab11
    ```

    これらのコマンドは、このラボのディレクトリを作成し、**lab11-setup.azcli** ファイルをそのディレクトリに移動してから、新しいディレクトリを現在の作業ディレクトリに変更します。 

1. **lab11-setup.azcli** に実行権限があることを確認するには、次のコマンドを入力します。 

    ```bash
    chmod +x lab11-setup.azcli
    ```

1. Cloud Shell ツール バーで、**lab11-setup.azcli** ファイルを編集するには、「**エディターを開く**」 (右から 2 番目のボタン - **{ }**) をクリックします。   

1. 「**ファイル**」 リストで、lab4 フォルダーを展開するには、「**lab11**」 をクリックしてから、「**lab11-setup.azcli**」 をクリックします。     

    エディターは **lab11-setup.azcli** ファイルの内容を表示します。 

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
    > DisplayName           Latitude    Longitude    Name
    > --------------------  ----------  -----------  ------------------
    > East Asia             22.267      114.188      eastasia
    > Southeast Asia        1.283       103.833      southeastasia
    > Central US            41.5908     -93.6208     centralus
    > East US               37.3719     -79.8164     eastus
    > East US 2             36.6681     -78.3889     eastus2
    > ```

1. エディター画面の右上で、ファイルに加えた変更を保存してエディタを閉じるには、「..」 をクリックし、「**エディタを閉じる**」 をクリックします。 

    保存を求められたら、「**保存**」 をクリックすると、エディタが閉じます。 

    > **注意**:  「**CTRL+S**」 を使っていつでも保存でき、 「**CTRL+Q**」 を押してエディターを閉じます。

1. この実習ラボに必要なリソースを作成するには、次のコマンドを入力します。

    ```bash
    ./lab11-setup.azcli
    ```

    これは、実行するのに数分かかります。各ステップが完了すると、JSON 出力が表示されます。

スクリプトが完了したら、ラボを続行できます。

### 演習 2: Azure IoT Edge 対応 Linux VM をデプロイする

この演習では、 Azure Marketplace の Azure IoT Edge ランタイム サポートを使用して Ubuntu サーバー VM をデプロイします。

1. 必要に応じて、Azure アカウントの認証情報を使用して Azure portal にログインします。

    複数の Azure アカウントをお持ちの場合は、このコースで使用するサブスクリプションに関連付けられているアカウントでログインしていることを確認してください。

1. Azure portal メニューで、「**リソースの作成**」 をクリックします。

1. 「**新規**」 ブレードの 「**マーケットプレースの検索**」 ボックスに、「**Azure IoT Edge on**」と入力してから、「**Azure IoT Edge on Ubuntu**」 をクリックする      

1. 「**Azure IoT Edge on Ubuntu**」 ブレードで 「**作成**」 をクリックします。

1. 「**仮想マシンの作成**」 ブレードの 「**サブスクリプション**」 ドロップダウンで、このコースで使用している Azure サブスクリプションを選択します。   

1. 「**リソース グループ**」 の右側で、「**新規作成**」 をクリックします。   

    VM 用の新しいリソース グループを作成します。これは完了後のクリーンアップを助けるものです。

1. 新しいリソース グループのポップアップで、「**名前**」 の下に「**rg-az220vm**」を入力してから、「**OK**」 をクリックします。     

1. **仮想マシン名** テキスト ボックスに、**vm-az220-training-edge0001-{your-id}** を入力します。

1. 「**地域**」 ドロップダウンで、Azure IoT Hub がプロビジョニングされているリージョンを選択します。 

1. **可用性オプション**を、「**インフラストラクチャ冗長は必要ありません**」 のままにします。

1. 「**イメージ**」 フィールドは、**Ubuntu Server 16.04 LTS + Azure IoT Edge ランタイム**イメージを使用するように構成されていることに注意してください。

1. **Azure Spot インスタンス**を 「**いいえ**」 に設定したままにします。

1. 「**サイズ**」 の右側にある 「**サイズの変更**」 をクリックします。

1. 「**VM サイズの選択**」 ブレードの 「**VM サイズ**」 で、「**DS1_v2**」、「**選択**」 の順にクリックします。

    DS1_v2 が一覧に表示されない場合は、「**すべてのフィルターをクリア**」 リンクを使用して、このサイズを一覧で表示されるようにする必要があります。

    > **注意**:  すべてのリージョンですべての VM サイズを使用できるわけではありません。後の手順で VM サイズを選択できない場合は、別のリージョンを試してください。たとえば、**米国西部** で利用できるサイズがない場合は、**米国西部 2** を試してみてください。 

1. 「**管理者アカウント**」 の 「**認証の種類**」 の右側で、「**パスワード**」 をクリックします。     

1. VM 管理者アカウントの場合は、「**ユーザー名**」、「**パスワード**」、「**パスワードの確認入力**」 の各フィールドの値を入力します。

    > **重要:** これらの値を失くしたり忘れたりしないでください - それらが無いと VM に接続できません。

1. **受信ポートの規則**は、VM の受信 **SSH** アクセスを有効にするために構成されていることに注意してください。

    これは、VM をにリモートで設定して管理するために使用されます。

1. **「確認および作成」** をクリックします。

1. 「**検証に成功しました**」というメッセージがブレードの上部に表示されるのを待ち、「**作成**」 をクリックします。   

    > **注意**:  デプロイが完了するには 5 分ほどかかる場合があります。デプロイ中に次の演習に進むことができます。

### 演習 3: Azure CLI を使用して IoT ハブで IoT Edge デバイス ID を作成する

この演習では、Azure CLI を使用して、Azure IoT Hub 内に新しい IoT Edge デバイス ID を作成します。

1. 必要に応じて、Azure アカウントの認証情報を使用して Azure portal にログインします。

    複数の Azure アカウントをお持ちの場合は、このコースで使用するサブスクリプションに関連付けられているアカウントでログインしていることを確認してください。

1. Azure portal のツール バーで、Azure Cloud Shell を開き、「**Cloud Shell**」 をクリックします。 

    左側のナビゲーション メニューではなく、Azure portal のツール バー上にある Cloud Shell ボタンには、コマンド プロンプトと似たようなアイコンがあります。

1. **Bash** 環境オプションを使用していることを確認します。

    Cloud Shell の左上隅の環境ドロップダウンで、Bash を選択する必要があります。

1. コマンド プロンプトで、IoT Hub に IoT Edge デバイス ID を作成するには、次のコマンドを入力します。

    ```cmd/sh
    az iot hub device-identity create --hub-name iot-az220-training-{your-id} --device-id sensor-th-0067 --edge-enabled
    ```

    このコースの開始時に作成した `{your-id}` のプレースホルダーを、YOU-ID 値に必ず置き換えてください。

    > **注意**: Azure portal で IoT Hub を使用して、この IoT Edge デバイスを作成することもできます。「**IoT Hub**」 -> 「**IoT Edge**」 -> 「**IoT Edge デバイスの追加**」

1. コマンドで作成された出力をレビューします。 

    出力には、IoT Edge デバイス用に作成された**デバイス ID** に関する情報が含まれています。たとえば、自動生成されたキーを使用した `symmetricKey` 認証が既定として設定されており、`iotEdge` 機能が、指定された `--edge-enabled` パラメーターで示されるとおり `true` に設定されてることを確認できます。

    ```json
    {
        "authentication": {
            "symmetricKey": {
                "primaryKey": "jftBfeefPsXgrd87UcotVKJ88kBl5Zjk1oWmMwwxlME=",
                "secondaryKey": "vbegAag/mTJReQjNvuEM9HEe1zpGPnGI2j6DJ7nECxo="
            },
            "type": "sas",
            "x509Thumbprint": {
                "primaryThumbprint": null,
                "secondaryThumbprint": null
            }
        },
        "capabilities": {
            "iotEdge": true
        },
        "cloudToDeviceMessageCount": 0,
        "connectionState": "Disconnected",
        "connectionStateUpdatedTime": "0001-01-01T00:00:00",
        "deviceId": "sensor-th-0067",
        "deviceScope": "ms-azure-iot-edge://sensor-th-0067-637093398936580016",
        "etag": "OTg0MjI1NzE1",
        "generationId": "637093398936580016",
        "lastActivityTime": "0001-01-01T00:00:00",
        "status": "enabled",
        "statusReason": null,
        "statusUpdatedTime": "0001-01-01T00:00:00"
    }
    ```

1. IoT Edge デバイスの**接続文字列**を表示するには、次のコマンドを入力します。 

    ```cmd/sh
    az iot hub device-identity show-connection-string --device-id sensor-th-0067 --hub-name iot-az220-training-{your-id}
    ```

    このコースの開始時に作成した `{your-id}` のプレースホルダーを、YOU-ID 値に必ず置き換えてください。

1. コマンドの JSON 出力から `connectionString` の値をコピーし、後で参照するために保存します。

    この接続文字列は、IoT Edge デバイスを構成して IoT ハブに接続するために使用されます。

    ```json
        {
          "connectionString": "HostName={IoTHubName}.azure-devices.net;DeviceId=sensor-th-0067;SharedAccessKey=jftBfeefPsXgrd87UcotVKJ88kBl5Zjk1oWmMwwxlME="
        }
    ```

    > **注意**:  IoT Edge デバイス接続文字列は、**IoT Hub** -> **IoT Edge** -> **Edge デバイス** -> **接続文字列 (主キー)** に移動することで、Azure portal 内からもアクセスできます。

### 演習 4: IoT Edge デバイスを IoT ハブに接続する

この演習では、IoT Edge デバイスを Azure IoT Hub に接続します。

1. IoT Edge 仮想マシンが正常にデプロイされたことを確認します。

    Azure portal で通知ウィンドウを確認できます。

1. Azure portal メニューで、**「リソース グループ」** をクリックします。

1. 「**リソース グループ**」 ブレードで、rg-az220vm リソース グループを探します。

1. **rg-az220vm** の右端の、「...」 をクリックする

1. コンテキスト メニューで、「**ダッシュボードにピン留めする**」 をクリックし、ダッシュボードに戻ります。 

    リソースへのアクセスが容易にする場合は、ダッシュボードを**編集**して、タイルを並べ替えることができます。 
 
1. **rg-az220vm** リソース グループ タイルで、IoT Edge 仮想マシンを開くために、 「**vm-az220-training-edge0001-{your-id}**」 をクリックします。   

1. 「**概要**」 ペインの上部にある 「**接続**」 をクリックしてから、「**SSH**」 をクリックします。     

1. 「**接続**」 ウィンドウの 「**4. 次のコマンドの例を実行して VM に接続する**」 で、サンプル コマンドをコピーします。

    これは、VM の IP アドレスと管理者ユーザー名を含む仮想マシンに接続するために、使用できるサンプル SSH コマンドです。コマンドは、`ssh username@52.170.205.79` のように形式化される必要があります。

    > **注意**: サンプル コマンドが `-i <private key path>` を含む場合は、テキスト エディターを使用してコマンドのその部分を削除し、更新されたコマンドをクリップボードにコピーします。

1. まだ Cloud Shell が開いていない場合は、「**Cloud Shell**」 をクリックします。 

1. Cloud Shell コマンド プロンプトで、テキスト エディターで更新した `ssh` コマンドを貼り付け、**Enter キー**を押します。 

1. 「**Are you sure you want to continue connecting (yes/no)?**」というメッセージが表示されたら、`yes` と入力して **Enter キー**を押します。   

    VM への接続を保護するために使用される証明書は自己署名されているので、このプロンプトはセキュリティの確認です。このプロンプトに対する答えは、その後の接続のために記憶され、最初の接続時にのみプロンプトが表示されます。

1. パスワードの入力を求められたら、VM のプロビジョニング時に作成した管理者パスワードを入力します。

1. 接続すると、ターミナル コマンド プロンプトが変更され、次のような Linux VM の名前が表示されます。

    ```cmd/sh
    username@vm-az220-training-edge0001-{your-id}:~$
    ```

    これにより、接続された VM が分かります。

    > **重要:** 接続すると、Edge VM に対して未処理の OS 更新プログラムがあることを知らせてくれます。  ラボの目的としてはこれを無視しますが、運用環境では、常に Edge デバイスを最新の状態に保つ必要があります。

1. Azure IoT Edge ランタイムが VM にインストールされていることを確認するには、次のコマンドを実行します。

    ```cmd/sh
    iotedge version
    ```

    このコマンドは、仮想マシンに現在インストールされている Azure IoT Edge ランタイムのバージョンを出力します。

1. Azure IoT Hub のデバイス接続文字列を使用して Edge デバイスを構成するには、次のコマンドを入力します。

    ```cmd/sh
    sudo /etc/iotedge/configedge.sh "{iot-edge-device-connection-string}"
    ```

    上記の `{iot-edge-device-connection-string}` のプレースホルダーを、IoT Edge デバイスの作成時に記録した接続文字列値に置き換えてください (コマンド ラインに引用符を必ず入れてください)。

    `/etc/iotedge/configedge.sh` スクリプトは、Azure IoT Hub に接続するために必要な接続文字列を使用して Edge デバイスを構成するために使用されます。このスクリプトは、Azure IoT Edge ランタイムの一部としてインストールされます。

1. 接続文字列が設定されていることを確認します。

    このコマンドが完了すると、入力された接続文字列を使用して Azure IoT Hub に接続するように IoT Edge デバイスが構成されます。コマンドは、設定された接続文字列を含む `接続文字列が ... に設定されました` というメッセージを出力します。

### 演習 5: Edge デバイスに Edge モジュールを追加する

この演習では、シミュレートされた温度センサーをカスタム IoT Edge モジュールとして追加し、IoT Edge デバイスで実行するようにデプロイします。

1. 必要に応じて、Azure アカウントの認証情報を使用して Azure portal にログインします。

    複数の Azure アカウントをお持ちの場合は、このコースで使用するサブスクリプションに関連付けられているアカウントでログインしていることを確認してください。

1. リソース グループ タイルで IoT ハブを開くには 「**iot-az220-training-{your-id}**」 をクリックします。

1. 「**IoT ハブ**」 ブレードの左側にある 「**デバイスの自動管理**」 で、「**IoT Edge**」 をクリックします。     

1. IoT Edge デバイスの一覧で、「**sensor-th-0067**」 をクリックします。 

1. 「**sensor-th-0067**」 ブレードで、「**モジュール**」 タブに、デバイスに対して現在構成されているモジュールのリストが表示されていることを確認します。   

    現在、IoT Edge デバイスは、IoT Edge ランタイムの一部である Edge エージェント (`$edgeAgent`) および Edge ハブ (`$edgeHub`) モジュールのみで構成されています。

1. 「**sensor-th-0067**」 ブレードの上部で、「**モジュールの設定**」 をクリックします。

1. 「**デバイスのモジュールを設定: sensor-th-0067**」 ブレードで、**IoT Edge モジュール**セクションを見つけます。   

1. 「**IoT Edge Modules**」 で 「**Add**」 をクリックしてから、「**IoT Edge Module**」 をクリックします。     

1. 「**Add IoT Edge Module**」 ペインの 「**IoT Edge Module Name**」 に、  「**tempsensor**」と入力します。   

    カスタム モジュールの名前を「tempsensor」と命名します

1. 「**Image URL**」 の下に**asaedgedockerhubtest/asa-edge-test-module:simulated-temperature-sensor** と入力します。  

    > **注意**: このイメージは、このテスト シナリオをサポートするために製品グループによって提供される Docker Hub で公開されたイメージです。

1. 選択したタブを変更するには、「**Module Twin Settings**」 をクリックします。 

1. モジュール ツインに必要なプロパティを指定するには、次の JSON を入力します。

    ```json
    {
        "EnableProtobufSerializer": false,
        "EventGeneratingSettings": {
            "IntervalMilliSec": 500,
            "PercentageChange": 2,
            "SpikeFactor": 2,
            "StartValue": 20,
            "SpikeFrequency": 20
        }
    }
    ```

    この JSON は、モジュール ツインの必要なプロパティを設定して Edge モジュールを構成します。

1. ブレードの最下部で、「**Add**」 をクリックします。

1. 「**デバイスのモジュールを設定: sensor-th-0067**」 ブレードで、ブレードの下部にある 「**Next: Routes >**」 をクリックします。

1. 既定のルートは既に構成されています。

    * Name: route
    * Value: FROM /messages/* INTO $upstream

    このルートによって、IoT Edge デバイス上のすべてのモジュールからのすべてのメッセージが IoT Hub に送信されます

1. **「Review + create」** をクリックします。

1. 「**Deployment**」 で、表示されている配置マニフェストを確認します。 

    見てのとおり、IoT Edge デバイスの配置マニフェストは非常に読みやすい JSON 形式です。

    `properties.desired` セクションの下に、IoT Edge デバイスにデプロイされる IoT Edge モジュールを宣言する `modules` セクションがあります。これには、コンテナー レジストリの資格情報を含むすべてのモジュールのイメージ URI が含まれます。

    ```json
    {
        "modulesContent": {
            "$edgeAgent": {
                "properties.desired": {
                    "modules": {
                        "tempsensor": {
                            "settings": {
                                "image": "asaedgedockerhubtest/asa-edge-test-module:simulated-temperature-sensor",
                                "createOptions": ""
                            },
                            "type": "docker",
                            "version": "1.0",
                            "status": "running",
                            "restartPolicy": "always"
                        },
    ```

    JSON の下部には、Edge Hub に必要なプロパティを含む **$edgeHub** セクションがあります。  このセクションには、モジュール間および IoT Hub へのイベントのルーティングのルーティング構成も含まれています。

    ```json
        "$edgeHub": {
            "properties.desired": {
                "routes": {
                  "route": "FROM /messages/* INTO $upstream"
                },
                "schemaVersion": "1.0",
                "storeAndForwardConfiguration": {
                    "timeToLiveSecs": 7200
                }
            }
        },
    ```

    JSON の下位のセクションには **tempsensor** モジュールのセクションがあり、その中の `properties.desired` セクションには、Edge モジュールの構成に必要なプロパティが含まれています。 

    ```json
                },
                "tempsensor": {
                    "properties.desired": {
                        "EnableProtobufSerializer": false,
                        "EventGeneratingSettings": {
                            "IntervalMilliSec": 500,
                            "PercentageChange": 2,
                            "SpikeFactor": 2,
                            "StartValue": 20,
                            "SpikeFrequency": 20
                        }
                    }
                }
            }
        }
    ```

1. ブレードの下部で、デバイスのモジュールの設定を完了するには、「**Create**」 をクリックします。 

1. 「**sensor-th-0067** ブレードの 「**モジュール**」 の下に、**tempsensor** が表示されていることを確認します。     

    > **注意**:  一覧表示されたモジュールを初めて表示するには、「**更新**」 をクリックする必要がある場合があります。

    **tempsensor** のランタイムの状態が報告されないことがあります。

1. ブレードの上部にある 「**更新**」 をクリックします。

1. **tempsensor** モジュールの **ランタイムの状態**が**running**に設定されています。

    それでも値が報告されない場合は、しばらく待ってからブレードを再度更新します。
 
1. Cloud Shell セッションを開きます (まだ開いていない場合)。

    `vm-az220-training-edge0001-{your-id}` 仮想マシンすでに接続されていない場合は、 以前と同様に **SSH** を使用して接続します。

1. Cloud Shell コマンド プロンプトで、IoT Edge デバイス上で現在実行されているモジュールを一覧表示するには、次のコマンドを入力します。

    ```cmd/sh
    iotedge list
    ```

1. コマンドの出力は、次のようなものになります。 

    ```cmd/sh
    demouser@vm-az220-training-edge0001-{your-id}:~$ iotedge list
    NAME             STATUS           DESCRIPTION      CONFIG
    edgeHub          running          Up a minute      mcr.microsoft.com/azureiotedge-hub:1.0
    edgeAgent        running          Up 26 minutes    mcr.microsoft.com/azureiotedge-agent:1.0
    tempsensor       running          Up 34 seconds    asaedgedockerhubtest/asa-edge-test-module:simulated-temperature-sensor
    ```

    `tempsensor` が動作中のモジュールの 1 つとしてリストされていることに注意してください。

1. モジュール ログを表示するには、次のコマンドを入力します。

    ```cmd/sh
    iotedge logs tempsensor
    ```

    コマンドの出力は、次のようなものになります。

    ```cmd/sh
    demouser@vm-az220-training-edge0001-{your-id}:~$ iotedge logs tempsensor
    11/14/2019 18:05:02 - Send Json Event : {"machine":{"temperature":41.199999999999925,"pressure":1.0182182583425192},"ambient":{"temperature":21.460937846433808,"humidity":25},"timeCreated":"2019-11-14T18:05:02.8765526Z"}
    11/14/2019 18:05:03 - Send Json Event : {"machine":{"temperature":41.599999999999923,"pressure":1.0185790159334602},"ambient":{"temperature":20.51992724976499,"humidity":26},"timeCreated":"2019-11-14T18:05:03.3789786Z"}
    11/14/2019 18:05:03 - Send Json Event : {"machine":{"temperature":41.999999999999922,"pressure":1.0189397735244012},"ambient":{"temperature":20.715225311096397,"humidity":26},"timeCreated":"2019-11-14T18:05:03.8811372Z"}
    ```

    `iotedge logs` コマンドを使用して、任意の Edge モジュールのモジュール ログを表示できます。

1. シミュレートされた温度センサー モジュールは、500 メッセージを送信した後に停止します。次のコマンドを実行して再開できます。

    ```cmd/sh
    iotedge restart tempsensor
    ```

    今すぐモジュールを再起動する必要はありませんが、後でテレメトリの送信を停止した場合は、Cloud Shell に戻って Edge VM にSSH 接続し、このコマンドを実行してリセットします。リセットされると、モジュールはテレメトリの送信を再開します。

### 演習 6: IoT Edge モジュールとして Azure Stream Analytics をデプロイする

これで、tempSensor モジュールが IoT Edge デバイスにデプロイされ、実行されるようになったので、IoT Edge デバイスでメッセージを処理できる Stream Analytics モジュールを追加してから、IoT Hub に送信できます。

#### タスク 1: Azure ストレージ アカウントを作成する

1. 必要に応じて、Azure アカウントの認証情報を使用して Azure portal にログインします。

    複数の Azure アカウントをお持ちの場合は、このコースで使用するサブスクリプションに関連付けられているアカウントでログインしていることを確認してください。

1. Azure portal メニューで、「**リソースの作成**」 をクリックします。

1. 「**新規**」 ブレードの検索テキスト ボックスに、「**storage**」と入力し、**Enter** を押します。     

1. 「**Marketplace** ブレードで、「**ストレージ アカウント**」 をクリックします。

1. 「**ストレージ アカウント**」 ブレードで、「**作成**」 クリックします。

1. 「**ストレージ アカウントの作成**」 ブレードで、「サブスクリプション」 ドロップダウンが、このコースで使用しているサブスクリプションを表示していることを確認します。 

1. 「**リソース グループ**」 ドロップダウンで、「**rg-az220**」 をクリックします。

1. 「**ストレージ アカウント名**」 テキストボックスに、「**az220store{YOUR-ID}**」と入力します。   

    > **注意**: このフィールドでは、{your-id} を小文字で入力する必要があり、ダッシュ文字や下線文字は使用できません。

1. 「**場所**」 フィールドを、Azure IoT Hub で使用されているのと同じ Azure リージョンに設定します。 

1. 「**レプリケーション**」 フィールド を**ローカル冗長ストレージ (LRS)** に設定します。

1. 他のすべての設定をそのままにします。

1. ブレードの最下部で、**確認および作成**をクリックします。

1. 「**検証が成功しました**」のメッセージが表示されるまで待ってから、「**作成**」 をクリックします。   

    デプロイが完了するまでに少し時間がかかる場合があります。作成している間に Stream Analytics のリソースの作成を継続できます。

#### タスク 2: Azure Stream Analytics の作成

1. Azure portal メニューで、「**リソースの作成**」 をクリックします。

1. 「**新規**」 ブレードの 「**Azure Marketplace**」 で、「**モノのインターネット**」 をクリックしてから、「**Stream Analytics ジョブ**」 をクリックします。       

1. 「**新しい Stream Analytics ジョブ**」 ブレードの 「**ジョブ名**」 フィールドに、「**asa-az220-training-{your-id}**」を入力します

1. 「**リソース グループ**」 ドロップダウンで、「**rg-az220**」 をクリックします。

1. 「**場所**」 ドロップダウンで、ストレージ アカウントと Azure IoT Hub に使用される Azure リージョンと同じリージョンを選択します。 

1. 「**ホスティング環境**」 フィールドを **Edge** に設定します。   

    これにより、Stream Analytics ジョブがオンプレミスの IoT ゲートウェイ Edge デバイスにデプロイされます。

1. ブレードの最下部で、**作成**をクリックします。

    このリソースがデプロイされるまで数分かかります。

#### タスク 3: Azure Stream Analytics ジョブを構成する

1. 「**デプロイが完了しました**」というメッセージが表示されたら 、「**リソースに移動**」 をクリックします。   

    新しい Stream Analytics ジョブの 「概要」 ウィンドウに表示されている必要があります。
 
1. 左側のナビゲーション メニューの 「**ジョブ トポロジ**」 で、「**入力**」 をクリックします。   

1. 「**入力**」 ペインで、「**ストリーム入力の追加**」 をクリックしてから、「**Event Hub**」 をクリックします。

1. 「**Edge Hub**」 ペインの 「**入力エイリアス**」 フィールドに、「**temperature**」を入力します。

1. 「**イベントのシリアル化形式**」 ドロップダウンで、**JSON** が選択されていることを確認します。    

    Stream Analytics は、メッセージ形式を理解する必要があります。JSON は標準形式です。

1. 「**エンコード**」 ドロップダウンで、**UTF-8** が選択されていることを確認します。

    > **注意**:  UTF-8 は、書き込み時にサポートされている唯一の JSON エンコードです。

1. 「**イベント圧縮タイプ**」 ドロップダウンで、「**なし**」 が選択されていることを確認します。   

    この課題では、圧縮を使用しません。GZip および  Deflate 形式もサービスでサポートされています。

1. 画面の最下部で、「**保存**」 をクリックします。

1. 左側のナビゲーション メニューの 「**ジョブ トポロジ**」 で、「**出力**」 をクリックします。   

1. 「**出力**」 ペインで、「**追加**」 をクリックしてから、「**Edge Hub**」 をクリックします。

1. 「**Edge Hub**」 ペインの 「**出力エイリアス**」 フィールドに、「**alert**」と入力します。   

1. 「**イベントのシリアル化形式**」 ドロップダウンで、**JSON** が選択されていることを確認します。   

    Stream Analytics は、メッセージ形式を理解する必要があります。JSON は標準形式ですが、サービスでは CSV もサポートされています。

1. 「**形式**」 ドロップダウンで、「**改行区切り**」 が選択されていることを確認します。

1. 「**エンコード**」 ドロップダウンで、**UTF-8** が選択されていることを確認します。

    > **注意**:  UTF-8 は、書き込み時にサポートされている唯一の JSON エンコードです。

1. 画面の最下部で、「**保存**」 をクリックします。

1. 左側のナビゲーション メニューの 「**ジョブ トポロジ**」 で、「**クエリ**」 をクリックします。

1. 「**クエリ**」 ペインで、既定のクエリを次に置き換えます。

    ```sql
    SELECT  
        'reset' AS command
    INTO
        alert
    FROM
        temperature TIMESTAMP BY timeCreated
    GROUP BY TumblingWindow(second,15)
    HAVING Avg(machine.temperature) > 25
    ```

    このクエリは、`temperature` 入力に入るイベントを調べ、15 秒間隔のタンブリング ウィンドウでグループ化して、そのグループ内の平均温度値が 25 よりも大きいかどうかをチェックします。平均が 25 より大きい場合、`command` プロパティが `reset` の値に設定されたイベントを `alert` 出力に送信します。

    `TumblingWindow` 関数の詳細については、次のリンクを参照してください: [https://docs.microsoft.com/ja-jp/stream-analytics-query/tumbling-window-azure-stream-analytics](https://docs.microsoft.com/ja-jp/stream-analytics-query/tumbling-window-azure-stream-analytics)

1. クエリ エディターの上部にある 「**クエリの保存**」 をクリックします。

#### タスク 4: ストレージ アカウント設定を構成する

IoT Edge デバイスにデプロイされるように Stream Analytics ジョブを準備するには、Azure BLOB Storage コンテナーに関連付ける必要があります。ジョブがデプロイされると、ジョブ定義がストレージ コンテナーにエクスポートされます。

1. 「**Stream Analytics ジョブ**」 ブレード上にある、左側のナビゲーション メニューの 「**構成**」 で、「**ストレージ アカウントの設定**」 をクリックします。     

1. 「**ストレージ アカウントの設定**」 ペインで 「**ストレージ アカウントの追加**」 をクリックします。

1. 「**ストレージ アカウントの設定**」 で、「**サブスクリプションからストレージ アカウントを選択**」 が選択されていることを確認します。   

1. 「**ストレージ アカウント**」 ドロップダウンで、**az220store{your-id}** ストレージ アカウントが選択されていることを確認します。   

1.  「**コンテナー**」 で 「**新規作成**」 をクリックし、コンテナーの名前として「**jobdefinition**」を入力します。     

1. ペインの上部にある 「**保存**」 をクリックします。

    変更を保存するかどうかを確認するメッセージが表示されたら、「**はい**」 をクリックします

#### タスク 5:  Streaming Analytics ジョブをデプロイする

1. Azure portal で、**iot-az220-training-{your-id}** IoT Hub リソースに移動します。 

1. 左側のナビゲーション メニューの 「**自動デバイス管理**」 で、「**IoT Edge**」 をクリックします。   

1. 「**デバイス ID**」 の 「**sensor-th-0067**」 をクリックします。   

1. 「**sensor-th-0067**」 ペインの上部にある 「**モジュールの設定**」 をクリックします。   

1. 「**デバイスのモジュールを設定: sensor-th-0067**」 ペインで、「**IoT Edge Modules**」 セクションを見つけます。   

1. 「**IoT Edge モジュール**」 の下で 「**Add**」 をクリックしてから、「**Azure Stream Analytics Module**」 をクリックします。     

1. 「**Edge の展開**」 ペインの 「**サブスクリプション**」 で、このコースに使用するサブスクリプションが選択されていることを確認します。   

1. 「**Edge ジョブ**」 ドロップダウンで、**asa-az220-training-{your-id}** Steam Analytics ジョブが選択されていることを確認します。   

    > **注意**:  ジョブは既に選択されていても、「**保存**」 ボタンが無効になっている可能性があります - 「**Edge ジョブ**」 のドロップダウンをもう一度開いて **asa-az220-training-{your-id}** ジョブをもう一度選択します。      これで 「**保存**」 ボタンが有効になるはずです。 

1. 画面の最下部で、「**保存**」 をクリックします。

    デプロイには数分かかります。

1. Edge パッケージが正常に公開されたら、新しい ASA モジュールが **IoT Edge Modules**セクションの下に表示されていることを確認します。

1. **IoT Edge Modules**で、**asa-az220-training-{your-id}** をクリックします。    

    これは、Edge デバイスに追加された Steam Analytics モジュールです。

1. 「**Update IoT Edge Module**」 ペインで、**Image URI** が「標準の Azure Stream Analytics」イメージを指していることに注意してください。 

    ```text
    mcr.microsoft.com/azure-stream-analytics/azureiotedge:1.0.6
    ```

    これは、IoT Edge デバイスに展開されるすべての ASA ジョブに使用されるイメージと同じです。

    > **注意**:  構成された**イメージ URI** の最後にあるバージョン番号には、Stream Analytics モジュールを作成したときの最新バージョンが反映されます。  このユニットを書いている時点では、バージョンは `1.0.6` でした。

1. すべての値を既定値のままにして、「**IoT Edge カスタム モジュール**」 ペインを閉じます。

1. 「**デバイスのモジュールを設定: sensor-th-0067**」 ペインで、「**Next: Routes >**」  をクリックします。。

    既存のルーティングが表示されることを確認します。

1. 定義されている既定のルートを次の 3 つのルートに置き換えます。

    * Route 1
        * NAME: **telemetryToCloud**
        * VALUE: `FROM /messages/modules/tempsensor/* INTO $upstream`
    * Route 2
        * NAME: **alertsToReset**
        * VALUE: `FROM /messages/modules/asa-az220-training-{your-id}/* INTO BrokeredEndpoint("/modules/tempsensor/inputs/control")`
    * Route 3
        * NAME: **telemetryToAsa**
        * VALUE: `FROM /messages/modules/tempsensor/* INTO BrokeredEndpoint("/modules/asa-az220-training-{your-id}/inputs/temperature")`

    > **注意**: `asa-az220-training-{your-id}` プレースホルダーを Azure Stream Analytics ジョブ モジュールの名前に必ず置き換えてください。  モジュールとその名前のリストを表示するために 「**Previous**」 をクリックしてから、このステップに戻るために 「**Next**」 をクリックします。   

    定義されるルートは次のとおりです。

    * **telemetryToCloud** ルートは、`tempsensor` モジュール出力から Azure IoT Hub にすべてのメッセージを送信します。 
    * **alertsToReset** ルートは、Stream Analytics モジュール出力から **tempsensor** モジュールの入力に、すべての警告メッセージを送信します。   
    * **telemetryToAsa** ルートは、`tempsensor` モジュール出力から Stream Analytics モジュール入力にすべてのメッセージを送信します。 

1. **「デバイスのモジュールを設定: sensor-th-0067**」 ブレードの下部にある、**「Review + create**」 をクリックします。   

1. 「**Review + create**」 タブで、**Deployment Manifest** JSON が、Stream Analytics モジュールと、たった今構成されたルーティング定義で更新されたことを確認します。

1. `tempsensor` でシミュレートされた温度センサー モジュールの JSON 構成を確認します。

    ```json
    "tempsensor": {
        "settings": {
            "image": "asaedgedockerhubtest/asa-edge-test-module:simulated-temperature-sensor",
            "createOptions": ""
        },
        "type": "docker",
        "version": "1.0",
        "status": "running",
        "restartPolicy": "always"
    },
    ```

1. 以前に構成されたルートの JSON 構成と、JSON デプロイ定義で構成されている方法を確認します。

    ```json
    "$edgeHub": {
        "properties.desired": {
            "routes": {
                "telemetryToCloud": "FROM /messages/modules/tempsensor/* INTO $upstream",
                "alertsToReset": "FROM /messages/modules/asa-az220-training-CP122619/* INTO BrokeredEndpoint(\\\"/modules/tempsensor/inputs/control\\\")",
                "telemetryToAsa": "FROM /messages/modules/tempsensor/* INTO BrokeredEndpoint(\\\"/modules/asa-az220-training-CP122619/inputs/temperature\\\")"
            },
            "schemaVersion": "1.0",
            "storeAndForwardConfiguration": {
                "timeToLiveSecs": 7200
            }
        }
    },
    ```

1. ブレードの最下部で、**Create**をクリックします。

#### タスク 6: データの表示

1. **SSH** 経由で **IoT Edge デバイス**に接続している **Cloud Shell** セッションに戻ります。       

    閉じているか、タイムアウトしている場合は、再接続します。`SSH` コマンドを実行し、前と同じログインを行います。

1. コマンド プロンプトで、デバイスにデプロイされたモジュールの一覧を表示するには、次のコマンドを入力します。

    ```cmd/sh
    iotedge list
    ```

    新しい Stream Analytics モジュールが IoT Edge デバイスにデプロイされるには、1 分かかる場合があります。それが表示されると、このコマンドの出力リストに表示されます。

    ```cmd/sh
    demouser@vm-az220-training-edge0001-{your-id}:~$ iotedge list
    NAME               STATUS           DESCRIPTION      CONFIG
    AZ-220-ASA-CP1119  running          Up a minute      mcr.microsoft.com/azure-stream-analytics/azureiotedge:1.0.5
    edgeAgent          running          Up 6 hours       mcr.microsoft.com/azureiotedge-agent:1.0
    edgeHub            running          Up 4 hours       mcr.microsoft.com/azureiotedge-hub:1.0
    tempsensor         running          Up 4 hours       asaedgedockerhubtest/asa-edge-test-module:simulated-temperature-sensor
    ```

    > **注意**:  Stream Analytics モジュールがリストに表示されない場合は、1?2 分待ってから、もう一度やり直してください。IoT Edge デバイスでモジュールのデプロイが更新されるには、1 分かかることがあります。

1. コマンド プロンプトで、`tempsensor` モジュールによって Edge デバイスから送信されるテレメトリを監視するには、次のコマンドを入力します。

    ```cmd/sh
    iotedge logs tempsensor
    ```

1. 出力を確認するには、少し時間がかかります。

    **tempsensor** によって送信される温度テレメトリを監視している間、`machine.temperature` の平均が `25`を超えると、Stream Analytics ジョブによって**リセット**コマンドが送信されることに注意してください。    これは、Stream Analytics ジョブ クエリで構成されたアクションです。

    このイベントの出力は、以下のようなものになります。

    ```cmd/sh
    11/14/2019 22:26:44 - Send Json Event : {"machine":{"temperature":231.599999999999959,"pressure":1.0095600761599359},"ambient":{"temperature":21.430643635304012,"humidity":24},"timeCreated":"2019-11-14T22:26:44.7904425Z"}
    11/14/2019 22:26:45 - Send Json Event : {"machine":{"temperature":531.999999999999957,"pressure":1.0099208337508767},"ambient":{"temperature":20.569532965342297,"humidity":25},"timeCreated":"2019-11-14T22:26:45.2901801Z"}
    Received message
    Received message Body: [{"command":"reset"}]
    Received message MetaData: {"MessageId":null,"To":null,"ExpiryTimeUtc":"0001-01-01T00:00:00","CorrelationId":null,"SequenceNumber":0,"LockToken":"e0e778b5-60ff-4e5d-93a4-ba5295b995941","EnqueuedTimeUtc":"0001-01-01T00:00:00","DeliveryCount":0,"UserId":null,"MessageSchema":null,"CreationTimeUtc":"0001-01-01T00:00:00","ContentType":"application/json","InputName":"control","ConnectionDeviceId":"sensor-th-0067","ConnectionModuleId":"asa-az220-training-CP1119","ContentEncoding":"utf-8","Properties":{},"BodyStream":{"CanRead":true,"CanSeek":false,"CanWrite":false,"CanTimeout":false}}
    Resetting temperature sensor..
    11/14/2019 22:26:45 - Send Json Event : {"machine":{"temperature":320.4,"pressure":0.99945886361358849},"ambient":{"temperature":20.940019742324957,"humidity":26},"timeCreated":"2019-11-14T22:26:45.7931201Z"}
    ```