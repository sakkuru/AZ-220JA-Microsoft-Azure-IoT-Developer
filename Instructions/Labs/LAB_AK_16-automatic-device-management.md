---
lab:
    title: '課題 16: Azure IoT Hub を使用した IoT デバイス管理の自動化'
    module: 'モジュール 8: デバイス管理'
---

# Azure IoT Hub を使用した IoT デバイス管理の自動化

IoT デバイスは、最適化されたオペレーティング システムを使用したり、シリコン上でコードを直接実行したりします (実際のオペレーティング システムは必要ありません)。このようなデバイスで実行されているソフトウェアを更新するために最も一般的な方法は、OSだけでなく実行されているアプリ (ファームウェアと呼ばれる) を含み、ソフトウェア パッケージ全体の新しいバージョンをフラッシュすることです。

各デバイスには特定の目的があるため、ファームウェアは非常に限定的で、デバイスの目的だけでなく、利用可能な制約されたリソースにも最適化されています。

ファームウェアの更新プロセスは、ハードウェアとハードウェアの製造元がボードを作成した方法に対して固有のプロセスにすることもできます。これは、ファームウェアの更新プロセスの一部が汎用的ではないことを意味し、ファームウェアの更新プロセスの詳細を取得するためにデバイスの製造元と協力する必要があります (ファームウェアの更新プロセスを知っている可能性が高い独自のハードウェアを開発している場合を除きます)。

ファームウェアの更新が個々のデバイスに手動で適用される場合、この方法では一般的な IoT ソリューションで使用されるデバイスの数を考慮することは意味がありません。ファームウェアの更新プログラムは、クラウドからリモートで管理される新しいファームウェアの展開により、より一般的に無線 (OTA) で行われるようになりました。

IoT デバイスのすべての無線ファームウェア更新プログラムに共通の分母のセットがあります。

1. ファームウェアのバージョンは一意に識別されます
1. ファームウェアは、デバイスがオンライン ソースから取得する必要があるバイナリ ファイル形式で提供されます
1. ローカルに保存されているファームウェアは、何らかの形の物理ストレージです (ROM メモリ、ハードドライブなど)
1. デバイスの製造元は、ファームウェアを更新するために必要なデバイスの操作の説明を提供します。

Azure IoT Hub は、単一のデバイスとデバイスのコレクションにデバイス管理操作を実装するための高度なサポートを提供します。[自動デバイス管理](https://docs.microsoft.com/azure/iot-hub/iot-hub-auto-device-config)機能を使用すると、一連の操作の構成、その操作のトリガー、進行状況の監視を簡単に行えます。

## ラボシナリオ

Contoso 社のチーズ熟成庫に実装した自動空気処理システムは、同社が既に高い品質水準をさらに上げるのに役立ちました。同社は、チーズでこれまで以上に多くの賞を受賞しています。

基本ソリューションは、センサーと気候制御システムと統合された IoT デバイスで構成されており、マルチチャンバー貯蔵庫システム内の温度と湿度をリアルタイムで制御します。また、ダイレクト メソッドとデバイス ツイン プロパティの両方を使用してデバイスを管理する機能を示す、シンプルなバックエンド アプリを開発しました。

Contoso 社は、初期ソリューションからシンプルなバックエンド アプリを拡張し、オペレーターが蔵環境の監視とリモート管理に使用できるオンライン ポータルを含めています。新しいポータルでは、オペレーターはチーズの種類に基づいて、またはチーズの熟成プロセス内の特定の段階に合わせて蔵内の温度と湿度をカスタマイズすることさえできます。蔵内の各チャンバーまたはゾーンは、別々に制御することができます。

IT 部署は、オペレータ向けに開発したバックエンド ポータルを保守しますが、管理者はソリューションのデバイス側を管理することに同意しています。 

この場合、これは次の 2 つのことを意味します。 

1. Contoso 社の運用チームは、改善方法を常に模索しています。これらの機能強化の結果、デバイス ソフトウェアの新機能のリクエストが寄せられることがよくあります。 

1. 洞窟の場所にデプロイされている IoT デバイスには、プライバシーを確保しハッカーがシステムを掌握するのを防ぐために、最新のセキュリティパッチが必要です。システムのセキュリティを維持するため、デバイスのファームウェアをリモートで更新して、デバイスを最新の状態に保つ必要があります。

自動デバイス管理とデバイス管理を大規模に行えるようにする IoT Hub の機能を実装する予定です。

次のリソースが作成されます。

![課題 16 アーキテクチャ](media/LAB_AK_16-architecture.png)

## この課題では

このラボでは、次のタスクを正常に達成します。

* ラボの前提条件を確認する
* ファームウェアの更新を実装するシミュレートされたデバイスのコードを記述します
* Azure IoT Hub の自動デバイス管理を使用して、1 つのデバイスでファームウェアの更新プロセスをテストする

## ラボの手順

### 演習 1: ラボの前提条件を確認する

このラボでは、次の Azure リソースが利用可能であることを前提としています。

| リソースの種類:  | リソース名 |
| :-- | :-- |
| リソース グループ | rg-az220 |
| IoT Hub | iot-az220-training-{your-id} |
| IoT デバイス | sensor-th-0155 |

これらのリソースが利用できない場合は、演習 2 に進む前に、以下の手順に従って**lab16-setup.azcli** スクリプトを実行する必要があります。スクリプト ファイルは、開発環境構成 (ラボ 3) の一部としてローカルに複製した GitHub リポジトリに含まれています。

>**注:** **sensor-th-0155**デバイスの接続文字列が必要です。このデバイスが Azure IoT Hub に登録されている場合は、Azure Cloud Shell で次のコマンドを実行して接続文字列を取得できます
>
> ```bash
> az iot hub device-identity show-connection-string --hub-name iot-az220-training-{your-id} --device-id SimulatedThermostat -o tsv
> ```

**lab16-setup.azcli** のスクリプトは、**bash**シェル環境内で実行するように記述されています - これを実行する最も簡単な方法は Azure Cloud Shell 内です。 

1. ブラウザーを使用して [Azure Shell](https://shell.azure.com/)を開き、このコースで使用している Azure サブスクリプションでログインします。

    Cloud Shell のストレージの設定に関するメッセージが表示された場合は、デフォルトをそのまま使用します。

1. Azure Cloud Shell が **Bash**を使用していることを確認 します。

    「Azure Cloud Shell」 ページの左上隅にあるドロップダウンは、環境を選択するために使用されます。選択されたドロップダウンの値が **Bash**であることを確認します。 

1. Azure Shell ツール バーで、「**ファイルのアップロード/ダウンロード**」 をクリックします(右から 4番目のボタン)。

1. ドロップダウンで、「**アップロード**」 をクリックします。

1. ファイル選択ダイアログで、開発環境を構成したときにダウンロードした GitHub ラボ ファイルのフォルダーの場所に移動します。

    _ラボ 3: 開発環境のセットアップ_:ZIP ファイルをダウンロードしてコンテンツをローカルに抽出することで、ラボ リソースを含む GitHub リポジトリを複製しました。抽出されたフォルダー構造には、次のフォルダー パスが含まれます。

    * Allfiles
      * Labs
          * 16-Automate IoT Device Management with Azure IoT Hub
            * Setup

    lab16-setup.azcli スクリプト ファイルは、課題 16 のセットアップ フォルダにあります。

1. **lab16-setup.azcli** ファイルを選択し、 **「開く」** をクリックします。   

    ファイルのアップロードが完了すると、通知が表示されます。

1. 正しいファイルが Azure Cloud Shell にアップロードされたことを確認するには、次のコマンドを入力します。

    ```bash
    ls
    ```

    `ls` コマンドを使用して、現在のディレクトリの内容を表示します。lab16-setup.azcli ファイルが一覧表示されます。

1. セットアップ スクリプトを含むこのラボのディレクトリを作成し、そのディレクトリに移動するには、次の Bash コマンドを入力します。

    ```bash
    mkdir lab16
    mv lab16-setup.azcli lab16
    cd lab16
    ```

1. **lab16-setup.azcli** に実行権限があることを確認するには、次のコマンドを入力します。 

    ```bash
    chmod +x lab16-setup.azcli
    ```

1. Cloud Shell ツールバーで、lab16-setup.azcli ファイルを編集するには、**「エディタを開く」** (右から 2 番目のボタン - { } ) をクリックします。 

1. **「ファイル」** の一覧で、課題16 フォルダを展開してスクリプト ファイルを開くには、**「lab16」** をクリックしてから、**「lab16-setup.azcli」** をクリックします。     

    エディタは、**lab16-setup.azcli**ファイルの内容を表示します。 

1. エディタで、割り当て済みの値 `{YOUR-ID}` と `{YOUR-LOCATION}` を更新します。

    以下の参考サンプルでは、このコースの開始時に作成した固有 ID に **(CAH191211)** などの `{YOUR-ID}` を設定し、`{YOUR-LOCATION}` をリソースにとって意味のある場所に設定する必要があります。

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

    > **注意**:  **CTRL+S**を使っていつでも保存でき、 **CTRL+Q**を押してエディターを閉じます。

1. このラボに必要なリソースを作成するには、次のコマンドを入力します。

    ```bash
    ./lab16-setup.azcli
    ```

    このスクリプトの実行には数分かかります。各ステップが完了すると、JSON 出力が表示されます。

    このスクリプトは、まず **rg-az220** という名前のリソース グループ と **AZ-220-ハブ-{YourID}** という名前の IoT ハブを作成します。  既に存在する場合は、対応するメッセージが表示されます。次にスクリプトは、**sensor-th-0155** の ID を持つデバイスを IoT ハブに追加し、デバイスの接続文字列を表示します。 

1. スクリプトが完了すると、デバイスの接続文字列が表示されることに注意してください。

    接続文字列は「ホスト名=」で始まります。

1. 接続文字列をテキスト ドキュメントにコピーし、**sensor-th-0155** デバイス用であることに注意してください。

    接続文字列を簡単に見つけることができる場所に保存したら、課題を続ける準備が整います。

### エクササイズ 2: ファームウェアの更新を実装するデバイスをシミュレートするコードを記述します。

この演習では、デバイス ツインの必要なプロパティの変更を管理し、ファームウェアの更新をシミュレートするローカル プロセスをトリガーする簡単なシミュレーターを作成します。実際のデバイスでは、ローカル ファームウェアの更新の実際の手順を除いて、全体のプロセスはまったく同じです。その後、Azure ポータルを使用して、1 つのデバイスのファームウェア更新プログラムを構成および実行します。IoT Hub は、デバイス ツインのプロパティを使用して、構成変更要求をデバイスに転送し、進行状況を監視します。

#### タスク 1: デバイス シミュレーター アプリを作成する

このタスクでは、Visual Studio Code を使用して、新しいコンソール アプリを作成します。

1. Visual Studio Code を起動します。

    このコースの課題 3 を修了すると、開発環境に [.NET Core](https://dotnet.microsoft.com/download) と [C# 拡張機能](https://marketplace.visualstudio.com/items?itemName=ms-vscode.csharp)がインストールされているはずです。

1. 「**ターミナル**」 メニューで、「**新しいターミナル**」 をクリックします。

1. ターミナル コマンド プロンプトで、次のコマンドを入力します。

    ```cmd/sh
    mkdir fwupdatedevice
    cd fwupdatedevice
    ```

    最初のコマンドは、 **fwupdatedevice** と呼ばれるフォルダを作成します。2 番目のコマンドで、 **fwupdatedevice**フォルダーに移動します。 

1. 新しいコンソール アプリを作成するには、次のコマンドを入力します。

    ```cmd/sh
    dotnet new console
    ```

    > **注意**: 新しい .NET コンソール アプリが作成されると、`dotnet restore` は作成後のプロセスとして実行されている必要があります。「ターミナル」 ペインに、この問題が発生したことを示すメッセージが表示されない場合、アプリは必要な .NET パッケージにアクセスできない場合があります。これに対処するためには、次のコマンドを入力します。`dotnet restore`

1. アプリに必要なライブラリをインストールするには、次のコマンドを入力します。

    ```cmd/sh
    dotnet add package Microsoft.Azure.Devices.Client
    dotnet add package Microsoft.Azure.Devices.Shared
    dotnet add package Newtonsoft.Json
    ```

    「ターミナル」 ウィンドウでメッセージを確認し、3 つのライブラリがすべてインストールされていることを確認します。

1. **「ファイル」** メニューで、**「フォルダを開く」** を選択します。

1. **「フォルダを開く」** ダイアログで、「ターミナル」 ウィンドウで指定したフォルダの場所に移動し、 **「fwupdatedevice」**をクリックしてから、**「フォルダの選択」** をクリックします。   

    「EXPLORER」 ウィンドウが Visual Studio Code で開き、`Program.cs` と `fwupdatedevice.csproj` ファイルが一覧表示されます。

1. **EXPLORER** ウインドウで、**EventsController.cs** をクリックします。

1. 「コード エディター」 ウィンドウで、Program.cs ファイルの内容を削除します。

#### タスク 2: アプリにコードを追加する

このタスクでは、IoT Hub で生成された要求に応答して、デバイスのファームウェア更新プログラムをシミュレートするためのコードを入力します。

1. **Program.cs**ファイルが Visual Studio Code で開かれていることを確認します。 

    「コード エディター」 ウインドウには、空のコード ファイルが表示されます。

1. 次のコードをコピーして Program.cs ファイルに貼り付けます。

    ```cs
    // Copyright (c) Microsoft. All rights reserved.
    // Licensed under the MIT license. See LICENSE file in the project root for full license information.

    using Microsoft.Azure.Devices.Shared;
    using Microsoft.Azure.Devices.Client;
    using System;
    using System.Threading.Tasks;

    namespace fwupdatedevice
    {
        class SimulatedDevice
        {
            // The device connection string to authenticate the device with your IoT hub.
            static string s_deviceConnectionString = "";

            // Device ID variable
            static string DeviceID="unknown";

            // Firmware version variable
            static string DeviceFWVersion = "1.0.0";

            // Simple console log function
            static void LogToConsole(string text)
            {
                // we prefix the logs with the device ID
                Console.WriteLine(DeviceID + ": " + text);
            }

            // Function to retrieve firmware version from the OS/HW
            static string GetFirmwareVersion()
            {
                // In here you would get the actual firmware version from the hardware. For the simulation purposes we will just send back the FWVersion variable value
                return DeviceFWVersion;
            }

            // Function for updating a device twin reported property to report on the current Firmware (update) status
            // Here are the values expected in the "firmware" update property by the firmware update configuration in IoT Hub
            //  currentFwVersion: The firmware version currently running on the device.
            //  pendingFwVersion: The next version to update to, should match what's
            //                    specified in the desired properties. Blank if there
            //                    is no pending update (fwUpdateStatus is 'current').
            //  fwUpdateStatus:   Defines the progress of the update so that it can be
            //                    categorized from a summary view. One of:
            //         - current:     There is no pending firmware update. currentFwVersion should
            //                    match fwVersion from desired properties.
            //         - downloading: Firmware update image is downloading.
            //         - verifying:   Verifying image file checksum and any other validations.
            //         - applying:    Update to the new image file is in progress.
            //         - rebooting:   Device is rebooting as part of update process.
            //         - error:       An error occurred during the update process. Additional details
            //                    should be specified in fwUpdateSubstatus.
            //         - rolledback:  Update rolled back to the previous version due to an error.
            //  fwUpdateSubstatus: Any additional detail for the fwUpdateStatus . May include
            //                     reasons for error or rollback states, or download %.
            //
            // reported: {
            //       firmware: {
            //         currentFwVersion: '1.0.0',
            //         pendingFwVersion: '',
            //         fwUpdateStatus: 'current',
            //         fwUpdateSubstatus: '',
            //         lastFwUpdateStartTime: '',
            //         lastFwUpdateEndTime: ''
            //   }
            // }

            static async Task UpdateFWUpdateStatus(DeviceClient client, string currentFwVersion, string pendingFwVersion, string fwUpdateStatus, string fwUpdateSubstatus, string lastFwUpdateStartTime, string lastFwUpdateEndTime)
            {
                TwinCollection properties = new TwinCollection();
                if (currentFwVersion!=null)
                    properties["currentFwVersion"] = currentFwVersion;
                if (pendingFwVersion!=null)
                    properties["pendingFwVersion"] = pendingFwVersion;
                if (fwUpdateStatus!=null)
                    properties["fwUpdateStatus"] = fwUpdateStatus;
                if (fwUpdateSubstatus!=null)
                    properties["fwUpdateSubstatus"] = fwUpdateSubstatus;
                if (lastFwUpdateStartTime!=null)
                    properties["lastFwUpdateStartTime"] = lastFwUpdateStartTime;
                if (lastFwUpdateEndTime!=null)
                    properties["lastFwUpdateEndTime"] = lastFwUpdateEndTime;

                TwinCollection reportedProperties = new TwinCollection();
                reportedProperties["firmware"] = properties;

                await client.UpdateReportedPropertiesAsync(reportedProperties).ConfigureAwait(false);
            }

            // Execute firmware update on the device
            static async Task UpdateFirmware(DeviceClient client, string fwVersion, string fwPackageURI, string fwPackageCheckValue)
            {
                LogToConsole("A firmware update was requested from version " + GetFirmwareVersion() + " to version " + fwVersion);
                await UpdateFWUpdateStatus(client, null, fwVersion, null, null, DateTime.UtcNow.ToString("yyyy-MM-ddTHH:mm:ssZ"), null);

                // Get new firmware binary. Here you would download the binary or retrieve it from the source as instructed for your device, then double check with a hash the integrity of the binary you downloaded
                LogToConsole("Downloading new firmware package from " + fwPackageURI);
                await UpdateFWUpdateStatus(client, null, null, "downloading", "0", null, null);
                await Task.Delay(2 * 1000);
                await UpdateFWUpdateStatus(client, null, null, "downloading", "25", null, null);
                await Task.Delay(2 * 1000);
                await UpdateFWUpdateStatus(client, null, null, "downloading", "50", null, null);
                await Task.Delay(2 * 1000);
                await UpdateFWUpdateStatus(client, null, null, "downloading", "75", null, null);
                await Task.Delay(2 * 1000);
                await UpdateFWUpdateStatus(client, null, null, "downloading", "100", null, null);
                // report the binary has been downloaded
                LogToConsole("The new firmware package has been successfully downloaded.");

                // Check binary integrity
                LogToConsole("Verifying firmware package with checksum " + fwPackageCheckValue);
                await UpdateFWUpdateStatus(client, null, null, "verifying", null, null, null);
                await Task.Delay(5 * 1000);
                // report the binary has been downloaded
                LogToConsole("The new firmware binary package has been successfully verified");

                // Apply new firmware
                LogToConsole("Applying new firmware");
                await UpdateFWUpdateStatus(client, null, null, "applying", null, null, null);
                await Task.Delay(5 * 1000);

                // On a real device you would reboot at the end of the process and the device at boot time would report the actual firmware version, which if successful should be the new version.
                // For the sake of the simulation, we will simply wait some time and report the new firmware version
                LogToConsole("Rebooting");
                await UpdateFWUpdateStatus(client, null, null, "rebooting", null, null, DateTime.UtcNow.ToString("yyyy-MM-ddTHH:mm:ssZ"));
                await Task.Delay(5 * 1000);

                // On a real device you would issue a command to reboot the device. Here we are simply running the init function
                DeviceFWVersion = fwVersion;
                await InitDevice(client);

            }

            // Callback for responding to desired property changes
            static async Task OnDesiredPropertyChanged(TwinCollection desiredProperties, object userContext)
            {
                LogToConsole("Desired property changed:");
                LogToConsole($"{desiredProperties.ToJson()}");

                // Execute firmware update
                if (desiredProperties.Contains("firmware") && (desiredProperties["firmware"]!=null))
                {
                    // In the desired properties, we will find the following information:
                    // fwVersion: the version number of the new firmware to flash
                    // fwPackageURI: URI from where to download the new firmware binary
                    // fwPackageCheckValue: Hash for validating the integrity of the binary downloaded
                    // We will assume the version of the firmware is a new one
                    TwinCollection fwProperties = new TwinCollection(desiredProperties["firmware"].ToString());
                    await UpdateFirmware((DeviceClient)userContext, fwProperties["fwVersion"].ToString(), fwProperties["fwPackageURI"].ToString(), fwProperties["fwPackageCheckValue"].ToString());

                }
            }

            static async Task InitDevice(DeviceClient client)
            {
                LogToConsole("Device booted");
                LogToConsole("Current firmware version: " + GetFirmwareVersion());
                await UpdateFWUpdateStatus(client, GetFirmwareVersion(), "", "current", "", "", "");
            }

            static async Task Main(string[] args)
            {
                // Get the device connection string from the command line
                if (string.IsNullOrEmpty(s_deviceConnectionString) && args.Length > 0)
                {
                    s_deviceConnectionString = args[0];
                } else
                {
                    Console.WriteLine("Please enter the connection string as argument.");
                    return;
                }

                DeviceClient deviceClient = DeviceClient.CreateFromConnectionString(s_deviceConnectionString, TransportType.Mqtt);

                if (deviceClient == null)
                {
                    Console.WriteLine("Failed to create DeviceClient!");
                    return;
                }

                // Get the device ID
                string[] elements = s_deviceConnectionString.Split('=',';');

                for(int i=0;i<elements.Length; i+=2)
                {
                    if (elements[i]=="DeviceId") DeviceID = elements[i+1];
                }

                // Run device init routine
                await InitDevice(deviceClient);

                // Attach callback for Desired Properties changes
                await deviceClient.SetDesiredPropertyUpdateCallbackAsync(OnDesiredPropertyChanged, deviceClient).ConfigureAwait(false);

                // Wait for keystroke to end app
                // TODO
                while (true)
                {
                    Console.ReadLine();
                    return;
                }
            }
        }
    }
    ```

    > **注意**: 
    > コード内のコメントを読み、デバイス ツインの変更に対してデバイスがどのように反応するかに関して、目的のプロパティ 「ファームウェア」で共有される構成に基づいてファームウェアの更新を実行します。また、デバイス ツインの報告されたプロパティを使用して、現在のファームウェアの更新の状態を報告する関数をメモすることもできます。

1. 「**ファイル**」 メニューの 「**上書き保存**」 をクリックします。   

これで、デバイス側のコードが完成しました。次に、このシミュレートされたデバイスでファームウェアの更新プロセスが期待どおりに動作することをテストします。

### エクササイズ 3: 単一のデバイスでファームウェアの更新をテストする

この演習では、Azure portal を使用して新しいデバイス管理構成を作成し、それを単一のシミュレートされたデバイスに適用します。

#### タスク 1: デバイス シミュレーターを起動する

1. 必要に応じて **fwupdatedevice** プロジェクトを Visual Studio Code で開きます。 

1. 「ターミナル」 ウィンドウが開いていることを確認します。

    コマンド プロンプトのフォルダーの場所は、`fwupdatedevice` フォルダーです。

1. `fwupdatedevice` アプリを実行するには、次のコマンドを入力します。

    ``` bash
    dotnet run "<device connection string>"
    ```

    > **注意**: プレースホルダーを実際のデバイス接続文字列に置き換え、接続文字列の周囲に "" を必ず含めることに注意してください。 
    > 
    > 例: `"HostName=AZ-220-HUB-{YourID}.azure-devices.net;DeviceId=sensor-th-0155;SharedAccessKey={}="`

1. 「ターミナル」 ウインドウの内容を確認します。

    ターミナルに次の出力が表示されます ("mydevice" はデバイス ID の作成時に使用したデバイス ID です)。

    ``` bash
        mydevice: Device booted
        mydevice: Current firmware version: 1.0.0
    ```

#### タスク 2: デバイス管理構成の作成

1. 必要に応じて、Azure アカウントの資格情報を使用して [Azure Portal](https://portal.azure.com/learn.docs.microsoft.com?azure-portal=true) にログインします。 

    複数の Azure アカウントをお持ちの場合は、このコースで使用するサブスクリプションに関連付けられているアカウントでログインしていることを確認してください。

1. Azure Portal ダッシュボードで、**「iot-az220-training-{your-id}」**」 をクリックします。

    IoT Hub ブレードが表示されるようになりました。
 
1. 左側のナビゲーション メニューの **「自動デバイス管理」** で 、**「IoT デバイスの構成」** をクリックします。 

1. **「IoT デバイスの構成」** ウィンドウで、**「デバイス構成の追加」** をクリックします。   

1. **「デバイス ツイン構成の作成」** ブレードの **「Name」** で、**firmwareupdate** と入力します。   

    **「ラベル」** の下ではなく、構成に必要な **「名前」** フィールドに `firmwareupdate` と入力していることを確認 します。    

1. ブレードの最下部で、「**Next: Twins Settings >**」 をクリックします。

1. **「デバイス ツインの設定」** で、**「Device Twin Property」** フィールドに **properties.desired.firmware** と入力します

1. **「Device Twin Property Content」** フィールドに、次のように入力します。 

    ``` json
    {
        "fwVersion":"1.0.1",
        "fwPackageURI":"https://MyPackage.uri",
        "fwPackageCheckValue":"1234"
    }
    ```

1. ブレードの最下部で、「**Next: Metrics >**」 をクリックします。

    カスタム メトリックを使用して、ファームウェアの更新が有効であったかどうかを追跡します。 

1. **「Metrics」** タブの **「METRIC NAME」** で **fwupdated** と入力します

1. **「METRIC CRITERIA」** で、次の項目を入力します。 

    ``` SQL
    SELECT deviceId FROM devices
        WHERE properties.reported.firmware.currentFwVersion='1.0.1'
    ```

1. ブレードの最下部で、「**Next: Target devices >**」 をクリックします。

1. **「Target Devices」** タブの **「Priority」**で、 **「10」** と入力します。       

1. **「ターゲット条件」** の **「ターゲット条件」** フィールドに、次のクエリを入力します。   

    ``` SQL
    deviceId='<your device id>'
    ```

    > **注意**: `'<your device id>'` をデバイスの作成に使用したデバイス ID に置き換えてください。例: `'sensor-th-0155'`

1. ブレードの最下部で、「**Next: Review + Create >**」 をクリックします。

    **「Review + create」** タブが開くと、新しい構成の 「Validation passed.」というメッセージが表示されます。  

1. **「Review + create」** タブで、「Validation passed.」というメッセージが表示されたら、**「Create」** をクリックします。   

    「Validation passed.」というメッセージが表示された場合は、設定を作成する前に、作業を確認する必要があります。

1. 「**IoT デバイスの構成**」 ペインの 「**構成名**」 に、新しい **firmwareupdat** の構成が一覧表示されていることを確認します。       

    新しい構成が作成されると、IoT ハブは構成のターゲット デバイスの条件に一致するデバイスを探し、ファームウェアの更新構成を自動的に適用します。

1. 「Visual Studio Code」 ウィンドウに切り替え、ターミナル ペインの内容を確認します。

    「ターミナル」 ペインには、トリガーされたファームウェア更新プロセスの進行状況を一覧表示するアプリによって生成された新しい出力が含まれている必要があります。

1. シミュレートされたアプリを停止し、Visual Studio Code を閉じます。

    ターミナルの「Enter」キーを押すだけで、デバイス シミュレーターを停止できます。

