# データバインディング

データバインディングは、JSON Pointer パス([RFC 6901](https://tools.ietf.org/html/rfc6901))を使って UI コンポーネントをアプリケーションの状態に結びつけます。これにより A2UI は、大きなデータ配列に対するレイアウトを効率的に定義したり、最初から作り直すことなく更新後の内容を表示したりできます。

## 構造と状態

A2UI は、次の 2 つを分離します。

1. **UI 構造(コンポーネント)**: インターフェースがどのように見えるか
2. **アプリケーションの状態(データモデル)**: どのデータを表示するか

これにより、次のことが可能になります。

- リアクティブな更新
- データドリブンな UI
- 再利用可能なテンプレート
- 双方向バインディング

## データモデル

各 Surface は、状態を保持する JSON オブジェクトを持ちます。

```json
{
  "user": {"name": "Alice", "email": "alice@example.com"},
  "cart": {
    "items": [{"name": "Widget", "price": 9.99, "quantity": 2}],
    "total": 19.98
  }
}
```

## JSON Pointer パス

**構文:**

- `/user/name` - オブジェクトのプロパティ
- `/cart/items/0` - 配列のインデックス(0 始まり)
- `/cart/items/0/price` - ネストされたパス

**例:**

```json
{"user": {"name": "Alice"}, "items": ["Apple", "Banana"]}
```

- `/user/name` → `"Alice"`
- `/items/0` → `"Apple"`

## リテラル値とパス値

=== "v0.8"

    **Literal(固定値):**
    ```json
    {
      "id": "title",
      "component": {
        "Text": {
          "text": { "literalString": "Welcome" }
        }
      }
    }
    ```

    **Data-bound(リアクティブ):**
    ```json
    {
      "id": "username",
      "component": {
        "Text": {
          "text": { "path": "/user/name" }
        }
      }
    }
    ```

=== "v0.9 and later"

    **Literal(固定値):**
    ```json
    {
      "id": "title",
      "component": "Text",
      "text": "Welcome"
    }
    ```

    **Data-bound(リアクティブ):**
    ```json
    {
      "id": "username",
      "component": "Text",
      "text": { "path": "/user/name" }
    }
    ```

`/user/name` が "Alice" から "Bob" に変わると、テキストは**自動的に**"Bob" へ更新されます。

## リアクティブな更新

データパスにバインドされたコンポーネントは、データが変更されると自動的に更新されます。

=== "v0.8"

    ```json
    {
      "id": "status",
      "component": {
        "Text": {
          "text": {"path": "/order/status"}
        }
      }
    }
    ```

=== "v0.9 and later"

    ```json
    {
      "id": "status",
      "component": "Text",
      "text": {"path": "/order/status"}
    }
    ```

- **初期状態:** `/order/status` = "Processing..." → "Processing..." と表示
- **更新:** `status: "Shipped"` を含むデータモデル更新を送信 → "Shipped" と表示

コンポーネントの更新は不要です。データの更新だけで済みます。

## 動的リスト

配列をレンダリングするにはテンプレートを使用します。

=== "v0.8"

    ```json
    {
      "id": "product-list",
      "component": {
        "Column": {
          "children": {
            "template": {
              "dataBinding": "/products",
              "componentId": "product-card"
            }
          }
        }
      }
    }
    ```

=== "v0.9 and later"

    ```json
    {
      "id": "product-list",
      "component": "Column",
      "children": {
        "path": "/products",
        "componentId": "product-card"
      }
    }
    ```

**データ:**

```json
{
  "products": [
    {"name": "Widget", "price": 9.99},
    {"name": "Gadget", "price": 19.99}
  ]
}
```

**結果:** 商品ごとに 1 枚ずつ、合計 2 枚のカードがレンダリングされます。

### スコープ付きパス

テンプレート内では、パスは配列の各アイテムにスコープされます。

=== "v0.8"

    ```json
    {
      "id": "product-name",
      "component": {
        "Text": {
          "text": {"path": "/name"}
        }
      }
    }
    ```

    - `/products/0` の場合、`/name` は `/products/0/name` に解決され → "Widget"
    - `/products/1` の場合、`/name` は `/products/1/name` に解決され → "Gadget"

=== "v0.9 and later"

    ```json
    {
      "id": "product-name",
      "component": "Text",
      "text": {"path": "name"}
    }
    ```

    - `/products/0` の場合、`name` は `/products/0/name` に解決され → "Widget"
    - `/products/1` の場合、`name` は `/products/1/name` に解決され → "Gadget"

アイテムを追加・削除すると、レンダリングされたコンポーネントは自動的に更新されます。

## 入力バインディング

インタラクティブなコンポーネントは、データモデルを双方向に更新します。

=== "v0.8"

    | コンポーネント      | 例                                           | ユーザー操作      | データ更新                  |
    | ------------------ | ------------------------------------------- | ---------------- | -------------------------- |
    | **TextField**      | `{"text": {"path": "/form/name"}}`          | "Alice" と入力    | `/form/name` = `"Alice"`   |
    | **CheckBox**       | `{"value": {"path": "/form/agreed"}}`       | チェックボックスをオン | `/form/agreed` = `true`    |
    | **MultipleChoice** | `{"selections": {"path": "/form/country"}}` | "Canada" を選択  | `/form/country` = `["ca"]` |

=== "v0.9 and later"

    | コンポーネント      | 例                                           | ユーザー操作      | データ更新                  |
    | ------------------ | ------------------------------------------- | ---------------- | -------------------------- |
    | **TextField**      | `{"value": {"path": "/form/name"}}`         | "Alice" と入力    | `/form/name` = `"Alice"`   |
    | **CheckBox**       | `{"value": {"path": "/form/agreed"}}`       | チェックボックスをオン | `/form/agreed` = `true`    |
    | **ChoicePicker**   | `{"value": {"path": "/form/country"}}`      | "Canada" を選択  | `/form/country` = `["ca"]` |

## ベストプラクティス

- **粒度の細かい更新を使う**: 変更されたパスだけを更新します。

=== "v0.8"

    ```json
    {
      "dataModelUpdate": {
        "path": "/user",
        "contents": [{"key": "name", "valueString": "Alice"}]
      }
    }
    ```

=== "v0.9 and later"

    ```json
    {
      "version": "v0.9.1",
      "updateDataModel": {
        "surfaceId": "user_profile",
        "path": "/user/name",
        "value": "Alice"
      }
    }
    ```

- **ドメインごとに整理する**: 関連するデータをグループ化します。

    ```json
    {"user": {...}, "cart": {...}, "ui": {...}}
    ```

- **表示用の値を事前計算する**: 送信前にエージェント側で(通貨や日付などの)データをフォーマットします。

    ```json
    {"price": "$19.99"} // 不可: {"price": 19.99}
    ```
