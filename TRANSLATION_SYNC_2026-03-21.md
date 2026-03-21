# 翻訳同期ログ 2026-03-21

## 作業単位

1. `mkdocs.yaml` を英語版の最新ナビゲーションに合わせて更新し、旧パス向けの redirect を追加した。
2. `index.md` を最新の英語版構成に合わせて日本語化し、仕様バージョン表と新しい導線を反映した。
3. `quickstart.md` を新しい英語版に合わせて全面更新し、`npm run demo:all` ベースの手順に揃えた。
4. 新規ページを追加した。
   - `docs/concepts/catalogs.md`
   - `docs/concepts/client_to_server_actions.md`
   - `docs/concepts/transports.md`
   - `docs/reference/agents.md`
   - `docs/reference/renderers.md`
   - `docs/ecosystem/community.md`
   - `docs/ecosystem/renderers.md`
   - `docs/ecosystem/a2ui-in-the-world.md`
   - `docs/guides/a2ui_over_mcp.md`
5. 旧パスのページは新しい構成に統合できるものを新パスへ移し、旧ルートは redirect 対象にした。
6. `docs/index.md` の埋め込み動画参照を、上流の命名に合わせて `assets/a2ui-custom-component.mp4` へ更新した。

## 変更のポイント

- 新しい英語版の文書構造に合わせて、`concepts` / `reference` / `ecosystem` の分割を導入した。
- 既存の日本語訳は、再利用できるものは活かしつつ、必要なところだけ最新内容へ合わせた。
- 仕様リンクやサンプルリンクは、元の英語版の意図を崩さない範囲で維持した。

## 注意点

- `docs/concepts/catalogs.md` と `docs/guides/a2ui_over_mcp.md` には、元の英語版由来の未完了/将来予定の箇所が残っている。
- `A2UI/docs` 側に存在する一部の JSON Schema 参照は、現行リポジトリ内には実体がないため、そのままのリンクを維持している。
