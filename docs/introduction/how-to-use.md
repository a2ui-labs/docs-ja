# A2UIはどのように使えますか？

あなたの役割とユースケースに合った統合パスを選択してください。

## 3つのパス

### パス 1：ホストアプリケーションの構築 (フロントエンド)

既存のアプリにA2UIレンダリング機能を統合するか、エージェント主導の新しいフロントエンドを構築します。

**レンダラーの選択：**

- **Web：** Lit, Angular
- **モバイル/デスクトップ：** Flutter GenUI SDK
- **React：** 2026年第1四半期予定

**クイック設定：**

Angularアプリを使用している場合、Angularレンダラーを追加できます：

```bash
npm install @a2ui/angular 
```

エージェントメッセージ (SSE, WebSockets または A2A) に接続し、ブランドに合わせてスタイルをカスタマイズしてください。

**次のステップ：** [クライアント設定ガイド](../guides/client-setup.md) | [テーマ設定](../guides/theming.md)

---

### パス 2：エージェントの構築 (バックエンド)

すべての互換性のあるクライアントのためにA2UI応答を生成するエージェントを作成します。

**フレームワークの選択：**

- **Python：** Google ADK, LangChain, カスタム実装
- **Node.js：** A2A SDK, Vercel AI SDK, カスタム実装

LLMプロンプトにA2UI仕様を含め、JSONLメッセージを生成して、SSE, WebSockets または A2A を介してクライアントにストリーミングします。

**次のステップ：** [エージェント開発ガイド](../guides/agent-development.md)

---

### パス 3：既存のフレームワークを活用する

すでにA2UIをサポートしているフレームワークを介して使用します：

- **[AG UI / CopilotKit](https://ag-ui.com/)** - A2UIレンダリング機能を含むフルスタックReactフレームワーク
- **[Flutter GenUI SDK](https://docs.flutter.dev/ai/genui)** - クロスプラットフォーム生成型UI（内部的にA2UIを使用）

**次のステップ：** [エージェントUIエコシステム](agent-ui-ecosystem.md) | [A2UIはどこで使われていますか？](where-is-it-used.md)
