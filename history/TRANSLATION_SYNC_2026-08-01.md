# 2026-08-01 翻訳同期・Upstream反映記録

上流 A2UI リポジトリ (`https://github.com/a2ui-project/a2ui.git`) の最新変更点 (コミット `d4723f29` → `2276f8cc`、計 37 コミット) を同期し、`docs-ja` ドキュメントを更新しました。対象範囲は `docs/public/**` のうち、上流で実際に変更があった以下の 7 ファイルです。

## 1. ドキュメント更新・翻訳内容 (`docs-ja`)

- **A2UI カタログ (`docs/public/concepts/catalogs.md`)**:
  - 「レンダラーの実装」セクションのサンプルコードを刷新。旧 Angular `DynamicComponent` / `inputBinding` ベースの実装例を、`ComponentApi`（Zod スキーマ）で API を定義 → `CatalogComponent` を継承して実装 → `AngularCatalog` に登録、という新しい 3 ステップ構成に翻訳し直した。
  - Orchestrator デモが現時点で v0.8 API のままであることと、v0.9 のカタログ登録例は Angular Explorer の `DemoCatalog` を参照する旨、およびクライアント側関数の実行境界（`clientOnly` など）がカタログ定義から実行時に解決される旨の NOTE を新規翻訳して追加。
  - このセクションはこれまで日本語訳がここで途切れていた箇所だった（後述の既知の未対応事項を参照）。今回の upstream diff がちょうどこの続きを書き換えたため、diff 該当部分に限って新規翻訳し、既存訳との段差を解消した。

- **エコシステムのレンダラー (`docs/public/ecosystem/renderers.md`)**:
  - コミュニティレンダラー表に `yessGlory17/generative-mui` (`@yessglory/generative-mui-react`、React + Material UI、v0.9.1 対応) を追加。
  - 「ハイライト」節に generative-mui の詳細説明段落を新規翻訳して追加（ホストアプリの `<ThemeProvider>` に統合される点、フレームワーク非依存コアと React/MUI アダプターへの分割、オプトインの Extended Catalog、双方向データバインディング、ストリーミング耐性、セキュリティガードなど）。
  - 冒頭の `!!! note` admonition 内に、過去の編集作業で混入したとみられるツール呼び出しの残骸文字列（`<ctrl42>call:default_api:replace_file_content`）を発見し削除した。upstream diff とは無関係だが、表示が崩れる明白な不具合だったため合わせて修正。

- **カスタムコンポーネントを作成する (`docs/public/guides/authoring-components.md`)**:
  - 手順 2「コンポーネントを実装する（クライアント）」を、v0.9 の新しいコンポーネント作成パターンに合わせて全面更新。`DynamicComponent` を継承する旧方式の説明・コード例を、`ComponentApi`（Zod）による API 定義 → `CatalogComponent` を継承する実装、という 2 段階の手順に翻訳し直した。あわせて、不要になった `{% raw %}...{% endraw %}` ラップを upstream に合わせて削除。
  - 手順 3「レンダラーに登録する（クライアント）」を、`Catalog` / `DEFAULT_CATALOG` / `inputBinding` を使う旧登録方式から、`AngularCatalog` クラスを使う新しい登録方式（Eager Registration）へ翻訳更新。

- **クライアント設定ガイド (`docs/public/guides/client-setup.md`)**:
  - Angular の「設定例 (v0.9)」を、`A2UI_RENDERER_CONFIG` トークン + `A2uiRendererService` プロバイダー方式から、新しい `provideA2Ui` ヘルパー方式に更新。
  - 新設された小節「アクションハンドラーでの依存性注入」を新規翻訳（`provideA2Ui` にファクトリー関数を渡し、Angular の `inject()` でサービスを注入する例）。

- **A2UIとは何ですか？ (`docs/public/introduction/what-is-a2ui.md`)**:
  - 「例」セクションが、これまで上流の v0.8/v0.9 タブ構成になっておらず v0.8 相当の単一例のみだった（過去の同期ノートで軽微な既知差異として記録済み）。今回の diff がこのセクション内の説明文一文を書き換えたのを機に、上流と同じ「v0.8 (レガシー) / v0.9 (安定版)」タブ構成に合わせて全面的に翻訳し直した。
  - 新しい v0.9 タブの説明文を「A2UI メッセージは `createSurface` でサーフェスを初期化し、フラットなコンポーネント構造を使い、`version` フィールドを含みます。」として翻訳。

- **クイックスタート (`docs/public/quickstart.md`)**:
  - 「A2UI メッセージの構造」内 v0.9 タブの注記文を、廃止済み API（`beginRendering` → `createSurface` の置き換え）への言及を削った、より簡潔な上流の文言に合わせて更新。
  - 「コンポーネントギャラリー（エージェント不要）」の起動手順を更新。新規チェックアウトから実行する場合はまず `renderers/lit/a2ui_explorer` で `yarn build` が必要になった点を追記し、起動コマンドを `yarn start gallery` から `yarn dev` に変更。

- **リファレンス: メッセージ (`docs/public/reference/messages.md`)**:
  - `createSurface` の説明文から、廃止済み `beginRendering` との比較表現を削除し、ルートコンポーネントの決定方法（`updateComponents` 内のいずれか 1 つが `"id": "root"` を持つ）と `catalogId` が必須である点のみに絞った文言に更新。
  - `updateComponents` の「コンポーネントオブジェクト」の導入文を「v0.9 では、コンポーネントの構造はよりフラットです。」から「コンポーネントの構造はフラットです。」へ簡潔化。

## 2. 対象外の確認

- root `README.md`、`mkdocs.yaml` は、対象コミット範囲 (`d4723f29`..`2276f8cc`) の upstream diff で変更がなかったことを `git diff` で確認済み。今回は更新していない。

## 3. 既知の未対応事項（引き続きスコープ外として保留）

- `concepts/catalogs.md`：「A2UI Catalog Negotiation」以降（Naming & Versioning、Schema Validation、Inline Catalogs 等、約 240 行相当）は、305336f1 時点から続く翻訳未了のまま据え置いた。今回の upstream diff は「レンダラーの実装」セクションまでだったため、そこまでは追いついたが、それ以降は今回のスコープ外。次回以降のフォローアップ推奨。
- `introduction/what-is-a2ui.md`：「例」セクションのタブ構成は今回追いついたが、それ以外の内容（コアバリュー、設計原則など）は既存訳のまま変更していない。

## 注意点

- `specification/` フォルダ本体（生の JSON スキーマや `--8<--` で読み込まれる実体ファイル）および `eval/` フォルダはこの翻訳プロジェクトの対象外であり、今回も一切触れていない。
- 今回の同期対象は `docs/public/**` のうち upstream で実際に差分があった 7 ファイルのみ。それ以外の `docs/public/**` 配下のファイルは、upstream 側に変更がなかったため一切変更していない。
