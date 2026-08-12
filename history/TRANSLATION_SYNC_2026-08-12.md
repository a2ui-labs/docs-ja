# 2026-08-12 翻訳同期・Upstream反映記録

上流 A2UI リポジトリ (`https://github.com/a2ui-project/a2ui.git`) の最新変更点 (コミット `2276f8cc` → `43a7bdd8`、計 27 コミット) を同期し、`docs-ja` ドキュメントを更新しました。この区間で実際に差分が発生したテキストファイルは `README.md`、`mkdocs.yaml` を含む 8 ファイルで、画像アセットが 6 点追加・1 点削除されています。

今回の区間のドキュメント変更は実質的に 1 つの大きな流れであり、**A2UI Composer が CopilotKit の外部 Widget Builder から、A2UI プロジェクト自身が運営する公式ツール (`https://a2ui-project.github.io/composer/`) へ置き換わった**こと、およびそれに伴い単一ページだった `composer.md` が `composer/` ディレクトリ配下の 2 ドキュメントへ拡張されたことによるものです (上流 PR #2201、#2215)。

## 1. ドキュメント更新・翻訳内容 (`docs-ja`)

- **A2UI Composer (`docs/public/composer.md` → `docs/public/composer/index.md`)**:
  - 既存の `composer.md` (CopilotKit Widget Builder を案内する 14 行のドキュメント) を削除し、上流で新設された `composer/index.md` を全文翻訳して新規作成しました。
  - 翻訳範囲: Composer を使い始める 3 ステップ、Composer UI の各パネル解説 (Gemini アシスタント、レンダリングされた A2UI プレビュー、A2UI JSON エディター、下部のデバッグ・検査タブの Data Model / Events / Errors / Raw Messages)、コンポーネントギャラリー、設定 (レンダラーアプリケーションの選択、Gemini API キーの取得手順および Web Crypto API によるローカル暗号化保存の説明)、進行中の作業、Raw Messages のメッセージ一覧 (`RENDERER_READY`、`A2UI_CATALOG`、`COMPONENT_USAGES`、`DATA_MODEL_CHANGE`、`LLM_REQUEST`、`LLM_RESPONSE`)。
  - 上流原文の `#gemini-api-key` アンカーは日本語見出し (「Gemini API キー」) からは同じものが生成されないため、`attr_list` 拡張を使って見出しに `{#gemini-api}` の明示的アンカーを付与し、本文中のリンクもそれに合わせました (`mkdocs.yaml` で `attr_list` が有効であることを確認済み)。

- **A2UI Composer 連携マニュアル (`docs/public/composer/composer_renderer_integration.md`、新規)**:
  - 上流で新設されたドキュメントを全文翻訳しました。背景 (Composer は特定のカタログ・レンダラーを知らず、「レンダラーアプリケーション」を iframe にホストして postMessage で通信する)、ブリッジ (bridge) とフレームワーク別ラッパー、サンプルレンダラーアプリ 3 種 (Angular/Lit/React) の案内、Angular ベースのレンダラーアプリ作成手順 (依存関係の追加 → ラッパーコンポーネントの作成 → `provideA2uiSandbox` によるブートストラップ)、Zone / Zoneless の変更検知互換性に関する NOTE を含みます。
  - リポジトリの慣例に従い、コードブロックとコード内コメントは上流原文のまま維持しました。

- **概念 - 用語集 (`docs/public/concepts/glossary.md`)**:
  - 上流に新しく追加された 4 つの用語を翻訳して反映しました (上流 PR #2021)。
    - `Catalog Transformer` (および下位セクション「必要な理由」「例」): システムプロンプト生成や検証スキーマのコンパイル前にカタログをフィルタリング・変形するルールセット。コンテキストウィンドウのトークン最適化、タスク固有の機能ガードレール、モデルシグネチャの削減という 3 つの動機と、`ComponentPruningTransformer` / `FunctionPruningTransformer` の例を含みます。
    - `A2UI Tag`、`Tag Unwrapping`、`Compilation`: LLM レスポンスのパースパイプラインに関する用語。
  - 上流の配置順をそのまま踏襲し、`Catalog Transformer` は `Basic Catalog` と `Surface` の間に、残りの 3 つは `Surface` と `エージェントアーキテクチャ` の間に挿入しました。
  - 用語の見出しは既存ドキュメントの慣例 (`Catalog`、`Basic Catalog`、`Surface`、`Action` などプロトコル固有の用語は英語のまま) に合わせ、英語表記を維持しました。

- **ホーム (`docs/public/index.md`)**:
  - 下部の「A2UI Composer」セクションを、CopilotKit Widget Builder の案内 + スクリーンショットリンクから、公式 Composer へのリンクと新しい Composer ドキュメント (`./composer/index.md`) へのリンクに差し替えました。
  - 上流原文と同様に `A2UI-widget-builder.png` の画像埋め込みは削除しました。

- **ガイド - A2UI における MCP Apps (`docs/public/guides/mcp-apps-in-a2ui.md`)**:
  - inner iframe の権限説明に、禁止項目として `allow-top-navigation` と `allow-top-navigation-by-user-activation` を追加しました。
  - 「最上位ウィンドウのハイジャック対策」の項目を新規翻訳して追加しました (frame busting 攻撃によって host ウィンドウがリダイレクトされるのを防ぐという説明、上流 PR #2218)。

## 2. ルートファイルとナビゲーション

- **`README.md`**: 「開始パス」表の Composer 行から `Widget Builder` (`go.copilotkit.ai`) のリンクを削除し、Composer の URL を `https://a2ui-composer.ag-ui.com/` → `https://a2ui-project.github.io/composer/` に更新しました。表の区切り線の桁も上流に合わせて調整しています。
- **`mkdocs.yaml`**: `A2UI Composer ⭐: composer.md` の単一項目を、上流と同じ 2 階層構造に変更しました (`composer/index.md` + `Composer 連携: composer/composer_renderer_integration.md`)。

## 3. アセット

- 追加 (上流からコピー、6 点): `composer_workspace.png`、`composer_components_gallery.png`、`composer_editor_tooltip.png`、`composer_paperclip.png`、`composer_camera.png`、`composer_copy.png`
- 削除: `A2UI-widget-builder.png` (参照していたドキュメントがすべて削除されたため)

## 4. 検証

- `mkdocs build` は、ローカルに `mkdocs-material`、`mkdocs-macros-plugin`、`mkdocs-mermaid2-plugin` が入っていないため実行できませんでした。代わりにスクリプトで以下を確認しています。
  - `mkdocs.yaml` の `nav` が指すファイルがすべて存在すること → 問題なし
  - 新規作成した composer ドキュメント 2 件の相対リンク・画像パスがすべて解決すること → 全件 OK
  - 削除した `composer.md` / `A2UI-widget-builder.png` を参照する残存リンクがないこと → なし

## 5. 対象外 (作業していない範囲)

- `docs/contributing/**`、`eval/`、トップレベルの `specification/` (原文の仕様 JSON など) — 従来の方針どおり完全に除外しました。
- 今回の差分対象以外の `docs/public/**` のファイルには手を加えていません。
- 今回の上流区間の変更の大半 (計 27 コミット、483 ファイル) は Swift/SwiftUI レンダラーの新規追加、v1.0 仕様スキーマの変更、クライアントのセキュリティ強化などコード・仕様領域であり、`docs/public/**` には影響していません。

## 6. 既知の未対応事項 (今後の整理を推奨)

- **`introduction/agent-ui-ecosystem.md` の構造ドリフト**: 上流の今回の差分は「A2UI vs AG-UI / CopilotKit」節から「CopilotKit チームは A2UI Composer にも貢献している」という記述と `../composer.md` へのリンクを削除するものでしたが、`docs-ja` 版の同ファイルにはそもそもこの節が存在しません (日本語版は「A2UI が提供するもの」「統合フレームワークとの関係」という別構成の古いバージョンのままで、上流の「A2UI vs MCP Apps」「A2UI vs AG-UI / CopilotKit」「A2UI vs ChatKit」「組み合わせて使う」という構成が反映されていません)。今回の差分は適用対象がないため無変更としましたが、ファイル単位での全面的な再翻訳が必要です。なお `docs-ko` 版は上流構成に追随済みで、今回の削除を適用しています。
- 上流の `docs/public/index.md` と `docs/public/guides/a2ui-with-any-agent-framework.md` には、いまだに旧 Composer URL (`https://a2ui-composer.ag-ui.com/`) が残っています。上流自体の不整合であるため今回は上流に追随し、JA 訳も変更していません。上流で整理され次第、合わせて反映が必要です。
- リンク検査スクリプトにより、`docs-ja` 全体で上流リポジトリのソースパスを指す相対リンク (`../../../samples/...`、`../../../renderers/...`、`../specification/...` など) が多数解決できないことを確認しました。`docs-ja` はドキュメントのみのリポジトリであるため構造的に発生する問題で、今回の差分とは無関係な既存の課題です (HEAD 時点でも同様に存在することを確認済み)。今後、これらを `https://github.com/a2ui-project/a2ui/blob/main/...` 形式の絶対 URL へ一括置換することを推奨します。`guides/client-setup.md` や `guides/mcp-apps-in-a2ui.md` など一部のファイルはすでに絶対 URL 方式に整理されています。
- 前回の記録 (`2026-08-01`) で挙げた未対応事項 — `concepts/catalogs.md` の上流に対する省略状態、`guides/`・`reference/`・`concepts/` の残りのファイルにおける v0.8 / v0.9 タブ構造の反映状況 — は、今回の差分範囲外のためそのまま残っています。
