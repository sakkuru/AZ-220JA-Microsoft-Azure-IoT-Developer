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
| リソース グループ | AZ-220-RG |
| IoT Hub | AZ-220-HUB-_{YOUR-ID}_ |
| デバイス プロビジョニング サービス | AZ-220-DPS-_{YOUR-ID}_ |

これらのリソースを利用できない場合は、演習 2 に進む前に、以下の手順に従って  **lab06-setup.azcli** スクリプトを実行する必要があります。スクリプト ファイルは、開発環境構成 (ラボ 3) の一部としてローカルに複製した GitHub リポジトリに含まれています。

**lab06-setup.azcli** スクリプトは **Bash** シェル環境で実行するように記述されています 。 これを実行する最も簡単な方法は Azure Cloud Shell で実行することです。

1. ブラウザーを使用して [Azure Cloud Shell](https://shell.azure.com/) を開き、このコースで使用している Azure サブスクリプションでログインします。

    Cloud Shell のストレージの設定に関するメッセージが表示された場合は、デフォルトをそのまま使用します。

1. Azure Cloud Shell が **Bash** を使用していることを確認 します。

    「Azure Cloud Shell」 ページの左上隅にあるドロップダウンは、環境を選択するために使用されます。選択されたドロップダウンの値が **Bash**であることを確認します。 

1. Azure Shell ツール バーで、「**ファイルのアップロード/ダウンロード**」 をクリックします (右から 4 番目のボタン)。

1. ドロップダウンで、「**アップロード**」 をクリックします。

1. ファイル選択ダイアログで、開発環境を構成したときにダウンロードした GitHub ラボ ファイルのフォルダーの場所に移動します。

    _ラボ 3: 開発環境の設定_:ZIP ファイルをダウンロードしてコンテンツをローカルに抽出することで、ラボ リソースを含む GitHub リポジトリを複製しました。抽出されたフォルダー構造には、次のフォルダー パスが含まれます。

    * Allfiles
      * ラボ
          * 06 - DPS でのデバイスの自動登録
            * セットアップ

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

    RGName="AZ-220-RG"
    IoTHubName="AZ-220-HUB-{YOUR-ID}"

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

1. ポータル ウィンドウの右上で、Azure Cloud Shell を開き、「**クラウド シェル**」をクリックします。 

    「Cloud Shell」 ボタンには、コマンド プロンプトを表すアイコン **`>_`** が表示されます。

    表示画面の下部付近に Cloud Shell  ウィンドウが開きます。

1. Cloud Shell ウィンドウの左上隅で、環境オプションとして「**Bash**」が選択されていることを確認します。

    > **注意**:  Azure Cloud Shell では、*Bash* インターフェイスと *PowerShell* インターフェイスの両方で **OpenSSL** の使用がサポートされています。この演習では、*Bash* シェル用に書かれたヘルパー スクリプトをいくつか使用します。 

1. Cloud Shell コマンド プロンプトで、使用する Azure IoT ヘルパー スクリプトをダウンロードするには、次のコマンドを入力します。

    ```sh
    # 証明書ディレクトリを作成する
    mkdir certificates
    # 証明書ディレクトリに移動
    cd certificates

    # ヘルパー スクリプト ファイルをダウンロードする
    curl https://raw.githubusercontent.com/Azure/azure-iot-sdk-c/master/tools/CACertificates/certGen.sh --output certGen.sh
    curl https://raw.githubusercontent.com/Azure/azure-iot-sdk-c/master/tools/CACertificates/openssl_device_intermediate_ca.cnf --output openssl_device_intermediate_ca.cnf
    curl https://raw.githubusercontent.com/Azure/azure-iot-sdk-c/master/tools/CACertificates/openssl_root_ca.cnf --output openssl_root_ca.cnf

    # スクリプトのアクセス許可を更新して、ユーザーがスクリプトの読み取り、書き込み、実行を行えるようにする
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

    これは、デバイス プロビジョニング サービス名 `AZ-220-DPS-{YOUR-ID}` です。

1. 「**デバイス プロビジョニング サービス**」 ブレードの左側にある 「**設定**」 セクションで、「**証明書**」 をクリックします。     

1. 「**証明書**」 ウィンドウの上部にある 「**追加**」 をクリックします。  

    「**追加**」 をクリックすると、X.509 CA 証明書を DPS サービスにアップロードするプロセスが開始されます。 

1. 「**証明書の追加**」ウィンドウの 「**証明書名**」に「**root-ca-cert**」と入力します。     

    ルート証明書と中間証明書、またはチェーン内の階層レベルにおける複数の証明書などの、証明書を区別できる名前を付ける必要があります。

    > **注意**: 入力したルート証明書名は、証明書ファイルの名前と同じか、または別の名前である可能性があります。指定した名前は、コンテンツ X.509 CA 証明書に埋め込まれている _共通名 _ とは相関関係のない論理名です。

1. **Certificate .pem または .cer ファイル**で、 _ファイルの選択_ テキスト ボックスの右側にある「**開く**」をクリックします。     

    テキスト フィールドの右の 「**開く**」 ボタンをクリックすると、「ファイルを開く」 ダイアログが開き、前にダウンロードした `azure-iot-test-only.root.ca.cert.pem` CA 証明書に移動できます。

1. 画面の下部で、「**保存**」 をクリックします。

    X.509 CA 証明書がアップロードされると、「証明書」 ペインに _「未確認」_ の状態の証明書が表示されます。  この CA 証明書を使用して DPS に対してデバイスを認証できるようになる前に、証明書の**所有証明**を確認する必要があります。

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

1. 「**証明書**」 ウィンドウで、証明書の 「**状態**」 が _検証済み_ と表示されていることを確認します。

    この変更を表示するには、ウィンドウの上部 (「**追加**」ボタンの右側) の 「**更新**」 ボタンを使用する必要があります。

### 演習 3: DPS でのグループ登録 (X.509 証明書) の作成

この練習では、_証明書の構成証明_を使用するデバイス プロビジョニング サービス (DPS) 内に新しい登録グループ を作成します。 

#### タスク 1: 登録の作成

1. 必要に応じて、Azure アカウントの認証情報を使用して Azure portal にログインします。

    複数の Azure アカウントをお持ちの場合は、このコースで使用するサブスクリプションに関連付けられているアカウントでログインしていることを確認してください。

1. リソース グループ タイルで、**AZ-220-DPS-_{YOUR-ID}_** をクリックします。

1. 「デバイス プロビジョニング サービス」 ブレードの左側にある 「**設定**」 で、「**登録の管理**」 をクリックします。   

1. ブレードの上部にある、「**登録グループの追加**」 をクリックします。

    登録グループは、基本的に自動プロビジョニングを通じて登録できるデバイスの記録であることを思い出してください。

1. 「**登録グループの追加**」 ブレードの 「**グループ名**」 に、「**simulated-devices**」と入力します

1. 「**証明タイプ**」 が 「**証明書**」に設定されていることを確認します。

1. 「**証明書の種類**」 フィールドが 「**CA 証明書**」 に設定されていることを確認します。

1. 「**プライマリ証明書**」 ドロップダウンで、以前に DPS にアップロードされた CA 証明書 (**root-ca-cert** など) を選択します。  

1. 「**セカンダリ証明書**」 ドロップダウン リストを 「**証明書なし**」 のままにします。   

    通常、セカンダリ証明書は、期限切れの証明書や侵害された証明書に対応するために、証明書のローテーションに使用されます。証明書のローリングに関する詳細については、 こちらをご覧ください。[https://docs.microsoft.com/ja-jp/azure/iot-dps/how-to-roll-certificates](https://docs.microsoft.com/ja-jp/azure/iot-dps/how-to-roll-certificates)

1. **ハブにデバイスを割り当てる方法**は、「**加重が均等に分布**」 のままにします。   

   複数の分散ハブがある大規模な環境では、この設定によって、デバイス登録を受信する IoT Hub の選択方法を制御できます。このラボの登録に関連付けられた IoT ハブが 1 つあるため、IoT Hub にデバイスを割り当てる方法は、このラボ シナリオでは実際には適用されません。 

1. 「**このデバイスを割り当て可能な IoT ハブの選択**」 ドロップダウンで、「**AZ-220-HUB-_{YOUR-ID}_**」 IoT ハブが選択されていることを確認します。

   このフィールドは、このデバイスを割り当てることができる IoT ハブを指定します。

1. 「**再プロビジョニング時にデバイス データを処理する方法を選択する**」 を 「**データの再プロビジョニングと移行**」 に既定値のままにします。

    このフィールドによって、同じデバイス (同じ登録 ID で示される) が、少なくとも 1 回正常にプロビジョニングされた後に、プロビジョニング要求を後で送信する再プロビジョニング動作を高度に制御できます。

1. 「**このグループを割り当てることができる IoT ハブの選択**」 ドロップダウンで **AZ-220-HUB-*{YOUR-ID}*** IoT ハブが選択されていることに注意してください。

    これにより、デバイスがプロビジョニングされると、この IoT ハブに追加されます。

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

#### タスク 2: 登録を検証する

1. 「**登録グループ**」 タブが表示され、新しい登録グループが一覧表示されていることを確認します。

    登録グループが一覧にない場合は、 ブレードの上部にある 「**更新**」 をクリックします。

1. グループ名の一覧で、**simulated-devices** をクリックします。

1. 「**登録グループの詳細**」 ブレードで、以下を確認します。

    * 「**証明書の種類**」 が 「**CA 証明書**」 に設定されていること
    * 「**プライマリ証明書**」 が 「**root-ca-cert**」 に設定されていること
    * 「**セカンダリ証明書**」 が 「**証明書が選択されていません**」 に設定されていること
    * 「**ハブにデバイスを割り当てる方法を選択**」 が 「**均等加重分布**」 に設定されていること
    * 「**このグループに割り当てることができる IoT ハブを選択する**」 が 「**AZ-220-HUB-{YOUR-ID}.azure-devices.net**」 に設定されていること
    * 「**初期デバイス ツイン状態**」 に、値 `"1"` に設定されている `telemetryDelay` プロパティが含まれていること

1. 登録グループの設定を確認したら、「**登録グループの詳細**」 ブレードを閉じます。

### 演習 4: X.509 証明書を使用してシミュレートされたデバイスを構成する

この演習では、X.509 証明書を使用してデバイス プロビジョニング サービス (DPS) を介して Azure IoT Hub に接続するように、C# で記述されたシミュレートされたデバイスを構成します。この演習では、 **シミュレートされたデバイス** ソース コード内のワークフロー、および DPS での認証と IoT ハブへのメッセージの送信のしくみについても説明します。

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
    ./certGen.sh create_device_certificate simulated-device1
    ```

    このコマンドは、以前に生成された CA 証明書によって署名された新しい X.509 証明書を作成します。デバイス ID (`simulated-device1`) が `certGen.sh` スクリプトの `create_device_certificate` コマンドに渡されることに注意してください。このデバイス ID は、 デバイス証明書の _common name_ `CN=` の値内に設定されます。この証明書は、シミュレートされたデバイスのリーフ デバイス X.509 証明書を生成し、デバイス プロビジョニング サービス (DPS) でデバイスを認証するために使用されます。

    `create_device_certificate` コマンドが完了すると、生成された X.509 デバイス証明書は `new-device.cert.pfx` という名前になり、`/certs` サブディレクトリ内に配置されます。

    > **注意**: このコマンドは、`/certs` サブディレクトリ内の既存のデバイス証明書を上書きします。複数デバイスの証明書を作成する場合は、コマンドを実行するたびに `new-device.cert.pfx` のコピーを保存してください。

1. 生成された X.509 デバイス証明書を Cloud Shell からローカル コンピューターにダウンロードするには、次のコマンドを入力します。

    ```sh
    download ~/certificates/certs/new-device.cert.pfx
    ```

    シミュレートされたデバイスは、この X.509 デバイス証明書を使用して、デバイス プロビジョニング サービスで認証するように構成されます。

1. Azure portal 内で、「**デバイス プロビジョニング サービス**」 ブレードに移動し、「**概要**」 ウィンドウに移動します。

1. 「**概要**」 ウィンドウで、デバイス プロビジョニング サービスの 「**ID スコープ**」 をコピーし、後で参照するために保存します。

    値の上にマウス ポインターを合わせると表示される値の右側に 「コピー」 ボタンがあります。

    **ID スコープ**は、次の値と似ています。  `0ne0004E52G`

1. Windows ファイル エクスプローラーを開き、`new-device.cert.pfx` がダウンロードされたフォルダーに移動します。

1. ファイル エクスプローラを使用して、`new-device.cert.pfx` ファイルのコピーを作成します。

1. ファイル エクスプローラーで、ラボ 6 (DPS のデバイスの自動登録) のスターター フォルダーに移動します。

    _ラボ 3: 開発環境の設定_:ZIP ファイルをダウンロードしてコンテンツをローカルに抽出することで、ラボ リソースを含む GitHub リポジトリを複製しました。抽出されたフォルダー構造には、次のフォルダー パスが含まれます。

    * Allfiles
      * ラボ
          * 06 - DPS でのデバイスの自動登録
            * スターター

1. `new-device.cert.pfx` ファイルをスターター フォルダに貼り付けます。

    ラボ 6 スターター フォルダのルート ディレクトリには、`Program.cs` ファイルが含まれています。  **シミュレートされたデバイス**プロジェクトは、デバイス プロビジョニング サービスに認証するときに、この証明書ファイルにアクセスする必要があります。

1. **Visual Studio Code** を開きます。

1. **ファイル** メニューで、**フォルダを開く** を選択します。

1. 「フォルダーを開く」 ダイアログで、「**06-DPS のデバイスの自動登録**」 フォルダーに移動します。

1. 「**スタート**」 ボタンをクリックし、「**フォルダーの選択**」 をクリックします。

    Visual Studio Code のエクスプローラー ウィンドウに次のファイルが一覧表示されます。

    * new-device.cert.pfx
    * Program.cs
    * SimulatedDevice.csproj

1. `SimulatedDevice.csproj` ファイルを開きます。

1. `SimulatedDevice.csproj` ファイルで、`<ItemGroup>` タグに次の内容が含まれていることを確認します。 

    ```xml
        <None Update="new-device.cert.pfx" CopyToOutputDirectory="PreserveNewest" />
    ```

    それがない場合は、それを追加します。完了すると、`<ItemGroup>` タグは次のようになります。

    ```xml
            <ItemGroup>
                <None Update="new-device.cert.pfx" CopyToOutputDirectory="PreserveNewest" />
                <PackageReference Include="Microsoft.Azure.Devices.Client" Version="1.21.1" />
                <PackageReference Include="Microsoft.Azure.Devices.Provisioning.Transport.Mqtt" Version="1.1.8" />
                <PackageReference Include="Microsoft.Azure.Devices.Provisioning.Transport.Amqp" Version="1.1.9" />
                <PackageReference Include="Microsoft.Azure.Devices.Provisioning.Transport.Http" Version="1.1.6" />
            </ItemGroup>
        </Project>
    ```

    この構成により、C# コードのコンパイル時に `new-device.cert.pfx` 証明書ファイルがビルド フォルダーにコピーされ、プログラムの実行時にアクセスできるようになります。

    > **注意**: 正確な `PackageReference` エントリは、使用しているラボ コードの正確なバージョンによって少し異なる場合があります。

1. 「Visual Studio Code **ファイル**」 メニューの 「**保存**」 をクリックします。

1. `Program.cs` ファイルを開きます。

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

1. Locate the `s_certificateFileName` 変数を見つけて、その値が、生成したデバイス証明書ファイル (`new-device.cert.pfx`) に設定されていることを確認します。

    `new-device.cert.pfx` ファイルは、クラウド シェル内のヘルパー スクリプトを使用して生成した X.509 デバイス証明書ファイル `certGen.sh`。この変数は、デバイス プロビジョニング サービスで認証するときに使用する X.509 デバイス証明書がどのファイルに含まれているかをデバイス コードに指示します。

1. `s_certificatePassword` 変数を見つけ、その値が `certGen.sh` スクリプトの既定のパスワードに設定されていることを確認します。

    `s_certificatePassword` 変数には、X.509 デバイス証明書のパスワードが含まれています。これは X.509 証明書を生成するときに `certGen.sh` ヘルパー スクリプトで使用される既定のパスワードであるため、`1234` に設定されています。

    > **注意**: このラボの目的のために、パスワードはハード コーディングされています。_運用_シナリオでは、Azure Key Vault などに、より安全な方法でパスワードを保存する必要があります。さらに、証明書ファイル (PFX) は、ハードウェア セキュリティ モジュール (HSM) を使用して、運用デバイスに安全に安全に格納する必要があります。
    >
    > HSM (ハードウェア セキュリティ モジュール) は、デバイス シークレットの安全なハードウェア ベースのストレージに使用され、最も安全なフォームのシークレット ストレージです。X.509 証明書と SAS トークンの両方を HSM に格納できます。HSM は、プロビジョニング サービスがサポートするすべての構成証明メカニズムで使用できます。

1. `public static int Main` メソッドを見つけ、その後コードのレビューまで少々お待ちください。 

    Main メソッドは、シミュレートされたデバイス アプリのエントリ ポイントです。最初に行うことは、`X509Certificate2` オブジェクトを戻す `LoadProvisioningCertificate` メソッドを呼び出すことです。

1. 下にスクロールして `LoadProvisioningCertificate` メソッドを見つけ、`X509Certificate2` オブジェクトの生成に使用するコードをレビューするまで少々お待ちください。

    `LoadProvisioningCertificate` は、(`new-device.cert.pfx` ファイルから) X.509 デバイス証明書を読み込むために `s_certificateFileName` を使用します。

1. `public static int Main` メソッドまで上にスクロールし、入れ子の `using` ステートメント内のコードをレビューするまで少々お待ちください。

    このコードは、X.509 デバイス証明書の `security` `SecurityProviderX509Certificate` オブジェクトを開始し、シミュレートされたデバイスが、Azure IoT Hub に接続するときに通信プロトコルとして AMQP を使用することを定義する `transport` `ProvisioningTransportHandlerAmqp` オブジェクトを作成します。

    `security` および `transport`オブジェクトは、DPS ID スコープと DPS グローバル デバイス エンドポイントと共に `ProvisioningDeviceClient.Create` メソッドに渡されることに注意してください。ProvisioningDeviceClient オブジェクト `provClient` は、デバイス プロビジョニング サービスにデバイスを登録するために使用されます。

    `ProvisioningDeviceLogic` は、`provClient` オブジェクトと `security` オブジェクトを渡すことによってインスタンス化されることに注意してください。`ProvisioningDeviceLogic` クラスは、デバイス (シミュレートされたデバイス) のロジックを定義するために使用されます。これには、シミュレートされたデバイス センサーから読み取り、device-to-cloud メッセージを Azure IoT Hub に送信することで、実行中のデバイスをシミュレートするためのコードが含まれています。また、後で変更され、デバイス ツインの必要なプロパティの変更に応じてデバイスを更新するコードが含まれ、クラウドからデバイスに送信されます。

1. `ProvisioningDeviceLogic` クラスまで下にスクロールし、`RunAsync` メソッドを見つけます。

    時間を割いて、`RunAsync`メソッドを確認し、いくつかの重要なポイントを指摘します。

1. `RunAsync` メソッド内で、ProvisioningDeviceClient.RegisterAsync メソッド (以下に示す) を含むコードに注意してください。

    ```csharp
    DeviceRegistrationResult result = await _provClient.RegisterAsync().ConfigureAwait(false);
    ```

    RegisterAsync メソッドは、デバイス プロビジョニング サービスを使用してデバイスを登録し、Azure IoT Hub に割り当てるために使用されます。

1. 新しい `DeviceAuthenticationWithX509Certificate` オブジェクトをインスタンス化するコードに注意してください (下図)。 

    ```csharp
    var auth = new DeviceAuthenticationWithX509Certificate(result.DeviceId, (_security as SecurityProviderX509).GetAuthenticationCertificate());
    ```

    これは、X.509 デバイス証明書を使用して、Azure IoT Hub に対するシミュレートされたデバイスの認証に使用されるデバイス認証オブジェクトです。コンストラクターは、DPS に登録されたデバイスのデバイス ID と、デバイスを認証するための X.509 デバイス証明書を渡されます。

1. `DeviceClient.Create` メソッドを呼び出すコードに注意してください。

    ```csharp
    using (iotClient = DeviceClient.Create(result.AssignedHub, auth, TransportType.Amqp))
    {
    ```

    `DeviceClient.Create` メソッドは、Azure IoT Hub サービスとの通信に使用される新しい IoT `DeviceClient` オブジェクトの作成のために使用されます。このコードは`TransportType.Amqp` の値を渡し、AMQP プロトコルを使用して Azure IoT Hub と通信するよう `DeviceClient` に指示します。または、ネットワーク アーキテクチャやデバイス要件などに応じて、MQTT または HTTP プロトコルを使用して Azure IoT Hub に接続し、通信することもできます。

1. `SendDeviceToCloudMessagesAsync` メソッドの呼び出しに注意してください。 

    ```csharp
    await SendDeviceToCloudMessagesAsync(iotClient);
    ```

    `SendDeviceToCloudMessagesAsync` メソッドは、コード内でさらに定義された別のメソッドです。このメソッドには、シミュレートされたセンサーから読み取り、device-to-cloud メッセージを Azure IoT Hub に送信するために使用されるコードが含まれています。このメソッドには、シミュレートされたデバイスの実行中に実行を継続するループも含まれます。

1. まだ RunAsync メソッド内での `DeviceClient.CloseAsync` メソッドの呼び出しに注意してください。

    ```csharp
    await iotClient.CloseAsync().ConfigureAwait(false);
    ```

    このメソッドは、`DeviceClient` オブジェクトを閉じるため、Azure IoT Hub との接続を閉じます。これは、シミュレートされたデバイスがシャットダウンしたときに実行されるコードの最後の行です。

1. 下にスクロールして、`SendDeviceToCloudMessagesAsync` メソッドを見つけます。

    ここでも、いくつかの主な詳細を指摘します。

1. シミュレートされたセンサー読み取り値を生成するコード (下図参照) に注目してください。

    ```csharp
    double currentTemperature = minTemperature + rand.NextDouble() * 15;
    double currentHumidity = minHumidity + rand.NextDouble() * 20;
    double currentPressure = minPressure + rand.NextDouble() * 12;
    double currentLatitude = minLatitude + rand.NextDouble() * 0.5;
    double currentLongitude = minLongitude + rand.NextDouble() * 0.5;
    ```

    各センサータイプについて、while ループ内の最小センサ値にランダムな値が追加されます。min 値はループの外側で初期化されます。これにより、IoT ハブに報告できるセンサー値の読み取り値の範囲が作成されます。

1. センサーの読み取り値を JSON オブジェクトに結合するために使用されるコードに注意してください。

    ```csharp
        var telemetryDataPoint = new
        {
            temperature = currentTemperature,
            humidity = currentHumidity,
            pressure = currentPressure,
            latitude = currentLatitude,
            longitude = currentLongitude
        };
        var messageString = JsonConvert.SerializeObject(telemetryDataPoint);
        var message = new Message(Encoding.ASCII.GetBytes(messageString));
    ```

    IoT ハブには、適切な形式のメッセージが必要です。

1. device-to-cloud `Message` に `temperatureAlert` プロパティを追加するコード行に注意してください。

    ```csharp
    message.Properties.Add("temperatureAlert", (currentTemperature > 30) ? "true" : "false");
    ```

    `temperatureAlert` プロパティの値は、 _温度_センサーの読み取り値が 30 を超えているかどうかを表す `boolean` 値に設定されています。20 ~ 35 の範囲の温度測定値を生成しているので、temperatureAlert は時間の約 3 分の 1 について "true" になります。 

    このコードは、Azure IoT Hub に送信する前にプロパティを `Message` オブジェクトに追加する方法の簡単な例です。これを使用すると、メッセージ本文の内容に加えて、送信されるメッセージにメタデータを追加できます。

1. `DeviceClient.SendEventAsync` メソッドの呼び出しに注意してください。

    ```csharp
    await deviceClient.SendEventAsync(message);
    ```

    `SendEventAsync` メソッドは、生成された `message` を受け取ってパラメーターとして送信し、device-to-cloud メッセージを Azure IoT Hub に送信する処理を行います。

1. device-to-cloud テレメトリ メッセージ間の時間の設定に使用される `Delay` メソッドの呼び出しに注意してください。

    この単純な遅延は、`_telemetryDelay` 変数を使用して、次のシミュレートされたセンサー読み取りを送信するまでの待機する秒数を定義します。次の演習では、デバイス ツインプロパティを使用して遅延時間を制御します。

### 演習 5: デバイス ツインの必要なプロパティの変更を処理します。

デバイス ツインは、メタデータ、構成、条件などのデバイス状態情報を格納する JSON ドキュメントです。Azure IoT Hub では、IoT Hub に接続するデバイスごとにデバイス ツインが維持されます。

デバイス プロビジョニング サービス (DPS) には、グループ登録を使用して登録されたデバイスの初期デバイス ツインの必要なプロパティが含まれています。デバイスが登録されると、DPS のこの初期デバイス ツイン構成を使用して IoT ハブ内にデバイスが作成されます。登録後、Azure IoT Hub は、IoT Hub デバイス レジストリ内の各デバイスのデバイス ツイン (およびそのプロパティ) を維持します。

デバイス ツインの必要なプロパティが Azure IoT Hub 内のデバイスに対して更新されると、必要な変更が IoT SDK (この場合は C# SDK) に含まれる `DesiredPropertyUpdateCallback` イベントを使用して IoT デバイスに送信されます。デバイス コード内でこのイベントを処理すると、(IoT ハブがアクセスを提供する) デバイスのデバイス ツイン状態を簡単に管理できるため、デバイスの構成とプロパティを必要に応じて更新できます。

この演習では、シミュレートされたデバイスの ソース コードを変更して、Azure IoT Hub からデバイスに送信されるデバイス ツインの必要なプロパティの変更に基づいてデバイス構成を更新するイベント ハンドラーを含めます。

> **注意**: ここで使用する一連の手順は、概念とプロセスが同じであるため、シミュレートされたデバイスを操作する場合の以前のラボの手順と非常によく似ています。  プロビジョニング プロセス内で認証に使用されるメソッドは、デバイスがプロビジョニングされた後に、デバイス ツイン プロパティの変更の処理を変更しません。

1. **Visual Studio Code** を使用して、ラボ 6 の**スターター** フォルダ－を開きます。

    前の演習からのコード プロジェクトを開いている場合は、同じコード ファイルで作業を続けます。

1. `Program.cs` ファイルを開きます。

1. `ProvisioningDeviceLogic` クラスを見つけて、`RunAsync` メソッドまで下にスクロールします。

   これは、`DeviceClient` オブジェクトを使用して、Azure IoT Hub にシミュレートされたデバイスを 接続するメソッドです。デバイス ツインの必要なプロパティの変更を受け取るデバイスの `DesiredPropertyUpdateCallback` イベント ハンドラーを統合するコードを追加します。このコードは、デバイスが Azure IoT Hub に接続した直後に実行されます。 

1. `// TODO 1` コメントを見つけて、次のコードを貼り付けます。

    ```csharp
    // TODO 1: 必要なプロパティの変更を受け取るために OnDesiredPropertyChanged イベント処理を設定する
    Console.WriteLine("Connecting SetDesiredPropertyUpdateCallbackAsync event handler...");
    await iotClient.SetDesiredPropertyUpdateCallbackAsync(OnDesiredPropertyChanged, null).ConfigureAwait(false);
    ```

    `iotClient` オブジェクトは、`DeviceClient` のインスタンスです。`SetDesiredPropertyUpdateCallbackAsync`メソッドは、デバイス ツインの必要なプロパティ変更を受け取るために `DesiredPropertyUpdateCallback` イベント ハンドラーを設定するために使用されます。このコードは、デバイス ツイン プロパティの変更イベントを受信したときに `OnDesiredPropertyChanged` という名前のメソッドを呼び出す `iotClient` を構成します。

    イベント ハンドラーを設定するための `SetDesiredPropertyUpdateCallbackAsync` メソッドが整ったので、呼び出す `OnDesiredPropertyChanged` メソッドを作成する必要があります。

1. `RunAsync` メソッドのすぐ下の空白のコード行にカーソルを置きます。

1. `OnDesiredPropertyChanged` メソッドを定義するには、次のコードを貼り付けます。

    ```csharp
        private async Task OnDesiredPropertyChanged(TwinCollection desiredProperties, object userContext)
        {
            Console.WriteLine("Desired Twin Property Changed:");
            Console.WriteLine($"{desiredProperties.ToJson()}");

            // 必要なツイン プロパティを読み取る
            if (desiredProperties.Contains("telemetryDelay"))
            {
                string desiredTelemetryDelay = desiredProperties["telemetryDelay"];
                if (desiredTelemetryDelay != null)
                {
                    this._telemetryDelay = int.Parse(desiredTelemetryDelay);
                }
                // 必要な telemetryDelay が null または未指定の場合は、変更しないでください
            }


            // ツイン プロパティをレポートする
            var reportedProperties = new TwinCollection();
            reportedProperties["telemetryDelay"] = this._telemetryDelay;
            await iotClient.UpdateReportedPropertiesAsync(reportedProperties).ConfigureAwait(false);
            Console.WriteLine("Reported Twin Properties:");
            Console.WriteLine($"{reportedProperties.ToJson()}");
        }
    ```

    `OnDesiredPropertyChanged` イベント ハンドラーが `TwinCollection` 型の `desiredProperties` パラメーターを受け入れることに注意してください。 

    `desiredProperties` パラメーターの値に `telemetryDelay` (デバイス ツインの必要なプロパティ) が含まれている `if`、コードはデバイス ツイン プロパティの値を `this._telemetryDelay` 変数に割り当てます。`SendDeviceToCloudMessagesAsync` メソッドには、IoT ハブに送信されるメッセージ間の遅延時間を設定するために `this._telemetryDelay` 変数を使用する `Task.Delay` 呼び出しが含まれていることを思い出すかもしれません。

    次のコード ブロックは、デバイスの現在の状態を Azure IoT Hub に報告するために使用されることに注意してください。このコードは、`DeviceClient.UpdateReportedPropertiesAsync` メソッドを呼び出し、デバイス プロパティの現在の状態を含む **TwinCollection** を渡します。これは、デバイス ツインの必要なプロパティ変更イベントを受信し、それに応じて構成を更新したことを IoT ハブに報告する方法です。これは、目的のプロパティのエコーではなく、プロパティに設定されている内容を報告することに注意してください。デバイスから送信された報告されたプロパティが、デバイスが受信した望ましい状態と異なる場合、IoT ハブはデバイスの状態を反映する正確なデバイス ツインを維持します。

    デバイスは、Azure IoT Hub からデバイス ツインの必要なプロパティに対する更新を受け取ることができるようになったので、デバイスの起動時に初期セットアップを構成するようにコード化する必要があります。これを行うには、デバイスは、Azure IoT Hub から現在のデバイス ツインの必要なプロパティを読み込み、それに応じてそれ自体を構成する必要があります。 

1. `RunAsync` メソッド内の `// TODO 2` コメントを見つけます。

1. デバイスの起動時に `OnDesiredPropertyChanged` メソッドを実行するコードを実装するには、次のコードを入力します。

    ```csharp
    // TODO 2: デバイスの起動が始まったばかりなので、デバイス ツイン プロパティを読み込みます
    Console.WriteLine("Loading Device Twin Properties...");
    var twin = await iotClient.GetTwinAsync().ConfigureAwait(false);
    // OnDesiredPropertyChanged イベント ハンドラーを使用して、読み込まれたデバイス ツインのプロパティを設定する (再利用!)
    await OnDesiredPropertyChanged(twin.Properties.Desired, null);
    ```

    `DeviceClient.GetTwinAsync` メソッドの呼び出しに注意してください。このメソッドは、デバイスが現在のデバイス ツインの状態をいつでも取得するために使用できます。この場合、デバイスが最初に実行を開始したときにデバイス ツインの必要なプロパティに一致するようにデバイス自身が構成できるように使用されます。

    この場合、`OnDesiredPropertyChanged` イベント ハンドラー メソッドは、デバイス ツインの目的のプロパティに基づく `telemetryDelay` プロパティの構成を 1 か所に保持するために再利用されます。これにより、コードの保守が時間の経過と共に容易になります。

1. 「Visual Studio Code **ファイル**」 メニューの 「**保存**」 をクリックします。

### 演習 6: シミュレートされたデバイスのテスト

この演習では、シミュレートされたデバイスを実行します。デバイスが初めて起動されると、デバイス プロビジョニング サービス (DPS) に接続され、構成されたグループ登録を使用して自動的に登録されます。DPS グループ登録に登録すると、デバイスは Azure IoT Hub デバイス レジストリ内に自動的に登録されます。登録が完了すると、デバイスは構成済みの X.509 証明書認証を使用して Azure IoT Hub との通信を安全に開始します。

#### タスク 1: デバイスをビルドおよび実行する

1. **Visual Studio Code** を使用して、ラボ 6 の**スターター** フォルダ－を開きます。

    前の演習からのコード プロジェクトを開いている場合は、同じコード ファイルで作業を続けます。

1. 「Visual Studio Code **ビュー**」 メニューで、「**ターミナル**」 をクリックします。

    これにより、統合されたターミナルが Visual Studio のコード ウィンドウの下部に開きます。

1. ターミナル コマンド プロンプトで、現在のディレクトリ パスが `/Starter` フォルダーに設定されていることを確認します。

    次のようなメッセージが表示されます。

    `Allfiles\Labs\06-Automatic Enrollment of Devices in DPS\Starter>`

1. **SimulatedDevice** プロジェクトをビルドして実行するには、次のコマンドを入力します。

    ```cmd/sh
    dotnet run
    ```

    > **注意**:  シミュレートされたデバイスに対して `dotnet run` を実行しているときに、`ProvisioningTransportException` 例外が表示される場合、最も一般的な原因は _無効な証明書_エラーです。これが発生する場合は、DPS の CA 証明書と、シミュレートされたデバイス アプリケーションのデバイス証明書が正しく構成されていることを確認します。
    >
    > ```text
    > localmachine:LabFiles User$ dotnet run
    > Found certificate: AFF851ED016CA5AEB71E5749BCBE3415F8CF4F37 CN=simulated-device1; PrivateKey: True
    > Using certificate AFF851ED016CA5AEB71E5749BCBE3415F8CF4F37 CN=simulated-device1
    > RegistrationID = simulated-device1
    > ProvisioningClient RegisterAsync . . . Unhandled exception. Microsoft.Azure.Devices.Provisioning.Client.ProvisioningTransportException: {"errorCode":401002,"trackingId":"2e298c80-0974-493c-9fd9-6253fb055ade","message":"Invalid certificate.","timestampUtc":"2019-12-13T14:55:40.2764134Z"}
    >   at Microsoft.Azure.Devices.Provisioning.Client.Transport.ProvisioningTransportHandlerAmqp.ValidateOutcome(Outcome outcome)
    >   at Microsoft.Azure.Devices.Provisioning.Client.Transport.ProvisioningTransportHandlerAmqp.RegisterDeviceAsync(AmqpClientConnection client, String correlationId, DeviceRegistration deviceRegistration)
    >   at Microsoft.Azure.Devices.Provisioning.Client.Transport.ProvisioningTransportHandlerAmqp.RegisterAsync(ProvisioningTransportRegisterMessage message, CancellationToken cancellationToken)
    >   at X509CertificateSimulatedDevice.ProvisioningDeviceLogic.RunAsync() in /Users/User/Documents/AZ-220/LabFiles/Program.cs:line 121
    >   at X509CertificateSimulatedDevice.Program.Main(String[] args) in /Users/User/Documents/AZ-220/LabFiles/Program.cs:line 55
    > ...
    > ```

1. ターミナル ウィンドウに表示されるシミュレートされたデバイス アプリからのコンソール出力に注意してください。

    シミュレートされたデバイスアプリケーションが実行されると、**ターミナル**は、アプリからのコンソール出力を表示します。 

    ターミナルウィンドウに表示される情報の一番上までスクロールします。

    X.509 証明書が読み込まれ、デバイスがデバイス プロビジョニング サービスに登録され、**AZ-220-HUB-_{YOUR-ID}_** IoT ハブに接続するように割り当てられ、デバイス ツインの必要なプロパティが読み込まれることに注意してください。

    ```text
    localmachine:LabFiles User$ dotnet run
    Found certificate: AFF851ED016CA5AEB71E5749BCBE3415F8CF4F37 CN=simulated-device1; PrivateKey: True
    Using certificate AFF851ED016CA5AEB71E5749BCBE3415F8CF4F37 CN=simulated-device1
    RegistrationID = simulated-device1
    ProvisioningClient RegisterAsync . . . Device Registration Status: Assigned
    ProvisioningClient AssignedHub: AZ-220-HUB-CP1119.azure-devices.net; DeviceID: simulated-device1
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

    シミュレートされたデバイスのソース コードを確認するには、`Program.cs` ソース コード ファイルを開きます。コンソールに表示されるメッセージを出力するために使用されるいくつかの `Console.WriteLine` ステートメントを探します。

1. JSON 形式のテレメトリ メッセージが Azure IoT Hub に送信されていることに注意してください。

    ```text
    Start reading and sending device telemetry...
    12/9/2019 5:47:00 PM > Sending message: {"temperature":24.047539159212047,"humidity":67.00504162675004,"pressure":1018.8478924248358,"latitude":40.129349260196875,"longitude":-98.42877188146265}
    12/9/2019 5:47:01 PM > Sending message: {"temperature":26.628804161040485,"humidity":68.09610794675355,"pressure":1014.6454375411363,"latitude":40.093269544242695,"longitude":-98.22227128174003}
    ```

    シミュレートされたデバイスが最初の起動を通過すると、Azure IoT Hub へのシミュレートされたセンサーのテレメトリ メッセージの送信が開始されます。

    IoT ハブに送信される各メッセージ間の `telemetryDelay` デバイス ツイン プロパティで定義されている遅延が、センサー テレメトリ メッセージの送信の間に現在**1 秒**遅延していることに注意してください。

    次のタスクのために、シミュレートされたデバイスを実行したままにします。

#### タスク 2: ツインを使用してデバイスの構成を変更する

シミュレートされたデバイスを実行すると、Azure IoT Hub 内でデバイス ツインの必要な状態を編集することで、`telemetryDelay` 構成を更新できます。これは、Azure portal 内の Azure IoT Hub でデバイスを構成することで実現できます。

1. **Azure portal** を開き、**Azure IoT Hub** サービスに移動します。

1. IoT ハブ ブレードの左側にある**エクスプローラ** セクションで、「**IoT デバイス**」 をクリックします。

1. IoT デバイスの一覧で、「**simulated-device1**」 をクリックします。

    > **重要**: このラボからデバイスを選択していることを確認します。前のラボの間に作成された _SimulatedDevice1_ という名前のデバイスも表示される場合があります。

1. デバイス ブレードのブレードの上部にある 「**デバイス ツイン**」 をクリックします。

    「**デバイス ツイン**」 ブレード内には、デバイス ツイン用の完全な JSON を含むエディターがあります。これにより、Azure portal 内でデバイス ツインの状態を直接表示および/または編集できます。

1. デバイス ツイン JSON 内の `properties.desired` ノードを見つけます。`telemetryDelay` プロパティを `"2"` の値を持つように更新します。保存されると、シミュレートされたデバイスの `telemetryDelay` が更新され、センサーのテレメトリが **2 秒**ごとに送信されます。

    デバイス ツインの目的のプロパティのこのセクションの結果の JSON は、次のようになります。

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

    JSON 内の `properties.desired` ノードの `$metadata` と `$version` 値をそのままにします。`telemetryDelay` 値のみを更新して、新しいデバイス ツインの必要なプロパティ値を設定する必要があります。

1. ブレードの上部で、デバイスに対して必要な新しいデバイス ツインのプロパティを設定するには、「**保存**」 をクリックします。 

    保存すると、更新されたデバイス ツインの必要なプロパティが、シミュレートされたデバイスに自動的に送信されます。

1. シミュレートされたデバイスが実行されている **Visual Studio Code ターミナル** ウィンドウに戻り、更新されたデバイス ツイン `telemetryDelay` の必要なプロパティ設定がアプリケーションに通知されていることに注意してください。

    アプリケーションは、新しいデバイス ツインの必要なプロパティが読み込まれ、変更が設定され、Azure IoT Hub に報告されたことを示すメッセージをコンソールに出力します。

    ```text
    Desired Twin Property Changed:
    {"telemetryDelay":2,"$version":2}
    Reported Twin Properties:
    {"telemetryDelay":2}
    ```

1. シミュレートされたデバイスのセンサー テレメトリ メッセージが、Azure IoT Hub に _2 秒_ごとに送信されていることに注意してください。

    ```text
    12/9/2019 5:48:06 PM > Sending message: {"temperature":33.89822140284731,"humidity":78.34939097908763,"pressure":1024.9467544610131,"latitude":40.020042418755764,"longitude":-98.41923808825841}
    12/9/2019 5:48:09 PM > Sending message: {"temperature":27.475786026323114,"humidity":64.4175510594703,"pressure":1020.6866468579678,"latitude":40.2089999240047,"longitude":-98.26223221770334}
    12/9/2019 5:48:11 PM > Sending message: {"temperature":34.63600901637041,"humidity":60.95207713588703,"pressure":1013.6262313688063,"latitude":40.25499096898331,"longitude":-98.51199886959347}
    ```

1. 「**ターミナル**」 画面で **Ctrl-C** キーを押して、シミュレートされたデバイス アプリを終了します。

1. Azure portal で、「**デバイス ツイン**」 ブレードに移動します。

1. 引き続き Azure portal で、「シミュレートされた device1」 ブレードの 「**デバイス ツイン**」 をクリックします。

1. 下にスクロールして `properties.reported` オブジェクトの JSON を見つけます。

    これには、デバイスによって報告された状態が含まれます。`telemetryDelay` プロパティもここにあり、`2` に設定されています。  また、値が最後に更新された時点と、特定のレポート値が最後に更新された時刻を示す `$metadata` 値もあります。

1. もう一度 **デバイス ツイン**ブレードを閉じます。

1. 「シミュレートされた device1」 ブレードを閉じて、Azure portal のダッシュボードに戻ります。

### 演習 7: グループ登録の廃止

この演習では、登録グループとそのデバイスをデバイス プロビジョニング サービスと Azure IoT Hub の両方から取り外します。

#### タスク 1: DPS から登録グループを削除する

1. 必要に応じて、Azure アカウントの認証情報を使用して Azure portal にログインします。

    複数の Azure アカウントをお持ちの場合は、このコースで使用するサブスクリプションに関連付けられているアカウントでログインしていることを確認してください。

1. リソース グループ タイルで 「**AZ-220-DPS-_{YOUR-ID}_**」 をクリックして、デバイス プロビジョニング サービスに移動します。

1. 左側のメニューの 「**設定**」 で、「**登録の管理**」 をクリックします。

1. 「**登録グループ**」 の一覧 で、「**シミュレートされたデバイス**」 をクリックします。

1. 「**登録グループの詳細**」 ブレードで、下にスクロールして 「**エントリの有効化**」 フィールドを見つけ、「**無効**」 をクリック します。

    DPS 内のグループ登録を無効にすると、この登録グループ内のデバイスを一時的に無効にできます。これにより、これらのデバイスで使用していた X.509 証明書の一時的なブラックリストが作成されます。

1. ブレードの上部で、「**保存**」 をクリックします。

    登録グループを完全に削除するには、登録グループを DPS から削除する必要があります。 

1. 「**登録の管理**」 ペインの 「**グループ名**」 で、「**シミュレートされたデバイス**」 の左側にあるチェック ボックスをオンにします。

    「**シミュレートされたデバイス**」 の左側にあるチェックボックス が既に選択されている場合は、オンのままにします。

1. 「**登録の管理**」 ペインの上部にある 「**削除**」 をクリックします。

1. 「**登録の削除**」 のアクションを確認するメッセージが表示されたら、「**はい**」 をクリックします。

   いったん削除するとグループ登録は DPS から完全に消去されるため、追加しなおすにはもう一度作成する必要があります。

    > **注意**:  証明書の登録グループを削除した場合、証明書チェーンに証明書を持つデバイスは、ルート証明書または証明書チェーンの上位にある別の中間証明書に対して、別の有効な登録グループがまだ存在する場合でも登録できる場合があります。

1. Azure portal ダッシュボードに戻ります。

#### タスク 2: IoT Hub からデバイスを削除する

登録グループがデバイス プロビジョニング サービス (DPS) から削除されても、デバイス登録は引き続き Azure IoT Hub 内に存在します。デバイスを完全に破棄するには、その登録も削除する必要があります。

1. Azure portal 内のリソース グループ タイルで **AZ-220-HUB-_{YOUR-ID}_** をクリックします。 

1. 「**IoT ハブ**」 ブレードの左側にある 「**エクスプローラー**」 で、 「**IoT デバイス**」をクリックします。

1. デバイス ID **simulated-device1** が Azure IoT Hub デバイス レジストリ内にまだ存在していることに注意してください。

1. デバイスを削除するには、 **simulated-device1** の左側にあるチェック ボックスをオンにし、「**削除**」をクリックします。

    デバイス(チェック ボックス) が選択されると、ブレードの上部にある「**削除**」ボタンが有効になります。 

1. 「選択したデバイスを削除してよろしいですか？」という確認メッセージが表示されたら、「**はい**」をクリック します。

#### タスク 3: 削除を確認する

デバイス プロビジョニング サービスからグループ登録を削除し、Azure IoT Hub デバイス レジストリからデバイスを削除したので、そのデバイスはソリューションから完全に削除されました。

1. SimulatedDevice コードのプロジェクトがある Visual Studio Code 画面に切り替えます。

    Visual Studio Code を前の演習の後に閉じていれば、Visual Studio Code を使用してラボ 6 スターター フォルダーを開きます。

1. 「Visual Studio Code **ビュー**」 メニューで、「**ターミナル**」 をクリックします。

1. コマンド プロンプトが 「**スターター**」 フォルダの場所にあることを確認します。

1. 次のコマンドを入力して、シミュレートされたデバイス アプリを実行します。

    ```cmd/sh
    dotnet run
    ```

1. デバイスがプロビジョニングを試みたときに、例外が表示されます。

    グループ登録と登録デバイスが削除されたので、シミュレートされたデバイスはもうプロビジョニングも接続もできません。設定した X.509 証明書を使用してアプリケーションが DPS へ接続しようとすると、`ProvisioningTransportException` エラー メッセージが返されます。

    ```txt
    Found certificate: AFF851ED016CA5AEB71E5749BCBE3415F8CF4F37 CN=simulated-device1; PrivateKey: True
    Using certificate AFF851ED016CA5AEB71E5749BCBE3415F8CF4F37 CN=simulated-device1
    RegistrationID = simulated-device1
    ProvisioningClient RegisterAsync . . . Unhandled exception. Microsoft.Azure.Devices.Provisioning.Client.ProvisioningTransportException: {"errorCode":401002,"trackingId":"df969401-c766-49a4-bab7-e769cd3cb585","message":"Unauthorized","timestampUtc":"2019-12-20T21:30:46.6730046Z"}
       at Microsoft.Azure.Devices.Provisioning.Client.Transport.ProvisioningTransportHandlerAmqp.ValidateOutcome(Outcome outcome)
       at Microsoft.Azure.Devices.Provisioning.Client.Transport.ProvisioningTransportHandlerAmqp.RegisterDeviceAsync(AmqpClientConnection client, String correlationId, DeviceRegistration deviceRegistration)
       at Microsoft.Azure.Devices.Provisioning.Client.Transport.ProvisioningTransportHandlerAmqp.RegisterAsync(ProvisioningTransportRegisterMessage message, CancellationToken cancellationToken)
    ```

    デバイス プロビジョニング サービスで IoT デバイスライフサイクルの一環として、登録、構成、および廃止を完了しました。
