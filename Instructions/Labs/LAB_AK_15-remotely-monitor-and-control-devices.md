---
lab:
    title: 'ラボ 15: Azure IoT Hub を使用してリモートによるデバイスの監視および制御'
    module: 'モジュール 8: デバイス管理'
---

# Azure IoT Hub を使用してリモートによるデバイスを監視および制御する

## ラボ シナリオ

Contoso は、受賞歴のあるチーズを誇りにしており、製造プロセス全体で完璧な温度と湿度を維持するように気を付けていますが、エージング プロセス中の状況には常に特別な注意を払ってきました。

近年、Contoso は環境センサーを使用して、ナチュラル チーズのエイジングのための洞窟の中の状態を記録し、そのデータを使用してほぼ完璧な環境を特定しています。最も成功した (賞に値する) 場所のデータによると、熟成チーズの理想的な温度はおよそ華氏 50 度 +/- 5 度 (摂氏 10 度 +/- 2.8 度) です。最大飽和度のパーセンテージで測定される理想的な湿度値は、約 85% +/- 10% です。

これらの理想的な温度と湿度の値は、ほとんどのタイプのチーズにも効果的です。ただし、特に硬いチーズや特に柔らかいチーズには、少々調整が必要です。また、チーズの外皮に望ましい条件など、特定の結果を得るためにはエージング プロセス中の重要な時期あるいは段階で環境を調整する必要があります。

Contoso は幸運にも、(特定の地域で) ほぼ一年中理想的な条件を自然に維持するチーズ洞窟を運営することができます。しかしこういった場所でも、エージングプロセス中の環境管理は重要です。また、自然の洞窟には多くの場合、異なる小部屋が多くあり、小部屋の環境はそれぞれに少しずつ違います。チーズの品種は、それぞれ特定の要件に合う小部屋 (ゾーン) に配置されます。環境条件を望ましい範囲内に保つために、温度と湿度の両方を制御する空気処理・調整システムが使われています。

現在、作業者は洞窟施設の各ゾーン内の環境条件を監視し、必要に応じて空気処理システムの設定を調整して、望ましい温度と湿度を維持しています。作業者は、4時間ごとに各ゾーンを訪問し、環境条件を確認することができます。昼は高温、夜は低温と温度が著しく変化する場所では、状況が望ましい制限の範囲外になる可能性があります。

Contoso から貯蔵庫の環境を制御制限の範囲内に保つ自動化システムの実装を任されました。

このラボでは、IoT デバイスを実装するチーズ貯蔵庫監視システムをプロトタイプとして作成します。各デバイスには温度および湿度センサーが装備されており、デバイスが配置されているゾーンの温度と湿度を制御する空気処理システムに接続されています。

### 簡略化されたラボ条件

テレメトリの出力頻度は、生産ソリューションにおいて重要な検討事項です。冷却装置の温度センサーは 1 分間に 1 回しか報告する必要がないのに対し、航空機の加速度センサーは毎秒 10 回報告する必要がある場合があります。テレメトリを送信する必要のある頻度は、現在の状況に依存する場合もあります。たとえば、チーズ貯蔵庫のシナリオにおいて夜に急速に温度が低下する傾向がある場合は、日没の 2 時間前からセンサーの読み取り頻度を高めると役立つことがあります。当然のことながら、テレメトリの頻度を変更する要件は予測可能なパターンの一部である必要はなく、IoT デバイス設定の変更を促すイベントは予測不能な可能性があります。

このラボをシンプルに進行するために、以下を前提とします。

* デバイスは数秒ごとに IoT ハブにテレメトリ (温度と湿度の値) を送信します。この頻度はチーズ貯蔵庫では非現実的ですが、15 分ごとではなく、もっと頻繁に変化を見る必要があるラボ環境には最適です。
* 空気処理システムは、次の 3 つの状態のいずれかになる送風機です：オン、オフ、エラー
  * 送風機はオフ状態に初期化されています。
  * IoTデバイス上でダイレクト メソッドによって、送風機への電力を制御 (オン/オフ) します。
  * デバイス ツインの必要なプロパティ値は、送風機の目的の状態を設定するために使用されます。必要なプロパティ値は、送風機/デバイスの規定の設定をオーバーライドします。
  * 温度は送風機をオン/オフにすることで制御できます (ファンをオンにすると温度が下がります)

このラボでのコーディングは、製品利用統計情報の送受信、ダイレクト メソッドの呼び出しと実行、デバイス ツイン プロパティの設定と読み取りの 3 つの部分に分かれています。

まず、製品利用統計情報を送信するデバイス用と、製品利用統計情報を受信する (クラウドで実行される) バックエンド サービス用の 2 つのアプリを作成します。

次のリソースが作成されます。

![ラボ 15 のアーキテクチャ](media/LAB_AK_15-architecture.png)

## このラボでは

このラボでは、次のタスクを完了します。

* ラボの前提条件を確認する
* IoT Hub ポータルを使用してカスタム Azure IoT Hub を作成する
* IoT Hub ポータルを使用して IoT Hub デバイス ID を作成する
* カスタム IoT Hub にデバイステレメトリを送信するアプリを作成する
* テレメトリをリッスンするバックエンド サービス アプリを作成する
* ダイレクト メソッドを実装し、リモートデバイスに設定を伝達する
* リモート デバイス のプロパティを管理するために、デバイス ツインを実装する

## ラボの手順

### 演習 1: ラボの前提条件を確認する

このラボでは、次の Azure リソースが利用可能であることを前提としています。

| リソースの種類:  | リソース名 |
| :-- | :-- |
| リソース グループ | AZ-220-RG |
| IoT Hub | AZ-220-HUB-_{YOUR-ID}_ |
| IoT デバイス | CheeseCaveID |

これらのリソースが利用できない場合は、演習 2 に進む前に、以下の手順に従って**lab15-setup.azcli**スクリプトを実行する必要があります。スクリプト ファイルは、開発環境構成 (ラボ 3) の一部としてローカルに複製した GitHub リポジトリに含まれています。

**lab15-setup.azcli** スクリプトは、Azure Cloud Shell でこれを実行するのが最も簡単な **Bash** シェル環境で実行するように記述されています。 

>**注:** **CheeseCaveID** デバイスの接続文字列が必要です。このデバイスが Azure IoT Hub に登録されている場合は、Azure Cloud Shell で次のコマンドを実行して接続文字列を取得できます
>
> ```bash
> az iot hub device-identity show-connection-string --hub-name AZ-220-HUB-{YOUR-ID} --device-id CheeseCaveID -o tsv
> ```

1. ブラウザーを使用して [Azure Shell](https://shell.azure.com/) を開き、このコースで使用している Azure サブスクリプションでログインします。

    Cloud Shell のストレージの設定に関するメッセージが表示された場合は、デフォルトをそのまま使用します。

1. Azure Cloud Shell が **Bash** を使用していることを確認 します。

    「Azure Cloud Shell」 ページの左上隅にあるドロップダウンは、環境を選択するために使用されます。選択されたドロップダウンの値が **Bash**であることを確認します。 

1. Azure Shell ツール バーで、「**ファイルのアップロード/ダウンロード**」 をクリックします (右から 4 番目のボタン)。

1. ドロップダウンで、「**アップロード**」 をクリックします。

1. ファイル選択ダイアログで、開発環境を構成したときにダウンロードした GitHub ラボ ファイルのフォルダーの場所に移動します。

    _ラボ 3: 開発環境の設定_:ZIP ファイルをダウンロードしてコンテンツをローカルに抽出することで、ラボ リソースを含む GitHub リポジトリを複製しました。抽出されたフォルダー構造には、次のフォルダー パスが含まれます。

    * Allfiles
      * ラボ
          * 15-Azure IoT Hub を使用してリモートによるデバイスを監視および制御する
            * セットアップ

    lab15-setup.azcli スクリプト ファイルは、ラボ 15 の「設定」フォルダー内にあります。

1. **lab15-setup.azcli** ファイルを選択し、「**開く**」 をクリック します。   

    ファイルのアップロードが完了すると、通知が表示されます。

1. 正しいファイルが Azure Cloud Shell にアップロードされたことを確認するには、次のコマンドを入力します。

    ```bash
    ls
    ```

    `ls` コマンドを実行すると、現在のディレクトリの内容が一覧表示されます。lab15-setup.azcli ファイルが一覧表示されます。

1. セットアップ スクリプトを含むディレクトリをこのラボ用に作成し、そのディレクトリに移動するには、次の Bash コマンドを入力します。

    ```bash
    mkdir lab15
    mv lab15-setup.azcli lab15
    cd lab15
    ```

1. **lab15-setup.azcli** に実行権限を持たせるには、次のコマンドを入力します。 

    ```bash
    chmod +x lab15-setup.azcli
    ```

1. Cloud Shell ツール バーで、lab15-setup.azcli ファイルを編集するには、「**エディタを開く**」 (右から 2 番目のボタン - **{ }**) をクリックします。 

1. 「**ファイル**」 リストの一覧で、ラボ 15 フォルダを展開してスクリプト ファイルを開くには、「**lab15**」をクリックし、「**lab15-setup.azcli**」をクリックします。     

    エディタが**lab15-setup.azcli** ファイルの内容を表示します。 

1. エディタで、割り当てられた値 `{YOUR-ID}` と `SETLOCATION` を更新します。

    例として次のサンプルを参照し、このコースの開始時に作成した一意の ID 、つまり **CAH191211** に `{YOUR-ID}` を設定し、`SETLOCATION` をリソースにとって意味のある場所に設定する必要があります。 

    ```bash
    #!/bin/bash

    YourID="{YOUR-ID}"
    RGName="AZ-220-RG"
    IoTHubName="AZ-220-HUB-$YourID"
    DeviceID="CheeseCaveID"

    Location="SETLOCATION"
    ```

    > **注意**: `Location` 変数は、保存先の短い名前に設定する必要があります。次のコマンドを入力すると、使用可能な場所と短い名前 (「**名前**」 の列) の一覧を表示できます。
    >
    > ```bash
    > az account list-locations -o Table
    > ```
    >
    > ```text
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
    ./lab15-setup.azcli
    ```

    このスクリプトの実行には数分かかります。各ステップが完了すると、JSON 出力が表示されます。

    このスクリプトは、まず **AZ-220-RG** という名前のリソース グループ と **AZ-220-ハブ-{YourID}** という名前の IoT ハブを作成します。  既に存在する場合は、対応するメッセージが表示されます。次に、スクリプトによって、**CheeseCaveID** の ID を持つデバイスが IoT ハブに追加され、デバイス接続文字列が表示されます。

1. スクリプトが完了すると、IoT Hub とデバイスに関する情報が表示されます。

    スクリプトは、以下のような情報を表示します。

    ```text
    Configuration Data:
    ------------------------------------------------
    AZ-220-HUB-{YourID} Service connectionstring:
    HostName=AZ-220-HUB-{YourID}.azure-devices.net;SharedAccessKeyName=iothubowner;SharedAccessKey=nV9WdF3Xk0jYY2Da/pz2i63/3lSeu9tkW831J4aKV2o=

    CheeseCaveID device connection string:
    HostName=AZ-220-HUB-{YourID}.azure-devices.net;DeviceId=CheeseCaveID;SharedAccessKey=TzAzgTYbEkLW4nWo51jtgvlKK7CUaAV+YBrc0qj9rD8=

    AZ-220-HUB-{YourID} eventhub endpoint:
    sb://iothub-ns-az-220-hub-2610348-5a463f1b56.servicebus.windows.net/

    AZ-220-HUB-{YourID} eventhub path:
    az-220-hub-{YourID}

    AZ-220-HUB-{YourID} eventhub SaS primarykey:
    tGEwDqI+kWoZroH6lKuIFOI7XqyetQHf7xmoSf1t+zQ=
    ```

1. スクリプトが表示する出力をテキスト ドキュメントにコピーして、このラボで後ほど使用します。

    情報を簡単に見つけることができる場所に保存したら、ラボを続ける準備が整います。

### 演習 2: テレメトリを送受信するコードを記述する

この演習では、IoT Hub にテレメトリを送信するシミュレートされたデバイス アプリ (CheeseCaveID デバイス用) を作成します。

#### タスク 1: Visual Studio Code で Console App を作成する

1. Visual Studio Code を起動します。

1. 「**ターミナル**」 メニューで、「**新しいターミナル**」 をクリックします。

1. Terminal コマンドプロンプトで、"cheesecavedevice" というディレクトリを作成し、現在のディレクトリをそのディレクトリに変更するには、次のコマンドを入力します。

    ```bash
    mkdir cheesecavedevice
    cd cheesecavedevice
    ```

1. 新しい .NET コンソール アプリケーションを作成するには、次のコマンドを入力します。

    ```bash
    dotnet new console
    ```

    このコマンドを実行すると、プロジェクト ファイルと共に、フォルダに **Program.cs** ファイルが作成されます。 

1. 必要なライブラリをインストールするには、次のコマンドを入力します。

    ```bash
    dotnet add package Microsoft.Azure.Devices.Client
    dotnet add package Microsoft.Azure.Devices.Shared
    dotnet add package Newtonsoft.Json
    ```

1. **「ファイル」** メニューで、**「フォルダを開く」** を選択します。

1. 「**フォルダーを開く**」 ダイアログで、「ターミナル」 ウィンドウで指定したフォルダーの場所に移動し、**cheesecavedevice** をクリックして、「**フォルダーの選択**」 をクリックします。

    エクスプローラーのウィンドウが Visual Studio Code で開き、`Program.cs` ファイルと `cheesecadedevice.csproj`ファイルが一覧表示されます。

1. **EXPLORER** ペインで、**EventsController.cs** をクリックします。

1. 「コード エディター」 ペインで、Program.cs ファイルの内容を削除します。

#### タスク 2: CheeseCaveID の IoT デバイスをシミュレートするコードを追加する

このタスクでは、シミュレートされたデバイスからテレメトリを送信するコードを追加します。デバイスは、バックエンド アプリがリッスンしているかどうかに関係なく、温度 (華氏度) と湿度 (パーセンテージ) を送信します。

1. **Program.cs**ファイルが Visual Studio Code で開かれていることを確認します。 

    「コード エディター」 ペインには、空のコード ファイルが表示されます。

1. 次のコードをコピーして 「コード エディター」 ペインに貼り付けます。

    ```csharp
    // Copyright (c) Microsoft.All rights reserved.
    // MITライセンスの下でライセンスされています。ライセンス情報の全容については、プロジェクト ルートのライセンス ファイルをご覧ください。

    using System;
    using Microsoft.Azure.Devices.Client;
    using Microsoft.Azure.Devices.Shared;
    using Newtonsoft.Json;
    using System.Text;
    using System.Threading.Tasks;
    using Newtonsoft.Json.Linq;

    namespace simulated_device
    {
        class SimulatedDevice
        {
        // グローバル定数。
            const float ambientTemperature = 70;                    // Ambient temperature of a southern cave, in degrees F.
            const double ambientHumidity = 99;                      // 空気飽和量の相対割合における周囲湿度
            const double desiredTempLimit = 5;                      // 華氏での望ましい温度の上または下の許容範囲
            const double desiredHumidityLimit = 10;                 // パーセンテージで表した、望ましい湿度の上下の許容範囲
            const int intervalInMilliseconds = 5000;                // テレメトリーがクラウドに送信される間隔

            // グローバル変数。
            private static DeviceClient s_deviceClient;
            private static stateEnum fanState = stateEnum.off;                      // Initial setting of the fan.
            private static double desiredTemperature = ambientTemperature - 10;     // Initial desired temperature, in degrees F.
            private static double desiredHumidity = ambientHumidity - 20;           // 空気飽和の相対パーセンテージでの初期の望ましい湿度。

            // 冷却/加熱、および加湿/除湿のためのファンの状態の列挙型。
            enum stateEnum
            {
                off,
                on,
                failed
            }

            // IoT ハブでデバイスを認証するためのデバイス接続文字列。
            private readonly static string s_deviceConnectionString = "<your device connection string>";

            private static void colorMessage(string text, ConsoleColor clr)
            {
                Console.ForegroundColor = clr;
                Console.WriteLine(text);
                Console.ResetColor();
            }
            private static void greenMessage(string text)
            {
                colorMessage(text, ConsoleColor.Green);
            }

            private static void redMessage(string text)
            {
                colorMessage(text, ConsoleColor.Red);
            }

            // シミュレーションされたテレメトリを送信する非同期メソッド。
            private static async void SendDeviceToCloudMessagesAsync()
            {
                double currentTemperature = ambientTemperature;         // 温度の初期設定。
                double currentHumidity = ambientHumidity;               // 湿度の初期設定。

                Random rand = new Random();

                while (true)
                {
                    // テレメトリをシミュレート
                    double deltaTemperature = Math.Sign(desiredTemperature - currentTemperature)
                    double deltaHumidity = Math.Sign(desiredHumidity - currentHumidity);

                    if (fanState == stateEnum.on)
                    {
                        // 扇風機が適正温度・湿度内の場合、たいていの時間ご希望の値内に向けて移行します。
                        currentTemperature += (deltaTemperature * rand.NextDouble()) + rand.NextDouble() - 0.5;
                        currentHumidity += (deltaHumidity * rand.NextDouble()) + rand.NextDouble() - 0.5;

                        // 無作為に扇風機が機能しなくなります。
                        if (rand.NextDouble() < 0.01)
                        {
                            fanState = stateEnum.failed;
                            redMessage("Fan has failed");
                        }
                    }
                    else
                    {
                        // ファンがオフであるか故障している場合、温度と湿度は周囲の値に達するまで上昇し、その後ランダムに変動します。
                        if (currentTemperature < ambientTemperature - 1)
                        {
                            currentTemperature += rand.NextDouble() / 10;
                        }
                        else
                        {
                            currentTemperature += rand.NextDouble() - 0.5;
                        }
                        if (currentHumidity < ambientHumidity - 1)
                        {
                            currentHumidity += rand.NextDouble() / 10;
                        }
                        else
                        {
                            currentHumidity += rand.NextDouble() - 0.5;
                        }
                    }

                    // チェック: 湿度が 100％ を超えることはありません。
                    currentHumidity = Math.Min(100, currentHumidity);

                    // Create JSON message.
                    var telemetryDataPoint = new
                    {
                        temperature = Math.Round(currentTemperature, 2),
                        humidity = Math.Round(currentHumidity, 2)
                    };
                    var messageString = JsonConvert.SerializeObject(telemetryDataPoint);
                    var message = new Message(Encoding.ASCII.GetBytes(messageString));

                    // メッセージにカスタム アプリケーション プロパティを追加します。
                    message.Properties.Add("sensorID", "S1");
                    message.Properties.Add("fanAlert", (fanState == stateEnum.failed) ? "true" : "false");

                    // 温度または湿度のアラートは、発生した場合にのみ送信します
                    if ((currentTemperature > desiredTemperature + desiredTempLimit) || (currentTemperature < desiredTemperature - desiredTempLimit))
                    {
                        message.Properties.Add("temperatureAlert", "true");
                    }
                    if ((currentHumidity > desiredHumidity + desiredHumidityLimit) || (currentHumidity < desiredHumidity - desiredHumidityLimit))
                    {
                        message.Properties.Add("humidityAlert", "true");
                    }

                    Console.WriteLine("Message data: {0}", messageString);

                    // テレメトリ メッセージを送信します。
                    await s_deviceClient.SendEventAsync(message);
                    greenMessage("Message sent\n");

                    await Task.Delay(intervalInMilliseconds);
                }
            }
            private static void Main(string[] args)
            {
                colorMessage("Cheese Cave device app.\n", ConsoleColor.Yellow);

                // MQTT プロトコルを使用して IoT ハブに接続します。
                s_deviceClient = DeviceClient.CreateFromConnectionString(s_deviceConnectionString, TransportType.Mqtt);

                SendDeviceToCloudMessagesAsync();
                Console.ReadLine();
            }
        }
    }
    ```

1. 少し時間をかけてコードについてレビューしてください。

    > **重要:** コードのコメントを読み、チーズ ケーブ シナリオの温度と湿度の設定がどのようにコードに取り組まれたかを把握します。

1. デバイス接続文字列の割り当てに使用するコード行を見つけます。

    ```csharp
    private readonly static string s_deviceConnectionString = "<your device connection string>";
    ```

1. `<your device connection string>` を、このラボで先ほど保存した CheeseCaveID デバイスの接続文字列に置き換えます。

    演習 1 の実行中に、lab15-setup.azcli セットアップ スクリプトによって生成された出力を保存しておく必要があります。

    他のコード行を変更する必要はありません。

1. 「**ファイル**」 メニューで 変更を Program.cs ファイルに保存するには、「**保存**」 をクリックします。   

#### タスク 3: テレメトリを送信するコードをテストする

1. Visual Studio Code で、ターミナルが開かれていることを確認します。

1. ターミナル コマンド プロンプトで、シミュレートされたデバイス アプリを実行するには、次のコマンドを入力します。

    ```bash
    dotnet run
    ```

   このコマンドは、 現在のフォルダー内の **Program.cs** ファイルを実行します。

1. 出力がターミナルに送信されていることに注意してください。

    すぐに次のようなコンソール出力が表示されます。

![コンソール出力](./Media/LAB_AK_15-cheesecave-telemetry.png)

    > **注意**:  緑色のテキストは、物事が本来のように機能していることを示すために使用され、赤いテキストは悪いことが起こっているときに表示されます。この画像に似た画面が表示されない場合は、まずデバイスの接続文字列を確認します。

1. このアプリを実行したままにします。

    このラボの後半で、IoT ハブにテレメトリを送信する必要があります。

### 演習 3: テレメトリを受信する 2 つ目のアプリを作成する

(シミュレートされた) CheeseCaveID デバイスから IoT ハブにテレメトリを送信できたので、IoT ハブに接続してそのテレメトリを "リッスン" できるバックエンド アプリを作成する必要があります。最終的には、このバックエンド アプリは、チーズ ケーブの温度の制御を自動化するために使用されます。

#### タスク 1: テレメトリを受信するアプリを作成する

1. Visual Studio Code の新しいインスタンスを開きます。

    シミュレートされたデバイス アプリは、既に開いている Visual Studio Code ウィンドウで実行されているため、バックエンド アプリの Visual Studio Code の新しいインスタンスが必要です。

1. 「**ターミナル**」 メニューで、「**新しいターミナル**」 をクリックします。

1. ターミナル コマンド プロンプトで、"cheesecaveoperator" という名前のディレクトリを作成し、現在のディレクトリをそのディレクトリに変更するために、次のコマンドを入力します。

   ```bash
   mkdir cheesecaveoperator
   cd cheesecaveoperator
   ```

1. 新しい .NET コンソール アプリケーションを作成するには、次のコマンドを入力します。

    ```bash
    dotnet new console
    ```

    このコマンドを実行すると、プロジェクト ファイルと共に、フォルダに **Program.cs** ファイルが作成されます。 

1. 必要なライブラリをインストールするには、次のコマンドを入力します。

    ```bash
    dotnet add package Microsoft.Azure.EventHubs
    dotnet add package Microsoft.Azure.Devices
    dotnet add package Newtonsoft.Json
    ```

1. **「ファイル」** メニューで、**「フォルダを開く」** を選択します。

1. 「**フォルダを開く**」 ダイアログで、ターミナル ペインで指定したフォルダの場所に移動し、「**cheesecaveoperator**」 をクリックしてから、「**フォルダの選択**」 をクリックします。   

    Explorer ウィンドウが Visual Studio Code で開き、`Program.cs` ファイルと `cheesecaveoperator.csproj` ファイルが表示されます。

1. **EXPLORER** ペインで、**EventsController.cs** をクリックします。

1. [コード エディター] ペインで、Program.cs ファイルの内容を削除します。

#### タスク 2: テレメトリを受信するコードを追加する

このタスクでは、IoT ハブ イベント ハブのエンドポイントからテレメトリを受信するために使用するコードを、バックエンド アプリに追加します。

1. **Program.cs**ファイルが Visual Studio Code で開かれていることを確認します。 

    [コード エディター] ペインには、空のコード ファイルが表示されます。

1. 次のコードをコピーして [コード エディター] ペインに貼り付けます。

    ```csharp
    // Copyright (c) Microsoft.All rights reserved.
    // MITライセンスの下でライセンスされています。ライセンス情報の全容については、プロジェクト ルートのライセンス ファイルをご覧ください。

    using System;
    using System.Threading.Tasks;
    using System.Text;
    using System.Collections.Generic;
    using System.Linq;

    using Microsoft.Azure.EventHubs;
    using Microsoft.Azure.Devices;
    using Newtonsoft.Json;

    namespace cheesecave_operator
    {
        class ReadDeviceToCloudMessages
        {
            // グローバル変数。
            // イベント ハブ互換エンドポイント。
            private readonly static string s_eventHubsCompatibleEndpoint = "<your event hub endpoint>";

            // イベント ハブと互換性のある名前。
            private readonly static string s_eventHubsCompatiblePath = "<your event hub path>";
            private readonly static string s_iotHubSasKey = "<your event hub Sas key>";
            private readonly static string s_iotHubSasKeyName = "service";
            private static EventHubClient s_eventHubClient;

            // IoT ハブの接続文字列。
            private readonly static string s_serviceConnectionString = "<your service connection string>";

            // パーティションの PartitionReceiver を非同期的に作成し、シミュレートされたクライアントから送信されたメッセージの読み取りを開始します。
            private static async Task ReceiveMessagesFromDeviceAsync(string partition)
            {
                // 既定のコンシューマー グループを使用して受信側を作成します。
                var eventHubReceiver = s_eventHubClient.CreateReceiver("$Default", partition, EventPosition.FromEnqueuedTime(DateTime.Now));
                Console.WriteLine("Created receiver on partition: " + partition);

                while (true)
                {
                    // EventData を確認する - このメソッドは、取得するものがない場合にタイムアウトします。
                    var events = await eventHubReceiver.ReceiveAsync(100);

                    // バッチにデータがある場合は、処理します。
                    if (events == null) continue;

                    foreach (EventData eventData in events)
                    {
                        string data = Encoding.UTF8.GetString(eventData.Body.Array);

                        greenMessage("Telemetry received: " + data);

                        foreach (var prop in eventData.Properties)
                        {
                            if (prop.Value.ToString() == "true")
                            {
                                redMessage(prop.Key);
                            }
                        }
                        Console.WriteLine();
                    }
                }
            }

            public static void Main(string[] args)
            {
                colorMessage("Cheese Cave Operator\n", ConsoleColor.Yellow);

                // IoT ハブ イベント ハブと互換性のあるエンドポイントに接続する EventHubClient インスタンスを作成します。
                var connectionString = new EventHubsConnectionStringBuilder(new Uri(s_eventHubsCompatibleEndpoint), s_eventHubsCompatiblePath, s_iotHubSasKeyName, s_iotHubSasKey);
                s_eventHubClient = EventHubClient.CreateFromConnectionString(connectionString.ToString());

                // ハブの各パーティションに対して PartitionReceiver を作成します。
                var runtimeInfo = s_eventHubClient.GetRuntimeInformationAsync().GetAwaiter().GetResult();
                var d2cPartitions = runtimeInfo.PartitionIds;

                // メッセージをリッスンする受信者を作成します。
                var tasks = new List<Task>();
                foreach (string partition in d2cPartitions)
                {
                    tasks.Add(ReceiveMessagesFromDeviceAsync(partition));
                }

                //  すべての PartitionReceivers が完了するのを待ちます。
                Task.WaitAll(tasks.ToArray());
            }

            private static void colorMessage(string text, ConsoleColor clr)
            {
                Console.ForegroundColor = clr;
                Console.WriteLine(text);
                Console.ResetColor();
            }
            private static void greenMessage(string text)
            {
                colorMessage(text, ConsoleColor.Green);
            }

            private static void redMessage(string text)
            {
                colorMessage(text, ConsoleColor.Red);
            }
        }
    }
    ```

1. 少し時間をかけてコードについてレビューしてください。

    > **重要:** コード内のコメントを読み進みます。この実装では、バックエンド アプリが起動された後にのみメッセージを読み取ります。これより前に送信されたテレメトリは処理されません。

1. サービス接続文字列の割り当てに使用するコード行を見つける

    ```csharp
    private readonly static string s_serviceConnectionString = "<your service connection string>";
    ```

1. `<your service connection string>` を、このラボで前に保存した IoT ハブの **iothubowner** 共有アクセス ポリシーのプライマリ接続文字列に置き換えます。

    演習 1 の実行中に、lab15-setup.azcli セットアップ スクリプトによって生成された出力を保存しておく必要があります。

    > **注意**: **サービス**共有ポリシーではなく、**iothubowner** 共有ポリシーがなぜ使用されているのか、不思議に思われるかもしれません。答えは、各ポリシーに割り当てられた IoT ハブのアクセス許可に関連しています。  **サービス**ポリシーには **ServiceConnect** アクセス許可があり、通常はバックエンド クラウド サービスによって使用されます。    これは、次の権利を付与します。
    >
    > * クラウド サービスに接続する通信および監視エンドポイントへのアクセスを許可します。
    > * デバイスからクラウドへのメッセージの受信、クラウドからデバイスへのメッセージの送信、および対応する配信確認応答の取得を行うアクセス許可を付与します。
    > * ファイル アップロードの配信確認を取得するアクセス許可を付与します。
    > * タグと必要なプロパティを更新する、報告されたプロパティを取得する、クエリを実行するために、ツインへのアクセス許可を付与します。
    >
    > ラボの最初の部分では、**serviceoperator** アプリケーションがファンの状態を切り替えるダイレクト メソッドを呼び出しており、 **サービス**ポリシーには十分な権限があります。ただし、ラボの後半では、デバイス レジストリが照会されます。これは `RegistryManager` クラスを通じて行われます。`RegistryManager` クラスを使用してデバイス レジストリをクエリするには、IoT ハブへの接続に使用される共有アクセス ポリシーに **レジストリ読み取り**アクセス許可が必要です。これは、次の権限を付与します。
    >
    > * ID レジストリへの読み取りアクセスを許可します。
    >
    > **iothubowner** ポリシーには **レジストリの書き込み** アクセス許可が与えられているため、 **レジストリの読み取り**権限が継承されるため、ニーズに適しています。     
    >
    > 運用シナリオでは、 **サービス接続**と**レジストリ読み取り**アクセス許可のみを持つ新しい共有アクセス ポリシーを追加することを検討してください。

1. `<your event hub endpoint>`、`<your event hub path>`、および `<your event hub Sas key>` を、このラボで前に保存した値に置き換えます。

1. 「**ファイル**」 メニューで 変更を Program.cs ファイルに保存するには、「**保存**」 をクリックします。   

#### タスク 3: テレメトリを受信するコードをテストする

このテストは、バックエンド アプリが、シミュレートされたデバイスから送信されるテレメトリをピックアップしているかどうかを確認する重要なものです。デバイス アプリがまだ実行されており、テレメトリを送信している点に注意してください。

1. ターミナルで `cheesecaveoperator` バックエンド アプリを実行するには、ターミナル ウィンドウを開き、次のコマンドを入力します。

    ```bash
    dotnet run
    ```

   このコマンドは、 現在のフォルダー内の **Program.cs** ファイルを実行します。

   > **注意**:  未使用の変数 `s_serviceConnectionString` に関する警告は無視できます 。この変数は後で使用します。

1. ターミナルへの出力をしばらく観察してください。

    コンソールの出力がすぐに表示されるはずで、IoT ハブにHub に正常に接続された場合、アプリがテレメトリ メッセージ データをほぼ即座に表示します。

    それ以外の場合は、IoT ハブサービスの接続文字列を慎重に確認し、文字列がサービス接続文字列であり、他の接続文字列ではないことを確認します。

![コンソール出力](./Media/LAB_AK_15-cheesecave-telemetry-received.png)

    > **注意**:  緑色のテキストは、物事が本来のように機能していることを示すために使用され、赤いテキストは悪いことが起こっているときに表示されます。この画像に似た画面が表示されない場合は、まずデバイスの接続文字列を確認します。

1. このアプリをしばらく実行したままにします。

1. 両方のアプリが実行されている状態で、送信されるテレメトリと受信しているテレメトリを視覚的に比較します。

    * 完全に一致するデータはありますか?
    * データが送信された時点から受信されるまでの遅延は多いですか。

    結果に満足したら、実行中のアプリを停止してから、VS Code の両方のインスタンスでターミナル ペインを閉じます。Visual Studio Code ウィンドウは閉じません。

    これで、デバイスからテレメトリを送信するアプリと、データの受信を確認するバックエンド アプリが作成されました。このユニットは、シナリオの監視側をカバーします。次のステップでは、データに問題が発生したときに実行する、コントロール側を処理します。明らかに問題があります。温度と湿度のアラートを取得しています!

### 演習 4: ダイレクト メソッドを呼び出すコードを記述する

この演習では、チーズ ケーブでファンをオンにするシミュレーションを行うダイレクト メソッドのコードを追加して、デバイス アプリを更新します。次に、このダイレクト メソッドを呼び出すコードをバックエンド サービス アプリに追加します。

直接メソッドを呼び出すバックエンド アプリからの呼び出しには、ペイロードの一部として複数のパラメーターを含めることができます。ダイレクト メソッドは、通常、デバイスの機能をオフまたはオンにしたり、デバイスの設定を指定したりするために使用します。

#### エラー条件を処理する

デバイスがダイレクト メソッドを実行する命令を受信したときにチェックする必要がある、エラー条件がいくつかあります。これらのチェックの 1 つは、ファンが故障状態にある場合にエラーで応答することです。報告するもう 1 つのエラー条件は、無効なパラメーターを受け取った場合です。デバイスの潜在的なリモート性を考えると、エラー報告を明確にすることが重要です。

#### ダイレクト メソッドを呼び出す

ダイレクト メソッドでは、バックエンド アプリがパラメーターを準備し、メソッドを呼び出すための単一のデバイスを指定して呼び出しを行う必要があります。バックエンド アプリは応答を待機し、応答を報告します。

デバイス アプリには、ダイレクト メソッドの機能コードが含まれています。関数名は、デバイスの IoT クライアントに登録されます。このプロセスにより、呼び出しが IoT ハブから取得されたときに実行する機能がクライアントに知られるようになります (多くの直接メソッドが存在する可能性があります)。

#### タスク 1: デバイス アプリでダイレクト メソッドを定義するコードを追加する

1. **cheesecavedevice** アプリを実行している Visual Studio Code インスタンスに戻ります。

    > **注意**: アプリがまだ実行されている場合は、ターミナル ウィンドウに入力フォーカスを置き、**Ctrl + C**を押してアプリを終了します。 

1. **Program.cs** がコード エディターで開かれていることを確認します。

1. コード エディター ペインで、**SimulatedDevice** クラスの下部に移動します。

1. ダイレクト メソッドを定義するには、**SimulatedDevice** クラスの閉じ波括弧 (`}` ) 内に次のコードを追加します。

    ```csharp
    // ダイレクト メソッド呼び出しを処理する
    private static Task<MethodResponse> SetFanState(MethodRequest methodRequest, object userContext)
    {
        if (fanState == stateEnum.failed)
        {
            //  400の成功メッセージで、ダイレクト メソッド 呼び出しを認識します。
            string result = "{\"result\":\"Fan failed\"}";
            redMessage("Direct method failed: " + result);
            return Task.FromResult(new MethodResponse(Encoding.UTF8.GetBytes(result), 400));
        }
        else
        {
            try
            {
                var data = Encoding.UTF8.GetString(methodRequest.Data);

                データから引用符を削除します。
                data = data.Replace("\"", "");

                // ペイロードを解析し、無効な場合は例外をトリガーします。
                fanState = (stateEnum)Enum.Parse(typeof(stateEnum), data);
                greenMessage("Fan set to: " + data);

                // 200の成功メッセージで、ダイレクト メソッド 呼び出しを認識します。
                string result = "{\"result\":\"Executed direct method: " + methodRequest.Name + "\"}";
                return Task.FromResult(new MethodResponse(Encoding.UTF8.GetBytes(result), 200));
            }
            catch
            {
                //  400の成功メッセージで、ダイレクト メソッド 呼び出しを認識します。
                string result = "{\"result\":\"Invalid parameter\"}";
                redMessage("Direct method failed: " + result);
                return Task.FromResult(new MethodResponse(Encoding.UTF8.GetBytes(result), 400));
            }
        }
    }
    ```

    > **注意**:  このコードは、ダイレクト メソッドの実装を定義し、ダイレクト メソッドが呼び出されたときに実行されます。ファンには次の 3 つの状態があります。*オン*、*オフ*、および *失敗*。     上記の方法で、ファンを*オン*または*オフ*に設定します。    ペイロード テキストがこれら 2 つの設定のいずれかと一致しない場合、またはファンが故障状態にある場合は、エラーが返されます。

1. コード エディター ペインで、少し上にスクロールして **Main** メソッドを見つけます。 

1. **Main** メソッド内で、デバイス クライアントを作成したすぐ後にある空白のコード行にカーソルを置きます。

1. ダイレクト メソッドを登録するには、次のコードを追加します。

    ```csharp
    // ダイレクト メソッド呼び出しのハンドラーを作成する
    s_deviceClient.SetMethodHandlerAsync("SetFanState", SetFanState, null).Wait();
    ```

    コードを追加すると、**Main** メソッドは次のようになるはずです。

    ```csharp
    private static void Main(string[] args)
    {
        colorMessage("Cheese Cave device app.\n", ConsoleColor.Yellow);

        // MQTT プロトコルを使用して IoT ハブに接続します。
        s_deviceClient = DeviceClient.CreateFromConnectionString(s_deviceConnectionString, TransportType.Mqtt);

        // ダイレクト メソッド呼び出しのハンドラーを作成する
        s_deviceClient.SetMethodHandlerAsync("SetFanState", SetFanState, null).Wait();

        SendDeviceToCloudMessagesAsync();
        Console.ReadLine();
    }
    ```

1. 「**ファイル**」 メニューで Program.cs ファイルを保存するために、「**保存**」 をクリックします。   

これで、デバイス側で必要なコーディングが完了しました。次に、ダイレクト メソッドを呼び出すバックエンド サービスにコードを追加する必要があります。

#### タスク 2: ダイレクトメソッドを呼び出すコードを追加する

1. **cheesecaveoperator** アプリを実行している Visual Studio Code インスタンスに戻ります。

    > **注意**: アプリがまだ実行されている場合は、ターミナル ウィンドウに入力フォーカスを置き、**Ctrl + C**を押してアプリを終了します。 

1. **Program.cs** がコード エディターで開かれていることを確認します。

1. **ReadDeviceToCloudMessages** クラスの一番上で、グローバル変数のリストに次のコードを追加します。 

    ```csharp
    private static ServiceClient s_serviceClient;
    ```

1. 下にスクロールして **Main** メソッドを見つけます。

1. **Main** メソッドの下の空白のコード行に、次のタスクを追加します。 

    ```csharp
    // ダイレクト メソッドの呼び出しを処理します。
    private static async Task InvokeMethod()
    {
        try
        {
            var methodInvocation = new CloudToDeviceMethod("SetFanState") { ResponseTimeout = TimeSpan.FromSeconds(30) };
            string payload = JsonConvert.SerializeObject("on");

            methodInvocation.SetPayloadJson(payload);

            // ダイレクト メソッドを非同期的に呼び出し、シミュレートされたデバイスから応答を取得します。
            var response = await s_serviceClient.InvokeDeviceMethodAsync("CheeseCaveID", methodInvocation);

            if (response.Status == 200)
            {
                greenMessage("Direct method invoked: " + response.GetPayloadAsJson());
            }
            else
            {
                redMessage("Direct method failed: " + response.GetPayloadAsJson());
            }
        }
        catch
        {
            redMessage("Direct method failed: timed-out");
        }
    }
    ```

    > **注意**: このコードは、 デバイス アプリで **SetFanState** ダイレクト メソッドを呼び出すために使用されます。

1. **Main** メソッド内で、`Create receivers to listen for messages` のコメント行のすぐ上にある、空のコード行にカーソルを置きます。 

1. メッセージをリッスンする受信者を作成するコードの前に、次のコードを追加します。

    ```csharp
    // ハブ上のサービスに接続するエンドポイントと通信する ServiceClient を作成します。
    s_serviceClient = ServiceClient.CreateFromConnectionString(s_serviceConnectionString);
    InvokeMethod().GetAwaiter().GetResult();
    ```

    > **注意**: このコードは、IoT ハブへの接続に使用する ServiceClient オブジェクトを作成します。IoT ハブへの接続により、デバイスでダイレクト メソッドを呼び出すことができます。

1. 「**ファイル**」 メニューで Program.cs ファイルを保存するために、「**保存**」 をクリックします。   

これで、**SetFanState** ダイレクト メソッドをサポートするためのコード変更が完了しました。

#### タスク 3: ダイレクト メソッドをテストする

ダイレクト メソッドをテストするには、正しい順序でアプリを起動する必要があります。登録されていないダイレクト メソッドを呼び出す方法はありません。

1. **cheesecavedevice** デバイス アプリを起動します。 

    端末への書き込みが開始され、テレメトリが表示されます。

1. **cheesecaveoperator** バックエンド アプリを起動します。 

    > **注意**:`Direct method failed: timed-out` というメッセージが表示された場合は、 **cheesecavedevice** に変更を保存し、アプリを再起動していることを再確認してください。

    cheesecaveoperator のバックエンド アプリは、すぐにダイレクト メソッドを呼び出します。

    次のような出力に注目してください。

![コンソール出力](./Media/LAB_AK_15-cheesecave-direct-method-sent.png)

1. これで、 **cheesecavedevice** デバイス アプリのコンソール出力を確認すると、ファンがオンになっていることがわかるはずです。 

![コンソール出力](./Media/LAB_AK_15-cheesecave-direct-method-received.png)

リモート デバイスの監視と制御が正常に行われています。クラウドから呼び出すことができるダイレクト メソッドをデバイスに実装しました。このシナリオでは、ダイレクト メソッドを使用してファンをオンにし、ケーブ内の環境を希望の設定にします。

チーズ蔵環境に対して希望する設定をリモートで指定したい場合はどうしますか? おそらく、熟成プロセスのある時点でチーズ ケーブの特定の目標温度を設定したいと思うでしょう。ダイレクト メソッド (有効なアプローチ) で目的の設定を指定することも、デバイス ツインと呼ばれる IoT ハブの別の機能を使用することもできます。次の演習では、ソリューション内でデバイス ツイン プロパティを実装する作業を行います。

### 演習 5: デバイス ツインのコードを書き込む

この演習では、デバイス アプリとバックエンド サービス アプリの両方に何らかのコードを追加して、操作中のデバイス ツイン同期を表示します。

デバイス ツインには、次の 4 種類の情報が含まれている点に留意してください。

* **タグ**: デバイスに表示されないデバイスの情報。
* **必要なプロパティ**: バックエンド アプリで指定された必要な設定。
* **報告されるプロパティ**: デバイスの設定の報告値。
* **デバイス ID のプロパティ**: デバイスを識別する読み取り専用情報。

IoT Hub で管理されるデバイス ツインはクエリ用に設計されており、実際の IoT デバイスと同期されます。デバイス ツインは、バックエンド アプリでいつでも照会できます。このクエリは、デバイスの現在の状態に関する情報を返すことができます。このデータを取得するには、デバイスとツインが同期するので、デバイスへの呼び出しは必要ありません。デバイス ツインの機能の多くは Azure IoT Hub が提供するため、利用する上でコードを記述する必要はありません。

デバイス ツインとダイレクト メソッドの機能には、いくつかの重複があります。ダイレクトメソッドを使用してデバイスプロパティを設定することができるので、直感的なやり方のように感じられるかもしれません。ただし、ダイレクト メソッドを使用するには、アクセスする必要がある場合は、バックエンド アプリがこれらの設定を明示的に記録する必要があります。デバイス ツインを使用すると、この情報は既定で保存および管理されます。

#### タスク 1: デバイス ツインを使用してデバイス のプロパティを同期するためのコードを追加する

1. **cheesecaveoperator** バックエンド アプリを実行している Visual Studio Code インスタンスに戻ります。

1. アプリがまだ実行されている場合は、端末に入力フォーカスを置き、**Ctrl + C** を押してアプリを終了します。 

1. **Program.cs** が開いていることを確認します。 

1. コード エディター ペインで、 **ReadDeviceToCloudMessages** クラスの下部に移動します。 

1. **ReadDeviceToCloudMessages** クラスの閉じ波括弧のすぐ上に、次のコードを追加します。 

    ```csharp
    // デバイス ツイン セクション。
    private static RegistryManager registryManager;

    private static async Task SetTwinProperties()
    {
        var twin = await registryManager.GetTwinAsync("CheeseCaveID");
        var patch =
            @"{
                tags: {
                    customerID: 'Customer1',
                    cellar: 'Cellar1'
                },
                properties: {
                    desired: {
                        patchId: 'set values',
                        temperature: '50',
                        humidity: '85'
                    }
                }
        }";
        await registryManager.UpdateTwinAsync(twin.DeviceId, patch, twin.ETag);

        var query = registryManager.CreateQuery(
          "SELECT * FROM devices WHERE tags.cellar = 'Cellar1'", 100);
        var twinsInCellar1 = await query.GetNextAsTwinAsync();
        Console.WriteLine("Devices in Cellar1: {0}",
          string.Join(", ", twinsInCellar1.Select(t => t.DeviceId)));

    }
    ```

    > **注意**:   **SetTwinProperties** メソッドは、デバイス ツインに追加されるタグとプロパティを定義する JSON の一部を作成し、ツインを更新します。  メソッドの次の部分では、**celler** タグが "Cellar1" に設定されているデバイスをリストするために、クエリをどのように実行できるかを示します。このクエリでは、接続に**レジストリ読み取り**アクセス許可が必要です。

1. コード  エディター ペインで、上にスクロールして **Main** メソッドを見つけます。 

1. **Main** メソッドで、サービス クライアントを作成するコード行を見つけます。

1. サービス クライアントを作成するコードの前に、次のコードを追加します。

    ```csharp
    // レジストリマネージャは、デジタル ツインへのアクセスに用います。
    registryManager = RegistryManager.CreateFromConnectionString(s_serviceConnectionString);
    SetTwinProperties().Wait();
    ```

    > **注意**: Main メソッドに含まれているコメントを読みます。

1. 「**ファイル**」 メニューで Program.cs ファイルを保存するために、「**保存**」 をクリックします。   

#### タスク 2: デバイスのデバイス ツイン設定を同期するためのコードを追加する

1. **cheesecavedevice** アプリを実行している Visual Studio Code インスタンスに戻ります。

1. アプリがまだ実行されている場合は、端末に入力フォーカスを置き、**Ctrl + C** を押してアプリを終了します。 

1. **Program.cs** ファイルが [コード エディター] ウィンドウで開かれていることを確認します。

1. [コード エディター] ウィンドウで下にスクロールし、**SimulatedDevice** クラスの末尾を見つけます。

1. **SimulatedDevice** クラスの右中括弧内に次のコードを追加します。

    ```csharp
    private static async Task OnDesiredPropertyChanged(TwinCollection desiredProperties, object userContext)
    {
        try
        {
            desiredHumidity = desiredProperties["humidity"];
            desiredTemperature = desiredProperties["temperature"];
            greenMessage("Setting desired humidity to " + desiredProperties["humidity"]);
            greenMessage("Setting desired temperature to " + desiredProperties["temperature"]);

            // IoT ハブにプロパティを報告します。
            var reportedProperties = new TwinCollection();
            reportedProperties["fanstate"] = fanState.ToString();
            reportedProperties["humidity"] = desiredHumidity;
            reportedProperties["temperature"] = desiredTemperature;
            await s_deviceClient.UpdateReportedPropertiesAsync(reportedProperties);

            greenMessage("\nTwin state reported: " + reportedProperties.ToJson());
        }
        catch
        {
            redMessage("Failed to update device twin");
        }
    }
    ```

    > **注意**: このコードは、デバイス ツインで必要なプロパティが変更されたときに呼び出されるハンドラーを定義します。変更を確認するために、新しい値が IoT ハブに報告されることに注意してください。

1. コード エディター ペインで、**Main** メソッドまで上にスクロールします。 

1. **Main** メソッド内で、ダイレクト メソッドのハンドラーを作成するコードを見つけます。

1. ダイレクト メソッドのハンドラーの下の空白行にカーソルを置きます。

1. 必要なプロパティ変更ハンドラーを登録するために、次のコードを追加します。

    ```csharp
    // デバイス ツインを取得して、最初に必要なプロパティを報告します。
    Twin deviceTwin = s_deviceClient.GetTwinAsync().GetAwaiter().GetResult();
    greenMessage("Initial twin desired properties: " + deviceTwin.Properties.Desired.ToJson());

    // デバイス ツインの更新コールバックを設定します。
    s_deviceClient.SetDesiredPropertyUpdateCallbackAsync(OnDesiredPropertyChanged, null).Wait();
    ```

1. 「**ファイル**」 メニューで Program.cs ファイルを保存するために、「**保存**」 をクリックします。   

    > **注意**:  これで、デバイス ツインのサポートがアプリに追加されました。**desiredHumidity** のように明示的な変数を使うことについて再考できます。その代わり、デバイス ツイン オブジェクトの変数を使用できます。

#### タスク 3: デバイス ツインのテスト

メソッドをテストするには、正しい順序でアプリを起動します。

1. **cheesecavedevice** デバイス アプリを起動します。  端末への書き込みが開始され、テレメトリが表示されます。

1. **cheesecaveoperator** バックエンド アプリを起動します。 

1. **cheesecavedevice** デバイス アプリのコンソール出力を検査し、デバイス ツインが正しく同期されていることを確認します。

![コンソール出力](./Media/LAB_AK_15-cheesecave-device-twin-received.png)

    ファンを動作させる場合、最終的に赤いアラートを取り除く必要があります!

![コンソール出力](./Media/LAB_AK_15-cheesecave-device-twin-success.png)

1. Visual Studio Code の両方のインスタンスに対して、アプリを停止してから、Visual Studio Code ウィンドウを閉じます。

このモジュールで提供されるコードは工業品質ではありません。ダイレクト メソッドとデバイス ツインの使用方法が示されています。ただし、メッセージは、バックエンド サービス アプリが最初に実行されたときにのみ送信されます。通常、バックエンド サービス アプリでは、必要に応じてオペレーターがダイレクト メソッドを送信したり、デバイス ツイン プロパティを設定したりするために、ブラウザー インターフェイスが必要になります。
