# クイックスタート：5分でA2UIを実行する

レストラン検索デモを通じてA2UIを直接体験してください。このガイドに従えば、5分以内にエージェントが生成したUIを体験できます。

## 構築するもの

このクイックスタートを完了する頃には、以下のものが手に入ります：

- ✅ A2UI Litレンダーラーが動作するWebアプリ
- ✅ 動的なUIを生成するGeminiベースのエージェント
- ✅ フォーム生成、時間選択、予約確認フローを含むインタラクティブなレストラン検索機能
- ✅ エージェントからUIへA2UIメッセージが配信される仕組みの理解

## 前提条件

開始する前に、以下の準備ができていることを確認してください：

- **Node.js** (v18以降) - [ここからダウンロード](https://nodejs.org/)
- **Gemini APIキー** - [Google AI Studioで無料で取得](https://aistudio.google.com/apikey)

!!! warning "セキュリティに関する通知"
    このデモは、Geminiを使用してA2UI応答を生成するA2Aエージェントを実行します。エージェントはあなたのAPIキーにアクセスし、GoogleのGemini APIにリクエストを送信します。本番環境で実行する前に、必ずエージェントのコードを確認してください。

## ステップ1：リポジトリのクローン

```bash
git clone https://github.com/google/a2ui.git
cd a2ui
```

## ステップ2：APIキーの設定

Gemini APIキーを環境変数としてエクスポートします：

```bash
export GEMINI_API_KEY="your_gemini_api_key_here"
```

## ステップ3：Litクライアントに移動

```bash
cd samples/client/lit
```

## ステップ4：インストールと実行

デモランチャーを単一のコマンドで実行します：

```bash
npm install
npm run demo:all
```

このコマンドは以下のことを行います：

1.  すべての依存関係のインストール
2.  A2UIレンダーラーのビルド
3.  A2Aレストラン検索エージェントの起動 (Pythonバックエンド)
4.  開発サーバーの起動
5.  ブラウザで `http://localhost:5173` を開く

!!! success "デモ実行中"
    すべてが正常に動作していれば、ブラウザにWebアプリが表示されているはずです。エージェントがUIを生成する準備ができました！

## ステップ5：試してみる

Webアプリで以下のプロンプトを試してください：

1.  **「2名で予約して」** - エージェントが予約フォームを生成するのを確認してください。
2.  **「近くのイタリアンレストランを探して」** - 動的な検索結果を確認してください。
3.  **「営業時間は？」** - 意図に応じて変化するUIレイアウトを体験してください。

### 内部の仕組み

```
┌─────────────┐         ┌──────────────┐         ┌────────────────┐
│  メッセージ │────────>│  A2Aエージェント│────────>│   Gemini API   │
│    入力     │         │   (Python)   │         │     (LLM)      │
└─────────────┘         └──────────────┘         └────────────────┘
                                │                         │
                                │ A2UI JSON 生成          │
                                │<────────────────────────┘
                                │
                                │ JSONLメッセージのストリーミング
                                v
                         ┌──────────────┐
                         │   Webアプリ  │
                         │ (A2UI Lit    │
                         │ レンダラー)  │
                         └──────────────┘
                                │
                                │ ネイティブコンポーネントのレンダリング
                                v
                         ┌──────────────┐
                         │   ユーザーUI │
                         └──────────────┘
```

1.  **ユーザー**がWeb UIを介してメッセージを送信します。
2.  **A2Aエージェント**がこれを受信し、会話の内容をGeminiに送信します。
3.  **Gemini**がUIを記述するA2UI JSONメッセージを生成します。
4.  **A2Aエージェント**がこれらのメッセージをWebアプリにストリーミングします。
5.  **A2UIレンダーラー**がこれらをネイティブWebコンポーネントに変換します。
6.  **ユーザー**がブラウザでレンダリングされたUIを確認します。

## A2UIメッセージ構造の分析

エージェントが何を送信しているか見てみましょう。以下は簡略化されたJSONメッセージの例です：

### UI定義

```json
{
  "surfaceUpdate": {
    "surfaceId": "main",
    "components": [
      {
        "id": "header",
        "component": {
          "Text": {
            "text": {"literalString": "テーブルを予約する"},
            "usageHint": "h1"
          }
        }
      },
      {
        "id": "date-picker",
        "component": {
          "DateTimeInput": {
            "label": {"literalString": "日付を選択"},
            "value": {"path": "/reservation/date"},
            "enableDate": true
          }
        }
      },
      {
        "id": "submit-btn",
        "component": {
          "Button": {
            "child": "submit-text",
            "action": {"name": "confirm_booking"}
          }
        }
      },
      {
        "id": "submit-text",
        "component": {
          "Text": {"text": {"literalString": "予約確定"}}
        }
      }
    ]
  }
}
```

このメッセージは、サーフェス（surface）上のUIコンポーネント（テキストヘッダー、日付選択、ボタン）を定義します。

### データ入力

```json
{
  "dataModelUpdate": {
    "surfaceId": "main",
    "contents": [
      {
        "key": "reservation",
        "valueMap": [
          {"key": "date", "valueString": "2025-12-15"},
          {"key": "time", "valueString": "19:00"},
          {"key": "guests", "valueInt": 2}
        ]
      }
    ]
  }
}
```

このメッセージは、コンポーネントがバインディングできるデータモデルを埋めます。

### レンダリング開始シグナル

```json
{"beginRendering": {"surfaceId": "main", "root": "header"}}
```

このメッセージは、UIをレンダリングするのに十分な情報が揃ったことをクライアントに知らせます。

!!! tip "シンプルなJSON構造"
    この構造がいかに読みやすく、整然としているかお分かりいただけますか？LLMはこれを容易に生成でき、コードを実行することなく安全に送信・レンダリングできます。

## 他のデモを探索する

リポジトリにはいくつかの他のデモが含まれています：

### コンポーネントギャラリー（エージェント不要）

利用可能なすべてのA2UIコンポーネントを確認してください：

```bash
npm start -- gallery
```

このクライアント専用デモは、すべての標準コンポーネント（Card、Button、TextField、Timelineなど）をライブ例とコードサンプルとともに紹介します。

### 連絡先検索（Contact Lookup）デモ

別のエージェントのユースケースを試してください：

```bash
npm run demo:contact
```

このデモは、検索フォームと結果リストを生成する連絡先検索エージェントを紹介します。

## 次のステップ

A2UIが動作する様子を確認できたので、次は以下のことに挑戦しましょう：

- **[コアコンセプトを学ぶ](concepts/overview.md)**：サーフェス、コンポーネント、データバインディングの理解
- **[独自のクライアントを設定する](guides/client-setup.md)**：自分のアプリへのA2UIの統合
- **[エージェントを構築する](guides/agent-development.md)**：A2UI応答を生成するエージェントの作成
- **[プロトコルを探索する](reference/messages.md)**：技術仕様を詳しく知る

## トラブルシューティング

### ポートがすでに使用されている場合

ポート 5173 がすでに使用されている場合、開発サーバーは自動的に次の利用可能なポートを試行します。ターミナル出力の実際のURLを確認してください。

### APIキーの問題

APIキーが見つからないというエラーが表示される場合：

1.  キーが正しくエクスポートされているか確認： `echo $GEMINI_API_KEY`
2.  [Google AI Studio](https://aistudio.google.com/apikey)で取得した有効なGemini APIキーであることを確認
3.  再エクスポートを試行： `export GEMINI_API_KEY="your_key"`

### Pythonの依存関係

デモではA2AエージェントにPythonを使用します。Pythonエラーが発生する場合：

```bash
# Python 3.10以降がインストールされていることを確認
python3 --version

# デモはnpmスクリプトを介して依存関係を自動的にインストールするはずです。
# そうならない場合は、手動でインストールしてください：
cd ../../agent/adk/restaurant_finder
pip install -r requirements.txt
```

### それでも解決しない場合は？

- [GitHub Issues](https://github.com/google/a2ui/issues)を確認してください。
- [samples/client/lit/README.md](https://github.com/google/a2ui/tree/main/samples/client/lit)を確認してください。
- コミュニティディスカッションに参加してください。

## デモコードの理解

仕組みが気になりますか？以下の項目を確認してください：

- **エージェントコード**: `samples/agent/adk/restaurant_finder/` - Python A2Aエージェント
- **クライアントコード**: `samples/client/lit/` - A2UIレン더ラーを含むLit Webクライアント
- **A2UIレンダラー**: `web-lib/` - Webレンダラー実装

各ディレクトリには、詳細なドキュメントを含むREADMEがあります。

---

**おめでとうございます！** 初めてのA2UIアプリケーションを正常に実行できました。AIエージェントが、安全で宣言的なJSONメッセージを通じて、WebアプリケーションでネイティブにレンダリングされるリッチでインタラクティブなUIを生成する方法を確認しました。
