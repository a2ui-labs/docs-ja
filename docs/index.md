---
hide:
  - toc
---

<!-- markdownlint-disable MD041 -->
<!-- markdownlint-disable MD033 -->
<div style="text-align: center; margin: 2rem 0 3rem 0;" markdown>

<!-- Logo for Light Mode (shows dark logo on light background) -->
<img src="assets/A2UI_dark.svg" alt="A2UI Logo" width="120" class="light-mode-only" style="margin-bottom: 1rem;">
<!-- Logo for Dark Mode (shows light logo on dark background) -->
<img src="assets/A2UI_light.svg" alt="A2UI Logo" width="120" class="dark-mode-only" style="margin-bottom: 1rem;">

# エージェント主導インターフェースのためのプロトコル

<p style="font-size: 1.2rem; max-width: 800px; margin: 0 auto 1rem auto; opacity: 0.9; line-height: 1.6;">
A2UIは、AIエージェントが任意のコードを実行することなく、Web、モバイル、デスクトップでネイティブにレンダリングされるリッチでインタラクティブなユーザーインターフェースを生成することを可能にします。
</p>

</div>

!!! warning "ステータス: 早期パブリックプレビュー"
    A2UIは現在 **v0.8 (Public Preview)** 段階です。仕様と実装は動作可能ですが、現在も進化を続けています。コラボレーションを促進し、フィードバックを収集し、貢献（例：クライアントレンダラー）を募るためにプロジェクトを公開しています。変更される可能性があることにご注意ください。

## 概要

A2UIは現在 [v0.8](specification/v0.8-a2ui.md) バージョンであり、
Apache 2.0ライセンスの下で公開されています。
CopilotKitおよびオープンソースコミュニティからの貢献とともに、Googleによって提供されており、
[GitHub](https://github.com/google/A2UI)で活発に開発されています。

A2UIが解決しようとしている課題は：**AIエージェントが信頼の境界を越えて、どのように安全にリッチなUIを送信できるか？** です。

テキストのみの応答や危険なコードの実行の代わりに、A2UIはエージェントが、クライアント側で独自のネイティブウィジェットを使用してレンダリングできる**宣言的なコンポーネント記述**を送信することを可能にします。これは、エージェントが共通のUI言語を話しているようなものです。

このリポジトリには以下が含まれています：
- [A2UI仕様](specification/v0.8-a2ui.md)
- クライアント側の[レンダラー](renderers.md)（例：Angular、Flutterなど）の実装
- エージェントとクライアント間でA2UIメッセージを運ぶ[トランスポート層](transports.md)（例：A2Aなど）

<div class="grid cards" markdown>

- :material-shield-check: **設計段階からのセキュリティ**

    ---

    実行可能なコードではなく、宣言的なデータ形式です。エージェントはカタログ内の事前承認されたコンポーネントのみを要求できるため、UIインジェクション攻撃から安全です。

- :material-rocket-launch: **LLMフレンドリー**

    ---

    生成を容易にするように設計された、フラットでストリーミング可能なJSON構造です。LLMは、一度に完璧なJSONを作成しなくても、UIを段階的に構築できます。

- :material-devices: **フレームワークに依存しない**

    ---

    1つのエージェント応答があらゆる場所で機能します。同じUIをAngular、Flutter、React、またはネイティブモバイルで、独自のブランドスタイルのコンポーネントとしてレンダリングします。

- :material-chart-timeline: **プログレッシブレンダリング**

    ---

    UIの更新を生成されると同時にストリーミングします。ユーザーは応答全体が完了するのを待つ必要がなく、インターフェースがリアルタイムで構築されるのを見ることができます。

</div>

## 5分で始めよう

<div class="grid cards" markdown>

- :material-clock-fast:{ .lg .middle } **[リファレンス](quickstart.md)**

    ---

    レストラン検索デモを実行し、Gemini搭載エージェントと連携するA2UIを確認してください。

    [:octicons-arrow-right-24: 開始する](quickstart.md)

- :material-book-open-variant:{ .lg .middle } **[コアコンセプト](concepts/overview.md)**

    ---

    サーフェス、コンポーネント、データバインディング、および隣接リスト（Adjacency List）モデルを学びます。

    [:octicons-arrow-right-24: コンセプトを学ぶ](concepts/overview.md)

- :material-code-braces:{ .lg .middle } **[開発者ガイド](guides/client-setup.md)**

    ---

    A2UIレンダラーをアプリに統合するか、UIを生成するエージェントを構築します。

    [:octicons-arrow-right-24: 開発を開始する](guides/client-setup.md)

- :material-file-document:{ .lg .middle } **[プロトコルリファレンス](specification/v0.8-a2ui.md)**

    ---

    完全な技術仕様とメッセージタイプについて詳しく学びます。

    [:octicons-arrow-right-24: 仕様を読む](specification/v0.8-a2ui.md)

</div>

## 仕組み

1. **ユーザー**がAIエージェントにメッセージを送信します。
2. **エージェントがUIを記述するA2UIメッセージ**（構造 + データ）を生成します。
3. **メッセージが**クライアントアプリケーションにストリーミングされます。
4. **クライアントが**ネイティブコンポーネント（Angular、Flutter、Reactなど）を使用してレンダリングします。
5. **ユーザーが**UIと対話し、アクションをエージェントに送り返します。
6. **エージェントが**更新されたA2UIメッセージで応答します。

![エンドツーエンドのデータフロー](assets/end-to-end-data-flow.png)

## A2UIの実際の例

### ランドスケープアーキテクト（Landscape Architect）デモ

<div style="margin: 2rem 0;">
  <div style="border-radius: .8rem; overflow: hidden; box-shadow: var(--md-shadow-z2);">
    <video width="100%" height="auto" controls playsinline style="display: block; aspect-ratio: 16/9; object-fit: cover;">
      <source src="assets/landscape-architect-demo.mp4" type="video/mp4">
      ブラウザがビデオタグをサポートしていません。
    </video>
  </div>
  <p style="text-align: center; margin-top: 1rem; opacity: 0.8;">
    エージェントがランドスケープアーキテクチャ・アプリケーションのインターフェース全体を生成する様子をご覧ください。ユーザーが写真をアップロードすると、エージェントはGeminiを使用してその内容を理解し、ランドスケープの要件に合わせてカスタマイズされたフォームを生成します。
  </p>
</div>

### カスタムコンポーネント：インタラクティブなチャートとマップ

<div style="margin: 2rem 0;">
  <div style="border-radius: .8rem; overflow: hidden; box-shadow: var(--md-shadow-z2);">
    <video width="100%" height="auto" controls playsinline style="display: block; aspect-ratio: 16/9; object-fit: cover;">
      <source src="assets/a2ui-custom-compnent.mp4" type="video/mp4">
      ブラウザがビデオタグをサポートしていません。
    </video>
  </div>
  <p style="text-align: center; margin-top: 1rem; opacity: 0.8;">
    エージェントが数値の要約に関する質問に答えるためにチャートコンポーネントを選択し、応答する様子をご覧ください。次に、エージェントは場所に関する質問に答えるためにGoogleマップコンポーネントを選択します。これらは両方とも、クライアントによって提供されるカスタムコンポーネントです。
  </p>
</div>

### A2UI Composer

CopilotKitによる公開されている[A2UIウィジェットビルダー](https://go.copilotkit.ai/A2UI-widget-builder)もお試しいただけます。

[![A2UI Composer](assets/A2UI-widget-builder.png)](https://go.copilotkit.ai/A2UI-widget-builder)