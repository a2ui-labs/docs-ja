# 2026-08-20 翻訳同期記録

上流 A2UI リポジトリ（`https://github.com/a2ui-project/a2ui.git`）の最新変更（コミット `44a420b6` → `ca09fac3`）を同期し、`docs-ja` ドキュメントの最新化および仕様（Specification）ドキュメントの表示問題を改善しました。

## 1. 主要な改善点: 仕様（Specification）ドキュメントのインクルード対応

### 1-1. 背景と原因
- `docs/public/specification/*.md` ファイルは、上流リポジトリのルート `specification/**` ディレクトリにあるマークダウンソースを `--8<--` スニペット構文で自動インクルードする仕組みになっています。
- これまで `docs-ja` リポジトリにはルート `specification/` ディレクトリが存在しなかったため、ビルド時にスニペットが解決されず、サイト上の仕様ページで上部の案内リンクのみが表示され、本文が空になる現象が発生していました。

### 1-2. 対応内容
- 上流の最新 `specification/` ディレクトリを `docs-ja/specification/` として同期しました。
- 各仕様ページ（v0.8、v0.9、v0.9.1、v1.0 プロトコル仕様、拡張仕様、進化ガイドなど全12ファイル）で完全な本文がレンダリングされることを確認しました。

## 2. ドキュメント更新・翻訳内容 (`docs-ja`)

- **クイックスタート (`docs/public/quickstart.md`)** — 上流 PR #2107
  - `### 他の言語・フレームワーク` セクションに Flutter サンプルパス（`- **Flutter**: samples/client/flutter`）を追加しました。
- **ガイド - 任意のエージェントフレームワークで A2UI を使用 (`docs/public/guides/a2ui-with-any-agent-framework.md`)** — 上流 PR #2328
  - `mkdocs.yaml` の設定変更に伴い不要となった `{% raw %}` および `{% endraw %}` ブロックを削除しました。
- **`mkdocs.yaml` プラグイン設定**
  - 上流 PR #2328 の変更を反映し、Jinja2 マクロレンダリングのデフォルト無効化（`render_by_default: false`, `on_error_fail: true`）を設定しました。

## 3. 検証結果

- `mkdocs build` がエラーなく完了（exit code 0）。
- `site/specification/*` の全仕様ページで本文が正しくレンダリングされていることを確認。
