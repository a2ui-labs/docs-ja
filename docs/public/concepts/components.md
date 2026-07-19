# コンポーネントと構造

A2UI は、コンポーネントの階層構造に**隣接リスト(adjacency list)モデル**を採用しています。ネストされた JSON ツリーの代わりに、コンポーネントは ID 参照を持つフラットなリストとして表現されます。

## なぜフラットなリストなのか

**従来のネスト方式:**

- LLM は一度の生成で完璧なネスト構造を作らなければなりません
- 深くネストされたコンポーネントの更新が困難です
- 段階的なストリーミングが難しくなります

**A2UI の隣接リスト:**

- フラットな構造で、LLM が生成しやすい
- コンポーネントを段階的に送信できる
- 任意のコンポーネントを ID で更新できる
- 構造とデータを明確に分離できる

## 隣接リストモデル

=== "v0.8"

    ```json
    {
      "surfaceUpdate": {
        "components": [
          {
            "id": "root",
            "component": {
              "Column": {
                "children": { "explicitList": ["greeting", "buttons"] }
              }
            }
          },
          {
            "id": "greeting",
            "component": {
              "Text": { "text": { "literalString": "Hello" } }
            }
          },
          {
            "id": "buttons",
            "component": {
              "Row": {
                "children": { "explicitList": ["cancel-btn", "ok-btn"] }
              }
            }
          },
          {
            "id": "cancel-btn",
            "component": {
              "Button": {
                "child": "cancel-text",
                "action": { "name": "cancel" }
              }
            }
          },
          {
            "id": "cancel-text",
            "component": {
              "Text": { "text": { "literalString": "Cancel" } }
            }
          },
          {
            "id": "ok-btn",
            "component": {
              "Button": {
                "child": "ok-text",
                "action": { "name": "ok" }
              }
            }
          },
          {
            "id": "ok-text",
            "component": {
              "Text": { "text": { "literalString": "OK" } }
            }
          }
        ]
      }
    }
    ```

=== "v0.9 and later"

    ```json
    {
      "version": "v0.9.1",
      "updateComponents": {
        "surfaceId": "main",
        "components": [
          {
            "id": "root",
            "component": "Column",
            "children": ["greeting", "buttons"]
          },
          {
            "id": "greeting",
            "component": "Text",
            "text": "Hello"
          },
          {
            "id": "buttons",
            "component": "Row",
            "children": ["cancel-btn", "ok-btn"]
          },
          {
            "id": "cancel-btn",
            "component": "Button",
            "child": "cancel-text",
            "action": { "event": { "name": "cancel" } }
          },
          {
            "id": "cancel-text",
            "component": "Text",
            "text": "Cancel"
          },
          {
            "id": "ok-btn",
            "component": "Button",
            "child": "ok-text",
            "action": { "event": { "name": "ok" } }
          },
          {
            "id": "ok-text",
            "component": "Text",
            "text": "OK"
          }
        ]
      }
    }
    ```

    v0.9 以降では、よりフラットなコンポーネント形式を採用しています。ネストされた `{"Text": {...}}` の代わりに `"component": "Text"` を使用し、children は `{"explicitList": [...]}` ではなく単純な配列になります。

コンポーネントは、ネストではなく ID によって子を参照します。

## コンポーネントの基本

すべてのコンポーネントは、以下を持ちます。

1. **ID**: 一意の識別子(`"welcome"`)
2. **Type**: コンポーネントの種類(`Text`、`Button`、`Card`)
3. **Properties**: その種類に固有の設定

=== "v0.8"

    ```json
    {
      "id": "welcome",
      "component": {
        "Text": {
          "text": { "literalString": "Hello" },
          "usageHint": "h1"
        }
      }
    }
    ```

=== "v0.9 and later"

    ```json
    {
      "id": "welcome",
      "component": "Text",
      "text": "Hello",
      "variant": "h1"
    }
    ```

## Basic Catalog

開発者がすばやく始められるように、A2UI チームは [Basic Catalog](../specification/v0_9_1/catalogs/basic/catalog.json) を維持しています。

これは、汎用的なコンポーネント(Button、Input、Card など)の基本セットを含む、あらかじめ定義されたカタログファイルです。特別な「種類」のカタログというわけではなく、オープンソースのレンダラーが用意されている、単なる 1 つのバージョンのカタログにすぎません。

サンプル付きの完全なコンポーネントギャラリーについては、[コンポーネントリファレンス](../reference/components.md) を参照してください。

## 静的な子要素と動的な子要素

=== "v0.8"

    **静的(`explicitList`)** - 子 ID の固定リスト:

    ```json
    {
      "children": {
        "explicitList": ["back-btn", "title", "menu-btn"]
      }
    }
    ```

    **動的(`template`)** - データ配列から子要素を生成:

    ```json
    {
      "children": {
        "template": {
          "dataBinding": "/items",
          "componentId": "item-template"
        }
      }
    }
    ```

    `/items` の各アイテムについて `item-template` をレンダリングします。詳細は [データバインディング](data-binding.md) を参照してください。

=== "v0.9 and later"

    **静的** - 子 ID の固定リスト:

    ```json
    {
      "children": ["back-btn", "title", "menu-btn"]
    }
    ```

    **動的** - データ配列から子要素を生成:

    ```json
    {
      "children": {
        "path": "/items",
        "componentId": "item-template"
      }
    }
    ```

    `/items` の各アイテムについて `item-template` をレンダリングします。詳細は [データバインディング](data-binding.md) を参照してください。

## 値のハイドレーション

コンポーネントは、次の 2 つの方法で値を取得します。

=== "v0.8"

    - **Literal(リテラル)** - 固定値: `{"text": {"literalString": "Welcome"}}`
    - **Data-bound(データバインド)** - データモデルから取得: `{"text": {"path": "/user/name"}}`

=== "v0.9 and later"

    - **Literal(リテラル)** - 固定値: `{"text": "Welcome"}`
    - **Data-bound(データバインド)** - データモデルから取得: `{"text": {"path": "/user/name"}}`

LLM は、リテラル値を持つコンポーネントを生成することも、動的なコンテンツのためにデータパスへバインドすることもできます。

## Surface の構成

コンポーネントは **Surface**(ウィジェット)を構成します。

=== "v0.8"

    1. LLM が `surfaceUpdate` でコンポーネント定義を生成します
    2. LLM が `dataModelUpdate` でデータを投入します
    3. LLM が `beginRendering` でレンダリングを指示します
    4. クライアントがすべてのコンポーネントをネイティブウィジェットとしてレンダリングします

=== "v0.9 and later"

    1. LLM が `createSurface` で Surface を作成します(カタログを指定。v1.0 では初期データモデルとコンポーネントも含められます)
    2. LLM が `updateComponents` でコンポーネント定義を生成します
    3. LLM が `updateDataModel` でデータを投入します
    4. クライアントがすべてのコンポーネントをネイティブウィジェットとしてレンダリングします

Surface は、完結した一貫性のある UI(フォーム、ダッシュボード、チャットなど)です。

## インクリメンタルな更新

インクリメンタルな更新は、次の操作をサポートします。

- **Add(追加)** - 新しい ID を持つ新しいコンポーネント定義を送信します
- **Update(更新)** - 既存の ID と新しいプロパティを持つコンポーネント定義を送信します
- **Remove(削除)** - 削除したい ID を除外するよう親の `children` リストを更新します

このフラットな構造により、あらゆる更新が ID ベースのシンプルな操作になります。

## 独自カタログの定義

Basic Catalog は始めるにあたって便利ですが、多くの本番アプリケーションは、自身のデザインシステムを反映した独自のカタログを定義します。

独自カタログを定義することで、エージェントを汎用的な入力欄やボタンではなく、アプリケーション内に存在するコンポーネントとビジュアル言語だけに制限できます。

実装の詳細については、[独自カタログの定義ガイド](../guides/defining-your-own-catalog.md) を参照してください。

## ベストプラクティス

1. **説明的な ID を使う**: `"c1"` ではなく `"user-profile-card"` を使用します
2. **階層を浅く保つ**: 深いネストを避けます
3. **構造とコンテンツを分離する**: リテラルではなくデータバインディングを使用します
4. **テンプレートで再利用する**: 1 つのテンプレートから、動的な子要素によって多数のインスタンスを生成します
