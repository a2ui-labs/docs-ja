# トランスポート（メッセージ伝送）

トランスポートは、エージェントからクライアントへ A2UI メッセージを届けます。A2UI は transport-agnostic なので、JSON を送れる手段なら何でも使えます。

実際のコンポーネント描画は [レンダラー](../reference/renderers.md) が担当し、[エージェント](../reference/agents.md) が A2UI メッセージを生成します。エージェントからクライアントへそのメッセージを届けるのがトランスポートの役割です。

## 仕組み

```
エージェント → トランスポート → クライアントレンダラー
```

A2UI は JSON メッセージのシーケンスを定義します。トランスポート層は、そのシーケンスをエージェントからクライアントへ届けます。一般的には、JSON Lines（JSONL）のような形式で 1 行ごとに 1 つの A2UI メッセージを送るストリームが使われます。

## 利用できるトランスポート

| トランスポート | ステータス | 用途 |
|-----------|--------|----------|
| **A2A Protocol** | ✅ 安定版 | マルチエージェントシステム、エンタープライズメッシュ |
| **AG-UI** | ✅ 安定版 | フルスタックの React、Vue、Angular アプリケーション(CopilotKit) |
| **REST API** | 📋 計画中 | シンプルな HTTP エンドポイント |
| **WebSockets** | 💡 提案中 | リアルタイム双方向通信 |
| **SSE (Server-Sent Events)** | 💡 提案中 | Web ストリーミング |

## A2A Protocol

[Agent2Agent (A2A) Protocol](https://a2a-protocol.org) は、安全で標準化されたエージェント間通信を提供します。A2A 拡張を使うと、A2UI と簡単に統合できます。

**利点:**

- セキュリティと認証が組み込み
- さまざまなメッセージ形式、認証、転送プロトコルへのバインディングをサポート
- 関心の分離が明確

A2A を使う場合、この連携はほぼ自動です。

TODO: 詳細ガイドを追加予定

**参照:** [A2A 拡張仕様](../specification/v0.8-a2a-extension.md)

## AG-UI

[AG-UI](https://ag-ui.com/) は、A2UI メッセージを AG-UI イベントへ変換し、トランスポートと状態同期を自動的に処理します。フルスタックの React、Vue、Angular アプリケーションでよく使われています。CopilotKit は AG-UI の開発元であり、主要な利用者です。

**参照:** [任意のエージェントフレームワークで A2UI を使う(AG-UI を使用)](../guides/a2ui-with-any-agent-framework.md): 任意のエージェントフレームワークに CopilotKit をセットアップし、A2UI のレンダリングを有効にする方法。

## カスタムトランスポート

JSON を送れるなら、好きなトランスポートを使えます。

**HTTP/REST:**

```javascript
// TODO: 例を追加
```

**WebSockets:**

```javascript
// TODO: 例を追加
```

**Server-Sent Events:**

```javascript
// TODO: 例を追加
```

## 次のステップ

- **[A2A Protocol ドキュメント](https://a2a-protocol.org)**: A2A について学ぶ
- **[A2A 拡張仕様](../specification/v0.8-a2a-extension.md)**: A2UI と A2A の詳細
