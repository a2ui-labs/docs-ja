# A2UI はどこで使われていますか？

A2UI は、Google 内の複数チームやパートナー組織で採用され、次世代のエージェント主導アプリケーションを支えています。ここでは、A2UI が実際にどのように使われているかを紹介します。

## 本番導入

### Google Opal: すべての人のための AI ミニアプリ

[Opal](http://opal.google) は、自然言語だけで AI ミニアプリを作成、編集、共有できるようにし、何十万人もの人に使われています。

**Opal での A2UI の使い方**

Google の Opal チームは、初期から **A2UI の主要な貢献者** です。Opal の AI ミニアプリを成立させる動的な生成 UI システムの基盤として A2UI を使っています。

- **高速な試作**: 新しい UI パターンを素早く作って試せる
- **ユーザー生成アプリ**: 誰でもカスタム UI 付きアプリを作れる
- **動的なインターフェース**: ユースケースごとに UI が自動で変わる

> 「A2UI は私たちの仕事の基盤です。固定されたフロントエンドに縛られず、AI が新しい方法でユーザー体験を導ける柔軟性を与えてくれます。宣言的で安全性に重きを置いた性質のおかげで、私たちは素早く安全に実験できます。」
>
> **— Dimitri Glazkov**, Principal Engineer, Opal Team

**詳細:** [opal.google](http://opal.google)

---

### Gemini Enterprise: ビジネス向けのカスタムエージェント

Gemini Enterprise は、企業が業務フローやデータに合わせた強力なカスタム AI エージェントを構築できるようにします。

**Gemini Enterprise での A2UI の使い方**

A2UI は、エンタープライズエージェントが業務アプリ内で **リッチで対話的な UI** を描画できるように統合されています。単なるテキスト応答を超えて、従業員を複雑なワークフローへ導きます。

- **データ入力フォーム**: 構造化データ収集のための AI 生成フォーム
- **承認ダッシュボード**: レビューと承認のための動的 UI
- **ワークフロー自動化**: 複雑なタスク向けのステップバイステップ UI
- **カスタム企業 UI**: 業界固有の要件に合わせた専用インターフェース

> 「顧客が求めているのは、質問に答えるだけのエージェントではありません。従業員を複雑なワークフローへ導くことです。Gemini Enterprise 上で開発する人たちは、A2UI によって、データ入力フォームから承認ダッシュボードまで、あらゆるタスクに必要な動的でカスタムな UI をエージェントに生成させることができます。ワークフロー自動化を劇的に加速できるでしょう。」
>
> **— Fred Jabbour**, Product Manager, Gemini Enterprise

**詳細:** [Gemini Enterprise](https://cloud.google.com/gemini)

---

### Flutter GenUI SDK: モバイル向けの生成 UI

[Flutter GenUI SDK](https://docs.flutter.dev/ai/genui) は、モバイル、デスクトップ、Web の Flutter アプリに動的な AI 生成 UI を届けます。

**GenUI での A2UI の使い方**

GenUI SDK は、サーバー側エージェントと Flutter アプリの間の通信に **基盤プロトコルとして A2UI** を使っています。GenUI を使っているなら、内部では A2UI を使っていることになります。

- **クロスプラットフォーム対応**: iOS、Android、Web、デスクトップで同じエージェントが動く
- **ネイティブ性能**: 各プラットフォームで Flutter ウィジェットがネイティブに描画される
- **ブランドの一貫性**: アプリのデザインシステムに沿った UI になる
- **サーバー主導 UI**: アプリ更新なしでエージェントが表示内容を制御できる

> 「Flutter は、表現力がありブランド性の高いカスタムデザインシステムを、どのプラットフォームでも素早く作れるから選ばれます。A2UI は Flutter GenUI SDK にぴったりでした。どのプラットフォームでも、すべてのユーザーに高品質でネイティブな体験を届けられるからです。」
>
> **— Vijay Menon**, Engineering Director, Dart & Flutter

**試すには:**

- [GenUI Documentation](https://docs.flutter.dev/ai/genui)
- [Getting Started Video](https://www.youtube.com/watch?v=nWr6eZKM6no)
- [Verdure Example](https://github.com/flutter/genui/tree/main/examples/verdure)（A2UI のクライアント・サーバー例）

---

### Google ADK: Agent Development Kit

[Agent Development Kit](https://google.github.io/adk-docs/)（ADK）は、AI エージェントを構築・展開する Google のオープンソースフレームワークです。内蔵の開発者 UI である [ADK Web](https://github.com/google/adk-web) には、A2UI のネイティブ描画が含まれます。

**ADK での A2UI の使い方**

ADK は A2UI v0.8 の標準カタログを統合し、仕様に準拠したエージェント部品をチャット内でネイティブ UI として自動描画します。ADK は A2UI と A2A のメッセージ変換も扱うため、ADK ベースのエージェントは A2UI 対応クライアントへリッチな UI を送れます。

- **組み込み描画**: ADK Web が開発 UI で A2UI コンポーネントをネイティブ描画
- **A2A 統合**: A2UI メッセージを A2A DataPart メタデータと ADK イベントの間で変換
- **Agent SDK**: [A2UI Python agent SDK](https://github.com/google/A2UI/tree/main/agent_sdks/python) が、エージェントから A2UI を生成するための ADK 拡張を提供

**試すには:**

- [ADK Documentation](https://google.github.io/adk-docs/)
- [ADK Web](https://github.com/google/adk-web)（A2UI 対応の開発者 UI）
- [Agent Development Guide](../guides/agent-development.md)（ADK で A2UI エージェントを作る）

## パートナー統合

### AG UI / CopilotKit: フルスタックのエージェントフレームワーク

[AG UI](https://ag-ui.com/) と [CopilotKit](https://www.copilotkit.ai/) は、**導入初日から A2UI に対応**した包括的なエージェントアプリ向けフレームワークを提供します。

**連携の仕組み**

AG UI は、カスタムフロントエンドと専用エージェントの間に高帯域な接続を作るのが得意です。ここに A2UI サポートを加えると、開発者は両方の利点を得られます。

- **状態同期**: AG UI がアプリ状態とチャット履歴を管理
- **A2UI 描画**: サードパーティエージェントの動的 UI を描画
- **マルチエージェント対応**: 複数エージェントの UI を調停
- **React 統合**: React アプリと自然に統合

> 「AG UI は、カスタムフロントエンドと専用エージェントの間に高帯域な接続を作るのが得意です。A2UI をサポートすることで、開発者に両方の良さを提供できます。今では、リッチで状態同期されたアプリを作りつつ、A2UI を通じてサードパーティエージェントの動的 UI も描画できます。マルチエージェントの世界にぴったりです。」
>
> **— Atai Barkai**, CopilotKit / AG UI Founder

**詳細:**

- [AG UI](https://ag-ui.com/)
- [CopilotKit](https://www.copilotkit.ai/)

### Google の AI 搭載プロダクト

Google 全体で AI 活用が進む中、A2UI は AI エージェントがテキストだけでなく **ユーザーインターフェースそのものをやり取りする標準的な方法** を提供します。

**社内エージェントでの採用**

- **マルチエージェントワークフロー**: 複数エージェントが同じ surface に寄与
- **リモートエージェント対応**: 別サービスで動くエージェントが UI を提供できる
- **標準化**: チーム間で共通プロトコルを使うことで統合コストを削減
- **外部公開**: 内部エージェントを Gemini Enterprise などへ簡単に拡張できる

> 「A2A がプラットフォームをまたいでエージェント同士の会話を標準化するように、A2UI はユーザーインターフェース層を標準化し、オーケストレーターを通じたリモートエージェントのユースケースを支えます。社内チームは、リッチな UI が例外ではなく当たり前になるエージェントを素早く作れるようになりました。Google が生成 UI をさらに推し進める中で、A2UI はどのクライアントにも描画できるサーバー主導 UI の理想的な土台です。」
>
> **— James Wren**, Senior Staff Engineer, AI Powered Google

## コミュニティプロジェクト

A2UI コミュニティは、魅力的なプロジェクトを作っています。

### オープンソースの例

- **Restaurant Finder** ([samples/agent/adk/restaurant_finder](https://github.com/google/A2UI/tree/main/samples/agent/adk/restaurant_finder))
  - 動的フォーム付きテーブル予約
  - Gemini ベースのエージェント
  - ソースコード公開

- **Contact Lookup** ([samples/agent/adk/contact_lookup](https://github.com/google/A2UI/tree/main/samples/agent/adk/contact_lookup))
  - 検索インターフェースと結果リスト
  - A2A エージェント例
  - データバインディングのデモ

- **Component Gallery** ([samples/client/angular - gallery mode](https://github.com/google/A2UI/tree/main/samples/client/angular))
  - すべてのコンポーネントの対話的な一覧
  - ライブ例とコード
  - 学習用として最適

### サードパーティ連携

- **[json-render](https://json-render.dev/docs/a2ui)** — Zod スキーマを使って A2UI コンポーネントカタログを描画する Vercel の React ライブラリ。 [比較記事](https://dipjyotimetia.medium.com/vercels-json-render-vs-google-s-a2ui-the-head-to-head-6f213cf1a23b) も参照してください。
- **[OpenClaw Canvas](https://docs.openclaw.ai/platforms/mac/canvas)** — OpenClaw は canvas システム経由で、接続済みデバイスに A2UI 生成 UI を描画します。 [アーキテクチャ概要](https://ppaolo.substack.com/p/openclaw-system-architecture-overview) もあります。

### コミュニティへの投稿

A2UI で何か作ったなら、[コミュニティと共有してください](community.md)。

## 次のステップ

- [クイックスタート](../quickstart.md) - デモを試す
- [エージェント開発](../guides/agent-development.md) - エージェントを作る
- [クライアント設定](../guides/client-setup.md) - レンダラーを統合する
- [コミュニティ](community.md) - コミュニティに参加する

---

**本番で A2UI を使っていますか？** [GitHub Discussions](https://github.com/google/A2UI/discussions) でぜひ体験を共有してください。
