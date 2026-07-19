# エージェント（サーバー側）

エージェントは、ユーザーのリクエストに応じて A2UI メッセージを生成するサーバー側プログラムです。

実際のコンポーネント描画は [レンダラー](renderers.md) が担当し、メッセージが [トランスポート](../concepts/transports.md) を通じてクライアントへ届けられます。エージェントの責務は、A2UI メッセージを生成することだけです。

## エージェントの動き

```
User Input → Agent Logic → LLM → A2UI JSON → Send to Client
```

1. ユーザーメッセージを **受け取る**
2. Gemini、GPT、Claude などの LLM で **処理する**
3. 構造化出力として A2UI JSON メッセージを **生成する**
4. トランスポート経由でクライアントへ **送信する**

クライアント側からのユーザー操作は、新しいユーザー入力として扱えます。

## サンプルエージェント

A2UI リポジトリには、学習用のサンプルエージェントが含まれています。

- [Restaurant Finder](https://github.com/a2ui-project/a2ui/tree/main/samples/agent/adk/restaurant_finder)
  - フォーム付きのテーブル予約
  - ADK で実装
- [Rizzcharts](https://github.com/a2ui-project/a2ui/tree/main/samples/community/agent/adk/rizzcharts/python)
  - A2UI カスタムコンポーネントのデモ
  - ADK で実装
- [Orchestrator](https://github.com/a2ui-project/a2ui/tree/main/samples/community/agent/adk/orchestrator)
  - リモートのサブエージェントから A2UI メッセージを中継
  - ADK で実装

## A2A で使うエージェントの種類

### 1. ユーザー向けエージェント（スタンドアロン）

ユーザーが直接対話するエージェントです。

### 2. リモートエージェントのホストとしてのユーザー向けエージェント

ユーザー向けエージェントが 1 つ以上のリモートエージェントのホストになるパターンです。ユーザー向けエージェントがリモートエージェントを呼び出し、リモート側が A2UI メッセージを生成します。これは、クライアントエージェントがサーバーエージェントを呼ぶ [A2A](https://a2a-protocol.org) でよくある形です。

- ユーザー向けエージェントは A2UI メッセージをそのままパススルーできます
- ユーザー向けエージェントは、クライアントへ送る前に A2UI メッセージを編集できます

### 3. リモートエージェント

リモートエージェントはユーザー向け UI の直接の一部ではありません。ユーザー向けエージェントから呼び出されるリモートエージェントとして登録されます。これも [A2A](https://a2a-protocol.org) でよく使われるパターンです。
