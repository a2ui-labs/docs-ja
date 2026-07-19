# メッセージタイプ

このリファレンスでは、すべての A2UI メッセージタイプについて詳細なドキュメントを提供します。

## メッセージ形式

すべての A2UI メッセージは、JSON Lines（JSONL）形式で送信される JSON オブジェクトです。各行にはちょうど1つのメッセージが含まれます。

=== "v0.8 メッセージタイプ"

    - `beginRendering` — クライアントにサーフェスのレンダリングを指示するシグナル
    - `surfaceUpdate` — コンポーネントを追加または更新する
    - `dataModelUpdate` — アプリケーションの状態を更新する
    - `deleteSurface` — サーフェスを削除する

=== "v0.9 メッセージタイプ"

    - `createSurface` — サーフェスを作成し、そのカタログを指定する
    - `updateComponents` — コンポーネントを追加または更新する
    - `updateDataModel` — アプリケーションの状態を更新する
    - `deleteSurface` — サーフェスを削除する

    すべての v0.9 メッセージには `"version": "v0.9"` フィールドが含まれます。

---

## beginRendering (v0.8) / createSurface (v0.9)

クライアントにサーフェスの初期化とレンダリングを指示します。

=== "v0.8 — `beginRendering`"

    ### スキーマ

    ```typescript
    {
      beginRendering: {
        surfaceId: string;      // Required: Unique surface identifier
        root: string;           // Required: The ID of the root component to render
        catalogId?: string;     // Optional: URL of component catalog
        styles?: object;        // Optional: Styling information
      }
    }
    ```

    ### プロパティ

    | Property    | Type   | Required | Description                                                                             |
    | ----------- | ------ | -------- | ---------------------------------------------------------------------------------------- |
    | `surfaceId` | string | ✅        | このサーフェスの一意の識別子です。                                                        |
    | `root`      | string | ✅        | このサーフェスの UI ツリーのルートとなるべきコンポーネントの `id` です。                  |
    | `catalogId` | string | ❌        | コンポーネントカタログの識別子です。省略した場合は v0.8 の basic catalog がデフォルトで使用されます。 |
    | `styles`    | object | ❌        | カタログで定義された、UI のスタイリング情報です。                                        |

    ### 例

    ```json
    {
      "beginRendering": {
        "surfaceId": "main",
        "root": "root-component"
      }
    }
    ```

    **カスタムカタログを使用する場合:**

    ```json
    {
      "beginRendering": {
        "surfaceId": "custom-ui",
        "root": "root-custom",
        "catalogId": "https://my-company.com/a2ui/v0.8/my_custom_catalog.json"
      }
    }
    ```

    コンポーネント定義の後に送信する必要があります。クライアントは `beginRendering` を受信するまで、`surfaceUpdate` と `dataModelUpdate` メッセージをバッファリングします。

=== "v0.9 — `createSurface`"

    ### スキーマ

    ```typescript
    {
      version: "v0.9";
      createSurface: {
        surfaceId: string;      // Required: Unique surface identifier
        catalogId: string;      // Required: URL of component catalog
        theme?: object;         // Optional: Theme configuration
        sendDataModel?: boolean; // Optional: Request client to send data model updates
      }
    }
    ```

    ### プロパティ

    | Property        | Type    | Required | Description                                                     |
    | --------------- | ------- | -------- | ----------------------------------------------------------------- |
    | `surfaceId`     | string  | ✅        | このサーフェスの一意の識別子です。                               |
    | `catalogId`     | string  | ✅        | コンポーネントカタログの識別子です。                             |
    | `theme`         | object  | ❌        | テーマの設定です（例: `primaryColor`）。                         |
    | `sendDataModel` | boolean | ❌        | true の場合、クライアントはデータモデルの変更をサーバーへ送り返します。 |

    ### 例

    ```json
    {
      "version": "v0.9",
      "createSurface": {
        "surfaceId": "main",
        "catalogId": "https://a2ui.org/specification/v0_9/catalogs/basic/catalog.json"
      }
    }
    ```

    v0.9 では、`createSurface` が `beginRendering` を置き換えます。ルートは規約によって決定されます。`updateComponents` 内のいずれか1つのコンポーネントが `"id": "root"` を持つ必要があります。`catalogId` は必須です。

---

## surfaceUpdate (v0.8) / updateComponents (v0.9)

サーフェス内のコンポーネントを追加または更新します。

=== "v0.8 — `surfaceUpdate`"

    ### スキーマ

    ```typescript
    {
      surfaceUpdate: {
        surfaceId: string;        // Required: Target surface
        components: Array<{       // Required: List of components
          id: string;             // Required: Component ID
          component: {            // Required: Wrapper for component data
            [ComponentType]: {    // Required: Exactly one component type
              ...properties       // Component-specific properties
            }
          }
        }>
      }
    }
    ```

    ### プロパティ

    | Property     | Type   | Required | Description                    |
    | ------------ | ------ | -------- | ------------------------------- |
    | `surfaceId`  | string | ✅        | 更新対象のサーフェスの ID です  |
    | `components` | array  | ✅        | コンポーネント定義の配列        |

    ### コンポーネントオブジェクト

    `components` 配列内の各オブジェクトは、次を持つ必要があります。

    - `id`（string、必須）: サーフェス内で一意の識別子
    - `component`（object、必須）: コンポーネントタイプ（例: `Text`、`Button`）を表すキーを1つだけ含むラッパーオブジェクト

    ### 例

    **単一のコンポーネント:**

    ```json
    {
      "surfaceUpdate": {
        "surfaceId": "main",
        "components": [
          {
            "id": "greeting",
            "component": {
              "Text": {
                "text": {"literalString": "Hello, World!"},
                "usageHint": "h1"
              }
            }
          }
        ]
      }
    }
    ```

    **複数のコンポーネント（隣接リスト）:**

    ```json
    {
      "surfaceUpdate": {
        "surfaceId": "main",
        "components": [
          {
            "id": "root",
            "component": {
              "Column": {
                "children": {"explicitList": ["header", "body"]}
              }
            }
          },
          {
            "id": "header",
            "component": {
              "Text": {
                "text": {"literalString": "Welcome"}
              }
            }
          },
          {
            "id": "body",
            "component": {
              "Card": {
                "child": "content"
              }
            }
          },
          {
            "id": "content",
            "component": {
              "Text": {
                "text": {"path": "/message"}
              }
            }
          }
        ]
      }
    }
    ```

    **既存のコンポーネントを更新する:**

    ```json
    {
      "surfaceUpdate": {
        "surfaceId": "main",
        "components": [
          {
            "id": "greeting",
            "component": {
              "Text": {
                "text": {"literalString": "Hello, Alice!"},
                "usageHint": "h1"
              }
            }
          }
        ]
      }
    }
    ```

    `id: "greeting"` を持つコンポーネントが（重複を作らず）更新されます。

    ### 使用上の注意

    次の点に注意してください。
    - 1つのコンポーネントを、ツリーのルートとして機能するように `beginRendering` メッセージ内で `root` として指定する必要があります。
    - コンポーネントは隣接リスト（ID 参照を持つフラットな構造）を形成します。
    - 既存の ID を持つコンポーネントを送信すると、そのコンポーネントが更新されます。
    - 子要素は ID によって参照されます。
    - コンポーネントは段階的に（ストリーミングで）追加できます。

=== "v0.9 — `updateComponents`"

    ### スキーマ

    ```typescript
    {
      version: "v0.9";
      updateComponents: {
        surfaceId: string;        // Required: Target surface
        components: Array<{       // Required: List of components
          id: string;             // Required: Component ID
          component: string;      // Required: Component type name
          ...properties           // Component-specific properties (flat)
        }>
      }
    }
    ```

    ### プロパティ

    | Property     | Type   | Required | Description                    |
    | ------------ | ------ | -------- | ------------------------------- |
    | `surfaceId`  | string | ✅        | 更新対象のサーフェスの ID です  |
    | `components` | array  | ✅        | コンポーネント定義の配列        |

    ### コンポーネントオブジェクト

    v0.9 では、コンポーネントの構造はよりフラットです。

    - `id`（string、必須）: サーフェス内で一意の識別子
    - `component`（string、必須）: コンポーネントタイプ名（例: `"Text"`、`"Button"`）
    - それ以外のすべてのプロパティは、コンポーネントオブジェクトの直下（トップレベル）に置かれます。

    ### 例

    **単一のコンポーネント:**

    ```json
    {
      "version": "v0.9",
      "updateComponents": {
        "surfaceId": "main",
        "components": [
          {
            "id": "greeting",
            "component": "Text",
            "text": "Hello, World!",
            "variant": "h1"
          }
        ]
      }
    }
    ```

    **複数のコンポーネント:**

    ```json
    {
      "version": "v0.9",
      "updateComponents": {
        "surfaceId": "main",
        "components": [
          {
            "id": "root",
            "component": "Column",
            "children": ["header", "body"]
          },
          {
            "id": "header",
            "component": "Text",
            "text": "Welcome"
          },
          {
            "id": "body",
            "component": "Card",
            "child": "content"
          },
          {
            "id": "content",
            "component": "Text",
            "text": {"path": "/message"}
          }
        ]
      }
    }
    ```

    **既存のコンポーネントを更新する:**

    ```json
    {
      "version": "v0.9",
      "updateComponents": {
        "surfaceId": "main",
        "components": [
          {
            "id": "greeting",
            "component": "Text",
            "text": "Hello, Alice!",
            "variant": "h1"
          }
        ]
      }
    }
    ```

    ### 使用上の注意

    次の点に注意してください。
    - 1つのコンポーネントが、ツリーのルートとして機能するために `"id": "root"` を持つ必要があります（独立したメッセージフィールドではなく、規約によるものです）。
    - コンポーネントタイプは、ラッパーオブジェクトではなく文字列（`"component": "Text"`）です。
    - プロパティはコンポーネントオブジェクト上にフラットに配置されます（タイプキーの下にネストされません）。
    - データバインディングには `{"path": "/pointer"}`（JSON Pointer）を使用します。キー名は v0.8 と同じですが、標準の JSON Pointer パスを使用します。
    - コンポーネントは段階的に（ストリーミングで）追加できます。

### エラー

| Error                  | Cause                                  | Solution                                                                                                                                                                                      |
| ---------------------- | --------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| サーフェスが既に存在する | `surfaceId` が既に使用されている          | `surfaceId` がレンダラーのライフタイムを通じてグローバルに一意であることを確認してください。サブエージェントを伴うオーケストレーターを使用している場合、オーケストレーターは競合を避けるために必要に応じてサーフェス ID を管理する権限を持ちます。 |
| サーフェスが見つからない | `surfaceId` が存在しない                | 作成済みのサーフェスと `surfaceId` が一致していることを確認してください。v0.8 ではサーフェスは暗黙的ですが、v0.9 以降では `createSurface` が必要です。                                          |
| 無効なコンポーネントタイプ | 不明なコンポーネントタイプ              | ネゴシエートされたカタログにコンポーネントタイプが存在するか確認してください。                                                                                                                |
| 無効なプロパティ       | そのタイプに存在しないプロパティ         | カタログのスキーマと照合して確認してください。                                                                                                                                                |
| 循環参照               | コンポーネントが自分自身を子として参照している | コンポーネントの階層構造を修正してください。                                                                                                                                                  |

---

## dataModelUpdate (v0.8) / updateDataModel (v0.9)

コンポーネントがバインドするデータモデルを更新します。

=== "v0.8 — `dataModelUpdate`"

    ### スキーマ

    ```typescript
    {
      dataModelUpdate: {
        surfaceId: string;      // Required: Target surface
        path?: string;          // Optional: Path to a location in the model
        contents: Array<{       // Required: Data entries
          key: string;
          valueString?: string;
          valueNumber?: number;
          valueBoolean?: boolean;
          valueMap?: Array<{...}>;
        }>
      }
    }
    ```

    ### プロパティ

    | Property    | Type   | Required | Description                                                                                          |
    | ----------- | ------ | -------- | ------------------------------------------------------------------------------------------------------ |
    | `surfaceId` | string | ✅        | 更新対象のサーフェスの ID です。                                                                     |
    | `path`      | string | ❌        | データモデル内の場所へのパスです（例: `user`）。省略した場合、更新はルートに適用されます。            |
    | `contents`  | array  | ✅        | 隣接リストとしてのデータエントリの配列です。各エントリは `key` と、型付きの `value*` プロパティを持ちます。 |

    ### `contents` 隣接リスト

    `contents` 配列は、キーと値のペアのリストです。配列内の各オブジェクトは、`key` と、ちょうど1つの `value*` プロパティ（`valueString`、`valueNumber`、`valueBoolean`、`valueMap` のいずれか）を持つ必要があります。この構造は LLMフレンドリーであり、汎用的な `value` フィールドから型を推論する際の問題を回避します。

    ### 例

    **モデル全体を初期化する:**

    ```json
    {
      "dataModelUpdate": {
        "surfaceId": "main",
        "contents": [
          {
            "key": "user",
            "valueMap": [
              { "key": "name", "valueString": "Alice" },
              { "key": "email", "valueString": "alice@example.com" }
            ]
          },
          { "key": "items", "valueMap": [] }
        ]
      }
    }
    ```

    **ネストされたプロパティを更新する:**

    ```json
    {
      "dataModelUpdate": {
        "surfaceId": "main",
        "path": "user",
        "contents": [
          { "key": "email", "valueString": "alice@newdomain.com" }
        ]
      }
    }
    ```

    これにより、`/user/name` に影響を与えずに `/user/email` が変更されます。

    ### 使用上の注意

    次の点に注意してください。
    - データモデルはサーフェスごとに存在します。
    - バインドされたデータが変更されると、コンポーネントは自動的に再レンダリングされます。
    - モデル全体を置き換えるよりも、特定のパスへの粒度の細かい更新を優先してください。
    - 型付きの値フィールド（`valueString`、`valueNumber`、`valueBoolean`、`valueMap`）を使用します。LLMフレンドリーで、型推論は不要です。
    - データの変換（日付のフォーマットなど）は、メッセージを送信する前にサーバー側で行う必要があります。

=== "v0.9 — `updateDataModel`"

    ### スキーマ

    ```typescript
    {
      version: "v0.9";
      updateDataModel: {
        surfaceId: string;      // Required: Target surface
        path?: string;          // Optional: JSON Pointer path (defaults to "/")
        value?: any;            // Optional: Value to set (omit to delete)
      }
    }
    ```

    ### プロパティ

    | Property    | Type   | Required | Description                                           |
    | ----------- | ------ | -------- | -------------------------------------------------------- |
    | `surfaceId` | string | ✅        | 更新対象のサーフェスの ID です。                       |
    | `path`      | string | ❌        | JSON Pointer パスです（例: `/user/email`）。デフォルトは `/`（ルート）です。 |
    | `value`     | any    | ❌        | 設定する値です。省略した場合、`path` にあるキーは削除されます。 |

    ### 例

    **モデル全体を初期化する:**

    ```json
    {
      "version": "v0.9",
      "updateDataModel": {
        "surfaceId": "main",
        "path": "/",
        "value": {
          "user": {
            "name": "Alice",
            "email": "alice@example.com"
          },
          "items": []
        }
      }
    }
    ```

    **ネストされたプロパティを更新する:**

    ```json
    {
      "version": "v0.9",
      "updateDataModel": {
        "surfaceId": "main",
        "path": "/user/email",
        "value": "alice@newdomain.com"
      }
    }
    ```

    ### 使用上の注意

    次の点に注意してください。
    - v0.9 は標準的な JSON Pointer パスとプレーンな JSON 値を使用します。型付きラッパーはありません。
    - `path` は省略した場合、`"/"`（ルート）がデフォルトになります。
    - `value` には任意の JSON 型（string、number、boolean、object、array、null）を指定できます。省略すると削除されます。
    - v0.8 の `contents` 隣接リストよりシンプルで、標準的な JSON Patch のセマンティクスに近い構造です。
    - `{"path": "/user/email"}` を参照しているコンポーネントは、そのパスが変更されると自動的に更新されます。

---

## deleteSurface

サーフェスと、それに関連するすべてのコンポーネントおよびデータを削除します。

=== "v0.8 — `deleteSurface`"

    ### スキーマ

    ```typescript
    {
      deleteSurface: {
        surfaceId: string;        // Required: Surface to delete
      }
    }
    ```

    ### 例

    ```json
    {
      "deleteSurface": {
        "surfaceId": "modal"
      }
    }
    ```

=== "v0.9 — `deleteSurface`"

    ### スキーマ

    ```typescript
    {
      version: "v0.9";
      deleteSurface: {
        surfaceId: string;        // Required: Surface to delete
      }
    }
    ```

    ### 例

    ```json
    {
      "version": "v0.9",
      "deleteSurface": {
        "surfaceId": "modal"
      }
    }
    ```

### プロパティ

| Property    | Type   | Required | Description                 |
| ----------- | ------ | -------- | ---------------------------- |
| `surfaceId` | string | ✅       | 削除対象のサーフェスの ID です |

### 使用上の注意

次の点に注意してください。

- サーフェスに関連するすべてのコンポーネントを削除します。
- サーフェスのデータモデルをクリアします。
- クライアントは UI からサーフェスを削除する必要があります。
- 存在しないサーフェスを削除しても安全です（何も起こりません）。
- モーダルやダイアログを閉じるとき、または画面遷移するときに使用します。
- 両バージョンで構造は同一です（v0.9 では `version` フィールドが追加されるのみです）。

---

## メッセージの順序

### 要件

メッセージの順序は、次の要件を満たす必要があります。

1. `beginRendering` は、そのサーフェスに対する最初の `surfaceUpdate` メッセージより後に送信する必要があります。
2. `surfaceUpdate` は `dataModelUpdate` の前後どちらでも構いません。
3. 異なるサーフェスに対するメッセージは互いに独立しています。
4. 複数のメッセージによって、同じサーフェスを段階的に更新できます。

### 推奨される順序

=== "v0.8"

    ```jsonl
    { "surfaceUpdate":    { "surfaceId": "main", "components": [...] } }
    { "dataModelUpdate":  { "surfaceId": "main", "contents": {...} } }
    { "beginRendering":   { "surfaceId": "main", "root": "root-id" } }
    ```

=== "v0.9"

    ```jsonl
    { "version": "v0.9", "createSurface":    { "surfaceId": "main", "catalogId": "..." } }
    { "version": "v0.9", "updateComponents": { "surfaceId": "main", "components": [...] } }
    { "version": "v0.9", "updateDataModel":  { "surfaceId": "main", "path": "/", "value": {...} } }
    ```

### 段階的な構築

=== "v0.8"

    ```jsonl
    { "surfaceUpdate":   { "surfaceId": "main", "components": [...] } }  // Header
    { "surfaceUpdate":   { "surfaceId": "main", "components": [...] } }  // Body
    { "beginRendering":  { "surfaceId": "main", "root": "root-id" } }   // Render
    { "surfaceUpdate":   { "surfaceId": "main", "components": [...] } }  // Footer
    { "dataModelUpdate": { "surfaceId": "main", "contents": {...} } }    // Data
    ```

=== "v0.9"

    ```jsonl
    { "version": "v0.9", "createSurface":    { "surfaceId": "main", "catalogId": "..." } }
    { "version": "v0.9", "updateComponents": { "surfaceId": "main", "components": [...] } }  // Header
    { "version": "v0.9", "updateComponents": { "surfaceId": "main", "components": [...] } }  // Body + Footer
    { "version": "v0.9", "updateDataModel":  { "surfaceId": "main", "path": "/", "value": {...} } }
    ```

## 検証

=== "v0.8"

    以下に対して検証してください。

    - **[server_to_client.json](../../../specification/v0_8/json/server_to_client.json)**: メッセージエンベロープのスキーマ
    - **[standard_catalog_definition.json](../../../specification/v0_8/json/standard_catalog_definition.json)**: コンポーネントのスキーマ

=== "v0.9"

    以下に対して検証してください。

    - **[server_to_client.json](../../../specification/v0_9/json/server_to_client.json)**: メッセージエンベロープのスキーマ
    - **[catalogs/basic/catalog.json](../../../specification/v0_9/catalogs/basic/catalog.json)**: コンポーネントのスキーマ

## 参考資料

- **[コンポーネントギャラリー](components.md)**: 利用可能なすべてのコンポーネントタイプ
- **[データバインディングガイド](../concepts/data-binding.md)**: データバインディングの仕組み
- **[エージェント開発ガイド](../guides/agent-development.md)**: 有効なメッセージを生成する
