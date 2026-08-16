# 2026-08-16 翻訳同期・Upstream反映記録

上流 A2UI リポジトリ (`https://github.com/a2ui-project/a2ui.git`) の最新変更点 (コミット `43a7bdd8` → `44a420b6`、計 21 コミット) を同期し、`docs-ja` ドキュメントを更新しました。

今回の区間の上流コミットは大半がコード・仕様領域 (v1.0 仕様スキーマの拡張、`a2ui_core` / `a2ui_agent` の移管、conformance テストの再配置、CSP・iframe のセキュリティ強化、スクリプト整備など) であり、`docs/public/**` で実際に差分が発生したのは Markdown 3 ファイルとスタイルシート 1 ファイル、およびルートの `mkdocs.yaml` の計 5 ファイルです。ドキュメントやアセットの新規追加・削除はありません。

## 1. ドキュメント更新・翻訳内容 (`docs-ja`)

- **コンセプト - カタログ (`docs/public/concepts/catalogs.md`)** — 上流 PR #2184 (issue #2152、`CatalogId` / `Id` の一貫性整理)
  - カタログのサンプル JSON に `catalogId` フィールドを追加しました。上流と同じく 3 つのサンプルブロック (`hello_world`、`hello_world_with_all_basic`、`hello_world_with_some_basic`) すべてに反映しています。
  - 上流では `### CatalogId Naming Convention` に **JSON Schema との互換性 (`$id` と `catalogId`)** の項目が追加されました。しかし `docs-ja` には、この項目が属する `## Catalog Naming & Versioning` セクション自体が存在せず、翻訳を配置する場所がない状態でした。
    - 今回の差分を反映できるよう、**`## カタログの命名とバージョニング` セクションと、その配下の `### CatalogId の命名規則` サブセクションを新規に翻訳して追加**しました (導入文 + 形式 / 目的 / 実行時のフェッチは行わない / JSON Schema との互換性 の 4 項目)。
    - 配置は上流の並び順に従い、`### レンダラーの実装` の後ろ (ファイル末尾) としました。
    - 見出し内の `CatalogId` は、既存の慣例 (プロトコル固有の識別子は英語のまま残す) に合わせて英語表記を維持しています。

- **ガイド - A2UI における MCP Apps (`docs/public/guides/mcp-apps-in-a2ui.md`)** — 上流 PR #2266、#2267
  - inner iframe のセキュリティ項目に「ハイパーリンクによる情報流出対策」を新規に翻訳して追加しました (`allow-popups` を除外し、リンクによるナビゲーションを捕捉することで、新たに開かれたウィンドウへのクリックジャッキングを介した情報の持ち出しを防げる、という内容)。

- **ガイド - レンダラーの開発 (`docs/public/guides/renderer-development.md`)** — 上流 PR #2210 (v1.0 の双方向関数呼び出しの導入)
  - `v1.0 (Candidate)` タブのプロトコル要件を、上流の改訂に合わせて差し替えました。
    - `- **アクションレスポンス(RPC)**` → `- **方向別の関数呼び出し(RPC)**`: `actionResponse` の処理に関する記述を、エージェントからの `callRendererFunction` を処理し `rendererFunctionResponse` (または `error`) を返す、という記述に置き換え。
    - `**クライアントからサーバーへ**`: `actionId` の生成・付与と `wantResponse: true` のサポートという 2 項目を削除し、リモートでの関数実行のためにエージェントへ `callAgentFunction` メッセージを送出することをサポートする、という項目に置き換え。
  - `a2uiClientCapabilities` の項目と `**Capabilities**` セクションは上流と同じく変更していません。

## 2. ルートファイルとスタイルシート

上流 PR #2227 (著作権表示の標準化) に合わせて、ライセンスヘッダーを上流と同一の形に正規化しました。ドキュメント本文への影響はありません。

- **`docs/public/stylesheets/custom.css`**: コメント開始記号 `/**` → `/*`、`Copyright 2025` → `Copyright 2024`、ライセンス URL `http://` → `https://`
- **`mkdocs.yaml`**: `# Copyright 2025 Google LLC` → `# Copyright 2024 Google LLC`

`mkdocs.yaml` の `nav` 構造については、今回の区間で変更はありません。

## 3. 付随修正: マクロエラーで壊れていた 3 ページの復旧

今回の区間から実際に `mkdocs build` で検証したことにより、**本文ではなくエラーページとしてビルドされていたドキュメント 3 件**を発見し、あわせて修正しました。いずれも今回の差分とは無関係な既存の問題です。

共通の対応は、該当ドキュメントの先頭に `render_macros: false` の YAML front matter を追加することです。3 件とも実際のマクロは使用していないため、副作用はありません。

### 3-1. `Macro Rendering Error` — 上流に由来するバグ

- **対象**: `concepts/catalogs.md`、`guides/authoring-components.md`
- **原因**: 両ドキュメントの Angular テンプレートのコードブロックに含まれる `{{ message() }}` / `{{ title() }}` を、`mkdocs-macros-plugin` (Jinja2) がマクロ変数として解釈し `UndefinedError` が発生します。コードブロック内であっても、マクロプラグインは Markdown パースより前の段階でテキスト全体をレンダリングするため保護されません。
- **範囲**: 上流 A2UI リポジトリをそのままビルドしても同一の症状が再現する**上流自体のバグ**であり、`docs-ko` でも同様に発生します。上流には存在しないローカル修正のため、上流側が独自に修正した場合はその方式に合わせて整理する必要があります。
- この修正を入れるまでは、今回翻訳した `catalogs.md` の変更分がビルド成果物にまったく反映されない状態でした。

### 3-2. `Macro Syntax Error` — 前回同期 (`2026-08-12`) で混入した回帰

- **対象**: `composer/index.md`
- **原因**: 前回の同期で日本語見出しのアンカーを揃えるために追加した `### Gemini API キー {#gemini-api}` の `{#...}` を、Jinja2 が**コメント開始タグ (`{# ... #}`) として解釈**し、`Missing end of comment tag` エラーが発生していました。その結果、Composer ドキュメント全体がエラーページに置き換わっていました。
- **範囲**: 上流にはこのアンカーが存在しないため上流では再現しない、`docs-ko` / `docs-ja` 固有の回帰です。
- **結果**: 修正後、意図していた `id="gemini-api"` アンカーと本文内リンクがいずれも正常に機能することをビルド成果物で確認しました。
- **今後の注意**: `attr_list` のアンカー (`{#...}`) を使うドキュメントには、あわせて `render_macros: false` を付与する必要があります。

## 4. 検証

`mkdocs-material` などのドキュメント依存を一時的な venv に導入し、今回の区間からは実際のビルドで検証しました。

- `mkdocs build` は成功 (exit 0)。変更前の状態 (`git stash`) と変更後の状態で警告一覧を比較し、**今回の作業で新たに発生した警告がない**ことを確認しました。
- 上記 3 項の修正により、マクロエラーの警告 3 件がすべて解消されました (警告 8 件 → 5 件)。
- ビルド成果物の HTML 上で、以下を直接確認しました。
  - `concepts/catalogs`: 見出しが `A2UI カタログ` として正常にレンダリングされること、`CatalogId の命名規則` / `JSON Schema との互換性` の新規セクションが表示されること、コードブロックの `catalogId` 行 (3 箇所) がレンダリングされること
  - `guides/authoring-components`、`composer/index`: 正常なレンダリングに復旧したこと (`composer/index` は `id="gemini-api"` アンカーと内部リンクまで確認)
  - `guides/mcp-apps-in-a2ui`: 新規の「ハイパーリンクによる情報流出対策」項目が表示されること
  - `guides/renderer-development`: `callRendererFunction` / `callAgentFunction` / `方向別の関数呼び出し` が表示され、`=== "v1.0 (Candidate)"` タブのインデントが保たれていること
- `mkdocs.yaml` の `nav` 参照先ファイルの存在、変更ファイルのコードフェンスの対応、`catalogs.md` 内の JSON ブロックのパース → 問題なし
- `--strict` ビルドは、後述 5 項の既存リンク警告により引き続き失敗します (今回の作業とは無関係で、変更前も同様)。

## 5. 対象外 (未対応)

- `docs/contributing/**`、`eval/`、ルート直下の `specification/` (原文の仕様 JSON、`specification/v1_0/docs/evolution_guide.md` など) — 従来の方針どおり完全に除外しました。
  - 今回の区間では上流 PR #2235 により `specification/v1_0/docs/evolution_guide.md` が改訂されています (41 行追加 / 27 行削除) が、`docs/public/specification/v1.0-evolution-guide.md` は当該ファイルを `--8<--` で取り込むだけの stub であり、`docs-ja` には原文の仕様ソースが存在しないため対象外としました。
- 今回の差分対象以外の `docs/public/**` のファイルには手を加えていません。

## 6. 既知の残課題 (今後の整理を推奨)

- **`concepts/catalogs.md` の上流に対する省略状態 (継続)**: 上流 475 行に対し `docs-ja` は 309 行です。今回は `## カタログの命名とバージョニング` の導入部と `### CatalogId の命名規則` のみを追加したため、以下のセクションは依然としてまるごと欠落しています。
  - `## A2UI Catalog Negotiation` (3 ステップのネゴシエーション手順全体)
  - `### Versioning Guidelines`、`### Graceful Degradation`、`### Versioning with CatalogId`、`### Handling Migrations`
  - `## A2UI Schema Validation & Fallback` (2 フェーズバリデーション、クライアントからサーバーへのエラー報告)
  - `## Inline Catalogs`
- **相対リンクが解決できない問題 (継続)**: `docs-ja` はドキュメント専用リポジトリのため、`../../../samples/...`、`../../../renderers/...`、`../specification/...` といった上流ソースへのパスが解決されません。3 項の修正後に残る警告 5 件はすべてこれが原因 (`concepts/glossary.md`、`concepts/components.md`) であり、`--strict` ビルドが失敗する理由でもあります。`https://github.com/a2ui-project/a2ui/blob/main/...` の絶対 URL への一括置換を推奨します。
- **上流ドキュメント自体の矛盾 (新規発見)**: `guides/mcp-apps-in-a2ui.md` の権限の項目には依然として `sandbox="allow-scripts allow-forms allow-popups allow-modals"` と `allow-popups` を含む記述が残っている一方、今回追加された「ハイパーリンクによる情報流出対策」の項目は `allow-popups` を除外するよう説明しています。上流の原文どおりに翻訳しており、上流側で整理された際に併せて反映する必要があります。
- **上流に残る旧 Composer URL (継続)**: `docs/public/index.md`、`guides/a2ui-with-any-agent-framework.md` の `https://a2ui-composer.ag-ui.com/` は、今回の区間でも整理されていません。
- 直前の記録 (`2026-08-12`) のその他の残課題 (`guides/`・`reference/`・`concepts/` の各ファイルにおける v0.8 / v0.9 タブ構成の反映状況) は、今回の差分の範囲外のため未対応のまま残っています。
