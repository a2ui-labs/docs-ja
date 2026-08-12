# A2UI Composer 連携マニュアル

## 背景

A2UI Composer は、特定のカタログやレンダラースタックについての知識を持たず、
それらと直接統合されてもいません。A2UI JSON がレンダリングされた結果を見るために、
Composer は「レンダラーアプリケーション」に依存します。A2UI Composer はレンダラー
アプリケーションを iframe 内にホストし、**postMessage** を介して通信します。
レンダラーアプリケーションをホストする iframe には
`sandbox='allow-scripts allow-same-origin allow-forms'` が設定されていますが、
単に A2UI JSON をレンダリングするだけであれば問題にはなりません。

レンダラーアプリケーションは、A2UI JSON を受け取り、自身のレンダラーを使って
結果を表示する役割を担います。

## ブリッジ (Bridge)

A2UI Composer との連携作業を簡単にするために、**ブリッジ**が用意されています。
これはレンダラーアプリケーションに組み込む少量の JavaScript コードで、
A2UI Composer とレンダラーアプリケーション間のすべての通信を調整します。

連携をさらに簡単にするために、フレームワークごとのラッパーも提供されています。

## 例

サンプルのレンダラーアプリをご覧ください。

- [Angular](https://github.com/a2ui-project/composer/tree/main/samples/ng-basic-catalog)
- [Lit](https://github.com/a2ui-project/composer/tree/main/samples/lit-basic-catalog)
- [React](https://github.com/a2ui-project/composer/tree/main/samples/react-basic-catalog)

これらはいずれも同じバージョンの Basic カタログを提供します。

ホストされている [A2UI Composer](https://a2ui-project.github.io/composer/) を実行し、
設定ページで Renderer ドロップダウンをクリックすると、これらのレンダラー
アプリケーションが実際に動作する様子を確認できます。

### Angular を使う

例として、Angular ベースのレンダラーアプリケーションを作成する手順を見ていきます。

### 依存関係の追加

まず、プロジェクトの依存関係リストにコア連携パッケージを追加します。

```
yarn add a2ui-bridge @a2ui/web_core @a2ui/angular
```

もちろん、別のパッケージマネージャーを使っている場合は、それに応じた手順に
従ってください。

#### ラッパーコンポーネントの作成

次のような新しいコンポーネントを作成します。

```ts
import {Component, inject} from '@angular/core';
import {SurfaceComponent} from '@a2ui/angular/v0_9';
import {A2uiSandboxConnection} from 'a2ui-bridge/angular';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [SurfaceComponent],
  template: `
    <main class="sandbox-shell">
      @if (sandbox.surfaceId()) {
        <a2ui-v09-surface [surfaceId]="sandbox.surfaceId()" />
      } @else {
        <p style="padding: 24px; color: #666; font-family: sans-serif; text-align: center;">
          Waiting for RENDER_A2UI payloads...
        </p>
      }
    </main>
  `,
})
export class AppComponent {
  protected sandbox = inject(A2uiSandboxConnection);
}
```

`@else` ブロックの内容は自由に変更してかまいませんが、テンプレートのそれ以外の
部分はそのままにしておいてください。

#### レンダラーアプリケーションのブートストラップ

標準的な Angular のブートストラップエントリーポイントファイル (`src/main.ts`) を
設定し、サンドボックスの provider マッピングにカタログクラスを動的に渡します。

```ts
import {bootstrapApplication} from '@angular/platform-browser';
import {AppComponent} from './app/app.component';
import {provideA2uiSandbox} from 'a2ui-bridge/angular';
import {BasicCatalog} from '@a2ui/angular/v0_9';

bootstrapApplication(AppComponent, {
  providers: [
    provideA2uiSandbox([BasicCatalog]), // Injects and exposes dynamic catalogs
  ],
}).catch((err) => console.error('A2UI Sandbox Bootstrap Failed:', err));
```

`BasicCatalog` は必ずご自身のカタログに置き換えてください。

> 変更検知 (change detection) の互換性に関する注意: `provideA2uiSandbox` ヘルパーは、
> 標準の Zone ベースの変更検知 (`zone.js` を使用) と Zoneless の変更検知
> (`provideZonelessChangeDetection()`) の両方に、追加設定なしで 100% 対応しています。
