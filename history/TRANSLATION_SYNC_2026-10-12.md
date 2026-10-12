# 2026-10-12 翻訳同期履歴

上流 A2UI リポジトリ（`https://github.com/a2ui-project/a2ui.git`）の最新の変更（コミット `04e6f07f` → `ce03d005`）を反映し、`docs-ja` のドキュメントを最新化しました。

## 1. 主な変更点と翻訳反映内容

### 1-1. 概念 (Concepts)
- **アクション (`docs/public/concepts/actions.md`)**
  - Python SDK のディレクトリ構成変更に伴い、`A2uiSubagentMap` のリンクパスを `agent_sdks/python/...` から `python/a2ui_agent/...` へ更新
- **用語集 (`docs/public/concepts/glossary.md`)**
  - 上流の用語集全面改訂を反映: レンダラー・エージェント・モデル間のシーケンス図、A2UI SDK の概要、および 3 カテゴリ構成（プロトコル用語、エージェント用語、生成 UI 用語）を導入
  - **Function / Function call / Action** セクションの刷新: レンダラー関数とエージェント関数の区別、LLM ツール呼び出しとの比較表、関数呼び出しの 4 つの文脈（動的値バインディング、検証ルール、コンポーネント操作トリガー、RPC）およびアクションとの違いを詳細化
  - **Inference format（推論フォーマット）** セクションの新設、および応答解析フェーズ（タグのアンラップ、コンパイル）の整理
  - **推奨されるコード構成（Recommended code organization）**: Agent SDK を含む最新のアーキテクチャ図と `typescript/web_core` パスを反映

### 1-2. エコシステム・リファレンス (Ecosystem & Reference)
- **エコシステムのレンダラー (`docs/public/ecosystem/renderers.md`)**
  - コミュニティレンダラー表に `v1.0` 対応状況の列を追加
  - 新規コミュニティレンダラー 2 件の追加とハイライト翻訳:
    - `Language-Lit/material3-expressive-a2ui` (`@language-lit/material3-expressive-a2ui`)
    - `uny/a2ui-compose` (`dev.ynagai.a2ui:a2ui-material3`)
  - 上流で削除された `TanXudong-Vivo/A2UI-Android-Renderer` を削除し、`BBC6BAE9/a2ui-swift` のプラットフォーム表記を更新
  - コミュニティレンダラー掲載基準の仕様バージョン表記に `v1.0` を追加
- **A2UI の活用事例 (`docs/public/ecosystem/a2ui-in-the-world.md`)**
  - Google ADK セクションの Python agent SDK リンク（`python/a2ui_agent`）の更新、および AG2 パートナー統合セクションの追加
- **レンダラーリファレンス (`docs/public/reference/renderers.md`)**
  - 保守されているレンダラー表を Web / モバイルに分割し、**Jetpack Compose** の `v0.9.1` ステータスを `✅ アルファ版` に更新（公式ドキュメント・コードへのリンクを追加）

### 1-3. ガイド・ホーム・クイックスタート (Guides, Home & Quickstart)
- **エージェント開発ガイド (`docs/public/guides/agent-development.md`)**、**MCP 経由の A2UI (`docs/public/guides/a2ui_over_mcp.md`)**、**コンポーネントの作成 (`docs/public/guides/authoring-components.md`)**
  - Python Agent SDK のリファクタリングを反映: `A2uiSchemaManager` から `DirectJsonFormat`、`BasicCatalog`、`resolve_catalogs`、`validate_payload` を用いる最新 API コード例および `python/a2ui_agent` パスへ更新
- **A2UI 内の MCP Apps (`docs/public/guides/mcp-apps-in-a2ui.md`)**
  - `Catalog` コンストラクタ呼び出し例に `'v0.9'` バージョン引数を追加
- **レンダラーの開発 (`docs/public/guides/renderer-development.md`)**、**テーマ設定 (`docs/public/guides/theming.md`)**、**クイックスタート (`docs/public/quickstart.md`)**
  - `@a2ui/web_core` パッケージのパスを `renderers/web_core` から `typescript/web_core` へ更新
- **ホーム (`docs/public/index.md`)**
  - `A2UI Composer` のリンクを `https://a2ui-project.github.io/composer/` に更新し、`A2UI Theater` カードを削除

### 1-4. 仕様・カタログ (Specification & Catalogs)
- **ベーシックカタログ実装ガイド (`docs/public/specification/v1.0-basic-catalog-implementation-guide.md`)**
  - 上流で v1.0 ベーシックカタログがトップレベルの `catalogs/basic/v1/` ディレクトリへ移動したことに伴い、インクルードパスを `catalogs/basic/v1/basic_catalog_implementation_guide.md` へ更新
- `specification/` および `catalogs/` ディレクトリを上流の最新状態と同期

## 2. 検証結果

- `docs-ja` 環境での `mkdocs build` 正常完了（exit code 0）。
- Markdown インクルード、生成されたすべてのドキュメントページおよびリンクの正常描画を確認。
