# 翻訳同期ログ 2026-07-19

upstream `a2ui-project/a2ui` の `305336f1` → `3708c069`（223 コミット分）を追従した同期作業です。

## 作業単位

1. `docs/` 以下の構成をアップストリームの `docs_dir: docs/public` 変更に合わせて `git mv` で移設した。`docs/glossary.md` は `docs/public/concepts/glossary.md` へ（`concepts` フォルダへ入れ直す形で）移設した。`docs/scripts/**` のみ現在地に残した。
2. 新規バイナリアセット (`assets/favicon.svg`) を追加し、`docs/scripts/convert_docs.py` / `test_convert_docs.py` を上流から無変換でコピーした。
3. `hooks.py` を上流の最新版に合わせて更新した（`docs/public` 再編、`--8<--` スニペット展開、バージョン付きラッパーページの相互リンク解決、GitHub ベース URL の `a2ui-project/a2ui` 化に対応するため）。
4. `mkdocs.yaml` を更新した。
   - `docs_dir: docs/public` を追加。
   - 「概念」ナビに「用語集」を追加。
   - 「ガイド」内の MCP 関連 3 ページを「A2UI + MCP」グループへネスト。
   - 「仕様」ナビを v1.0（候補版）・v0.9.1（現行版）・v0.9（旧安定版）・v0.8（レガシー）の 4 系統に再編。
   - `repo_name` / `repo_url` / `edit_uri` / `favicon` を上流の変更に合わせて更新。
   - `exclude_docs` に `specification/v*/**/*.md` を追加し、コメントアウトされていた `llmstxt` プラグイン設定ブロックを削除。
5. 全体の翻訳作業は、ファイル群ごとに並行するサブエージェントへ分担し、最後に本セッションで統合・検証した。
   - `README.md`、`index.md`、`quickstart.md`、`roadmap.md`：v0.9.1 現行/v1.0 候補のステータス反映、Corepack/Yarn ベースのセットアップ手順への刷新、`a2ui-project/a2ui` へのリポジトリ URL 更新、AG-UI 表記統一を反映した。
   - `concepts/*`、`guides/*`、`introduction/*`、`reference/*`、`ecosystem/*`：上流の差分を翻訳し、既存の正しい訳文はそのまま維持した。
   - `ecosystem/renderers.md`、`guides/a2ui-with-any-agent-framework.md`、`guides/renderer-development.md`：上流で全面書き換えされたため、新しい英語版を一から翻訳した。
   - `specification/v0.8-*.md`、`v0.9-*.md`：上流の最新フォーマット（バージョンバッジ span + Living Document 注記 + 関連ドキュメント一覧）に合わせて全面更新。
   - `specification/v0.9.1-*.md`、`v1.0-*.md`（計 8 ファイル）：新規ページとして翻訳。
   - `concepts/glossary.md`：新しい mermaid シーケンス図を日本語化し、`docs/` 直下から `concepts/` へ移動したことに伴う相対リンク（`data-flow.md`、`data-binding.md`、`actions.md`、`renderers/*`、`specification/*` など）をすべて上流の新しいパスに合わせて修正した。

## 既存の翻訳負債への対応

作業中、一部のページは 305336f1 時点の英語版とも既に乖離した独自内容（フォーマットの古さ、または独自に要約・再構成された内容）になっていることが判明した。今回の同期範囲を超えるが、実利用上の正確性を優先し、以下のページは現行 (3708c069) の英語版から新規に全訳し直した。

- `README.md`、`index.md`、`quickstart.md`、`roadmap.md`（一部セクションが上流に存在しない独自追加だったため、上流と一致する構成に揃えた）
- `specification/` 配下の v0.8 / v0.9 スタブページ（旧フォーマットのまま放置されていた）
- `guides/agent-development.md`、`guides/client-setup.md`、`guides/theming.md`（上流と無関係な独自ガイドになっていた）
- `reference/components.md`、`reference/messages.md`（上流の完全なリファレンス内容の代わりに大幅に要約された独自ページになっていた）
- `concepts/components.md`、`concepts/data-binding.md`（同上）

## 既知の未対応事項（今回のスコープ外として保留）

- `concepts/catalogs.md`：上流後半（Catalog Negotiation、Naming & Versioning、Schema Validation、Inline Catalogs 等、約 215 行相当）が翻訳されていない状態が 305336f1 以前から続いている。
- `concepts/overview.md`、`concepts/transports.md`、`concepts/data-flow.md`：意図的に簡略化された既存構成を維持しつつ、今回の差分該当箇所のみ追記した。上流の全セクションはまだカバーしていない。
- `introduction/what-is-a2ui.md`：「例」セクションが v0.8 相当の単一例のみで、上流の v0.8/v0.9 タブ構成になっていない（軽微、内容自体は誤りではない）。
- `guides/a2ui_over_mcp.md`：上流が動画・Prerequisites・Quick Start 実行手順などを含む約 419 行に拡張されたのに対し、既存の日本語版（約 240 行）は要約版のまま。今回の差分に該当する箇所（MIME タイプ変更、ツール名変更など）のみ反映した。
- `reference/components.md` の "Property / Type / Required / Description" のような表ヘッダーは英語のまま残し、セル内の説明文のみ翻訳した（既存の他ページとの表記揺れが残る可能性あり）。

いずれも上流との構造的な乖離が大きく、この同期の対象コミット範囲（305336f1..3708c069）の差分そのものとは独立した、より大きな追いつき作業が必要になる。次回以降のフォローアップ推奨。

## 注意点

- `specification/` フォルダ本体（生の JSON スキーマや `--8<--` で読み込まれる実体ファイル）および `eval/` フォルダはこの翻訳プロジェクトの対象外であり、今回も一切触れていない。
- `docs/public/CNAME` は元々存在せず、今回も追加していない。
- `docs/scripts/README.md` は上流で変更がなかったため、そのまま維持した。
