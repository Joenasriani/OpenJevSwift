[English](README.md) | [简体中文](README.zh-CN.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [Português](README.pt-BR.md) | [Français](README.fr.md) | [Deutsch](README.de.md) | [العربية](README.ar.md)

# OpenJevSwift

[![CI](https://github.com/Algorythm-Canada/OpenJevSwift/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/Algorythm-Canada/OpenJevSwift/actions/workflows/ci.yml) [![Fixtures](https://github.com/Algorythm-Canada/OpenJevSwift/actions/workflows/fixtures.yml/badge.svg?branch=main)](https://github.com/Algorythm-Canada/OpenJevSwift/actions/workflows/fixtures.yml) [![Documentation](https://github.com/Algorythm-Canada/OpenJevSwift/actions/workflows/docs.yml/badge.svg?branch=main)](https://github.com/Algorythm-Canada/OpenJevSwift/actions/workflows/docs.yml)

> この翻訳と[英語の README](README.md) に相違がある場合は、英語版が優先されます。

テキストについて型付きの質問（はいかいいえか、複数の選択肢のうちどれか、ある尺度でどの程度か）をモデルに尋ね、生成されたテキストではなく確率を受け取ります。OpenJevSwift はこれらの質問にローカルで答えます。Mac 上で Jev 互換の HTTP API として動かすことも、自分の iPhone アプリや Mac アプリの中で動かすこともできます。こうした意思決定をデバイス上で行いたい Swift 開発者と、Python 環境ではなく単一のネイティブバイナリで OpenJev の API を使いたい Mac ユーザーのためのプロジェクトです。[デモアプリ](#デモアプリを試す)では、入力に合わせて答える様子を見られます。

[OpenJev](https://github.com/razorback16/openjev) のネイティブ Swift 実装です。OpenJev は、オープンで Jev 互換の「System One」意思決定サーバーです。状態と型付きの質問（`noul`、`choice`、`score`）を送ると、テキストを生成するのではなく、モデルの確率からすべての回答を読み取ります。そのため、回答がスキーマから外れることはありません。noul の回答は「はい」の確率で、choice や score の回答はすべての選択肢の確率と信頼度です。Mac 上では 1 つの `openjev` バイナリで、MLX 上の DiffusionGemma 26B-A4B や JevK5、あるいは Core ML 上の Verdict や Laya を提供します。リクエスト・レスポンス形式がアップストリームとまったく同じなので、TypeSafe の SDK を変更なしでそのまま使えます。同じライブラリが iPhone アプリや Mac アプリの中でもリクエストに応答します。

OpenJevSwift は独立したプロジェクトです。TypeSafe AI（Jev の開発元）、Google DeepMind や NVIDIA（DiffusionGemma）、およびこのプロジェクトが提供するその他のモデルの作者とは提携しておらず、承認も受けていません。Jev、TypeSafe、Gemma などの名称は、それぞれの所有者に帰属します。

## クイックスタート

Appleシリコン搭載の Mac で、macOS 15 以降と Xcode 27 がある場合（最初のコマンドは Xcode の Metal Toolchain を 1 回だけインストールするもので、[docs/development.md](docs/development.md#xcode-27-needs-the-metal-toolchain) で説明しています）：

```bash
xcodebuild -downloadComponent MetalToolchain
git clone https://github.com/Algorythm-Canada/OpenJevSwift.git
cd OpenJevSwift
swift build -c release --product openjev
.build/release/openjev serve --backend verdict
```

Xcode 26.4 から 26.6 では、`swift build` に `--build-system swiftbuild` を追加してください。追加しないと、`mlx` と `jevk5` のバックエンドに MLX の Metal シェーダーが含まれません。

初回起動時に、Verdict の変換済み Core ML パッケージ、トークナイザー、キャリブレーター（約 310 MB）を Application Support にダウンロードし、各ファイルの SHA-256 を確認します。モデルの読み込みとウォームアップが終わると、サーバーは `127.0.0.1:8080` で待ち受けます。別のターミナルから：

```bash
curl -s localhost:8080/v1/systemone -H 'content-type: application/json' \
    -d '{"model":"jev-latest","state":"The deploy failed twice and the site is down.","questions":{"urgent":{"type":"noul","instructions":"Is this urgent?"}}}'
```

```json
{"model":"verdict-1.4","answers":{"urgent":{"type":"noul","noul":0.5910340547561646}},"usage":{"input_tokens":45,"output_tokens":0}}
```

`openjev decide` はサーバーなしで 1 件のリクエストに答え、サーバーが送信するバイト列を出力します：

```bash
echo '{"model":"jev-latest","state":"The deploy failed twice and the site is down.","questions":{"urgent":{"type":"noul","instructions":"Is this urgent?"}}}' | .build/release/openjev decide --backend verdict
```

Ctrl-C でサーバーをグレースフルに停止できます。[docs/deployment.md](docs/deployment.md) では、設定、launchd ジョブ、ログ、終了ステータスについて説明しています。

アプリはリリースに依存し、`OpenJevCore` と、読み込む各バックエンドのモジュールをリンクします。Verdict と Laya には `OpenJevEncoders`、DiffusionGemma には `OpenJevDiffusionGemma`、JevK5 には `OpenJevLetterReadout` を使います。DocC の記事 [Making decisions in an app](https://algorythm-canada.github.io/OpenJevSwift/documentation/openjevcore/gettingstarted/) では、同じモデルをアプリ内で読み込みます：

```swift
dependencies: [
    .package(url: "https://github.com/Algorythm-Canada/OpenJevSwift.git", from: "0.1.0"),
],
targets: [
    .target(
        name: "MyApp",
        dependencies: [
            .product(name: "OpenJevCore", package: "OpenJevSwift"),
            .product(name: "OpenJevEncoders", package: "OpenJevSwift"),
        ]),
]
```

## デモアプリを試す

![iPhone シミュレーター上で、入力中の請求に関する苦情に回答するトリアージデモ](docs/assets/triage-demo.gif)

[Examples/TriageDemo](Examples/TriageDemo) は、アップストリームの README にある例にデバイス上で答える iPhone アプリです。顧客メッセージに 1 時間以内の返信が必要か（`noul`）、どのチームが対応すべきか（`choice`）、顧客がどれほど不満を抱いているか（`score`）を、各選択肢の確率をバーで表示しながら、入力に合わせて答えます。Xcode 26.4 以降と、iOS 18 以降の iPhone またはシミュレーターが必要です：

1. OpenJevSwift パッケージを開いている Xcode ウィンドウをすべて閉じます。Xcode では、ローカルパッケージを使えるウィンドウは 1 つだけです。
2. `Examples/TriageDemo/TriageDemo.xcodeproj` を開きます。
3. iPhone で実行するには、Signing & Capabilities でチームを選択します。シミュレーターでは不要です。
4. `TriageDemo` スキームを実行します。

初回起動時には、ライブラリ自身のストアを通じて Verdict をダウンロードします。openjev-models リリースと Hugging Face から約 310 MB を取得し、各ファイルの SHA-256 を確認します。以降の起動はオフラインで動作します。詳細とテストは [Examples/TriageDemo/README.md](Examples/TriageDemo/README.md) にあります。

## 動作要件

| バックエンド | モデル | Mac | メモリ |
|---|---|---|---|
| `verdict` | `verdict-1.4`、151M パラメータ、Core ML | Appleシリコン、macOS 15 以降 | 1 問の読み取りが読み込む関数では 1.6 GB、6 つすべてでは 2.8 GB |
| `laya` | `laya-1.0`、421M パラメータ、Core ML | Appleシリコン、macOS 15 以降 | 1 問の読み取りが読み込む関数では 4.7 GB、8 つすべてでは 8.9 GB で、ピーク時は最大 9.7 GB。`OPENJEV_ENCODER_FUNCTIONS=2` では、1 問の読み取りで 2.1 GB、ピーク時は最大 4.4 GB |
| `mlx` | `openjev-0.1`、DiffusionGemma 26B-A4B、4-bit、MLX | Appleシリコン | 読み込みに約 17 GB（ビジョンタワーの 1.06 GiB を含む、D-054）。短いプロンプトでの運用時は 17.3 GiB（タワーの読み込み前に測定）で、キャッシュされた長いプロンプトにはさらに最大約 3.6 GB。32 GB 以上を推奨 |
| `jevk5` | `jevk5-0.2`、JevK5（Qwen3.5-4B）、8-bit、MLX | Appleシリコン | 読み込み後は 6.0 GB、`OPENJEV_MLX_CACHE_LIMIT_GB=4` での運用時は最大 11.0 GB。4-bit 変換版ではそれぞれ 3.6 GB と 8.9 GB |

ビルドには Xcode 26.4 以降が必要です。`mlx` バックエンドには MLX の Metal シェーダーも必要で、これは Swift Build が Metal Toolchain を使ってコンパイルします。Xcode 27 では Swift Build がデフォルトで、Xcode 26 では `--build-system swiftbuild` を指定します。アプリ内では、`OpenJevCore` は macOS 14 および iOS 17 以降で、Verdict と Laya のバックエンドは macOS 15 および iOS 18 以降で、JevK5 は Appleシリコンで動作します（iOS 向けにビルドはできますが、まだ iPhone 上で実行したことはありません）。Linux では、テスト用にコア、サーバー、`openjev` ツールをビルドします。バックエンドは含みません。

## ステータス

リリース 0.1.0 は、パッケージが依存できる最初のバージョンです。含まれる内容は [CHANGELOG.md](CHANGELOG.md) に記載しています。完了したもの：

- **マイルストーン 0 から 5**（含まれる作業 Issue はすべてクローズ済み）：基盤、意思決定エンジンのコア、MLX 上の DiffusionGemma の読み取り、`openjev` ツールを備えた Jev 互換 HTTP サーバー、読み取りの拡張と画像、`think` と chat completions によるテキスト生成（#50 から #53）。
- **DiffusionGemma（チェックポイントで検証済み）：** `steps`、`samples`、`sequential` のエンドツーエンド（#43、#44、#45）と、オラクルのカーネル上でアップストリームとビット単位で一致する画像の読み取り（#46 から #48）。
- **マイルストーン 6 から：** Verdict と Laya、および JevK5（#55）。JevK5 は JevBench の 231 項目中 230 項目で、作者が公開したトップの回答を返します（D-052）。
- **マイルストーン 7 から：** アップストリームとの JevBench 比較と、DiffusionGemma のキャリブレーションレポート（#61 と #62）。

未完了のもの：

- CLM モデル。誰かが必要とするまで延期しています（[D-011](docs/06-decisions.md#d-011-encoder-models-core-ml-for-verdict-and-laya-jevk5-first-among-the-extra-models)）。必要な場合は、ユースケースを添えて [Issue](https://github.com/Algorythm-Canada/OpenJevSwift/issues) を作成するか、[Discussions](https://github.com/Algorythm-Canada/OpenJevSwift/discussions) に投稿してください。
- DiffusionGemma の読み取りの高速化：#100、#101、#102。

マイルストーンと Issue の索引は [docs/08-implementation-plan.md](docs/08-implementation-plan.md) にあります。

## 互換性

記録済みの差異を除き、モデルの確率に至るまでのすべてがアップストリームとバイト単位で同一です。プロンプト、回答テンプレート、キャンバスとシード、リクエストの検証、エラーボディ、ヘッダー、`/v1/models` の一覧のすべてを、アップストリーム自身のコードが書き出したフィクスチャと照合しています。確率は測定された範囲内で一致します：

- **DiffusionGemma：** 63 回のオラクル読み取りで決定 D-014 と D-048 の範囲内に収まり（91.7% のスロットでトップのラベルが一致し、mlx-vlm の上位 2 つが 0.5 以上離れている 140 スロットのうち 139 スロットで一致）、333 の JevBench および TypeSafe 項目では 2 つのサーバー間でも範囲内に収まります。
- **Verdict と Laya：** 666 項目すべてでアップストリームのトップの回答と一致します。
- **JevK5（8-bit 変換版）：** JevBench の 231 項目中 230 項目で作者が公開したトップの回答と一致し、すべての項目でトークン数も同じです。

差異には、より厳格な JSON パーサー、あらゆるバックエンド障害に対する 503、未実装の機能などがあり、それぞれに決定記録があります。[docs/compatibility.md](docs/compatibility.md) には、3 つの表と、macOS、iOS、Linux で何が動作するかのマトリクスがあります。

## ドキュメント

- **API ドキュメント。** `OpenJevCore`、`OpenJevEncoders`、`OpenJevDiffusionGemma`、`OpenJevLetterReadout`、`OpenJevServer` の DocC カタログ：アプリでの入門、リクエストと回答の型、バックエンドの実装、サーバーの実行、設定リファレンス。Documentation ワークフローがソースの変更のたびにこれらをビルドし、<https://algorythm-canada.github.io/OpenJevSwift/> に公開します。`make docs` で同じサイトをローカルにビルドできます（[docs/development.md](docs/development.md)）。
- **[docs/deployment.md](docs/deployment.md)**：Mac で `openjev serve` を実行する方法。
- **[docs/compatibility.md](docs/compatibility.md)**：アップストリームと同一のもの、許容範囲内のもの、異なるもの。
- **[docs/credits.md](docs/credits.md)**：モデル、その作者とライセンス。
- **[docs/quality.md](docs/quality.md)** と **[docs/benchmarks.md](docs/benchmarks.md)**：アップストリームと比較した回答品質、および速度とメモリ。
- **[docs/README.md](docs/README.md)**：設計ドキュメントの索引。アップストリームの仕組みから、各決定、適合性の戦略まで。
- **[CHANGELOG.md](CHANGELOG.md)**：各リリースとバージョニングポリシー。
- **[SECURITY.md](SECURITY.md)**：脆弱性の報告方法、コードの接続先、キーの扱い方。
- **[ADOPTERS.md](ADOPTERS.md)**：OpenJevSwift を使用している組織。自分の組織はプルリクエストで追加してください。

## コントリビュート

バグ報告、互換性レポート、ドキュメントの修正、プルリクエストは、どなたからでも歓迎します。[CONTRIBUTING.md](CONTRIBUTING.md) では、ビルド、テスト、プルリクエストの作成方法を説明しています。[help wanted](https://github.com/Algorythm-Canada/OpenJevSwift/labels/help%20wanted) ラベルの付いた Issue には誰でも取り組めます。[Discussions](https://github.com/Algorythm-Canada/OpenJevSwift/discussions) は質問やアイデアのための場所です。

## ライセンスとクレジット

Apache-2.0 で、アップストリームの OpenJev と同じです。mlx-vlm（MIT）から移植したコードは、各ファイルのヘッダーに著作権表示を残しています。[THIRD_PARTY.md](THIRD_PARTY.md) には、参照しているすべてのプロジェクトを、固定したリビジョンとともに記載しています。モデルは他の人々の成果物であり、それぞれ独自のライセンスに従います。[docs/credits.md](docs/credits.md) では各モデルのクレジットと、このプロジェクトが重みを取得する場所を記載しています。このリポジトリに重みは含まれていません。
