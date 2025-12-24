# データバインディング

データバインディングは、UIコンポーネントをアプリケーションの状態（データモデル）に結びつける仕組みです。これにより、エージェントはUIの構造を一度定義すれば、あとはデータだけを更新して表示内容を変えることができます。

## データモデル

A2UIのデータモデルは、単純なキーと値のペア、またはネストされたマップ（Map）の集合です。

```json
{
  "dataModelUpdate": {
    "surfaceId": "main",
    "contents": [
      {
        "key": "user",
        "valueMap": [
          {"key": "name", "valueString": "Alice"},
          {"key": "age", "valueInt": 30}
        ]
      }
    ]
  }
}
```

## バインディングの仕組み

コンポーネントのプロパティをデータモデルにバインドするには、`path` を使用します。A2UIは [JSON Pointer (RFC 6901)](https://datatracker.ietf.org/doc/html/rfc6901) に基づいたパス指定を採用しています。

### 例：テキストコンポーネントのバインディング

以下の例では、`Text` コンポーネントの内容をデータモデルの `/user/name` パスにバインドしています。

**UI定義 (surfaceUpdate):**
```json
{
  "id": "welcome-text",
  "component": {
    "Text": {
      "text": {"path": "/user/name"}
    }
  }
}
```

この場合、データモデルに `Alice` という値があれば、UIには "Alice" と表示されます。

## リテラル値とパスの使い分け

A2UIの多くのプロパティは、静的な「リテラル値」と動的な「パス」のどちらかを受け取ることができます。

- **literalString / literalInt / literalBool**: 固定の値を直接指定します。
- **path**: データモデル内の値への参照を指定します。

```json
{
  "component": {
    "Text": {
      "text": {"literalString": "こんにちは！"}
    }
  }
}
```

## 双方向バインディング

`TextField` や `DateTimeInput` などの入力コンポーネントでは、ユーザーが入力した値が自動的にデータモデルに反映されます。

```json
{
  "component": {
    "TextField": {
      "label": {"literalString": "お名前"},
      "value": {"path": "/user/name"}
    }
  }
}
```

ユーザーがテキストフィールドに「Bob」と入力すると、データモデルの `/user/name` の値が「Bob」に更新されます。

## リストのバインディング

`Timeline` や `List` などのコンポーネントは、複数のアイテムを表示するためにデータバインディングを使用します。

```json
{
  "component": {
    "Timeline": {
      "items": {"path": "/events"}
    }
  }
}
```

## メリット

1. **通信量の削減**: UI全体を再送することなく、変更されたデータのみを送信できます。
2. **LLMの負荷軽減**: エージェント（LLM）は複雑なUI構造を何度も生成する必要がなく、データ更新に集中できます。
3. **リアクティブなUI**: データが更新されると、クライアント側で瞬時にUIが更新されます。

## 次のステップ

- **[コンポーネントリファレンス](../reference/components.md)**：各コンポーネントのバインディング可能なプロパティを確認
- **[メッセージリファレンス](../reference/messages.md)**：`dataModelUpdate` メッセージの詳細を確認
