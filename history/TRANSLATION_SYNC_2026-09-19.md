# 2026-09-19 翻訳同期履歴

上流 A2UI リポジトリ（`https://github.com/a2ui-project/a2ui.git`）の最新の変更（コミット `a6a2eba2` → `04e6f07f`）を反映し、`docs-ja` のドキュメントを最新化しました。

## 1. 主な変更点と翻訳反映内容

### 1-1. 概念 (Concepts)
- **カタログ (`docs/public/concepts/catalogs.md`)**
  - **バージョニングガイドラインの全面刷新**: エージェントとレンダラー間のバージョンの差異（version skew）を吸収できるよう、前方・後方互換性の維持原則を詳細化
  - **カタログスキーマの進化ルール 7箇条の追加**:
    1. 追加のみ許可（Additive Only）
    2. 削除ではなく非推奨化（Deprecate Rather than Delete）
    3. 型の不変性（Type Invariance）
    4. オープンな Enum（Open Enums）
    5. グレースフルデグラデーション（Graceful Degradation）
    6. 未知のフィールドの往復保持（Round-Trip Unknown Preservation）
    7. メジャーバージョン引き上げの原則
  - **プロパティの非推奨化ルール**: `deprecated: true` および `x-deprecated-reason` の説明と JSON スキーマ例を追加
  - **CatalogId によるメジャーバージョン引き上げと無停止移行パターン**: URI ベースのネゴシエーション手順と表を提供
  - **2段階のスキーマ検証とフォールバック戦略**: エージェント送信前検証、クライアント検証、実行時フォールバック、クライアントエラー報告（`VALIDATION_FAILED`）手順の補完
  - **インラインカタログ** 案内を追記

- **用語集 (`docs/public/concepts/glossary.md`)**
  - **Capabilities Object (機能オブジェクト) セクションの新設**: サポートカタログ集合の通知（advertise）、トランスポートメタデータ交換方式、およびプロトコルバージョンごとの名称差異を整理

### 1-2. クイックスタート (Quickstart)
- **クイックスタート (`docs/public/quickstart.md`)**
  - **Flutter レンダラーのパス更新**: 上流のディレクトリ構成変更に伴い、`renderers/flutter/` から `dart/a2ui_flutter/` へパスを更新

### 1-3. ガイド (Guides)
- **エージェントフレームワーク連携 (`docs/public/guides/a2ui-with-any-agent-framework.md`)**
  - CopilotKit スキルリンクの最新化（`a2ui-renderer` スキル → `copilotkit` スキル）

### 1-4. 仕様 (Specification)
- `specification/v1_0/` の最新変更を同期:
  - プロトコル仕様（`a2ui_protocol.md`）および進化ガイド（`evolution_guide.md`）の最新化
  - カタログ定義（`catalog_definition.json`）、共通型（`common_types.json`）、Basic カタログ（`catalog.json`）の最新化
  - 上流の構成変更に合わせた `specification/*/eval` の整理

## 2. 検証結果

- `docs-ja` 環境での `mkdocs build` 正常完了（exit code 0）。
- Markdown インクルード、生成されたすべてのドキュメントページおよびリンクの正常描画を確認。
