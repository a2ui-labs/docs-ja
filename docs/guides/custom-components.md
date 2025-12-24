# カスタムコンポーネントガイド

A2UIは拡張可能です。標準コンポーネントカタログに含まれていない独自のウィジェットや複雑なUI要素を、カスタムコンポーネントとして定義・利用することができます。

## なぜカスタムコンポーネントを使用するのか？

- **リッチな機能**: Googleマップ、インタラクティブなチャート、データグリッドなどの表示。
- **ブランド固有のUI**: あなたの製品特有のデザインを持つコンポーネントの再利用。
- **ビジネスロジックの統合**: ローカルの状態やAPIとやり取りする複雑なウィジェット。

## ステップ 1：コンポーネントの定義

各プラットフォーム（Web, Flutterなど）のやり方に従って、通常のコンポーネントとしてUIを作成します。

### Web (Lit) の例：

```typescript
import { LitElement, html } from 'lit';
import { property } from 'lit/decorators.js';

export class MyCustomMap extends LitElement {
  @property({ type: String }) location = '';

  render() {
    return html`
      <div class="map-container">
        マップを表示中: ${this.location}
      </div>
    `;
  }
}
```

## ステップ 2：レンダラーへの登録

クライアント側で、レンダラーに新しいコンポーネントタイプを教えます。

```typescript
const renderer = document.querySelector('a2ui-renderer');

renderer.registerComponent('CustomMap', {
  element: 'my-custom-map',
  // A2UIプロパティをコンポーネントのプロパティにマッピング
  propsMap: {
    location: 'location'
  }
});
```

## ステップ 3：エージェントから呼び出す

エージェント（LLM）は、登録された名前を使用してコンポーネントを要求できます。

```json
{
  "id": "my-map-1",
  "component": {
    "CustomMap": {
      "location": {"literalString": "東京駅"}
    }
  }
}
```

## 考慮事項

### データバインディング
カスタムコンポーネントのプロパティも、標準コンポーネントと同様に `literalValue` または `path` を介してデータモデルにバインドできます。

### 互換性
エージェントにカスタムコンポーネントを生成させる場合、そのコンポーネントをサポートしていないクライアントでも代替手段（フォールバック）が提供されるように考慮してください。

### プロンプト
エージェントがカスタムコンポーネントを正しく使えるように、システムプロンプトにコンポーネントの名前と期待されるプロパティの仕様を含める必要があります。

## 次のステップ

- **[レンダラー開発ガイド](renderer-development.md)**：より深くレンダリングの仕組みを知る
- **[エージェント開発ガイド](agent-development.md)**：エージェントにカスタムコンポーネントを教える
