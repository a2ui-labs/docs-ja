# メッセージリファレンス

A2UIプロトコルは、エージェントとクライアントレンダラー間で交換される主要なメッセージタイプを定義しています。メッセージは一般的にJSONL形式で送信され、各行が1つのJSONオブジェクト（メッセージ）となります。

## メッセージの概要

| メッセージタイプ | 送信元 | 説明 |
| :--- | :--- | :--- |
| `surfaceUpdate` | エージェント | UIのコンポーネント構造を定義または更新します。 |
| `dataModelUpdate` | 両方 | データモデル（状態）を更新します。 |
| `beginRendering` | エージェント | クライアントにUIのレンダリング開始を指示します。 |
| `deleteSurface` | エージェント | 特定のサーフェスとそのリソースを破棄します。 |
| `action` | クライアント | ユーザーの操作（ボタンクリックなど）をエージェントに通知します。 |

---

## 1. surfaceUpdate (エージェント → クライアント)

UIを構成するコンポーネントツリー（隣接リスト形式）を定義します。

```json
{
  "surfaceUpdate": {
    "surfaceId": "unique-surface-id",
    "components": [
      {
        "id": "root-id",
        "component": {
          "Card": {
            "title": "カードのタイトル",
            "children": ["child-1", "child-2"]
          }
        }
      },
      {
        "id": "child-1",
        "component": {
          "Text": {"text": {"literalString": "コンテンツ1"}}
        }
      }
    ]
  }
}
```

- **surfaceId**: この更新が適用されるサーフェスのID。
- **components**: コンポーネント定義のリスト。既存のIDがある場合は更新され、新しいIDは追加されます。

---

## 2. dataModelUpdate (エージェント ↔ クライアント)

データモデル内の値を更新します。

```json
{
  "dataModelUpdate": {
    "surfaceId": "unique-surface-id",
    "contents": [
      {
        "key": "user",
        "valueMap": [
          {"key": "name", "valueString": "Alice"},
          {"key": "active", "valueBool": true}
        ]
      }
    ]
  }
}
```

- **contents**: 更新するデータのリスト。キーと値のペアで指定します。
- **階層構造**: `valueMap` を使用してネストされたデータ構造を表現できます。

---

## 3. beginRendering (エージェント → クライアント)

クライアントにレンダリングを開始するように指示します。これは、必要なコンポーネントとデータがすべて送信された後に送信されるべきです。

```json
{
  "beginRendering": {
    "surfaceId": "unique-surface-id",
    "root": "root-component-id"
  }
}
```

- **root**: UIのルート（親）となるコンポーネントのID。

---

## 4. deleteSurface (エージェント → クライアント)

サーフェスを削除し、関連するすべてのリソースを解放します。

```json
{
  "deleteSurface": {
    "surfaceId": "unique-surface-id"
  }
}
```

---

## 5. action (クライアント → エージェント)

ユーザーがUIを操作したときに送信されます。

```json
{
  "action": {
    "surfaceId": "unique-surface-id",
    "name": "confirm_booking",
    "parameters": {
      "selection": "option_1"
    }
  }
}
```

- **name**: 実行されたアクションの名前。エージェントが定義した名前に対応します。
- **parameters**: アクションに関連する追加データ（引数）。

## データ型リファレンス

A2UIのメッセージで使用される一般的なデータ表現：

### TextValue
- `literalString`: 直接の文字列。
- `path`: データモデルへのJSON Pointerパス。
- `usageHint`: レンダリングの推奨（`h1`, `h2`, `bold` など）。

### NumericValue
- `literalInt`: 直接の整数。
- `literalDouble`: 直接の浮動小数点数。
- `path`: データモデルへのパス。

### BooleanValue
- `literalBool`: 直接の真偽値。
- `path`: データモデルへのパス。

---

詳細は [個別コンポーネントリファレンス](components.md) を参照してください。
