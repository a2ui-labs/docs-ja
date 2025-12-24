# コンポーネントリファレンス

A2UIプロトコルで定義されている標準コンポーネントの詳細な仕様です。

## 概要

すべてのコンポーネントは、以下の共通構造を持ちます：

- **id**: 隣接リスト内で使用される一意の識別子。
- **component**: コンポーネントの型とプロパティを格納するオブジェクト。

---

## 基本コンポーネント

### Text
プレーンテキストを表示します。

| プロパティ | 型 | 説明 |
| :--- | :--- | :--- |
| `text` | TextValue | 表示するテキスト。 |
| `usageHint` | String | スタイルのヒント (`h1`, `h2`, `body`, `caption`, `bold` など)。 |

### Image
画像を表示します。

| プロパティ | 型 | 説明 |
| :--- | :--- | :--- |
| `src` | TextValue | 画像のURL。 |
| `alt` | TextValue | 代替テキスト。 |

### Button
ユーザーがクリックできるアクション要素です。

| プロパティ | 型 | 説明 |
| :--- | :--- | :--- |
| `child` | ID | ボタンの中に表示するコンポーネント（Textなど）のID。 |
| `action` | Action | クリック時に実行されるアクション。 |
| `enabled` | BooleanValue | ボタンが有効かどうか。 |

---

## 入力コンポーネント

### TextField
テキスト入力を受け付けます。

| プロパティ | 型 | 説明 |
| :--- | :--- | :--- |
| `value` | TextValue (Path) | 入力された値を格納するデータモデルのパス。 |
| `label` | TextValue | 入力フィールドのラベル。 |
| `placeholder` | TextValue | ヒントテキスト。 |
| `multiline` | BooleanValue | 複数行入力を許可するかどうか。 |

### DateTimeInput
日付や時間を選択します。

| プロパティ | 型 | 説明 |
| :--- | :--- | :--- |
| `value` | TextValue (Path) | 選択された値を格納するパス (ISO 8601形式推奨)。 |
| `enableDate` | BooleanValue | 日付選択を有効にするか。 |
| `enableTime` | BooleanValue | 時間選択を有効にするか。 |

---

## レイアウト・コンテナ

### FlexBox
水平または垂直方向に子要素を配置します。

| プロパティ | 型 | 説明 |
| :--- | :--- | :--- |
| `children` | [ID] | 子コンポーネントのIDリスト。 |
| `direction` | String | `horizontal` または `vertical`。 |
| `gap` | NumericValue | 子要素間の間隔。 |

### Card
コンテンツを視覚的なボックスでグループ化します。

| プロパティ | 型 | 説明 |
| :--- | :--- | :--- |
| `title` | TextValue | カードのタイトル。 |
| `children` | [ID] | カード内の子コンポーネントのIDリスト。 |

### Timeline / List
繰り返される複数のアイテムを表示します。

| プロパティ | 型 | 説明 |
| :--- | :--- | :--- |
| `items` | TextValue (Path) | 表示するデータの配列を指すパス。 |
| `template` | ID | 各アイテムを表示するために使用するコンポーネントのID。 |

---

## データ型の定義

### TextValue
```json
{
  "literalString": "こんにちは",
  "path": "/data/greeting",
  "usageHint": "h1"
}
```

### NumericValue
```json
{
  "literalInt": 10,
  "literalDouble": 3.14,
  "path": "/data/count"
}
```

### BooleanValue
```json
{
  "literalBool": true,
  "path": "/data/active"
}
```

---

各プロップの詳細は、各レンダラー（Lit, Angular, Flutter）の実装によって若干異なる場合があります。
