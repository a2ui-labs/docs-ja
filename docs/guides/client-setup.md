# クライアント設定ガイド

A2UIレンダラーを既存のアプリケーションに統合する方法、またはエージェント主導のインターフェースを表示する新しいホストアプリケーションを構築する方法を学びます。

## ステップ 1：レンダラーのインストール

あなたのアプリが使用しているフレームワーク、または構築したいものに合ったレンダラーを選択してインストールします。

### Web (Lit / Web Components)
フレームワークを問わず、あらゆるWebアプリで使用可能です。

```bash
npm install @a2ui/lit
```

### Angular
Angularプロジェクトにネイティブに統合できます。

```bash
npm install @a2ui/angular
```

### Flutter (GenUI SDK)
モバイル、デスクトップ、Webなど、Flutterがサポートするすべてのプラットフォームで使用可能です。

```bash
flutter pub add flutter_genui
```

## ステップ 2：レンダラーの設定

レンダラーをアプリのコンポーネントツリーに配置し、初期設定を行います。

### Web (Lit) の例：

```typescript
import { A2UIRenderer } from '@a2ui/lit';

// カスタムエレメントとして登録
customElements.define('a2ui-renderer', A2UIRenderer);

// HTMLでの使用
// <a2ui-renderer id="my-renderer"></a2ui-renderer>
```

### Angular の例：

```typescript
import { A2UIModule } from '@a2ui/angular';

@NgModule({
  imports: [
    A2UIModule,
    // ...
  ]
})
export class AppModule { }
```

## ステップ 3：トランスポート層の構築

エージェントから配信されるA2UIメッセージを受信し、レンダラーに渡すためのトランスポート層を設定します。

汎用的なSSEを使用した例：

```typescript
const renderer = document.querySelector('a2ui-renderer');
const eventSource = new EventSource('/api/agent/stream');

eventSource.onmessage = (event) => {
  const message = JSON.parse(event.data);
  // メッセージをレンダラーに渡す
  renderer.handleMessage(message);
};
```

トランスポート方式の詳細は [トランスポート層](../transports.md) を参照してください。

## ステップ 4：テーマ設定とスタイリング

A2UIコンポーネントがあなたのアプリのデザインシステムに馴染むように、テーマを定義します。

```css
:root {
  --a2ui-primary-color: #007bff;
  --a2ui-font-family: 'Inter', sans-serif;
  --a2ui-border-radius: 8px;
}
```

詳細は [テーマ設定ガイド](theming.md) を参照してください。

## ステップ 5：カスタムコンポーネントの登録 (オプション)

標準カタログにない独自のウィジェット（マップ、チャート、高度なフォーム要素など）を表示したい場合に設定します。

```typescript
renderer.registerComponent('MyMap', MyMapComponent);
```

詳細は [カスタムコンポーネントガイド](custom-components.md) を参照してください。

## チェックリスト

✅ レンダラーがインストールされ、アプリに組み込まれている
✅ トランスポート層がエージェントメッセージを受信できている
✅ メッセージが `renderer.handleMessage()` を介して処理されている
✅ テーマが設定され、ブランドに合っている
✅ 必要に応じてカスタムコンポーネントが登録されている

## 次のステップ

- **[テーマ設定ガイド](theming.md)**
- **[カスタムコンポーネントガイド](custom-components.md)**
- **[エージェント開発ガイド](agent-development.md)**
