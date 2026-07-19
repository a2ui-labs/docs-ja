# クライアント設定ガイド

お使いのプラットフォーム向けのレンダラーを使って、A2UI をアプリケーションに統合します。

## レンダラー

| レンダラー                                                                                | プラットフォーム    | v0.8 | v0.9 | ステータス       |
| ------------------------------------------------------------------------------------------ | ------------------- | ---- | ---- | ---------------- |
| **[React](https://github.com/a2ui-project/a2ui/tree/main/renderers/react)**                | Web                 | ✅   | ✅   | ✅ 安定版         |
| **[Lit (Web Components)](https://github.com/a2ui-project/a2ui/tree/main/renderers/lit)**   | Web                 | ✅   | ✅   | ✅ 安定版         |
| **[Angular](https://github.com/a2ui-project/a2ui/tree/main/renderers/angular)**            | Web                 | ✅   | ✅   | ✅ 安定版         |
| **[Flutter (GenUI SDK)](https://docs.flutter.dev/ai/genui)**                                | Mobile/Desktop/Web  | ✅   | ✅   | ✅ 安定版         |
| **Jetpack Compose**                                                                         | Android             | —    | —    | 🚧 2026年 Q2 予定 |

詳細については、[A2UI レンダラー](../reference/renderers.md) と [コミュニティ A2UI レンダラー](../ecosystem/renderers.md) の一覧を参照してください。

## コンポーネントカタログ

コンポーネントカタログとは、コンポーネントの集合のことです。A2UI は「Basic Catalog」を提供していますが、独自のコンポーネントや共有ライブラリを追加したり、Basic Catalog のコンポーネントを完全に独自のものへ置き換えたりすることを想定しています。

**重要なのは、あなたのデザインシステムです。** コンポーネントと関数の任意の集合を登録でき、A2UI はそれらと連携して動作します。カタログは、エージェントとレンダラーの間の契約にすぎません。

自分のデザインシステムに合わせたカタログを定義する方法については、[独自カタログを定義する](defining-your-own-catalog.md) を参照してください。

## 共有 Web ライブラリ

すべての Web レンダラー(Lit、Angular、React)は、共通の基盤である **`@a2ui/web_core`** を共有しています。このライブラリは、あらゆる Web レンダラーが必要とするメッセージプロセッサー、状態管理、データバインディングのロジックを提供します。各フレームワーク固有のレンダラーはこの上に構築され、そのフレームワーク用のレンダリング層だけを追加します。

つまり、コアのプロトコル処理は Web プラットフォーム間で一貫しており、異なるのはコンポーネントのレンダリング方法だけです。

共有 `web_core` ライブラリが提供するもの:

- **メッセージプロセッサー**: A2UI の状態を管理し、受信メッセージを処理します。

## Web Components (Lit)

```bash
npm install @a2ui/lit @a2ui/web_core
```

インストールが完了すると、アプリ内でレンダラーを使用できます。Lit レンダラーは次を使用します。

- **メッセージプロセッサー**: A2UI のメッセージプロセッサーをラップします。
- **`<a2ui-surface>` コンポーネント**: アプリ内で surface をレンダリングします。
- **Lit Signals**: UI の自動更新のためのリアクティブな状態管理を提供します。

**動作するサンプルを見る:** [Lit shell サンプル](https://github.com/a2ui-project/a2ui/tree/main/samples/client/lit/shell) — 詳しい実行手順は README を確認してください。

## Angular

```bash
npm install @a2ui/angular @a2ui/web_core
```

インストールが完了すると、アプリ内でレンダラーを使用できます。Angular レンダラーは次を提供します。

- **`A2uiRendererService`**: A2UI のメッセージプロセッサーとリアクティブモデルを管理するサービスです。
- **`a2ui-v09-component-host` コンポーネント**: surface から A2UI コンポーネントをレンダリングする動的コンポーネントホストです。
- **`A2UI_RENDERER_CONFIG` トークン**: カタログとアクションハンドラーでレンダラーを設定するために使用します。

### 設定例 (v0.9)

A2UI は、プロトコル固有の実装に対してバージョン管理されたインポートを使用します。v0.9 の場合、アプリケーションのプロバイダーを次のように設定します。

```typescript
import {ApplicationConfig} from '@angular/core';
import {A2UI_RENDERER_CONFIG, A2uiRendererService, BasicCatalog} from '@a2ui/angular/v0_9';

export const appConfig: ApplicationConfig = {
  providers: [
    {
      provide: A2UI_RENDERER_CONFIG,
      useValue: {
        catalogs: [new BasicCatalog()],
        actionHandler: action => {
          console.log('Action dispatched:', action);
        },
      },
    },
    A2uiRendererService,
  ],
};
```

**動作するサンプルを見る:** [Angular サンプル](https://github.com/a2ui-project/a2ui/tree/main/samples/client/angular)

### ストリーミング

Angular クライアントはデフォルトでストリーミング API を使用します。ストリーミングを無効にするには、アプリを起動する前に `ENABLE_STREAMING` 環境変数を `false` に設定します。

```bash
export ENABLE_STREAMING=false
yarn start restaurant
```

> [!NOTE]
> **パッケージマネージャーの使用について:** 上記の `yarn start` コマンドは、A2UI monorepo リポジトリ内でサンプルアプリケーションを実行する場合に特有のものです。このリポジトリ外でのご自身の通常の利用やスタンドアロンプロジェクトでは、お好みのパッケージマネージャー(npm、pnpm など)を使用してください。

## React

```bash
npm install @a2ui/react @a2ui/web_core
```

React レンダラーは次を提供します。

- **`MessageProcessor` クラス**: A2UI のメッセージプロセッサーとリアクティブモデルを管理するクラスです。
- **`<A2UISurface>` コンポーネント**: React アプリ内で A2UI surface をレンダリングします。
- **`useA2UI()` フック**: 任意のコンポーネントからメッセージプロセッサーにアクセスします。

**動作するサンプルを見る:** [React shell](https://github.com/a2ui-project/a2ui/tree/main/samples/client/react/shell)

## Flutter (GenUI SDK)

```bash
flutter pub add flutter_genui
```

Flutter は、ネイティブな A2UI レンダリングを提供する GenUI SDK を使用します。

**ドキュメント:** [GenUI SDK](https://docs.flutter.dev/ai/genui) | [GitHub](https://github.com/flutter/genui) | [GenUI Flutter パッケージの README](https://github.com/flutter/genui/blob/main/packages/genui/README.md#getting-started-with-genui)

## エージェントへの接続

クライアントアプリケーションは次を行う必要があります。

1. エージェントから(トランスポート経由で)**A2UI メッセージを受信する**
2. メッセージプロセッサーを使って**メッセージを処理する**
3. **ユーザーのアクションをエージェントへ送り返す**

一般的なトランスポートの選択肢:

- **Server-Sent Events (SSE)**: サーバーからクライアントへの一方向ストリーミング
- **WebSockets**: 双方向のリアルタイム通信
- **A2A Protocol**: A2UI をサポートする標準化されたエージェント間通信

A2A プロトコルクライアントの使用例については、[samples/client/lit/shell/client.ts](https://github.com/a2ui-project/a2ui/tree/main/samples/client/lit/shell/client.ts) を参照してください。

**参照:** [トランスポート層](../concepts/transports.md)

## ユーザーアクションの処理

ユーザーが A2UI コンポーネントを操作すると(ボタンのクリック、フォームの送信など)、クライアントは次を行います。

1. コンポーネントからアクションイベントをキャプチャする
2. アクションに必要なデータコンテキストを解決する
3. アクションをエージェントへ送信する
4. エージェントの応答メッセージを処理する

ボタンのクリックとフォーム送信の処理例については、[samples/client/lit/shell/app.ts](https://github.com/a2ui-project/a2ui/tree/main/samples/client/lit/shell/app.ts) の `#maybeRenderData` 内にある `@a2uiaction` イベントハンドラーを参照してください。

## エラー処理

処理すべき一般的なエラー:

- **Invalid Surface ID**: `beginRendering`(v0.8)または `createSurface`(v0.9)を受信する前に surface が参照された。
- **Invalid Component ID**: コンポーネント ID は surface 内で一意である必要があります。
- **Invalid Data Path**: データモデルの構造と JSON Pointer の構文を確認してください。
- **Schema Validation Failed**: メッセージ形式が A2UI 仕様と一致しているか確認してください。

通信エラーの処理例については、[samples/client/lit/shell/app.ts](https://github.com/a2ui-project/a2ui/tree/main/samples/client/lit/shell/app.ts) の `#sendMessage` 内にある `try...catch` ブロックを参照してください。

## 次のステップ

- **[クイックスタート](../quickstart.md)**: デモアプリケーションを試す
- **[テーマとスタイリング](theming.md)**: 見た目をカスタマイズする
- **[独自カタログを定義する](defining-your-own-catalog.md)**: コンポーネントカタログを拡張する
- **[エージェント開発](agent-development.md)**: A2UI を生成するエージェントを構築する
- **[リファレンスドキュメント](../reference/messages.md)**: プロトコルを深く理解する
