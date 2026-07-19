# レンダラーの開発

このドキュメントは、A2UI プロトコルの新しいレンダラー実装に求められる機能をまとめたものです。新しいレンダラー(React、Flutter、iOS 向けなど)を構築する開発者を対象としています。

> NOTE: バージョン対応ガイド
>
> このガイドは、v0.8、v0.9.1(本番運用中の現行バージョン)、v1.0(候補)向けの実装チェックリストを提供します。対象とするバージョンを選ぶには、以下のタブを使用してください。

## Web レンダラー: `@a2ui/web_core`(`web_core`)を使う

Web 向けのレンダラー(React、Vue、Svelte など)を構築する場合、メッセージ処理・状態管理・スキーマ検証をゼロから実装する必要はありません。**[`@a2ui/web_core`](https://github.com/a2ui-project/a2ui/tree/main/renderers/web_core)** パッケージ(`web_core`)は、メンテナンスされている Lit、Angular、React の各レンダラーが共有する、フレームワークに依存しないロジックをすべて提供します。

### `web_core` が提供するもの

| モジュール | 機能 |
| ---------------------------------------- | ------------------------------------------------------------------------------------------------ |
| **`MessageProcessor`**                   | A2UI の JSONL ストリームを処理し、メッセージをディスパッチし、サーフェスのライフサイクルを管理します |
| **`SurfaceModel` / `SurfaceGroupModel`** | サーフェス、コンポーネント、データモデルの状態管理                                                 |
| **`DataModel` / `DataContext`**          | データバインディングの解決、パスベースのルックアップ、テンプレートリストのレンダリング              |
| **`ComponentModel`**                     | コンポーネントツリーの状態、隣接リストからツリーへの解決                                            |
| **型とスキーマ**                          | すべての A2UI コンポーネント、プリミティブ、カラー、スタイルの TypeScript 型、および JSON スキーマ検証 |
| **式パーサー**                            | クライアント側関数の評価(v0.9+)                                                                   |

### メンテナンスされているレンダラーでの使い方

3 つの Web レンダラーはすべて同じパターンに従っています。`web_core` がプロトコルを処理し、レンダラーが UI を処理します。

```typescript
// 型 — すべてのレンダラーで共有されます
import type * as Types from '@a2ui/web_core/types/types';
import type * as Primitives from '@a2ui/web_core/types/primitives';

// v0.8: メッセージ処理と状態
import {A2uiMessageProcessor} from '@a2ui/web_core/data/model-processor';

// v0.9.1 / v1.0: メッセージ処理、サーフェス、カタログ
import {MessageProcessor} from '@a2ui/web_core/v0_9';
import {SurfaceModel} from '@a2ui/web_core/v0_9';

// スタイルとレイアウトのヘルパー
import * as Styles from '@a2ui/web_core/styles/index';
```

あなたのレンダラーがすべきことは、次の 3 つだけです。

1. **A2UI のコンポーネントタイプをフレームワークのコンポーネントにマッピングする**(例: `Text` → `<p>`、`Button` → `<button>`)
2. `web_core` からの**状態変化を購読して**再レンダリングする
3. **ユーザーアクションを転送し**、`MessageProcessor` を通じてサーバーへ送り返す

このパターンの動作例については、[React レンダラー](https://github.com/a2ui-project/a2ui/tree/main/renderers/react)、[Lit レンダラー](https://github.com/a2ui-project/a2ui/tree/main/renderers/lit)、[Angular レンダラー](https://github.com/a2ui-project/a2ui/tree/main/renderers/angular)を参照してください。

### バージョンサポート

`web_core` は、バージョンごとに API セットをエクスポートします。

- `@a2ui/web_core/v0_8` — 安定版の v0.8
- `@a2ui/web_core/v0_9` — `createSurface`、カスタムカタログ、クライアント側関数に対応した v0.9/v0.9.1 サポート
- `@a2ui/web_core/v1_0` — RPC アクションレスポンスを含む、候補版の v1.0 サポート

> TIP: `web_core` から始める
>
> `web_core` を使わずに Web レンダラーを構築するということは、メッセージ処理・状態管理・スキーマ検証のおよそ 3,000 行を自前で再実装することを意味します。あえて外れる特別な理由がない限り、`web_core` を使いましょう。

---

## I. コアプロトコル実装チェックリスト

このセクションでは、A2UI プロトコルの基本的な仕組みについて詳しく説明します。準拠するレンダラーは、サーバーのストリームを正しく解析し、状態を管理し、ユーザーの操作を処理するために、これらの仕組みを実装する必要があります。

=== "v0.8"

    - **JSONL ストリームのパース**: ストリーミングされたレスポンスを 1 行ずつ読み取り、各行を個別の JSON オブジェクトとしてデコードします。
    - **メッセージディスパッチャー**: メッセージタイプ(`beginRendering`、`surfaceUpdate`、`dataModelUpdate`、`deleteSurface`)を識別し、適切なハンドラーへルーティングします。
    - **サーフェス管理**:
        - `surfaceId` によってサーフェスをキー管理します。
        - `surfaceUpdate` を処理します: サーフェスのバッファ内のコンポーネントを追加・更新します。
        - `deleteSurface` を処理します: サーフェスと、それに関連するすべてのデータ・コンポーネントを削除します。
    - **コンポーネントのバッファリング**:
        - サーフェスごとにコンポーネントバッファ(例: `Map<String, Component>`)を維持します。
        - `id` の参照(`children.explicitList`、`child`、`contentChild` など)を解決して UI ツリーを再構築します。
    - **データモデルストア**:
        - サーフェスごとにデータモデルの状態を維持します。
        - `dataModelUpdate` を処理します: 隣接リスト表現(`[{ "key": "name", "valueString": "Bob" }]`)を使って、パス上の値を更新します。
    - **プログレッシブレンダリング**:
        - `beginRendering` を受信するまで更新をバッファリングします。
        - `beginRendering` を受信したら、指定された `root` ID からレンダリングを開始します。テーマスタイルを適用します。
    - **データバインディングの解決**:
        - `literalString` / `literalNumber` / `path` を使って `BoundValue` オブジェクトを解決します。
    - **動的リスト**:
        - `children.template` については、`template.dataBinding` にあるデータリストを反復処理し、`template.componentId` を使ってコンポーネントをレンダリングします。
    - **クライアントからサーバーへ**:
        - 解決済みのパスを含むコンテキストを持つ `userAction` をサーバーへ送出します。
        - トランスポートのメタデータに `a2uiClientCapabilities` を含めます。

=== "v0.9.1 (Current)"

    - **JSONL ストリームのパース**: ストリーミングされたレスポンスを 1 行ずつ読み取り、各行を個別の JSON オブジェクトとしてデコードします。
    - **メッセージディスパッチャー**: メッセージタイプ(`createSurface`、`updateComponents`、`updateDataModel`、`deleteSurface`)を識別し、適切なハンドラーへルーティングします。
    - **MIME タイプの検証**: 標準化された `application/a2ui+json` MIME タイプに基づいてペイロードをインターセプトします。
    - **サーフェス管理**:
        - `surfaceId` によってサーフェスをキー管理します。
        - `createSurface` を処理します: サーフェスを作成し、`catalogId` を紐付け、`theme` と `sendDataModel` を登録します。
        - `updateComponents` を処理します: `"component": "Type"` の判別子を使ったフラット形式で、コンポーネントを追加・更新します。
        - `deleteSurface` を処理します: サーフェスと、それに関連するすべてのデータ・コンポーネントを削除します。
    - **コンポーネントのバッファリング**:
        - サーフェスごとにコンポーネントバッファ(例: `Map<String, Component>`)を維持します。
        - コンテナコンポーネントの `children` 配列や `child` フィールド内の ID 参照を解決して UI ツリーを再構築します。
    - **データモデルストア**:
        - サーフェスごとにデータモデルの状態を維持します。
        - `updateDataModel` を処理します: upsert セマンティクスを持つ標準的な JSON オブジェクトを使って、パス上のデータを更新します。
    - **プログレッシブレンダリング**:
        - `updateComponents` 内で有効なルートコンポーネント(ID `root`)をパースした時点で、ただちにレンダリングします。特別なレンダリング開始シグナルを待つ必要はありません。
    - **データバインディングの解決**:
        - 簡略化されたバインド値(リテラル値、または `{"path": "..."}`)を解決します。
    - **動的リスト**:
        - 子テンプレートについては、`path` にあるデータ配列を反復処理し、`componentId` で指定されたテンプレートをレンダリングします。
    - **クライアント側関数**:
        - 登録済みのカタログ定義関数(例: `formatString` の補間)を評価します。
    - **クライアントからサーバーへ**:
        - 解決済みのパスを含むコンテキストを持つ `action`(`userAction` を置き換えるもの)を送出します。
        - `sendDataModel` が要求されていた場合、クライアント側のデータモデル全体を自動的にメタデータへ含めます。
        - スキーマ検証に失敗した場合、構造化された `ValidationFailed` エラーメッセージをサーバーへ送信します。

=== "v1.0 (Candidate)"

    v0.9.1 のすべての要件に加えて、以下の拡張が必要です。
    - **サーフェスプロパティ**:
        - `surfaceProperties`(`theme` から改名)を伴う `createSurface` を処理します。カスタムのプライマリブランドカラーは、サーフェススキーマ内ではサポートされなくなりました。
    - **アクションレスポンス(RPC)**:
        - `actionId` と、戻り値の `value` または `error` を含む、サーバーからの `actionResponse` メッセージを処理します。
    - **クライアントからサーバーへ**:
        - `action` ペイロード内に `actionId` を生成して含めます。
        - クライアントがレスポンスを期待する場合、アクションで `wantResponse: true` をサポートします。
        - A2A を使用している場合、サーバーへ送信するすべての A2A `Message` は、その `metadata` フィールドに `a2uiClientCapabilities` オブジェクトを含めなければなりません。
    - **Capabilities**:
        - Capabilities 交換の際に、`theme` の代わりに `surfaceProperties` を公開します。

---

## II. 基本コンポーネントカタログチェックリスト

プラットフォーム間で一貫したユーザー体験を確保するために、A2UI は基本的なコンポーネントのセットを定義しています。クライアントは、これらの抽象的な定義を、対応するネイティブの UI ウィジェットにマッピングする必要があります。

=== "v0.8"

    - **Text**: テキストを描画します。`usageHint`(h1〜h5、body、caption)をサポートします。
    - **Image**: URL から画像を描画します。`fit` と `usageHint`(avatar、hero など)をサポートします。
    - **Icon**: システムアイコンを描画します。
    - **Video**: 動画プレーヤーを描画します。
    - **AudioPlayer**: 説明付きの音声プレーヤーを描画します。
    - **Divider**: 水平・垂直の区切り線を描画します。
    - **Row** / **Column**: 子要素を水平・垂直方向に配置します。`distribution` と `alignment` をサポートします。子要素の `weight` をサポートします。
    - **List**: スクロール可能なリストを描画します。
    - **Card**: 角丸と影を持つボックスレイアウトです。
    - **Tabs**: `tabItems` を使ったタブナビゲーションです。
    - **Modal**: `entryPointChild` によってトリガーされ、`contentChild` を表示するポップアップです。
    - **Button**: `userAction` をトリガーするクリック可能なボタンです。`primary` バリアントをサポートします。
    - **CheckBox**: 真偽値のチェックボックスです。
    - **TextField**: `label`、`textFieldType`(`shortText`、`longText` など)、`validationRegexp` をサポートする入力フィールドです。
    - **MultipleChoice**: `options`、`maxAllowedSelections`、単一・複数の値選択をサポートします。
    - **Slider**: `minValue`、`maxValue` を使った範囲をサポートします。

=== "v0.9.1 (Current)"

    - **Text**: テキストを描画します。(`usageHint` を置き換える)`variant` をサポートします。
    - **Image**: URL から画像を描画します。`fit` と `variant` をサポートします。
    - **Icon**: システムアイコンを描画します。
    - **Video**: 動画プレーヤーを描画します。
    - **AudioPlayer**: 説明付きの音声プレーヤーを描画します。
    - **Divider**: 水平・垂直の区切り線を描画します。
    - **Row** / **Column**: 子要素を水平・垂直方向に配置します。`justify` と `align` をサポートします。子要素の `weight` をサポートします。
    - **List**: スクロール可能なリストを描画します。
    - **Card**: 角丸と影を持つボックスレイアウトです。
    - **Tabs**: `tabs` を使ったタブナビゲーションです。
    - **Modal**: `trigger` によってトリガーされ、`content` を表示するポップアップです。
    - **Button**: `action` をトリガーするクリック可能なボタンです。`variant`(primary、borderless)をサポートします。
    - **CheckBox**: 真偽値のチェックボックスです。
    - **TextField**: `label`、(`text` を置き換える)`value`、`variant`(`shortText`、`longText` など)、`checks`(検証関数)をサポートする入力フィールドです。
    - **ChoicePicker**:(MultipleChoice を置き換える)`options` と `variant`(`mutuallyExclusive`、`multipleSelection`)をサポートします。
    - **Slider**:(`minValue`、`maxValue` を置き換える)`min`、`max` を使った範囲をサポートします。

=== "v1.0 (Candidate)"

    v0.9.1 のすべてのコンポーネントに加えて、以下の拡張があります。
    - **Video**: プレビュー画像を表示する `posterUrl` プロパティをサポートします。
    - **TextField**: `placeholder` プロパティをサポートします。
