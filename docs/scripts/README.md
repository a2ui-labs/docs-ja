# ドキュメント変換スクリプト

このディレクトリには、**MkDocs** のビルドプロセスに向けてドキュメントを準備するためのユーティリティスクリプトが含まれています。

## 目的

GitHub 上でもホストされたサイト上でも読みやすい体験を提供するため、このプロジェクトでは **GitHub-flavored Markdown** を主要なソース形式として使用しています。このスクリプトは、ビルドパイプライン内で GitHub のネイティブ構文を **MkDocs 互換構文** に変換します。特に `pymdown-extensions` 向けの変換を行います。

## 対応している変換(一方向)

このスクリプトは **GitHub Markdown → MkDocs Syntax** の一方向変換を行います。

### アラート/Admonition 変換

このスクリプトは次の変換を処理します。

- GitHub はアラートに blockquote ベースの構文を使います。
- MkDocs は色付きの callout ボックスをレンダリングするために `!!!` または `???` 構文を必要とします。

## 変換の実行

変換はビルドパイプラインの一部として実行されます。追加の手順は不要です。手動で変換を実行する必要がある場合は、リポジトリルートで `convert_docs.py` スクリプトを実行できます。

```bash
python docs/scripts/convert_docs.py
```

### 例

- **ソース(GitHub-flavored Markdown):**

    ```markdown
    > ⚠️ **Attention**
    >
    > This is an alert.
    ```

- **変換後(MkDocs Syntax):**

    ```markdown
    !!! warning "Attention"
    This is an alert.
    ```
