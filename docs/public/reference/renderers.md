# レンダラー（クライアントライブラリ）

レンダラーは、A2UI JSON メッセージを各プラットフォームのネイティブ UI コンポーネントへ変換します。

[エージェント](agents.md) が A2UI メッセージを生成し、[トランスポート](../concepts/transports.md) がそれをクライアントへ届けます。クライアント側のレンダラーライブラリは、A2UI メッセージをバッファリングし、A2UI のライフサイクルを実装し、サーフェス（ウィジェット）を描画する必要があります。

カスタムコンポーネントをレンダラーへ持ち込んだり、自分の UI コンポーネントフレームワークを支える独自レンダラーを作ったりできる柔軟性があります。

## 保守されているレンダラー

| レンダラー | プラットフォーム | v0.8 | v0.9.1 | v1.0 | リンク |
|----------|----------|------|------|------|-------|
| **React** | Web | ✅ 安定版 | ✅ 安定版 | 🚧 予定 | [コード](https://github.com/a2ui-project/a2ui/tree/main/renderers/react) |
| **Lit (Web Components)** | Web | ✅ 安定版 | ✅ 安定版 | 🚧 予定 | [コード](https://github.com/a2ui-project/a2ui/tree/main/renderers/lit) |
| **Angular** | Web | ✅ 安定版 | ✅ 安定版 | 🚧 予定 | [コード](https://github.com/a2ui-project/a2ui/tree/main/renderers/angular) |
| **Flutter (GenUI SDK)** | モバイル/デスクトップ/Web | ✅ 安定版 | ✅ 安定版 | 🚧 予定 | [ドキュメント](https://docs.flutter.dev/ai/genui) · [コード](https://github.com/flutter/genui) |
| **SwiftUI** | iOS/macOS | — | — | 🚧 予定 | — |
| **Jetpack Compose** | Android | — | — | 🚧 予定 | — |

詳細は [ロードマップ](../roadmap.md) を確認してください。

## エコシステムのレンダラー

コミュニティは、さらに多くのプラットフォーム向け A2UI レンダラーを作っています。

- **[json-render](https://json-render.dev/docs/a2ui)** — Zod スキーマ経由で A2UI カタログを描画する Vercel の React ライブラリ（[比較記事](https://dipjyotimetia.medium.com/vercels-json-render-vs-google-s-a2ui-the-head-to-head-6f213cf1a23b)）
- **[A2UI-Android](https://github.com/lmee/A2UI-Android)** — Jetpack Compose のコミュニティレンダラー。20+ コンポーネント、約 15 ⭐、v0.8
- **[a2ui-react-native](https://github.com/sivamrudram-eng/a2ui-react-native)** — iOS/Android 向け React Native レンダラー。約 9 ⭐、v0.8
- **[Lynx A2UI](https://lynxjs.org/next/react/genui/a2ui.html)** — A2UI 向け ReactLynx レンダラー（v0.9）

**[エコシステムのレンダラー一覧](../ecosystem/renderers.md)** で、さらに多くのコミュニティプロジェクトや投稿方法を確認できます。

## レンダラーの仕組み

```
A2UI JSON → レンダラー → ネイティブコンポーネント → あなたのアプリ
```

1. トランスポートから A2UI メッセージを **受信** する
2. JSON を **パース** し、スキーマを検証する
3. プラットフォームのネイティブコンポーネントで **描画** する
4. アプリのテーマに合わせて **スタイルを適用** する

## レンダラーの使い方

アプリへの統合は、使いたいレンダラーのセットアップガイドに従ってください。

- **[React](../guides/client-setup.md#react)**
- **[Lit (Web Components)](../guides/client-setup.md#web-components-lit)**
- **[Angular](../guides/client-setup.md#angular)**
- **[Flutter (GenUI SDK)](../guides/client-setup.md#flutter-genui-sdk)**

## レンダラーを作る

独自プラットフォーム向けのレンダラーを作りたい場合は、次を確認してください。

- 実装予定フレームワークは [ロードマップ](../roadmap.md) を確認する
- 既存レンダラーの実装パターンを参考にする
- 詳細は [レンダラー開発ガイド](../guides/renderer-development.md) を読む

### 主な要件

- A2UI JSON メッセージ、特に隣接リスト形式の解析
- A2UI コンポーネントとネイティブウィジェットの対応付け
- データバインディングとライフサイクルイベントの処理
- 段階的な A2UI メッセージ列を処理して UI を構築・更新する
- サーバー主導更新のサポート
- ユーザーアクションのサポート

### 次のステップ

- **[クライアント設定ガイド](../guides/client-setup.md)**: 統合手順
- **[クイックスタート](../quickstart.md)**: Lit レンダラーを試す
- **[コンポーネントリファレンス](components.md)**: サポートすべきコンポーネントを確認する
