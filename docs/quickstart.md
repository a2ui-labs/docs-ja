# A2UI クイックスタート: 5 分で実行する

レストラン検索デモを動かして、A2UI を実際に触ってみましょう。このガイドを終える頃には、エージェントが生成する UI を 5 分以内で体験できます。

## 作成するもの

このクイックスタートを終えると、次のものが手元にできます。

- A2UI Lit レンダラーが動く Web アプリ
- 動的 UI を生成する Gemini ベースのエージェント
- フォーム生成、時間選択、確認フローを備えたインタラクティブなレストラン検索画面
- エージェントから UI まで A2UI メッセージが流れる仕組みの理解

## 前提条件

開始前に、次を用意してください。

- **Node.js**（v18 以降） - [ダウンロード](https://nodejs.org/)
- **uv**（Python パッケージマネージャー） - [インストール手順](https://docs.astral.sh/uv/getting-started/installation/)
  Python エージェントバックエンドの起動に使用します。
- **Gemini API キー** - [Google AI Studio で取得](https://aistudio.google.com/apikey)

> ⚠️ **セキュリティ注意**
>
> このデモは Gemini を使って A2UI 応答を生成する A2A エージェントを実行します。エージェントは API キーにアクセスし、Google の Gemini API にリクエストを送信します。本番環境で使う前に、必ずエージェントコードを確認してください。

## ステップ 1: リポジトリをクローンする

```bash
git clone https://github.com/google/a2ui.git
cd a2ui
```

## ステップ 2: API キーを設定する

Gemini API キーを環境変数として設定します。

```bash
export GEMINI_API_KEY="your_gemini_api_key_here"
```

## ステップ 3: Lit クライアントへ移動する

```bash
cd samples/client/lit
```

## ステップ 4: インストールして実行する

デモランチャーを 1 つのコマンドで実行します。

```bash
npm install
npm run demo:all
```

このコマンドは次のことを行います。

1. すべての依存関係をインストールする
2. A2UI レンダラーをビルドする
3. A2A レストラン検索エージェント（Python バックエンド）を起動する
4. 開発サーバーを起動する
5. ブラウザで `http://localhost:5173` を開く

> ✅ **デモ起動完了**
>
> うまくいっていれば、ブラウザに Web アプリが表示されます。これでエージェントは UI を生成できる状態です。

## ステップ 5: 試してみる

Web アプリで次のプロンプトを試してください。

1. **"Book a table for 2"** - 予約フォームが生成される様子を見る
2. **"Find Italian restaurants near me"** - 動的な検索結果を見る
3. **"What are your hours?"** - 意図ごとに異なる UI レイアウトを体験する

### 裏側で起きていること

```
┌─────────────┐         ┌──────────────┐         ┌────────────────┐
│   You Type  │────────>│ A2A Agent    │────────>│  Gemini API    │
│  a Message  │         │  (Python)    │         │  (LLM)         │
└─────────────┘         └──────────────┘         └────────────────┘
                               │                         │
                               │ Generates A2UI JSON     │
                               │<────────────────────────┘
                               │
                               │ Streams JSONL messages
                               v
                        ┌──────────────┐
                        │   Web App    │
                        │ (A2UI Lit    │
                        │  Renderer)   │
                        └──────────────┘
                               │
                               │ Renders native components
                               v
                        ┌──────────────┐
                        │   Your UI    │
                        └──────────────┘
```

1. **あなた**が Web UI からメッセージを送る
2. **A2A エージェント**がそれを受け取り、会話を Gemini に渡す
3. **Gemini** が UI を記述する A2UI JSON メッセージを生成する
4. **A2A エージェント**がそれらを Web アプリへストリーミングする
5. **A2UI レンダラー**がネイティブ Web コンポーネントへ変換する
6. **あなた**がブラウザでレンダリング済み UI を確認する

## A2UI メッセージの構造

エージェントが何を送っているのか見てみましょう。以下は、JSON メッセージを簡略化した例です。

=== "v0.8 (Stable)"

    **UI の定義:**

    ```json
    {"surfaceUpdate": {"surfaceId": "main", "components": [
      {"id": "header", "component": {"Text": {"text": {"literalString": "Book Your Table"}, "usageHint": "h1"}}},
      {"id": "date-picker", "component": {"DateTimeInput": {"label": {"literalString": "Select Date"}, "value": {"path": "/reservation/date"}, "enableDate": true}}},
      {"id": "submit-text", "component": {"Text": {"text": {"literalString": "Confirm Reservation"}}}},
      {"id": "submit-btn", "component": {"Button": {"child": "submit-text", "action": {"name": "confirm_booking"}}}}
    ]}}
    ```

    **データの投入:**

    ```json
    {"dataModelUpdate": {"surfaceId": "main", "contents": [
      {"key": "reservation", "valueMap": [
        {"key": "date", "valueString": "2025-12-15"},
        {"key": "time", "valueString": "19:00"},
        {"key": "guests", "valueInt": 2}
      ]}
    ]}}
    ```

    **レンダリング開始の通知:**

    ```json
    {"beginRendering": {"surfaceId": "main", "root": "header"}}
    ```

=== "v0.9 (Draft)"

    **サーフェスの作成:**

    ```json
    {"version": "v0.9", "createSurface": {"surfaceId": "main", "catalogId": "https://a2ui.org/specification/v0_9/basic_catalog.json"}}
    ```

    **UI の定義:**

    ```json
    {"version": "v0.9", "updateComponents": {"surfaceId": "main", "components": [
      {"id": "header", "component": "Text", "text": "# Book Your Table", "variant": "h1"},
      {"id": "date-picker", "component": "DateTimeInput", "label": "Select Date", "value": {"path": "/reservation/date"}, "enableDate": true},
      {"id": "submit-text", "component": "Text", "text": "Confirm Reservation"},
      {"id": "submit-btn", "component": "Button", "child": "submit-text", "variant": "primary", "action": {"event": {"name": "confirm_booking"}}}
    ]}}
    ```

    **データの投入:**

    ```json
    {"version": "v0.9", "updateDataModel": {"surfaceId": "main", "path": "/reservation", "value": {"date": "2025-12-15", "time": "19:00", "guests": 2}}}
    ```

    v0.9 では `createSurface` が `beginRendering` に代わり、コンポーネントはよりフラットな形式を使い、データモデルは型付きの隣接リストではなく通常の JSON 値を使います。

> 💡 **ただの JSON です**
>
> 読みやすく構造化されていることが分かるはずです。LLM はこれを簡単に生成でき、コード実行なしで安全に送信・レンダリングできます。

## 他のデモを見る

このリポジトリには、ほかにもいくつかのデモがあります。

### コンポーネントギャラリー（エージェント不要）

利用可能な A2UI コンポーネントをすべて確認できます。

```bash
npm start -- gallery
```

これは、標準コンポーネント（Card、Button、TextField、Timeline など）をすべて紹介するクライアント専用デモです。ライブ例とコードサンプルも含まれます。

### Contact Lookup デモ

別のユースケースも試せます。

```bash
npm run demo:contact
```

検索フォームと結果リストを生成する Contact Lookup エージェントのデモです。

## 次のステップ

A2UI の動きを確認できたら、次に進みましょう。

- **[コアコンセプトを学ぶ](concepts/overview.md)**: サーフェス、コンポーネント、データバインディングを理解する
- **[クライアントを設定する](guides/client-setup.md)**: 自分のアプリに A2UI を統合する
- **[エージェントを構築する](guides/agent-development.md)**: A2UI 応答を生成するエージェントを作る
- **[プロトコルを読む](reference/messages.md)**: 技術仕様を掘り下げる

## トラブルシューティング

### ポートがすでに使用されている

ポート 5173 がすでに使われている場合、開発サーバーは次の空いているポートを自動的に試します。ターミナル出力で実際の URL を確認してください。

### API キーの問題

API キーが見つからないというエラーが出る場合は、次を確認してください。

1. キーがエクスポートされているか確認する: `echo $GEMINI_API_KEY`
2. [Google AI Studio](https://aistudio.google.com/apikey) で取得した有効な Gemini API キーであることを確認する
3. 再度エクスポートする: `export GEMINI_API_KEY="your_key"`

### 起動時の接続エラー

ブラウザを開いたときに `ERR_CONNECTION_REFUSED` が出ても心配いりません。これは既知の競合です ([#587](https://github.com/google/A2UI/issues/587))。Web アプリが Python エージェントより先に起動するためです。数秒待ってからページを再読み込みしてください。

### Python / uv の問題

デモエージェントの実行には [uv](https://docs.astral.sh/uv/) が必要です。`uv: command not found` が出る場合は、次を試してください。

```bash
# uv をインストール
curl -LsSf https://astral.sh/uv/install.sh | sh

# 確認
uv --version
```

その他の Python エラーが出る場合は、次を確認してください。

```bash
# Python 3.10 以降が利用可能か確認
python3 --version

# エージェントを手動で実行してみる
cd samples/agent/adk/restaurant_finder
uv run .
```

### まだ問題が解決しない場合

- [GitHub Issues](https://github.com/google/A2UI/issues) を確認する
- [samples/client/lit/README.md](https://github.com/google/A2UI/tree/main/samples/client/lit) を読む
- コミュニティの議論に参加する

## デモコードの理解

仕組みを見たい場合は、次を確認してください。

- **エージェントコード**: `samples/agent/adk/restaurant_finder/` - Python の A2A エージェント
- **クライアントコード**: `samples/client/lit/` - A2UI レンダラーを含む Lit Web クライアント
- **A2UI レンダラー**: `renderers/lit/`（Lit）と `renderers/web_core/`（フレームワーク非依存のコア）

各ディレクトリには、詳しいドキュメントを含む README があります。

---

**おめでとうございます。** これで最初の A2UI アプリケーションを実行できました。安全な宣言的 JSON メッセージだけで、AI エージェントが Web アプリケーション内にリッチでインタラクティブな UI をネイティブにレンダリングできることを確認できました。
