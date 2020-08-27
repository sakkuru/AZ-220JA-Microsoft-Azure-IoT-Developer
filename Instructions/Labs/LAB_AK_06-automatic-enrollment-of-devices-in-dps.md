---
lab:
    title: 'ラボ 06: DPS を使用して IoT デバイスを安全かつ大規模に自動プロビジョニングする'
    module: 'モジュール 3: 大規模なデバイス プロビジョニング'
---

# DPS を使用して IoT デバイスを安全かつ大規模に自動プロビジョニングする

## ラボ シナリオ

Contoso の資産の監視および追跡ソリューションに対する現在までの作業により、個々の登録を使用してデバイスのプロビジョニングとプロビジョニング解除のプロセスを検証することが可能になりました。管理チームから、より大規模なプロセスの展開を開始するよう指示を受けました。

プロジェクトを進めるには、デバイス プロビジョニング サービスを使用して、より多くのデバイスを自動的に登録し、X.509 証明書認証を使用して、安全に登録できることを実証する必要があります。

次のリソースが作成されます。

![ラボ 6 アーキテクチャ](media/LAB_AK_06-architecture.png)

## このラボでは

このラボでは、次のタスクを完了します。

* ラボの前提条件が満たされていることを確認する (必要な Azure リソースがあること)
* OpenSSL を使用した X.509 CA 証明書の生成と構成
* DPS でグループ登録を作成する
* X.509 証明書を使用したシミュレートされたデバイスを構成する
* デバイス ツインの必要なプロパティの変更を処理します。
* シミュレートされたデバイスの自動登録
* シミュレートされたデバイスのグループ登録を削除

## ラボの手順

### 演習 1: ラボの前提条件を確認する

このラボでは、次の Azure リソースが使用可能であることを前提としています。

| リソースの種類:  | リソース名 |
| :-- | :-- |
| リソース グループ | rg-az220 |
| IoT Hub | iot-az220-training-{your-id} |
| デバイス プロビジョニング サービス | dps-az220-training-{your-id} |

これらのリソースを利用できない場合は、演習 2 に進む前に、以下の手順に従って  **lab06-setup.azcli**スクリプトを実行する必要があります。スクリプト ファイルは、開発環境構成 (ラボ 3) の一部としてローカルに複製した GitHub リポジトリに含まれています。

**lab06-setup.azcli**スクリプトは**Bash** シェル環境で実行するように記述されています 。 これを実行する最も簡単な方法は Azure Cloud Shell で実行することです。

1. ブラウザーを使用して [Azure Cloud Shell](https://shell.azure.com/) を開き、このコースで使用している Azure サブスクリプションでログインします。

    Cloud Shell のストレージの設定に関するメッセージが表示された場合は、デフォルトをそのまま使用します。

1. Azure Cloud Shell が **Bash** を使用していることを確認 します。

    「Azure Cloud Shell」 ページの左上隅にあるドロップダウンは、環境を選択するために使用されます。選択されたドロップダウンの値が **Bash**であることを確認します。 

1. Azure Shell ツール バーで、「**ファイルのアップロード/ダウンロード**」 をクリックします (右から 4 番目のボタン)。

1. ドロップダウンで、「**アップロード**」 をクリックします。

1. ファイル選択ダイアログで、開発環境を構成したときにダウンロードした GitHub ラボ ファイルのフォルダーの場所に移動します。

    _ラボ 3: 開発環境の設定_:ZIP ファイルをダウンロードしてコンテンツをローカルに抽出することで、ラボ リソースを含む GitHub リポジトリを複製しました。抽出されたフォルダー構造には、次のフォルダー パスが含まれます。

    * Allfiles
      * Labs
          * 06-Automatic Enrollment of Devices in DPS
            * Setup

    lab06-setup.azcli スクリプト ファイルは、ラボ 6 のセットアップ フォルダにあります。

1. **lab06-setup.azcli** ファイルを選択し、**「開く」** をクリックします。

    ファイルのアップロードが完了すると、通知が表示されます。

1. 正しいファイルが Azure Cloud Shell にアップロードされたことを確認するには、次のコマンドを入力します。

    ```bash
    ls
    ```

    `ls` コマンドを実行すると、現在のディレクトリの内容が一覧表示されます。lab06-setup.azcli ファイルが一覧表示されます。

1. セットアップ スクリプトを含むディレクトリをこのラボ用に作成し、そのディレクトリに移動するには、次の Bash コマンドを入力します。

    ```bash
    mkdir lab6
    mv lab06-setup.azcli lab6
    cd lab6
    ```

1. **lab06-setup.azcli** スクリプトに実行権限があることを確認するには、次のコマンドを入力します。 

    ```bash
    chmod +x lab06-setup.azcli
    ```

1. Cloud Shell ツール バーで、lab06-setup.azcli ファイルを編集するには、「**エディターを開く**」 (右から 2 番目のボタン - **{ }**) をクリックします。 

1. 「**ファイル**」 の一覧で、lab6 フォルダを展開してスクリプト ファイルを開き、**lab6** をクリックし、**lab06-setup.azcli** をクリックします。

    エディターに、**lab06-setup.azcli** ファイルの内容が表示されます。

1. エディターで、`{YOUR-ID}` と `{YOUR-LOCATION}` 変数の値を更新します。

    サンプル例として、このコースの最初に作成した一意の ID 、つまり **CAH191211** に `{YOUR-ID}` を設定し、リソースにとって意味のある場所に `{YOUR-LOCATION}` を設定する必要があります。

    ```bash
    #!/bin/bash

    RGName="rg-az220"
    IoTHubName="iot-az220-training-{your-id}"

    Location="{YOUR-LOCATION}"
    ```

    > **注意**:  `{YOUR-LOCATION}` 変数は、リージョンの短い名前に設定する必要があります。次のコマンドを入力すると、使用可能な領域とその短い名前 (「**名前**」 列) の一覧を表示できます。
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
    ./lab06-setup.azcli
    ```

    これを実行するには数分かかります。各ステップが完了すると、JSON 出力が表示されます。

    スクリプトが完了したら、ラボを続行できます。

### 演習 2: OpenSSL を使用した X.509 CA 証明書の生成と構成

この演習では、 Azure Cloud Shell 内で OpenSSL を使用して X.509 CA 証明書を生成します。この証明書は、デバイス プロビジョニング サービス (DPS) 内のグループ登録を構成するために使用されます。

#### タスク 1: 証明書の生成

1. 必要に応じて、 このコースで使用している Azure アカウントの認証情報を使用して [Azure portal](https://portal.azure.com) にログインします。 

    複数の Azure アカウントをお持ちの場合は、このコースで使用するサブスクリプションに関連付けられているアカウントでログインしていることを確認してください。

1. ポータル ウィンドウの右上で、Azure Cloud Shell を開きます。 

    「Cloud Shell」 ボタンには、コマンド プロンプトを表すアイコン **`>_`** が表示されます。

    表示画面の下部付近に Cloud Shell  ウィンドウが開きます。

1. Cloud Shell ウィンドウの左上隅で、環境オプションとして「**Bash**」が選択されていることを確認します。

    > **注意**:  Azure Cloud Shell では、*Bash* インターフェイスと *PowerShell* インターフェイスの両方で **OpenSSL** の使用がサポートされています。この演習では、*Bash* シェル用に書かれたヘルパー スクリプトをいくつか使用します。 

1. Cloud Shellコマンドプロンプトで、新しいディレクトリを作成してそこに移動するには、次のコマンドを入力します。

    ```sh
    # make a directory named "certificates"
    mkdir certificates

    # change directory to the "certificates" directory
    cd certificates
    ```

1. Cloud Shell コマンド プロンプトで、使用する Azure IoT ヘルパー スクリプトをダウンロードするには、次のコマンドを入力します。

    ```sh
    # download helper script files
    curl https://raw.githubusercontent.com/Azure/azure-iot-sdk-c/master/tools/CACertificates/certGen.sh --output certGen.sh
    curl https://raw.githubusercontent.com/Azure/azure-iot-sdk-c/master/tools/CACertificates/openssl_device_intermediate_ca.cnf --output openssl_device_intermediate_ca.cnf
    curl https://raw.githubusercontent.com/Azure/azure-iot-sdk-c/master/tools/CACertificates/openssl_root_ca.cnf --output openssl_root_ca.cnf

    # update script permissions so user can read, write, and execute it
    chmod 700 certGen.sh
    ```

    ヘルパー スクリプトとサポート ファイルは、Github でホストされている `Azure/azure-iot-sdk-c` オープン ソース プロジェクトからダウンロードされます。これは、Azure IoT SDK の一部であるオープン ソース プロジェクトです。ヘルパー スクリプトの `certGen.sh` は、このラボの範囲外の OpenSSL 構成の詳細に取り込まずに CA 証明書の目的を示す手助けとなります。

    このヘルパー スクリプトの使用方法、および Bash の代わりに PowerShell を使用する方法については、次のリンクを参照 「してください[https://github.com/Azure/azure-iot-sdk-c/blob/master/tools/CACertificates/CACertificateOverview.md](https://github.com/Azure/azure-iot-sdk-c/blob/master/tools/CACertificates/CACertificateOverview.md)

    > **警告**: このヘルパー スクリプトで作成された証明書は、プロダクションで **使用しないでください**。これらのパスワードには、ハードコードされたパスワード (「1234」) が含まれ、30 日後に期限切れになり、最も重要なのは、CA 証明書を理解しやすくするためのデモ用に提供されています。CA 証明書に対して製品を構築する場合は、証明書の作成と有効期間の管理に独自のセキュリティのベスト プラクティスを使用する必要があります。

    興味がある場合は、Cloud Shell に組み込まれているエディターを使用して、ダウンロードしたスクリプト ファイルの内容をすばやくスキャンできます。

    * Cloud Shell でエディターを開くには、**`{}`** をクリックします。 
    * 「ファイル」 の一覧で 「**証明書**」 をクリックし、「**certGen.sh**」 をクリックします

    > **注意**: Bash 環境で他のテキスト ファイル表示ツール (`more` コマンドや `vi` コマンドなど) を使用した経験がある場合は、それらのツールを使用することもできます。

    次の手順では、スクリプトを使用してルート証明書と中間証明書を作成します。

1. ルート証明書と中間証明書を生成するには、次のコマンドを入力します。

    ```sh
    ./certGen.sh create_root_and_intermediate
    ```

    スクリプトを `create_root_and_intermediate` オプションで実行していることに注意してください。このコマンドは、`~/certificates` ディレクトリからスクリプトを実行していることを前提としています。

    このコマンドは、`azure-iot-test-only.root.ca.cert.pem` という名前のルート CA 証明書を生成し、それを `./certs` ディレクトリ (作成した証明書ディレクトリの下) に配置しました。

1. ルート証明書を (DPS にアップロードできるように) ローカル マシンにダウンロードするには、次のコマンドを入力します。

    ```sh
    download ~/certificates/certs/azure-iot-test-only.root.ca.cert.pem
    ```

    ファイルをローカル コンピュータに保存するように求められます。ファイルの保存場所をメモしておきます。次のタスクで必要になります。

#### タスク 2: ルート証明書を信頼するように DPS を構成する

1. Azure portal で、デバイス プロビジョニング サービスを開きます。

    これは、デバイス プロビジョニング サービス名 `dps-az220-training-{your-id}` です。

1. 「**デバイス プロビジョニング サービス**」 ブレードの左側にある 「**設定**」 セクションで、「**証明書**」 をクリックします。     

1. 「**証明書**」 ウィンドウの上部にある 「**追加**」 をクリックします。  

    「**追加**」 をクリックすると、X.509 CA 証明書を DPS サービスにアップロードするプロセスが開始されます。 

1. 「**証明書の追加**」ウィンドウの 「**証明書名**」に「**root-ca-cert**」と入力します。     

    ルート証明書と中間証明書、またはチェーン内の階層レベルにおける複数の証明書などの、証明書を区別できる名前を付ける必要があります。

    > **注意**: 入力したルート証明書名は、証明書ファイルの名前と同じか、または別の名前である可能性があります。指定した名前は、コンテンツ X.509 CA 証明書に埋め込まれている _共通名 _ とは相関関係のない論理名です。

1. **Certificate .pem または .cer ファイル**で、 _ファイルの選択_ テキスト ボックスの右側にある「**開く**」をクリックします。     

    テキスト フィールドの右の 「**開く**」 ボタンをクリックすると、「ファイルを開く」 ダイアログが開き、前にダウンロードした `azure-iot-test-only.root.ca.cert.pem` CA 証明書に移動できます。

1. 画面の下部で、「**保存**」 をクリックします。

    X.509 CA 証明書がアップロードされると、「証明書」 ペインに _「Unverified」_ の状態の証明書が表示されます。  この CA 証明書を使用して DPS に対してデバイスを認証できるようになる前に、証明書の**所有証明**を確認する必要があります。

1. 証明書の**所有証明**の確認プロセスを開始するには  **root-ca-cert** をクリックします。   

1. 「**証明書の詳細**」 ペインで、「**確認コードの生成**」 をクリックします。

    「**確認コードの生成**」 ボタンを表示するには、下にスクロールする必要があります。

    ボタンをクリックすると、生成されたコードが 「確認コード」 フィールドに入力されます。

1. 「**確認コード**」 の右側にある 「**クリップボードにコピー**」 をクリックします。   

    CA 証明書から生成された証明書を DPS 内で生成した確認コードと共にアップロードすることにより、CA 証明書の _所有証明_ が DPS に提供されます。これは、CA 証明書を実際に所有していることを証明する方法です。

    > **重要**: 検証証明書を生成する間は、「**証明書の詳細**」ウィンドウを開いたままにする必要があります。  ウィンドウを閉じると、確認コードが無効になり、新しいコードを生成する必要があります。

1. まだ開いていない場合は**Azure Cloud Shell** を開いて、`~/certificates` ディレクトリに移動します。

1. 検証証明書を作成するには、次のコマンドを入力します。

    ```sh
    ./certGen.sh create_verification_certificate <verification-code>
    ```

    必ず、`<verification-code>` プレースホルダーを Azure portal によって生成された **確認コード**に置き換えてください。

    たとえば、実行するコマンドは次のようになります。

    ```sh
    ./certGen.sh create_verification_certificate 49C900C30C78D916C46AE9D9C124E9CFFD5FCE124696FAEA
    ```

    これにより、CA 証明書にチェーンされた _検証証明書_ が生成されます。証明書の件名は、確認コードです。生成された検証証明書 `verification-code.cert.pem` は、Azure Cloud Shell の `./certs` ディレクトリ内にあります。

    次の手順では、検証証明書をローカル コンピューターにダウンロードし (ルート証明書で以前に行った場合と同様)、DPS にアップロードできるようにします。

1. 検証証明書をローカル コンピューターにダウンロードするには、次のコマンドを入力します。

    ```sh
    download ~/certificates/certs/verification-code.cert.pem
    ```

    > **注意**: Web ブラウザーによっては、この時点で複数のダウンロードを許可するように求められることがあります。ダウンロード コマンドに応答がないように見える場合、ダウンロードを許可するアクセス許可を要求するプロンプトが画面上のどこかに表示されていないことを確認してください。

1. 「**証明書の詳細**」 ウィンドウに移動します。

    DPS 内で CA 証明書の作業をしている間、このペインを Azure portal で開いたままにしていたことを思い出してください。

1. 「**証明書の詳細**」 ペインの下部にある**検証証明書の .pem または .cer ファイル**の右側にある 「**開く**」 をクリックします。

1. 「ファイルを開く」 ダイアログで、ダウンロード フォルダーに移動して、**verification-code.cert.pem** をクリックして、「**開く**」 をクリックします。   

1. 「**証明書の詳細**」 ペインの下部にある 「**確認**」 をクリックします。   

1. 「**証明書**」 ウィンドウで、証明書の 「**状態**」 が _Verified_ と表示されていることを確認します。

    この変更を表示するには、ウィンドウの上部 (「**追加**」ボタンの右側) の 「**更新**」 ボタンを使用する必要があります。

#### タスク 3: DPS でのグループ登録 (X.509 証明書) の作成

この練習では、_証明書の構成証明_を使用するデバイス プロビジョニング サービス (DPS) 内に新しい登録グループ を作成します。 

1. 必要に応じて、Azure アカウントの認証情報を使用して Azure portal にログインします。

    複数の Azure アカウントをお持ちの場合は、このコースで使用するサブスクリプションに関連付けられているアカウントでログインしていることを確認してください。

1. リソース グループ タイルで、**dps-az220-training-{your-id}** をクリックします。

1. 「デバイス プロビジョニング サービス」 ブレードの左側にある 「**設定**」 で、「**登録の管理**」 をクリックします。   

1. ブレードの上部にある、「**登録グループの追加**」 をクリックします。

    登録グループは、基本的に自動プロビジョニングを通じて登録できるデバイスの記録であることを思い出してください。

1. 「**登録グループの追加**」 ブレードの 「**グループ名**」 に、「**simulated-devices**」と入力します

1. 「**証明タイプ**」 が 「**証明書**」に設定されていることを確認します。

1. 「**証明書の種類**」 フィールドが 「**CA 証明書**」 に設定されていることを確認します。

1. 「**プライマリ証明書**」 ドロップダウンで、以前に DPS にアップロードされた CA 証明書 (**root-ca-cert** など) を選択します。  

1. 「**セカンダリ証明書**」 ドロップダウン リストを 「**証明書なし**」 のままにします。   

    通常、セカンダリ証明書は、期限切れの証明書や侵害された証明書に対応するために、証明書のローテーションに使用されます。証明書のローリングに関する詳細については、 こちらをご覧ください。[https://docs.microsoft.com/ja-jp/azure/iot-dps/how-to-roll-certificates](https://docs.microsoft.com/ja-jp/azure/iot-dps/how-to-roll-certificates)

1. **デバイスをハブに割り当てる方法**は、「**加重が均等に分布**」 のままにします。   

   複数の分散ハブがある大規模な環境では、この設定によって、デバイス登録を受信する IoT Hub の選択方法を制御できます。このラボの登録に関連付けられた IoT ハブが 1 つあるため、IoT Hub にデバイスを割り当てる方法は、このラボ シナリオでは実際には適用されません。 

1. 「**このグループを割り当てることができる IoT ハブの選択**」 ドロップダウンで **AZ-220-HUB-*{YOUR-ID}*** IoT ハブが選択されていることに注意してください。

    これにより、デバイスがプロビジョニングされると、この IoT ハブに追加されます。

1. 「**再プロビジョニング時にデバイス データを処理する方法を選択する**」 を 「**データの再プロビジョニングと移行**」 に既定値のままにします。

    このフィールドによって、同じデバイス (同じ登録 ID で示される) が、少なくとも 1 回正常にプロビジョニングされた後に、プロビジョニング要求を後で送信する再プロビジョニング動作を高度に制御できます。

1. 「**初期デバイス ツインの状態**」 フィールドで、JSON オブジェクトを次のように変更します。

    ```json
    {
        "tags": {},
        "properties": {
            "desired": {
                "telemetryDelay": "1"
            }
        }
    }
    ```

    この JSON データは、この登録グループに参加するすべてのデバイスに必要なデバイス ツインのプロパティの初期構成を表します。

    デバイスは、`properties.desired.telemetryDelay` プロパティを使用して、テレメトリを読み取り IoT ハブに送信するための遅延時間を設定します。

1. 「**エントリの有効化**」 を 「**有効**」 に設定したままにします。

    一般的に、新しい登録エントリを有効にし、有効を維持する必要があります。

1. 「**登録の追加**」 ブレードの上部にある 「**保存**」 をクリック します。


### 演習3: X.509 証明書を使用してシミュレートされたデバイスを構成する

この演習では、ルート証明書を使用してデバイス証明書を生成し、デバイス証明書を使用して接続するシミュレートされたデバイスを構成して構成します。

#### デバイス証明書の生成

1. 必要に応じて、Azure アカウントの認証情報を使用して Azure portal にログインします。

    複数の Azure アカウントをお持ちの場合は、このコースで使用するサブスクリプションに関連付けられているアカウントでログインしていることを確認してください。

1. Azure portal の上部にあるツールバーで、「**Cloud Shell**」 をクリックします。

    Cloud Shell の左上隅で、**Bash** シェル オプションが選択されていることを確認します。

1. **Cloud Shell** 内で、次のコマンドを実行して `~/certificates` ディレクトリに移動します。

    ```sh
    cd ~/certificates
    ```

    `~/certificates` ディレクトリは、`certGen.sh` ヘルパー スクリプトがダウンロードされた場所です。以前にこれらを使用して、DPS の CA 証明書を生成しました。このヘルパー スクリプトは、CA 証明書チェーン内でデバイス証明書を生成するためにも使用されます。

1. CA 証明書チェーン内で _X.509 デバイス証明書_を生成するには、次のコマンドを入力します。

    ```sh
    ./certGen.sh create_device_certificate sensor-thl-2000
    ```

    このコマンドは、以前に生成された CA 証明書によって署名された新しい X.509 証明書を作成します。デバイス ID (`sensor-thl-2000`) が `certGen.sh` スクリプトの `create_device_certificate` コマンドに渡されることに注意してください。このデバイス ID は、 デバイス証明書の _common name_ `CN=` の値内に設定されます。この証明書は、シミュレートされたデバイスのリーフ デバイス X.509 証明書を生成し、デバイス プロビジョニング サービス (DPS) でデバイスを認証するために使用されます。

    `create_device_certificate` コマンドが完了すると、生成された X.509 デバイス証明書は `new-device.cert.pfx` という名前になり、`/certs` サブディレクトリ内に配置されます。

    > **注意**: このコマンドは、`/certs` サブディレクトリ内の既存のデバイス証明書を上書きします。複数デバイスの証明書を作成する場合は、コマンドを実行するたびに `new-device.cert.pfx` のコピーを保存してください。

1. 作成したデバイス証明書の名前を変更するには、次のコマンドを入力します。

    ```sh
    mv ~/certificates/certs/new-device.cert.pfx ~/certificates/certs/sensor-thl-2000-device.cert.pfx
    mv ~/certificates/certs/new-device.cert.pem ~/certificates/certs/sensor-thl-2000-device.cert.pem
    ```

1. 4つの追加のデバイス証明書を作成するには、次のコマンドを入力します。

    ```sh
    ./certGen.sh create_device_certificate sensor-thl-2001
    mv ~/certificates/certs/new-device.cert.pfx ~/certificates/certs/sensor-thl-2001-device.cert.pfx
    mv ~/certificates/certs/new-device.cert.pem ~/certificates/certs/sensor-thl-2001-device.cert.pem

    ./certGen.sh create_device_certificate sensor-thl-2002
    mv ~/certificates/certs/new-device.cert.pfx ~/certificates/certs/sensor-thl-2002-device.cert.pfx
    mv ~/certificates/certs/new-device.cert.pem ~/certificates/certs/sensor-thl-2002-device.cert.pem

    ./certGen.sh create_device_certificate sensor-thl-2003
    mv ~/certificates/certs/new-device.cert.pfx ~/certificates/certs/sensor-thl-2003-device.cert.pfx
    mv ~/certificates/certs/new-device.cert.pem ~/certificates/certs/sensor-thl-2003-device.cert.pem

    ./certGen.sh create_device_certificate sensor-thl-2004
    mv ~/certificates/certs/new-device.cert.pfx ~/certificates/certs/sensor-thl-2004-device.cert.pfx
    mv ~/certificates/certs/new-device.cert.pem ~/certificates/certs/sensor-thl-2004-device.cert.pem
    ```

1. 生成されたX.509デバイス証明書をCloud Shellからローカルマシンにダウンロードするには、次のコマンドを入力します。

    ```sh
    download ~/certificates/certs/sensor-thl-2000-device.cert.pfx
    download ~/certificates/certs/sensor-thl-2001-device.cert.pfx
    download ~/certificates/certs/sensor-thl-2002-device.cert.pfx
    download ~/certificates/certs/sensor-thl-2003-device.cert.pfx
    download ~/certificates/certs/sensor-thl-2004-device.cert.pfx
    ```

    次のタスクでは、X.509デバイス証明書を使用してデバイスプロビジョニングサービスで認証するシミュレートされたデバイスの構築を開始します。

####　タスク2:シミュレートされたデバイスを構成する

このタスクでは、以下を実行します。

* コードに配置されるDPSからIDスコープを取得する
* ダウンロードしたデバイス証明書をアプリケーションのルートフォルダーにコピーします
* Visual Studio Codeでアプリケーションを構成する

1. Azure portal 内で、「**デバイス プロビジョニング サービス**」 ブレードに移動し、「**概要**」 ウィンドウに移動します。

1. 「**概要**」 ウィンドウで、デバイス プロビジョニング サービスの 「**ID スコープ**」 をコピーし、後で参照するために保存します。

    値の上にマウス ポインターを合わせると表示される値の右側に 「コピー」 ボタンがあります。

    **ID スコープ**は、次の値と似ています。  `0ne0004E52G`

1. Windows ファイル エクスプローラーを開き、`sensor-thl-2000-device.cert.pfx` がダウンロードされたフォルダーに移動します。

1. ファイル エクスプローラを使用して、5つのデバイス証明書ファイルのコピーを作成します。

1. ファイル エクスプローラーで、ラボ 6 (DPS のデバイスの自動登録) のスターター フォルダーに移動します。

    _ラボ 3: 開発環境の設定_:ZIP ファイルをダウンロードしてコンテンツをローカルに抽出することで、ラボ リソースを含む GitHub リポジトリを複製しました。抽出されたフォルダー構造には、次のフォルダー パスが含まれます。
    
    * Allfiles
      * Labs
          * 06-Automatic Enrollment of Devices in DPS
            * Starter
              * ContainerDevice

1. コピーしたデバイス証明書ファイルをContainerDeviceフォルダに貼り付けます。

    ContainerDeviceフォルダーのルートディレクトリには、シミュレートされたデバイスアプリの `Program.cs`ファイルが含まれています。 シミュレートされたデバイスアプリは、デバイスプロビジョニングサービスへの認証時にデバイス証明書ファイルを使用します。


1. **Visual Studio Code** を開き、**ContainerDevice**フォルダを開きます。

    Visual Studio Code のエクスプローラー ウィンドウに次のファイルが一覧表示されます。

    * ContainerDevice.csproj
    * Program.cs
    * sensor-thl-2000-device.cert.pfx, 他4つのpfxファイル
    
    **注**:このフォルダーにコピーした他のデバイス証明書ファイルは、このラボの後半で使用されますが、ここでは、最初のデバイス証明書ファイルのみの実装に焦点を当てます。

1. `ContainerDevice.csproj` ファイルを開きます。

1. コードエディターペインの `<ItemGroup>`タグ内で、証明書ファイル名を次のように更新します:

    ```xml
    <ItemGroup>
        <None Update="sensor-thl-2000-device.cert.pfx" CopyToOutputDirectory="PreserveNewest" />
        <PackageReference Include="Microsoft.Azure.Devices.Client" Version="1.*" />
        <PackageReference Include="Microsoft.Azure.Devices.Provisioning.Transport.Mqtt" Version="1.*" />
        <PackageReference Include="Microsoft.Azure.Devices.Provisioning.Transport.Amqp" Version="1.*" />
        <PackageReference Include="Microsoft.Azure.Devices.Provisioning.Transport.Http" Version="1.*" />
    </ItemGroup>
    ```

    この構成により、C＃コードのコンパイル時に `sensor-thl-2000-device.cert.pfx`証明書ファイルがビルドフォルダーにコピーされ、プログラムの実行時にプログラムがアクセスできるようになります。

1. Visual Studio Code **ファイル**メニューの 「**保存**」 をクリックします。

1. `Program.cs` ファイルを開きます。

    ざっと一目見ると、このバージョンの**ContainerDevice**アプリケーションは、前のラボで使用したバージョンと実質的に同じであることがわかります。 唯一の変更は、証明メカニズムとしてのX.509証明書の使用に特に関連する変更です。 アプリケーションの観点からは、このデバイスがグループ登録と個別登録のどちらで接続するかはほとんど問題ではありません。

1. `GlobalDeviceEndpoint` 変数を見つけ、その値が Azure デバイス プロビジョニング サービス (`global.azure-devices-provisioning.net`) のグローバル デバイス エンドポイントに設定されていることを確認します。 

    パブリック Azure クラウド内では、`global.azure-devices-provisioning.net` は、デバイス プロビジョニング サービス (DPS) のグローバル デバイス エンドポイントです。Azure DPS に接続しているすべてのデバイスは、このグローバル デバイス エンドポイントの DNS 名で構成されます。次のようなコードが表示されます。

    ```csharp
    private const string GlobalDeviceEndpoint = "global.azure-devices-provisioning.net";
    ```

1. `dpsIdScope` 変数を見つけて、割り当てられた値をデバイス プロビジョニング サービスの 「概要」 ウィンドウからコピーした **ID スコープ**に置き換えます。

    コードを更新すると、次のようになります。

    ```csharp
    private static string dpsIdScope = "0ne000CBD6C";
    ```

1. Locate the `certificateFileName` 変数を見つけて、その値が、生成したデバイス証明書ファイル (`new-device.cert.pfx`) に設定されていることを確認します。

    以前のラボのように対称鍵を使用するのではなく、今回はアプリケーションがX.509証明書を使用しています。 **new-device.cert.pfx**ファイルは、Cloud Shell内で**certGen.sh**ヘルパースクリプトを使用して生成したX.509デバイス証明書ファイルです。 この変数は、デバイスプロビジョニングサービスで認証するときに使用するX.509デバイス証明書を含むファイルをデバイスコードに通知します。

1. **certificateFileName**変数に割り当てられた値を次のように更新します。

    ```csharp
    private static string certificateFileName = "sensor-thl-2000-device.cert.pfx";
    ```
    
1. `certificatePassword` 変数を見つけ、その値が `certGen.sh` スクリプトの既定のパスワードに設定されていることを確認します。

    `certificatePassword` 変数には、X.509 デバイス証明書のパスワードが含まれています。これは X.509 証明書を生成するときに `certGen.sh` ヘルパー スクリプトで使用される既定のパスワードであるため、`1234` に設定されています。

    >**注意**: このラボの目的のために、パスワードはハード コーディングされています。_運用_シナリオでは、Azure Key Vault などに、より安全な方法でパスワードを保存する必要があります。さらに、証明書ファイル (PFX) は、ハードウェア セキュリティ モジュール (HSM) を使用して、運用デバイスに安全に安全に格納する必要があります。
    >
    > HSM (ハードウェア セキュリティ モジュール) は、デバイス シークレットの安全なハードウェア ベースのストレージに使用され、最も安全なフォームのシークレット ストレージです。X.509 証明書と SAS トークンの両方を HSM に格納できます。HSM は、プロビジョニング サービスがサポートするすべての構成証明メカニズムで使用できます。

#### タスク 3: プロビジョニングコードの追加

このタスクでは、Mainメソッド、デバイスプロビジョニング、デバイスツインプロパティに関連する実装を完了するコードを入力します。

1. Program.csファイルのコードエディターペインで、 `// INSERT Main method below here`コメントを見つけます。

1. Mainメソッドを実装するには、次のコードを入力します。

    ```csharp
    public static async Task Main(string[] args)
    {
        X509Certificate2 certificate = LoadProvisioningCertificate();

        using (var security = new SecurityProviderX509Certificate(certificate))
        using (var transport = new ProvisioningTransportHandlerAmqp(TransportFallbackType.TcpOnly))
        {
            ProvisioningDeviceClient provClient =
                ProvisioningDeviceClient.Create(GlobalDeviceEndpoint, dpsIdScope, security, transport);

            using (deviceClient = await ProvisionDevice(provClient, security))
            {
                await deviceClient.OpenAsync().ConfigureAwait(false);

                // INSERT Setup OnDesiredPropertyChanged Event Handling below here

                // INSERT Load Device Twin Properties below here

                // Start reading and sending device telemetry
                Console.WriteLine("Start reading and sending device telemetry...");
                await SendDeviceToCloudMessagesAsync();

                await deviceClient.CloseAsync().ConfigureAwait(false);
            }
        }
    }
    ```

このMainメソッドは、以前のラボで使用したものと非常に似ています。 2つの重要な変更は、X.509証明書をロードする必要があることと、セキュリティプロバイダーとして**SecurityProviderX509Certificate**を使用することへの変更です。 残りのコードは同じです。デバイスツインプロパティ変更コードも存在することに注意してください。

1. `// INSERT LoadProvisioningCertificate method below here`コメントを見つけます。

1. LoadProvisioningCertificateメソッドを実装するには、次のコードを挿入します。

    ```csharp
    private static X509Certificate2 LoadProvisioningCertificate()
    {
        var certificateCollection = new X509Certificate2Collection();
        certificateCollection.Import(certificateFileName, certificatePassword, X509KeyStorageFlags.UserKeySet);

        X509Certificate2 certificate = null;

        foreach (X509Certificate2 element in certificateCollection)
        {
            Console.WriteLine($"Found certificate: {element?.Thumbprint} {element?.Subject}; PrivateKey: {element?.HasPrivateKey}");
            if (certificate == null && element.HasPrivateKey)
            {
                certificate = element;
            }
            else
            {
                element.Dispose();
            }
        }

        if (certificate == null)
        {
            throw new FileNotFoundException($"{certificateFileName} did not contain any certificate with a private key.");
        }

        Console.WriteLine($"Using certificate {certificate.Thumbprint} {certificate.Subject}");
        return certificate;
    }
    ```
    
    名前から想像できるように、このメソッドの目的はX.509証明書をディスクからロードすることです。ロードが成功した場合、メソッドは**X509Certificate2**クラスのインスタンスを返します。

    >**情報**:結果が**X509Certificate**ではなく**X509Certificate2**タイプである理由を知りたいと思うかもしれません。**X509Certificate**は以前の実装であり、機能が制限されています。**X509Certificate2**は**X509Certificate**のサブクラスであり、X509標準のV2とV3の両方をサポートする追加機能を備えています。

    このメソッドは**X509Certificate2Collection**クラスのインスタンスを作成し、ハードコーディングされたパスワードを使用して、証明書ファイルをディスクからインポートしようとします。**X509KeyStorageFlags.UserKeySet**値は、秘密鍵がローカルコンピューターストアではなく現在のユーザーストアに格納されることを指定します。これは、証明書でキーがローカルコンピュータストアに格納されるように指定されている場合でも発生します。

    次に、メソッドはインポートされた証明書（この場合は1つだけである必要があります）を反復処理し、証明書に秘密鍵があることを確認します。インポートされた証明書がこの基準に一致しない場合、例外がスローされます。それ以外の場合、メソッドはインポートされた証明書を返します。
    
    
1. `// INSERT ProvisionDevice method below here`コメントを見つけます。

1. ProvisionDeviceメソッドを実装するには、次のコードを入力します。
    
    ```csharp
    private static async Task<DeviceClient> ProvisionDevice(ProvisioningDeviceClient provisioningDeviceClient, SecurityProviderX509Certificate security)
    {
        var result = await provisioningDeviceClient.RegisterAsync().ConfigureAwait(false);
        Console.WriteLine($"ProvisioningClient AssignedHub: {result.AssignedHub}; DeviceID: {result.DeviceId}");
        if (result.Status != ProvisioningRegistrationStatusType.Assigned)
        {
            throw new Exception($"DeviceRegistrationResult.Status is NOT 'Assigned'");
        }

        var auth = new DeviceAuthenticationWithX509Certificate(
            result.DeviceId,
            security.GetAuthenticationCertificate());

        return DeviceClient.Create(result.AssignedHub, auth, TransportType.Amqp);
    }
    ```
    
このバージョンの**ProvisionDevice**は、以前のラボで使用したものとよく似ています。 主な変更は、**セキュリティ**パラメータのタイプが**SecurityProviderX509Certificate**になっていることです。 これは、**DeviceClient**の作成に使用される**auth**変数が**DeviceAuthenticationWithX509Certificate**タイプである必要があり、 `security.GetAuthenticationCertificate（）`値を使用することを意味します。 実際のデバイス登録は以前と同じです。

#### タスク 4: デバイスツインインテグレーションコードの追加

デバイスで（Azure IoT Hubからの）デバイスツインプロパティを使用するには、デバイスツインプロパティにアクセスして適用するコードを作成する必要があります。この場合、シミュレートされたデバイスコードを更新してデバイスツインのDesiredプロパティを読み取り、その値を**telemetryDelay**変数に割り当てます。また、デバイスツインレポートプロパティを更新して、現在デバイスに実装されている遅延値を示します。

1. Visual Studio Codeエディターで、**Main**メソッドを見つけます。

1. コードを確認し、 `// INSERT Setup OnDesiredPropertyChanged Event Handling below here`コメントを見つけます。

    デバイスツインプロパティの統合を開始するには、デバイスツインプロパティが更新されたときにシミュレートされたデバイスに通知できるようにするコードが必要です。

    これを実現するには、 **DeviceClient.SetDesiredPropertyUpdateCallbackAsync**メソッドを使用し、**OnDesiredPropertyChanged** メソッドを作成してイベントハンドラーを設定します。

1. OnDesiredPropertyChangedイベントのDeviceClientを設定するには、次のコードを入力します。

    ```csharp
    await deviceClient.SetDesiredPropertyUpdateCallbackAsync(OnDesiredPropertyChanged, null).ConfigureAwait(false);
    ```

**SetDesiredPropertyUpdateCallbackAsync**メソッドを使用して、**DesiredPropertyUpdateCallback**イベントハンドラーをセットアップし、デバイスツインの必要なプロパティの変更を受信します。 このコードは、デバイスツインのプロパティ変更イベントを受信したときに**OnDesiredPropertyChanged**という名前のメソッドを呼び出すように**deviceClient**を構成します。

     これで、**SetDesiredPropertyUpdateCallbackAsync**メソッドを使用してイベントハンドラーを設定できたので、それが呼び出す**OnDesiredPropertyChanged**メソッドを作成する必要があります。

1. `// INSERT OnDesiredPropertyChanged method below here`コメントを見つけます。

1. イベントハンドラーのセットアップを完了するには、次のコードを入力します。

    ```csharp
    private static async Task OnDesiredPropertyChanged(TwinCollection desiredProperties, object userContext)
    {
        Console.WriteLine("Desired Twin Property Changed:");
        Console.WriteLine($"{desiredProperties.ToJson()}");

        // Read the desired Twin Properties
        if (desiredProperties.Contains("telemetryDelay"))
        {
            string desiredTelemetryDelay = desiredProperties["telemetryDelay"];
            if (desiredTelemetryDelay != null)
            {
                telemetryDelay = int.Parse(desiredTelemetryDelay);
            }
            // if desired telemetryDelay is null or unspecified, don't change it
        }

        // Report Twin Properties
        var reportedProperties = new TwinCollection();
        reportedProperties["telemetryDelay"] = telemetryDelay.ToString();
        await deviceClient.UpdateReportedPropertiesAsync(reportedProperties).ConfigureAwait(false);
        Console.WriteLine("Reported Twin Properties:");
        Console.WriteLine($"{reportedProperties.ToJson()}");
    }
    ```

**OnDesiredPropertyChanged**イベントハンドラーは、**TwinCollection**タイプの**desiredProperties**パラメーターを受け入れることに注意してください。

   **desiredProperties**パラメーターの値に**telemetryDelay**（デバイスツインの目的のプロパティ）が含まれている場合、コードはデバイスツインプロパティの値を**telemetryDelay**変数に割り当てます。**SendDeviceToCloudMessagesAsync**メソッドには、**telemetryDelay**変数を使用してIoTハブに送信されるメッセージ間の遅延時間を設定する**Task.Delay**呼び出しが含まれていることを思い出してください。

    次のコードブロックを使用して、デバイスの現在の状態をAzure IoT Hubに報告します。このコードは、**DeviceClient.UpdateReportedPropertiesAsync**メソッドを呼び出し、デバイスプロパティの現在の状態を含む**TwinCollection**を渡します。これは、デバイスがデバイスツインの必要なプロパティ変更イベントを受信したことをIoT Hubに報告し、それに応じて構成を更新した方法です。これは、目的のプロパティのエコーではなく、プロパティの現在の設定を報告することに注意してください。デバイスから送信されたレポートされたプロパティがデバイスが受信した望ましい状態と異なる場合、IoT Hubはデバイスの状態を反映する正確なデバイスツインを維持します。

    デバイスがAzure IoT Hubからデバイスツインの必要なプロパティへの更新を受信できるようになったので、デバイスの起動時に初期セットアップを構成するようにコード化する必要もあります。これを行うには、デバイスは、Azure IoT Hubから現在のデバイスツインの必要なプロパティを読み込み、それに従って構成する必要があります。

1. **Main**メソッドで、 `// INSERT Load Device Twin Properties below here`コメントを見つけます。

1. デバイスツインの必要なプロパティを読み取り、デバイスの起動時に一致するようにデバイスを構成するには、次のコードを入力します。

    ```csharp
    var twin = await deviceClient.GetTwinAsync().ConfigureAwait(false);
    await OnDesiredPropertyChanged(twin.Properties.Desired, null);
    ```

このコードは**DeviceTwin.GetTwinAsync**メソッドを呼び出して、シミュレートされたデバイスのデバイスツインを取得します。次に、**Properties.Desired**プロパティオブジェクトにアクセスしてデバイスの現在のDesired Stateを取得し、それを**OnDesiredPropertyChanged**メソッドに渡して、シミュレートされたデバイスの**telemetryDelay**変数を構成します。

    このコードは、_OnDesiredPropertyChanged_イベントを処理するためにすでに作成されている**OnDesiredPropertyChanged**メソッドを再利用していることに注意してください。これにより、デバイスツインの望ましい状態プロパティを読み取り、起動時にデバイスを1か所で構成するコードを維持できます。結果のコードは、保守が簡単で簡単です。

1. Visual Studio Code**[ファイル]**メニューで、[**保存**]をクリックします。

    シミュレートされたデバイスは、Azure IoT Hubのデバイスツインプロパティを使用して、テレメトリメッセージ間の遅延を設定します。

1. [**ターミナル**]メニューで、[**新しいターミナル**]をクリックします。

1. ターミナルコマンドプロンプトで、コードが正しくビルドされることを確認するには、「**dotnet build**」と入力します

    リストされたビルドエラーが表示された場合は、すぐに修正してから次の演習に進んでください。必要に応じて、インストラクターと協力してください。

### 演習 4: シミュレートされたデバイスの追加インスタンスの作成

この演習では、シミュレートしたデバイスプロジェクトのコピーを作成し、コードを更新して、作成してプロジェクトフォルダーに追加したさまざまなデバイス証明書を使用します。

#### タスク1:コードプロジェクトのコピーをモークする

1. Windowsファイルエクスプローラーを開きます。

1. ファイルエクスプローラーで、ラボ6（DPSでのデバイスの自動登録）のスターターフォルダーに移動します。

     _ラボ3:開発環境のセットアップ_では、ZIPファイルをダウンロードしてローカルでコンテンツを抽出することにより、ラボリソースを含むGitHubリポジトリのクローンを作成しました。 抽出されたフォルダー構造には、次のフォルダーパスが含まれます。

    * Allfiles
      * Labs
          * 06-Automatic Enrollment of Devices in DPS
            * Starter

1. [**ContainerDevice**]を右クリックして、[**Copy**]をクリックします。

     ContainerDeviceフォルダーは、シミュレートされたデバイスコードを含むフォルダーでなければなりません。

1. **ContainerDevice**の下の空のスペースを右クリックして、[**貼り付け**]をクリックします

     「ContainerDevice-Copy」という名前のフォルダーが作成されているはずです。

1. [**ContainerDevice-Copy**]を右クリックし、[**Rename**]をクリックして、「**ContainerDevice2001**」と入力します

1. 手順3〜5を繰り返して、次の名前のフォルダーを作成します。

     * **ContainerDevice2002**
     * **ContainerDevice2003**
     * **ContainerDevice2004**


#### タスク2:コードプロジェクトの証明書ファイル参照を更新する

1. 必要に応じて、Visual Studio Codeを開きます。

1. [**ファイル**]メニューの[**フォルダを開く**]をクリックします。

1. ラボ6のStarterフォルダーに移動します。

1. [**ContainerDevice2001**]をクリックし、[**Select Folder**]をクリックします。

1. EXPLORERペインで、**Program.cs**をクリックします。

1. コードエディターで**certificateFileName**変数を見つけ、**certificateFileName**変数に割り当てられた値を次のように更新します。

    ```csharp
    private static string certificateFileName = "sensor-thl-2001-device.cert.pfx";
    ```

1. **EXPLORER**ペインで、ContainerDevice.csprojファイルを開くには、**ContainerDevice.csproj**をクリックします。

1. コードエディターペインの `<ItemGroup>`タグ内で、証明書ファイル名を次のように更新します:

    ```xml
    <ItemGroup>
        <None Update="sensor-thl-2001-device.cert.pfx" CopyToOutputDirectory="PreserveNewest" />
        <PackageReference Include="Microsoft.Azure.Devices.Client" Version="1.*" />
        <PackageReference Include="Microsoft.Azure.Devices.Provisioning.Transport.Mqtt" Version="1.*" />
        <PackageReference Include="Microsoft.Azure.Devices.Provisioning.Transport.Amqp" Version="1.*" />
        <PackageReference Include="Microsoft.Azure.Devices.Provisioning.Transport.Http" Version="1.*" />
    </ItemGroup>
    ```
    
1. [**ファイル**]メニューで[**すべて保存**]をクリックします。

1. 上記の手順3〜9を繰り返して、残りの各コードプロジェクトの**Program.cs**ファイルと**ContainerDevice.csproj**ファイルを次のように更新します。

    | Project Folder | Certificate Name |
    |----------------|------------------------------|
    | ContainerDevice2002 | sensor-thl-2002-device.cert.pfx |
    | ContainerDevice2003 | sensor-thl-2003-device.cert.pfx |
    | ContainerDevice2004 | sensor-thl-2004-device.cert.pfx |

    **注**:次のフォルダに進む前に、必ず**すべて保存**してください。

### 演習 5: シミュレートされたデバイスをテストする

この演習では、シミュレートされたデバイスを実行します。デバイスを初めて起動すると、デバイスはデバイスプロビジョニングサービス（DPS）に接続し、構成されたグループ登録を使用して自動的に登録されます。 DPSグループ登録に登録されると、デバイスはAzure IoT Hubデバイスレジストリ内に自動的に登録されます。登録および登録されると、デバイスは構成されたX.509証明書認証を使用してAzure IoT Hubと安全に通信を開始します。

#### タスク 1: シミュレートされたデバイスプロジェクトをビルドして実行する

1. Visual Studio Codeが開いていることを確認します。

1. [**ファイル**]メニューの[**フォルダを開く**]をクリックします。

1. ラボ6のStarterフォルダーに移動します。

1. [**ContainerDevice**]をクリックし、[**Select Folder**]をクリックします。

1. [**表示**]メニューで[**ターミナル**]をクリックします。

    これにより、Visual Studioコードウィンドウの下部に統合されたターミナルが開きます。

1. ターミナルのコマンドプロンプトで、現在のディレクトリパスが `\ContainerDevice`フォルダーに設定されていることを確認します。

    次のようなものが表示されます。
    
    
    `Allfiles\Labs\06-Automatic Enrollment of Devices in DPS\Starter\ContainerDevice>`

1. **ContainerDevice**プロジェクトをビルドして実行するには、次のコマンドを入力します。

    ```cmd/sh
    dotnet run
    ```
    
    > **注**:シミュレートされたデバイスを初めて実行するとき、最も一般的なエラーは_Invalid certificate_エラーです。 `ProvisioningTransportException`例外が表示される場合、これはおそらくこのエラーが原因です。 以下に示すようなメッセージが表示された場合は、続行する前に、DPSのCA証明書と、シミュレートされたデバイスアプリケーションのデバイス証明書が正しく構成されていることを確認する必要があります。
    >
    > ```text
    > localmachine:LabFiles User$ dotnet run
    > Found certificate: AFF851ED016CA5AEB71E5749BCBE3415F8CF4F37 CN=sensor-thl-2000; PrivateKey: True
    > Using certificate AFF851ED016CA5AEB71E5749BCBE3415F8CF4F37 CN=sensor-thl-2000
    > RegistrationID = sensor-thl-2000
    > ProvisioningClient RegisterAsync . . . Unhandled exception. Microsoft.Azure.Devices.Provisioning.Client.ProvisioningTransportException: {"errorCode":401002,"trackingId":"2e298c80-0974-493c-9fd9-6253fb055ade","message":"Invalid certificate.","timestampUtc":"2019-12-13T14:55:40.2764134Z"}
    >   at Microsoft.Azure.Devices.Provisioning.Client.Transport.ProvisioningTransportHandlerAmqp.ValidateOutcome(Outcome outcome)
    >   at Microsoft.Azure.Devices.Provisioning.Client.Transport.ProvisioningTransportHandlerAmqp.RegisterDeviceAsync(AmqpClientConnection client, String correlationId, DeviceRegistration deviceRegistration)
    >   at Microsoft.Azure.Devices.Provisioning.Client.Transport.ProvisioningTransportHandlerAmqp.RegisterAsync(ProvisioningTransportRegisterMessage message, CancellationToken cancellationToken)
    >   at X509CertificateContainerDevice.ProvisioningDeviceLogic.RunAsync() in /Users/User/Documents/AZ-220/LabFiles/Program.cs:line 121
    >   at X509CertificateContainerDevice.Program.Main(String[] args) in /Users/User/Documents/AZ-220/LabFiles/Program.cs:line 55
    > ...
    > ```

1. シミュレートされたデバイスアプリがターミナルウィンドウに出力を送信することに注意してください。

     シミュレートされたデバイスアプリケーションが正しく実行されている場合、**ターミナル**はアプリからのコンソール出力を表示します。

     ターミナルウィンドウに表示されている情報の一番上までスクロールします。

     X.509証明書が読み込まれ、デバイスがデバイスプロビジョニングサービスに登録され、**iot-az220-training- {your-id} **IoT Hubに接続するように割り当てられ、デバイスツインの必要なプロパティに注意してください が読み込まれます。
     
    ```text
    localmachine:LabFiles User$ dotnet run
    Found certificate: AFF851ED016CA5AEB71E5749BCBE3415F8CF4F37 CN=sensor-thl-2000; PrivateKey: True
    Using certificate AFF851ED016CA5AEB71E5749BCBE3415F8CF4F37 CN=sensor-thl-2000
    RegistrationID = sensor-thl-2000
    ProvisioningClient RegisterAsync . . . Device Registration Status: Assigned
    ProvisioningClient AssignedHub: iot-az220-training-CP1119.azure-devices.net; DeviceID: sensor-thl-2000
    Creating X509 DeviceClient authentication.
    simulated device. Ctrl-C to exit.
    DeviceClient OpenAsync.
    Connecting SetDesiredPropertyUpdateCallbackAsync event handler...
    Loading Device Twin Properties...
    Desired Twin Property Changed:
    {"$version":1}
    Reported Twin Properties:
    {"telemetryDelay":1}
    Start reading and sending device telemetry...
    ```
    
    シミュレートされたデバイスのソースコードを確認するには、 `Program.cs`ソースコードファイルを開きます。 コンソールに表示されるメッセージを出力するために使用されるいくつかの `Console.WriteLine`ステートメントを探します。

1. JSON形式のテレメトリメッセージがAzure IoT Hubに送信されていることに注意してください。

    ```text
    Start reading and sending device telemetry...
    12/9/2019 5:47:00 PM > Sending message: {"temperature":24.047539159212047,"humidity":67.00504162675004,"pressure":1018.8478924248358,"latitude":40.129349260196875,"longitude":-98.42877188146265}
    12/9/2019 5:47:01 PM > Sending message: {"temperature":26.628804161040485,"humidity":68.09610794675355,"pressure":1014.6454375411363,"latitude":40.093269544242695,"longitude":-98.22227128174003}
    ```

    シミュレートされたデバイスが最初の起動を通過すると、シミュレートされたセンサーテレメトリメッセージのAzure IoT Hubへの送信が開始されます。

     IoT Hubに送信される各メッセージ間の「telemetryDelay」デバイスツインプロパティで定義された遅延は、センサーテレメトリメッセージの送信間で**1秒**遅延していることに注意してください。

1. シミュレートされたデバイスを実行したままにします。

#### タスク 2: 他のシミュレートされたデバイスを起動する

1. Visual Studio Codeの新しいインスタンスを開きます。

    これは、Windows 10の[スタート]メニューから次のように実行できます。Windows10の[**スタート**]メニューで、[**Visual Studio Code**]を右クリックし、[**新しいウィンドウ**]をクリックします。

1. 新しいVisual Studioコードウィンドウの[**ファイル**]メニューで、[**フォルダーを開く**]をクリックします。

1. ラボ6のStarterフォルダーに移動します。

1. [**ContainerDevice2001**]をクリックし、[**Select Folder**]をクリックします。

1. [**表示**]メニューで[**ターミナル**]をクリックします。

    これにより、Visual Studioコードウィンドウの下部に統合されたターミナルが開きます。

1. ターミナルのコマンドプロンプトで、現在のディレクトリパスが `\ContainerDevice2001`フォルダーに設定されていることを確認します。

    次のようなものが表示されます。

    `Allfiles\Labs\06-Automatic Enrollment of Devices in DPS\Starter\ContainerDevice2001>`

1. **ContainerDevice**プロジェクトをビルドして実行するには、次のコマンドを入力します。

    ```cmd/sh
    dotnet run
    ```

1. 上記の手順1〜7を繰り返して、次のように他のシミュレートされたデバイスプロジェクトを開いて開始します。

    |プロジェクトフォルダ|
    | ---------------- |
    | ContainerDevice2002 |
    | ContainerDevice2003 |
    | ContainerDevice2004 |

#### タスク 3: ツインを介してデバイス構成を変更する

シミュレートされたデバイスが実行されている状態で、Azure IoT Hub内のデバイスツインのDesired Stateを編集することにより、 `telemetryDelay`構成を更新できます。

1. **Azureポータル**を開き、**Azure IoT Hub**サービスに移動します。

1. IoTハブブレードの左側のメニューで、[**エクスプローラー**]の下にある[**IoTデバイス**]をクリックします。

1. IoTデバイスのリスト内で、[**sensor-thl-2000**]をクリックします。

    > **重要**:このラボからデバイスを選択してください。コースの前半で作成された_sensor-th-0001_という名前のデバイスも表示される場合があります。

1. **sensor-thl-2000**デバイスブレードの上部にある[**デバイスツイン**]をクリックします。

    **デバイスツイン**ブレードには、デバイスツインの完全なJSONを備えたエディターがあります。これにより、Azureポータル内で直接デバイスツインの状態を表示または編集できます。

1. Device Twin JSON内で `properties.desired`ノードを見つけます。

1. `"2"`の値を持つように `telemetryDelay`プロパティを更新します。

    これにより、シミュレートされたデバイスの「telemetryDelay」が**2秒**ごとにセンサーテレメトリを送信するように更新されます。

    デバイスツインの必要なプロパティのこのセクションの結果のJSONは、次のようになります。

    ```json
    "properties": {
        "desired": {
          "telemetryDelay": "2",
          "$metadata": {
            "$lastUpdated": "2019-12-09T22:48:05.9703541Z",
            "$lastUpdatedVersion": 2,
            "telemetryDelay": {
              "$lastUpdated": "2019-12-09T22:48:05.9703541Z",
              "$lastUpdatedVersion": 2
            }
          },
          "$version": 2
        },
    ```

    JSON内の `properties.desired`ノードの` $metadata`と `$version`の値をそのままにします。新しいデバイスツインの必要なプロパティ値を設定するには、 `telemetryDelay`値のみを更新する必要があります。

1. ブレードの上部で、デバイスツインに必要なプロパティをデバイスに適用するには、[**保存**]をクリックします。

    保存すると、更新されたデバイスツインの必要なプロパティが自動的にシミュレートされたデバイスに送信されます。

1. 元の**ContainerDevice**プロジェクトを含む**Visual Studio Code**ウィンドウに切り替えます。

1. 更新されたデバイスツインの「telemetryDelay」の必要なプロパティ設定がアプリケーションに通知されていることに注意してください。

    アプリケーションは、新しいデバイスツインの必要なプロパティが読み込まれ、変更が設定されてAzure IoT Hubに報告されたことを示すメッセージをコンソールに出力します。

    ```text
    Desired Twin Property Changed:
    {"telemetryDelay":2,"$version":2}
    Reported Twin Properties:
    {"telemetryDelay":2}
    ```

1. シミュレートされたデバイスセンサーのテレメトリメッセージが_2_秒ごとにAzure IoT Hubに送信されていることに注意してください。

    ```text
    12/9/2019 5:48:07 PM > Sending message: {"temperature":33.89822140284731,"humidity":78.34939097908763,"pressure":1024.9467544610131,"latitude":40.020042418755764,"longitude":-98.41923808825841}
    12/9/2019 5:48:09 PM > Sending message: {"temperature":27.475786026323114,"humidity":64.4175510594703,"pressure":1020.6866468579678,"latitude":40.2089999240047,"longitude":-98.26223221770334}
    12/9/2019 5:48:11 PM > Sending message: {"temperature":34.63600901637041,"humidity":60.95207713588703,"pressure":1013.6262313688063,"latitude":40.25499096898331,"longitude":-98.51199886959347}
    ```

1. **ターミナル**ペイン内で、シミュレートされたデバイスアプリを終了するには、**Ctrl-C**を押します。

1. 各Visual Studio Codeウィンドウに切り替え、**ターミナル**プロンプトを使用して、シミュレートされたデバイスアプリを閉じます。

1. Azureポータルウィンドウを切り替えます。

1. **デバイスツイン**ブレードを閉じます。

1. **sensor-thl-2000**ブレードで、再度[**デバイスツイン**]をクリックします。

1. 下にスクロールして、 `properties.reported`オブジェクトのJSONを見つけます。

    これには、デバイスによって報告された状態が含まれます。

1. `telemetryDelay`プロパティもここに存在し、`2`にも設定されていることに注意してください。

    `reported`値が最後に更新された日時を示す` $ metadata`値もあります。

1. **デバイスツイン**ブレードを再度閉じます。

1. **sensor-thl-2000**ブレードを閉じて、Azureポータルダッシュボードに戻ります。


### 演習 6: グループ登録から1台のデバイスのプロビジョニングを解除する

グループ登録の一部として登録されているデバイスの一部のみをプロビジョニング解除する必要がある理由は多数あります。たとえば、デバイスが不要になった場合、デバイスの新しいバージョンが利用可能になった場合、デバイスが破損または侵害された場合などです。

登録グループから1つのデバイスをプロビジョニング解除するには、次の2つのことを行う必要があります。

*デバイスのリーフ（デバイス）証明書の無効な個別登録を作成します。

    これにより、そのデバイスのプロビジョニングサービスへのアクセスが取り消され、チェーンに登録グループの署名証明書を持つ他のデバイスへのアクセスが許可されます。デバイスの無効化された個別の登録を削除しないでください。削除すると、デバイスが登録グループを介して再登録できるようになります。

* IoTハブのIDレジストリからデバイスを無効にするか削除します。

    ソリューションに複数のIoTハブが含まれている場合は、登録グループのプロビジョニングされたデバイスのリストを使用して、デバイスがプロビジョニングされたIoTハブを見つける必要があります（デバイスを無効化または削除できるようにするため）。この場合、単一のIoTハブがあるため、どのIoTハブが使用されたかを調べる必要はありません。

この演習では、1つのデバイスを登録グループからプロビジョニング解除します。

#### タスク 1: デバイスの無効な個別登録を作成します。

このタスクでは、**sensor-thl-2004**デバイスを個別の登録に使用します。

1. 必要に応じて、Azureアカウントの資格情報を使用してAzureポータルにログインします。

    複数のAzureアカウントを持っている場合は、このコースで使用するサブスクリプションに関連付けられているアカウントでログインしていることを確認してください。

1. Azureポータルツールバーで、[**Cloud Shell**]をクリックします

    Azureポータルツールバーは、ポータルウィンドウの上部に表示されます。 Cloud Shellボタンは右から6番目です。

1. Cloud Shellが**Bash**を使用していることを確認します。

    Azure Cloud Shellページの左上隅にあるドロップダウンを使用して、環境を選択します。選択したドロップダウン値が**Bash**であることを確認します。

1. Cloud Shellコマンドプロンプトで、「〜/ certificates」ディレクトリに移動するには、次のコマンドを入力します。

    ```sh
    cd ~/certificates
    ```

1. .pemデバイス証明書をCloud Shellからローカルマシンにダウンロードするには、次のコマンドを入力します。

    ```sh
    download ~/certificates/certs/sensor-thl-2004-device.cert.pem
    ```

1. Azureダッシュボードに切り替えます。

1. リソースタイルで、[**dps-az220-training-{your-id}**]をクリックします。

1. DPSブレードの左側のメニューの[**設定**]で、[**登録の管理**]をクリックします。

1. [**登録の管理**]ペインで、[**個別登録の追加**]をクリックします。

1. [**登録**]ブレードの[**メカニズム**]で、[**X.509**]が選択されていることを確認します。

1. [**プライマリ証明書.pemまたは.cerファイル**]で、[**開く**]をクリックします。

1. [**開く**]ダイアログで、ダウンロードフォルダーに移動します。

1. ダウンロードフォルダーで**sensor-thl-2004-device.cert.pem**をクリックし、[**開く**]をクリックします。

1. [**登録の追加**]ブレードの[**IoT Hub のデバイス ID**]で、「**sensor-thl-2004**」と入力します

1. [**エントリを有効にする**]で[**無効にする**]をクリックします。

1. ブレードの上部にある[**保存**]をクリックします。

1. Azureダッシュボードに戻ります。

#### タスク 2: IoTハブからデバイスを登録解除する

1. [リソース]タイルで、[**iot-az220-training-{your-id}**]をクリックします。

1. IoTハブブレードの左側のmnuで、[**エクスプローラー**]の下にある[**IoTデバイス**]をクリックします。

1. [**IoTデバイス**]ペインの[**デバイスID**]で、**sensor-thl-2004**デバイスを見つけます。

1. **sensor-thl-2004**のロフトにチェックボックスをクリックします。

1. [**IoTデバイス**]ペインの上部にある[**削除**]をクリックし、[**はい**]をクリックします。

#### タスク 3: デバイスがプロビジョニング解除されていることを確認する

1. ContainerDevice2004コードプロジェクトを含むVisual Studioコードウィンドウに切り替えます。

    前の演習の後でVisual Studio Codeを閉じた場合は、Visual Studio Codeを使用してContainerDevice2004フォルダーを開きます。

1. [**表示**]メニューで[**ターミナル**]をクリックします。

1. コマンドプロンプトが**ContainerDevice2004**フォルダーの場所にあることを確認します。

1. シミュレートされたデバイスアプリの実行を開始するには、次のコマンドを入力します。

    ```cmd/sh
    dotnet run
    ```

1. デバイスがプロビジョニングを試みたときにリストされる例外に注意してください。

    デバイスがデバイスプロビジョニングサービスに接続して認証しようとすると、サービスはまずデバイスの資格情報と一致する個々の登録を探します。次に、サービスは登録グループを検索して、デバイスをプロビジョニングできるかどうかを判断します。サービスがデバイスの無効な個別登録を検出した場合、デバイスは接続できなくなります。デバイスの証明書チェーンに中間またはルートCAの有効な登録グループが存在する場合でも、サービスは接続を防止します。

    アプリケーションが構成されたX.509証明書を使用してDPSに接続しようとすると、DPSはDeviceRegistrationResult.Statusが「割り当てられていません」と報告します。


    ```txt
    Found certificate: 13F32448E03F451E897B681758BAC593A60BFBFA CN=sensor-thl-2004; PrivateKey: True
    Using certificate 13F32448E03F451E897B681758BAC593A60BFBFA CN=sensor-thl-2004
    ProvisioningClient AssignedHub: ; DeviceID:
    Unhandled exception. System.Exception: DeviceRegistrationResult.Status is NOT 'Assigned'
    at ContainerDevice.Program.ProvisionDevice(ProvisioningDeviceClient provisioningDeviceClient, SecurityProviderX509Certificate security) in C:\Users\howdc\Allfiles\Labs\06-Automatic Enrollment of Devices 
    in DPS\Starter\ContainerDevice2004\Program.cs:line 107
    at ContainerDevice.Program.Main(String[] args) in C:\Users\howdc\Allfiles\Labs\06-Automatic Enrollment of Devices in DPS\Starter\ContainerDevice2004\Program.cs:line 49
    at ContainerDevice.Program.<Main>(String[] args)
    ```

    Azureポータルに戻り、個別の登録を有効にするか、個別の登録を削除すると、デバイスは再びDPSで認証され、IoTハブに接続できるようになります。個別の登録が削除されると、デバイスは自動的にグループ登録に追加されます。
    
 ### 演習 7: グループ登録のプロビジョニングを解除する

この演習では、登録グループ全体をプロビジョニング解除します。この場合も、デバイスプロビジョニングサービスからの登録解除とIoT Hubからのデバイスの登録解除が含まれます。

#### タスク 1: 登録グループをDPSから登録解除する

このタスクでは、登録グループを削除します。これにより、登録済みデバイスが削除されます。

1. Azureダッシュボードに移動します。

1. リソースタイルで、[**dps-az220-training-{your-id}**]をクリックします。

1. DPSブレードの左側のメニューの[**設定**]で、[**登録の管理**]をクリックします。

1. **登録グループ**のリストの[**グループ名**]で、[**eg-test-simulated-devices**]をクリックします。

1. [**登録グループの詳細**]ブレードで、下にスクロールして[**エントリの有効化**]フィールドを見つけ、[**無効**]をクリックします。

    DPS内のグループ登録を無効にすると、この登録グループ内のデバイスを一時的に無効にすることができます。これにより、これらのデバイスで使用されるX.509証明書の一時的なブラックリストが提供されます。

1. ブレードの上部にある[**保存**]をクリックします。

    ここでシミュレートされたデバイスの1つを実行すると、個別の登録を無効にした場合と同様のエラーメッセージが表示されます。

    登録グループを完全に削除するには、DPSから登録グループを削除する必要があります。

1. [**登録の管理**]ペインの[**グループ名**]で、[**eg-test-simulated-devices**]の左側にあるチェックボックスをオンにします。

    **simulated-devices**の左側にあるチェックボックスがすでにオンになっている場合は、オンのままにします。

1. [**登録の管理**]ペインの上部にある[**削除**]をクリックします。

1. **登録を削除する**ためのアクションを確認するプロンプトが表示されたら、[**はい**]をクリックします。

    削除すると、グループ登録はDPSから完全に削除されるため、再度追加するには、再作成する必要があります。

    > **注**:証明書の登録グループを削除しても、ルート証明書または上位の別の中間証明書に対して別の有効な登録グループがまだ存在する場合、証明書チェーンに証明書を持つデバイスは引き続き登録できる可能性があります証明書チェーンに含まれています。

1. Azureポータルダッシュボードに戻ります

#### タスク 2: IoT Hubからデバイスを登録解除する

登録グループがデバイスプロビジョニングサービス（DPS）から削除されても、デバイス登録はAzure IoT Hub内に存在します。デバイスを完全にプロビジョニング解除するには、その登録も削除する必要があります。

1. リソースタイルで、[**iot-az220-training-{your-id}**]をクリックします。

1. IoTハブブレードの左側のメニューで、[**エクスプローラー**]の下にある[**IoTデバイス**]をクリックします。

1. **sensor-thl-2000**および他のグループ登録済みデバイスのデバイスIDがAzure IoT Hubデバイスレジストリ内にまだ存在していることに注意してください。

1. sensor-thl-2000デバイスを削除するには、**sensor-thl-2000**の左側にあるチェックボックスをオンにして、[**削除**]をクリックします。

1. 「_選択したデバイスを削除してもよろしいですか_」と表示されたら、[**はい**]をクリックします。

1. 上記の手順4〜5を繰り返して、次のデバイスを削除します。

    * sensor-thl-2001
    * sensor-thl-2002
    * sensor-thl-2003

#### タスク 3: デバイスがプロビジョニング解除されていることを確認する

グループ登録がデバイスプロビジョニングサービスから削除され、デバイスがAzure IoT Hubデバイスレジストリから削除されたため、デバイスはソリューションから完全に削除されました。

1. ContainerDeviceコードプロジェクトを含むVisual Studioコードウィンドウに切り替えます。

    前の演習の後でVisual Studio Codeを閉じた場合は、Visual Studio Codeを使用してラボ6のスターターフォルダーを開きます。

1. Visual Studio Codeの[**表示**]メニューで、[**ターミナル**]をクリックします。

1. コマンドプロンプトが**ContainerDevice**フォルダーの場所にあることを確認します。

1. シミュレートされたデバイスアプリの実行を開始するには、次のコマンドを入力します。

    ```cmd/sh
    dotnet run
    ```

1. デバイスがプロビジョニングを試みたときにリストされる例外に注意してください。

    グループ登録と登録済みデバイスが削除されたので、シミュレートされたデバイスはプロビジョニングまたは接続できなくなります。アプリケーションが設定されたX.509証明書を使用してDPSに接続しようとすると、 `ProvisioningTransportException`エラーメッセージが返されます。


    ```txt
    Found certificate: AFF851ED016CA5AEB71E5749BCBE3415F8CF4F37 CN=sensor-thl-2000; PrivateKey: True
    Using certificate AFF851ED016CA5AEB71E5749BCBE3415F8CF4F37 CN=sensor-thl-2000
    RegistrationID = sensor-thl-2000
    ProvisioningClient RegisterAsync . . . Unhandled exception. Microsoft.Azure.Devices.Provisioning.Client.ProvisioningTransportException: {"errorCode":401002,"trackingId":"df969401-c766-49a4-bab7-e769cd3cb585","message":"Unauthorized","timestampUtc":"2019-12-20T21:30:46.6730046Z"}
       at Microsoft.Azure.Devices.Provisioning.Client.Transport.ProvisioningTransportHandlerAmqp.ValidateOutcome(Outcome outcome)
       at Microsoft.Azure.Devices.Provisioning.Client.Transport.ProvisioningTransportHandlerAmqp.RegisterDeviceAsync(AmqpClientConnection client, String correlationId, DeviceRegistration deviceRegistration)
       at Microsoft.Azure.Devices.Provisioning.Client.Transport.ProvisioningTransportHandlerAmqp.RegisterAsync(ProvisioningTransportRegisterMessage message, CancellationToken cancellationToken)
    ```
    
デバイスプロビジョニングサービスを使用して、IoTデバイスのライフサイクルの一部として、登録、構成、プロビジョニング解除を完了しました。
