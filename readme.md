﻿# AZ-220: Microsoft Azure IoT Developer

- **[最新の受講生用ハンドブックとコンテンツをダウンロードしてください](../../releases/latest)**
- **あなたは MCT ですか。** - [MCT 向け GitHub ユーザー ガイド](https://microsoftlearning.github.io/MCT-User-Guide-JA/)をご覧ください
- **ラボの手順を手動でビルドする必要がありますか。** - 手順は、[MicrosoftLearning/Docker-Build](https://github.com/MicrosoftLearning/Docker-Build) リポジトリで利用可能です
- **[ラボ VM にインストールされているもの。](lab.md)**

## 演習内容

- このコースをサポートするために、コースで使用される Azure サービスを最新の状態に保つために、コース コンテンツを頻繁に更新する必要があります。 コース作成者と MCT の間のオープンな貢献を可能にし、Azure プラットフォームの変更に伴ってコンテンツを最新の状態に保つため、GitHub でラボの手順とラボ ファイルを公開しています。

- これにより、これまでになかったようなコラボレーションの感覚がラボにもたらされることと思います。Azure が変更され、ライブ配信中に最初に見つかった場合は、ラボ ソースですぐに拡張を行ってください。 仲間の MCT を支援しましょう。

## リリースされた MOC ファイルに関連してこれらのファイルを使用するには

- インストラクター ハンドブックと PowerPoint は、コース コンテンツを教えるための主要なソースになります。

- GitHub のこれらのファイルは、受講生用ハンドブックと組み合わせて使用するように設計されていますが、MCT とコース作成者が最新のラボファイルの共有ソースを有するができるように、中央リポジトリとして GitHub に用意されています。

- すべての配信について、トレーナーは最新の Azure サービスをサポートするために行われた変更がないか GitHub を確認し、配信用の最新ファイルを取得することをお勧めします。

## 受講生用ハンドブックの変更について

- 受講生用ハンドブックは四半期ごとに確認し、必要に応じて通常の MOC リリース チャンネルを通じて更新します。

## リクエストやフィードバックについて

- コースラボの更新は開発中です。いくつかのマイナーな変更が実装されており、さらにいくつかのグローバル アップデートが開発されています。プル要求を作成する前に、共同作成者が問題を提起して変更を提案することをお勧めします。

- どの MCT も GitHub repro のコードまたはコンテンツに (マスター ブランチに対して) プル要求を送信でき、Microsoft とコースの作成者は、必要に応じてコンテンツとラボ コードの変更をトリアージして含めます。ラボでブロッキングの問題が発生した場合は、すぐに問題を提起してください。

- バグ、変更、改善、アイデアを提出できます。新しい Azure 機能を見つけましたか？新しいデモを提出しましょう。

## これらのラボを自習に使用するにはどうすればよいですか。

これらのラボは、講師主導の AZ-220 コースのコンパニオンとして作成されています。 ラボが教育ツールとなるように各ステップの説明が含まれるように多大な努力を行っていますが、正常に完了するためのすべての要件が文書化されているわけではありません。 たとえば、教室環境では、受講者は比較的制限のない Azure サブスクリプションにアクセスできます。その結果、ラボで使用されるリソース プロバイダーの完全なリストは存在しません。 これは、ロックダウンされたサブスクリプション (たとえば、[Azure ポリシー](https://docs.microsoft.com/azure/governance/policy/overview)の制限があるサブスクリプション) のラボ ステップが失敗する可能性があることを意味します。 これは、講師主導の環境以外では、1 対 1 のラボ サポートを利用できないことも意味します。1 対 1 のラボ サポートを要求する問題を開かないでください。

Microsoft Learn は、サンドボックス化されたラボ環境を含む IoT の[ラーニング パス](https://docs.microsoft.com/ja-jp/learn/browse/?resource_type=learning%20path&products=azure-iot&roles=developer)をいくつか提供します。 これらは、IoT の自習の機会を探している多くの受講者により適しています。

## メモ

### 教材

MCT およびパートナーがこれらの資料にアクセスし、順番に受講生に個別に提供することを強くお勧めします。 受講生が GitHub に直接アクセスして、進行中のクラスの一部としてラボの手順にアクセスすると、コースの一部として別の UI にアクセスしなければならず、受講生にとっては混乱を招くことになります。受講生が個別のラボの指示を受ける理由に関する説明から、常に変化するクラウドベースのインターフェイスとプラットフォームの性質がよく分かります。GitHub 上のファイルにアクセスするための Microsoft Learning サポートと GitHub サイトのナビゲーションのサポートは、このコースのみを教える MCT に限定されます。

マークダウン ファイルの処理済みのフォーマット済みバージョンは、https://aka.ms/az220labs-jpn にあります。

### 現在のリリースでの既知の問題

**新しい**問題が既存の問題と重複していないことを確認するために、現在の未解決問題を参照してください。 以下は、既に作業を行っている未解決な問題ではない既知の問題です。

- ほとんどのラボでは、具体的に名前を付けられた IoT デバイスを作成するために、セットアップ スクリプトを実行する必要があります。ラボ内およびラボ全体でデバイス リソースを "クリーンアップ" する作業が行われています。

- ラボ演習の多くは、定義されたタスクに分割されておらず、迷ったり、ステップを見逃しやすくなります。この書式設定に対処する作業が行われています。

- ラボの手順には、イメージによって指示の明確さを改善できる箇所があります。現在、必要とされる箇所に画像がラボに追加されています。ただし、Azure UI の流動的な性質のため、画像の数を少なく抑える予定です。
