# コンポーネントギャラリー

このページでは、使用例と利用パターンを交えながら、A2UI のすべてのコンポーネントを紹介します。

> NOTE: スキーマファイル
>
> === "v0.8"
>
>     [:material-code-json: Standard Catalog Definition (JSON Schema)](../../../specification/v0_8/json/standard_catalog_definition.json)
>
> === "v0.9"
>
>     [:material-code-json: Basic Catalog Definition (JSON Schema)](../../../specification/v0_9/catalogs/basic/catalog.json)

---

## レイアウトコンポーネント

### Row

水平方向のレイアウトコンテナです。子要素は左から右に配置されます。

=== "v0.8"

    **プロパティ:** `children`（`explicitList` または `template`）、`distribution`、`alignment`

    ```json
    {
      "id": "toolbar",
      "component": {
        "Row": {
          "children": { "explicitList": ["btn1", "btn2", "btn3"] },
          "distribution": "spaceBetween",
          "alignment": "center"
        }
      }
    }
    ```

=== "v0.9"

    **プロパティ:** `children`（array または template）、`justify`、`align`

    ```json
    {
      "id": "toolbar",
      "component": "Row",
      "children": ["btn1", "btn2", "btn3"],
      "justify": "spaceBetween",
      "align": "center"
    }
    ```

### Column

垂直方向のレイアウトコンテナです。子要素は上から下に配置されます。

=== "v0.8"

    **プロパティ:** `children`（`explicitList` または `template`）、`distribution`、`alignment`

    ```json
    {
      "id": "content",
      "component": {
        "Column": {
          "children": { "explicitList": ["header", "body", "footer"] },
          "distribution": "start",
          "alignment": "stretch"
        }
      }
    }
    ```

=== "v0.9"

    **プロパティ:** `children`（array または template）、`justify`、`align`

    ```json
    {
      "id": "content",
      "component": "Column",
      "children": ["header", "body", "footer"],
      "justify": "start",
      "align": "stretch"
    }
    ```

### List

スクロール可能なアイテムのリストです。静的な子要素と動的なテンプレートの両方をサポートします。

=== "v0.8"

    **プロパティ:** `children`（`explicitList` または `template`）、`direction`、`alignment`

    ```json
    {
      "id": "message-list",
      "component": {
        "List": {
          "children": {
            "template": {
              "dataBinding": "/messages",
              "componentId": "message-item"
            }
          },
          "direction": "vertical"
        }
      }
    }
    ```

=== "v0.9"

    **プロパティ:** `children`（array または template）、`direction`、`align`

    ```json
    {
      "id": "message-list",
      "component": "List",
      "children": {
        "componentId": "message-item",
        "path": "/messages"
      },
      "direction": "vertical"
    }
    ```

---

## 表示コンポーネント

### Text

スタイルのヒントを伴うテキストコンテンツを表示します。

=== "v0.8"

    **プロパティ:** `text`（BoundValue）、`usageHint`

    `usageHint` の値: `h1`, `h2`, `h3`, `h4`, `h5`, `caption`, `body`

    ```json
    {
      "id": "title",
      "component": {
        "Text": {
          "text": { "literalString": "Welcome to A2UI" },
          "usageHint": "h1"
        }
      }
    }
    ```

=== "v0.9"

    **プロパティ:** `text`（string または DataBinding）、`variant`

    `variant` の値: `h1`, `h2`, `h3`, `h4`, `h5`, `caption`, `body`

    ```json
    {
      "id": "title",
      "component": "Text",
      "text": "Welcome to A2UI",
      "variant": "h1"
    }
    ```

### Image

URL から画像を表示します。

=== "v0.8"

    **プロパティ:** `url`（BoundValue）、`fit`、`usageHint`

    ```json
    {
      "id": "hero",
      "component": {
        "Image": {
          "url": { "literalString": "https://example.com/hero.png" },
          "fit": "cover",
          "usageHint": "hero"
        }
      }
    }
    ```

=== "v0.9"

    **プロパティ:** `url`（string または DataBinding）、`fit`、`variant`

    ```json
    {
      "id": "hero",
      "component": "Image",
      "url": "https://example.com/hero.png",
      "fit": "cover",
      "variant": "hero"
    }
    ```

### Icon

カタログで定義された基本セットからアイコンを表示します。

=== "v0.8"

    **プロパティ:** `name`（BoundValue）

    ```json
    {
      "id": "check-icon",
      "component": {
        "Icon": {
          "name": { "literalString": "check" }
        }
      }
    }
    ```

=== "v0.9"

    **プロパティ:** `name`（string または DataBinding）

    ```json
    {
      "id": "check-icon",
      "component": "Icon",
      "name": "check"
    }
    ```

### Divider

視覚的な区切り線です。

=== "v0.8"

    **プロパティ:** `axis`

    ```json
    {
      "id": "separator",
      "component": {
        "Divider": {
          "axis": "horizontal"
        }
      }
    }
    ```

=== "v0.9"

    **プロパティ:** `axis`

    ```json
    {
      "id": "separator",
      "component": "Divider",
      "axis": "horizontal"
    }
    ```

---

## インタラクティブコンポーネント

### Button

アクションをトリガーするクリック可能なボタンです。

=== "v0.8"

    **プロパティ:** `child`（コンポーネントID）、`primary`（boolean）、`action`

    ```json
    {
      "id": "submit-btn",
      "component": {
        "Button": {
          "child": "submit-text",
          "primary": true,
          "action": {
            "name": "submit_form"
          }
        }
      }
    }
    ```

=== "v0.9"

    **プロパティ:** `child`（コンポーネントID）、`variant`、`action`

    ```json
    {
      "id": "submit-btn",
      "component": "Button",
      "child": "submit-text",
      "variant": "primary",
      "action": {
        "event": {
          "name": "submit_form"
        }
      }
    }
    ```

### TextField

オプションで検証を行えるテキスト入力フィールドです。

=== "v0.8"

    **プロパティ:** `label`（BoundValue）、`text`（BoundValue）、`textFieldType`、`validationRegexp`

    `textFieldType` の値: `shortText`, `longText`, `number`, `obscured`, `date`

    ```json
    {
      "id": "email-input",
      "component": {
        "TextField": {
          "label": { "literalString": "Email Address" },
          "text": { "path": "/user/email" },
          "textFieldType": "shortText"
        }
      }
    }
    ```

=== "v0.9"

    **プロパティ:** `label`（string）、`value`（string または DataBinding）、`textFieldType`、`validationRegexp`

    `textFieldType` の値: `shortText`, `longText`, `number`, `obscured`, `date`

    ```json
    {
      "id": "email-input",
      "component": "TextField",
      "label": "Email Address",
      "value": { "path": "/user/email" },
      "textFieldType": "shortText"
    }
    ```

### CheckBox

真偽値の切り替えです。

=== "v0.8"

    **プロパティ:** `label`（BoundValue）、`value`（BoundValue、boolean）

    ```json
    {
      "id": "terms-checkbox",
      "component": {
        "CheckBox": {
          "label": { "literalString": "I agree to the terms" },
          "value": { "path": "/form/agreedToTerms" }
        }
      }
    }
    ```

=== "v0.9"

    **プロパティ:** `label`（string）、`value`（DataBinding、boolean）

    ```json
    {
      "id": "terms-checkbox",
      "component": "CheckBox",
      "label": "I agree to the terms",
      "value": { "path": "/form/agreedToTerms" }
    }
    ```

### Slider

数値範囲の入力コンポーネントです。

=== "v0.8"

    **プロパティ:** `value`（BoundValue）、`minValue`、`maxValue`

    ```json
    {
      "id": "volume",
      "component": {
        "Slider": {
          "value": { "path": "/settings/volume" },
          "minValue": 0,
          "maxValue": 100
        }
      }
    }
    ```

=== "v0.9"

    **プロパティ:** `value`（DataBinding）、`minValue`、`maxValue`

    ```json
    {
      "id": "volume",
      "component": "Slider",
      "value": { "path": "/settings/volume" },
      "minValue": 0,
      "maxValue": 100
    }
    ```

### DateTimeInput

日付および/または時刻を選択するピッカーです。

=== "v0.8"

    **プロパティ:** `value`（BoundValue）、`enableDate`、`enableTime`

    ```json
    {
      "id": "date-picker",
      "component": {
        "DateTimeInput": {
          "value": { "path": "/booking/date" },
          "enableDate": true,
          "enableTime": false
        }
      }
    }
    ```

=== "v0.9"

    **プロパティ:** `value`（DataBinding）、`enableDate`、`enableTime`

    ```json
    {
      "id": "date-picker",
      "component": "DateTimeInput",
      "value": { "path": "/booking/date" },
      "enableDate": true,
      "enableTime": false
    }
    ```

### MultipleChoice (v0.8) / ChoicePicker (v0.9)

リストから1つ以上の選択肢を選びます。

=== "v0.8"

    **プロパティ:** `options`（array）、`selections`（BoundValue）、`maxAllowedSelections`

    ```json
    {
      "id": "country-select",
      "component": {
        "MultipleChoice": {
          "options": [
            { "label": { "literalString": "USA" }, "value": "us" },
            { "label": { "literalString": "Canada" }, "value": "ca" }
          ],
          "selections": { "path": "/form/country" },
          "maxAllowedSelections": 1
        }
      }
    }
    ```

=== "v0.9"

    **プロパティ:** `options`（array）、`selections`（DataBinding）、`maxAllowedSelections`

    ```json
    {
      "id": "country-select",
      "component": "ChoicePicker",
      "options": [
        { "label": "USA", "value": "us" },
        { "label": "Canada", "value": "ca" }
      ],
      "selections": { "path": "/form/country" },
      "maxAllowedSelections": 1
    }
    ```

---

## コンテナコンポーネント

### Card

影(elevation)や枠線(border)、パディングを持つコンテナです。

=== "v0.8"

    **プロパティ:** `child`（コンポーネントID）

    ```json
    {
      "id": "info-card",
      "component": {
        "Card": {
          "child": "card-content"
        }
      }
    }
    ```

=== "v0.9"

    **プロパティ:** `child`（コンポーネントID）

    ```json
    {
      "id": "info-card",
      "component": "Card",
      "child": "card-content"
    }
    ```

### Modal

エントリーポイントとなるコンポーネントによってトリガーされるオーバーレイダイアログです。

=== "v0.8"

    **プロパティ:** `entryPointChild`（コンポーネントID）、`contentChild`（コンポーネントID）

    ```json
    {
      "id": "confirmation-modal",
      "component": {
        "Modal": {
          "entryPointChild": "open-modal-btn",
          "contentChild": "modal-content"
        }
      }
    }
    ```

=== "v0.9"

    **プロパティ:** `entryPointChild`（コンポーネントID）、`contentChild`（コンポーネントID）

    ```json
    {
      "id": "confirmation-modal",
      "component": "Modal",
      "entryPointChild": "open-modal-btn",
      "contentChild": "modal-content"
    }
    ```

### Tabs

コンテンツを切り替え可能なパネルに整理するためのタブインターフェースです。

=== "v0.8"

    **プロパティ:** `tabItems`（`{ title, child }` を要素とする array）

    ```json
    {
      "id": "settings-tabs",
      "component": {
        "Tabs": {
          "tabItems": [
            { "title": { "literalString": "General" }, "child": "general-tab" },
            { "title": { "literalString": "Privacy" }, "child": "privacy-tab" }
          ]
        }
      }
    }
    ```

=== "v0.9"

    **プロパティ:** `tabItems`（`{ title, child }` を要素とする array）

    ```json
    {
      "id": "settings-tabs",
      "component": "Tabs",
      "tabItems": [
        { "title": "General", "child": "general-tab" },
        { "title": "Privacy", "child": "privacy-tab" }
      ]
    }
    ```

---

## 共通プロパティ

すべてのコンポーネントは、以下のプロパティを共有します。

- `id`（必須）: サーフェス内で一意の識別子です。
- `accessibility`: アクセシビリティ属性（label、role）です。
- `weight`: Row や Column の内部にある場合の flex-grow 値です。

## バージョン間の違いのまとめ

コンポーネント名とプロパティはバージョン間でほぼ同じですが、構造上の違いは次のとおりです。

| 項目               | v0.8                                | v0.9                              |
| ------------------ | ----------------------------------- | ---------------------------------- |
| コンポーネントのラッパー | `"component": { "Text": { ... } }` | `"component": "Text", ...props`  |
| 文字列値            | `{ "literalString": "Hello" }`     | `"Hello"`                        |
| 子要素              | `{ "explicitList": ["a", "b"] }`   | `["a", "b"]`                     |
| データバインディング  | `{ "path": "/data" }`              | `{ "path": "/data" }`（変更なし）  |
| Text/Image のスタイル指定 | `usageHint`                    | `variant`                        |
| Button のスタイル指定 | `primary: true`                    | `variant: "primary"`             |
| アクションの形式     | `{ "name": "..." }`                | `{ "event": { "name": "..." } }` |
| 選択コンポーネント   | `MultipleChoice`                   | `ChoicePicker`                   |
| レイアウトの配置     | `distribution`, `alignment`        | `justify`, `align`               |
| TextField の値      | `text`                             | `value`                          |

## ライブサンプル

すべてのコンポーネントの動作を確認するには、次を実行します。

```bash
cd samples/client/angular
yarn start gallery
```

> [!NOTE]
> **パッケージマネージャーの使用について:** A2UI リポジトリに組み込まれているサンプルアプリケーションを実行するには、Corepack ワークスペースで設定されている Yarn（`yarn start gallery`）が必要です。このリポジトリ以外での通常の利用やスタンドアロンのプロジェクトでは、お好みのパッケージマネージャー（npm、pnpm など）を使用してください。

## 参考資料

> NOTE: スキーマファイル
>
> === "v0.8"
>
>     [:material-code-json: Standard Catalog Definition (JSON Schema)](../../../specification/v0_8/json/standard_catalog_definition.json)
>
> === "v0.9"
>
>     [:material-code-json: Basic Catalog Definition (JSON Schema)](../../../specification/v0_9/catalogs/basic/catalog.json)

- **[独自カタログを定義する](../guides/defining-your-own-catalog.md)**: 独自のコンポーネントを構築する
- **[テーマ設定ガイド](../guides/theming.md)**: ブランドに合わせてコンポーネントをスタイリングする
