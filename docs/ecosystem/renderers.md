# エコシステムのレンダラー

コミュニティやサードパーティによる A2UI レンダラー実装の一覧です。

!!! note
    これらのレンダラーは A2UI チームではなく、それぞれの作者が保守しています。
    互換性、対応バージョン、メンテナンス状況は各プロジェクトで確認してください。

## コミュニティレンダラー

| レンダラー | プラットフォーム | v0.8 | v0.9 | 活動状況 | リンク |
|----------|----------|------|------|----------|-------|
| **@a2ui-sdk/react** | React（Web） | ✅ | ❌ | ![Stars](https://img.shields.io/github/stars/easyops-cn/a2ui-sdk?style=flat-square&label=⭐) ![Last commit](https://img.shields.io/github/last-commit/easyops-cn/a2ui-sdk?style=flat-square&label=updated) | [GitHub](https://github.com/easyops-cn/a2ui-sdk) · [npm](https://www.npmjs.com/package/@a2ui-sdk/react) · [Docs](https://a2ui-sdk.js.org/) |
| **A2UI-Android** | Android（Compose） | ✅ | ❌ | ![Stars](https://img.shields.io/github/stars/lmee/A2UI-Android?style=flat-square&label=⭐) ![Last commit](https://img.shields.io/github/last-commit/lmee/A2UI-Android?style=flat-square&label=updated) | [GitHub](https://github.com/lmee/A2UI-Android) |
| **a2ui-react-native** | React Native | ✅ | ❌ | ![Stars](https://img.shields.io/github/stars/sivamrudram-eng/a2ui-react-native?style=flat-square&label=⭐) ![Last commit](https://img.shields.io/github/last-commit/sivamrudram-eng/a2ui-react-native?style=flat-square&label=updated) | [GitHub](https://github.com/sivamrudram-eng/a2ui-react-native) |
| **@zhama/a2ui** | React（Web） | ✅ | ❌ | — | [npm](https://www.npmjs.com/package/@zhama/a2ui) |
| **A2UI-react** | React（Web） | ✅ | ❌ | ![Stars](https://img.shields.io/github/stars/jem-computer/A2UI-react?style=flat-square&label=⭐) ![Last commit](https://img.shields.io/github/last-commit/jem-computer/A2UI-react?style=flat-square&label=updated) | [GitHub](https://github.com/jem-computer/A2UI-react) |

### 注目のプロジェクト

これらのプロジェクトは、初期段階または実験的です。

- **[@xpert-ai/a2ui-react](https://www.npmjs.com/package/@xpert-ai/a2ui-react)** — ShadCN UI コンポーネントを使った React レンダラー（v0.0.1、2026 年 1 月公開）
- **[a2ui-3d-renderer](https://github.com/josh-english-2k18/a2ui-3d-renderer)** — A2UI 向けの実験的 Three.js/WebGL 3D レンダラー（約 2 ⭐）
- **[ai-kit-a2ui](https://github.com/AINative-Studio/ai-kit-a2ui)** — AIKit フレームワーク向けの React + ShadCN レンダラー（約 2 ⭐）

### ハイライト

**@a2ui-sdk/react** は、現時点で最も成熟したコミュニティ製 React レンダラーです。11 版の公開、Radix UI プリミティブ、Tailwind CSS スタイリング、専用ドキュメントサイトがあります。A2UI の [Discussions で告知](https://github.com/google/A2UI/discussions/489) されました。

**A2UI-Android** は重要なギャップを埋める実装です。Android 5.0 以降を対象とする唯一のコミュニティ製 Jetpack Compose レンダラーで、20+ コンポーネント、データバインディング、アクセシビリティを備えています。

**a2ui-react-native** は唯一の React Native レンダラーで、単一コードベースで iOS と Android の両方で A2UI を使えます。

### Python / PyPI

2026 年 3 月時点で、PyPI に信用できる A2UI レンダラーパッケージは見つかっていません。A2UI レンダラーはクライアント側の UI ライブラリなので、エコシステムは自然に JavaScript/TypeScript とネイティブモバイルフレームワークに集中しています。

## レンダラーの投稿

A2UI レンダラーを作成したなら、ぜひここに掲載しましょう。

### 投稿方法

1. [google/A2UI](https://github.com/google/A2UI) リポジトリを Fork する
2. このファイル (`docs/ecosystem/renderers.md`) を編集し、コミュニティレンダラー表に名前、プラットフォーム、npm パッケージ（あれば）、対応バージョン、ソースリンクを追加する
3. 自分のレンダラーについて短い説明付きで `google/A2UI` へ PR を出す
4. [GitHub Discussions](https://github.com/google/A2UI/discussions) に投稿して、作ったものをコミュニティへ知らせる。短いデモ動画があると効果的です。

参考がほしければ、リポジトリ内の **[コミュニティサンプル](https://github.com/google/A2UI/tree/main/samples)** を見てください。Angular、Lit、ADK ベースのエージェントが含まれています。

### 良いコミュニティレンダラーの条件

次の条件を満たすほど、掲載されやすく実際に使われやすくなります。

- **公開済みソースコード** があること（オープンソース推奨、MIT または Apache 2.0）
- どの **A2UI 仕様バージョン** に対応しているかを明示すること（v0.8、v0.9、または両方）
- **コアコンポーネント**（text、button、input、基本レイアウト）をカバーしていること
- インストール手順と最小使用例を含む **README** があること
- **継続的に保守** されていること。サポートを終了した場合は archived と明記すること

コミュニティレンダラーは本番品質でなくても掲載できます。実験的・初期段階のプロジェクトも「注目のプロジェクト」欄に歓迎します。
