# Model Context Protocol (MCP) 上での A2UI

このガイドでは、**Tools** と **Embedded Resources** を使って **MCP サーバー** から **リッチで対話的な A2UI インターフェース** を配信する方法を説明します。最後まで進めると、あらゆる MCP 互換クライアントに A2UI コンポーネントを返す、実際に動作する MCP サーバーを構築できます。

<video width="100%" height="auto" controls playsinline style="display: block; aspect-ratio: 16/9; object-fit: cover; border-radius: 8px; margin-bottom: 24px;">
  <source src="../assets/guides-a2ui-over-mcp-tour.mp4" type="video/mp4">
  お使いのブラウザは video タグをサポートしていません。
</video>

## 前提条件

開始する前に、以下がインストールされていることを確認してください。

- **Python** (バージョン 3.10 以降)
- 高速な Python パッケージ管理用の **[uv](https://docs.astral.sh/uv/)**
- MCP Inspector 用の **Node.js** (バージョン 18 以降)

## クイックスタート: サンプルの実行

プロトコルの詳細に入る前に、実際に動作するサンプルを実行してみましょう。A2UI リポジトリには、すぐに使える MCP レシピデモが含まれています。

```bash
# リポジトリをクローン (まだ行っていない場合)
git clone https://github.com/a2ui-project/a2ui.git
cd a2ui/samples/community/mcp/a2ui-over-mcp-recipe

# MCP サーバーを起動 (ポート 8000 で SSE トランスポート)
uv run .
```

### オプション A: MCP Inspector を使った対話

別のターミナルで [MCP Inspector](https://github.com/modelcontextprotocol/inspector) を起動し、サーバーと対話します。

```bash
npx @modelcontextprotocol/inspector@latest --web --transport sse --server-url http://localhost:8000/sse
```

`http://localhost:6274` を開きます:

1. **List Resources** をクリック → `a2ui://recipe-form` および `a2ui://recipe-card` が表示されます。
2. リソースを読み取る → データバインディングを含む静的な A2UI プレゼンテーションテンプレート (`createSurface` と `updateComponents`) が含まれます。
3. **List Tools** をクリック → それぞれのプレゼンテーションテンプレートリソースへの `_meta.ui` リンクを持つ `get_recipe_form_a2ui` と `get_recipe_a2ui` が表示されます。
4. `get_recipe_form_a2ui` を実行 → ツールが `updateDataModel` メッセージでラップされた初期フォーム選択状態を返します。
5. カスタムパラメータで `get_recipe_a2ui` を実行 → ツールが `updateDataModel` メッセージでラップされた動的なレシピ詳細を返します。

> [!NOTE]
> このサンプルは A2UI Agent SDK へのローカルパス参照を使用しています。ご自身のプロジェクトでは PyPI からインストールしてください:
>
> ```bash
> pip install a2ui-agent-sdk
> ```

### オプション B: レシピクライアント Web アプリの実行

対話型の Web クライアントを実行するには:

> [!NOTE]
> A2UI リポジトリ内の組み込みサンプルアプリケーションを実行するには、Yarn ワークスペース (`yarn install` / `yarn dev`) を使用します。このリポジトリ外では、任意のパッケージマネージャー (npm, pnpm, yarn) を使用できます。

1. 新しいターミナルウィンドウで、client ディレクトリに移動します:
    ```bash
    cd client
    ```
2. 依存関係をインストールします:
    ```bash
    yarn install
    ```
3. Vite 開発サーバーを起動します:
    ```bash
    yarn dev
    ```
4. ブラウザで `http://localhost:5173` を開きます。

アプリケーションがロードされると、クライアントは SSE 経由で MCP サーバーに接続し、`get_recipe_form_a2ui` を実行します。`_meta.ui` を読み取って `a2ui://recipe-form` プレゼンテーションテンプレートを取得・キャッシュし、返された `updateDataModel` を適用してデフォルトの選択肢 (`Grilled`, `Chicken`) を入力します。オプションを選択して **「Get Recipe」** をクリックすると `get_recipe_a2ui` が実行され、`a2ui://recipe-card` を取得して右側のカラムにレシピ詳細を動的にレンダリングします。

![選択フォームと動的なレシピカード生成を示す Dynamic Recipe Studio デモ](../assets/recipe_sample.gif)

すべてのサンプルは [`samples/community/mcp/`](../../../samples/community/mcp) で確認できます。

## 分離アーキテクチャ: プレゼンテーションとデータの分離

MCP 上の A2UI は、ユーザーインターフェースを 2 つのレイヤーに分離します:

1. **MCP Resources による静的プレゼンテーションテンプレート (`resources/read`)**:
   データバインディング (`/title`, `/cookTime`, `/image` など) を持つコンポーネントツリー (`createSurface` および `updateComponents`) を含むレイアウトは、カスタム URI (例: `a2ui://recipe-form`, `a2ui://recipe-card`) のもとで MIME タイプ `application/a2ui+json` を持つ MCP リソースとして配信されます。テンプレートにはハードコードされたデータ値が含まれないため、クライアントはローカルに取得してキャッシュできます。
2. **MCP Tools による動的データ更新 (`tools/call`)**:
   ツールが実行されると、サーバーはテンプレートに必要な動的値のみを A2UI `updateDataModel` メッセージとしてパッケージ化して返します。
3. **ツール UI メタデータ (`_meta.ui`)**:
   ツールは、ツール定義および `CallToolResult` に `_meta.ui` オブジェクトを含めることで、プレゼンテーションテンプレートにリンクします:
    ```json
    "_meta": {
      "ui": {
        "resourceUri": "a2ui://recipe-card",
        "mimeType": "application/a2ui+json"
      }
    }
    ```
4. **クライアント側での解決とハイドレーション (Client-Side Resolution & Hydration)**:
   クライアントホストは `_meta.ui.resourceUri` を検査し、ローカルテンプレートキャッシュを確認 (初回ロード時はサーバーからリソースを取得) してサーフェスレイアウトを初期化し、ツールレスポンスからの動的 `updateDataModel` を適用します。

> [!IMPORTANT]
> **MIME タイプの一貫性**
> 静的テンプレートリソースと動的ツールペイロードの双方が `application/a2ui+json` MIME タイプを使用します。ツールレスポンスでは、データモデル更新はフォールバック用 `TextContent` とともに `EmbeddedResource` 内にラップされて返されます。この識別により、クライアントアプリケーションはペイロードを A2UI プロセッサへ直接ルーティングできます。

### 配信フロー

```
1. ツール呼び出し (Tool Invocation)
クライアント → tools/call (例: get_recipe_a2ui) → MCP サーバー
                                                      ↓
                                            動的な値を計算
                                                      ↓
クライアント ← CallToolResult (updateDataModel) ← MCP サーバー
         + _meta.ui: { resourceUri: "a2ui://recipe-card" }

2. テンプレート解決 (初回取得後にキャッシュ)
クライアントのキャッシュに "a2ui://recipe-card" がない場合:
  クライアント → resources/read ("a2ui://recipe-card") → MCP サーバー
  クライアント ← テンプレート (createSurface, updateComponents) ← MCP サーバー

3. サーフェスハイドレーション (Surface Hydration)
クライアントがツールレスポンスからの updateDataModel をサーフェスに適用
A2UI レンダラーが表示を更新
```

### 1. MCP Resources によるプレゼンテーションテンプレートの定義

`resources/list` および `resources/read` を通じて静的レイアウトテンプレートを公開します:

```python
@app.list_resources()
async def list_resources() -> list[types.Resource]:
    return [
        types.Resource(
            uri="a2ui://recipe-form",
            name="Recipe Form",
            mimeType="application/a2ui+json",
            description="Static form allowing users to pick cuisine and protein.",
        ),
        types.Resource(
            uri="a2ui://recipe-card",
            name="Recipe Card",
            mimeType="application/a2ui+json",
            description="Static recipe card layout template.",
        ),
    ]


@app.read_resource()
async def read_resource(uri: str) -> list[ReadResourceContents]:
    if str(uri) == "a2ui://recipe-form":
        return [
            ReadResourceContents(
                content=json.dumps(recipe_form_json),
                mime_type="application/a2ui+json",
            )
        ]
    if str(uri) == "a2ui://recipe-card":
        return [
            ReadResourceContents(
                content=json.dumps(recipe_a2ui_json),
                mime_type="application/a2ui+json",
            )
        ]
    raise ValueError(f"Unknown resource: {uri}")
```

### 2. ツール UI メタデータの宣言

ツール定義でプレゼンテーションリソースの URI を宣言します:

```python
types.Tool(
    name="get_recipe_a2ui",
    title="Get Recipe A2UI",
    description="Returns recipe data and links to the recipe-card template.",
    inputSchema={
        "type": "object",
        "properties": {
            "cookingStyle": {
                "type": "array",
                "items": {"type": "string"},
                "description": "Selected cooking styles",
            },
            "protein": {
                "type": "array",
                "items": {"type": "string"},
                "description": "Selected proteins",
            },
        },
        "additionalProperties": True,
    },
    _meta={
        "ui": {
            "resourceUri": "a2ui://recipe-card",
            "mimeType": "application/a2ui+json",
        }
    },
)
```

### 3. ツール実行による動的データの返却

ツール呼び出しハンドラーで、`_meta.ui` とともに動的状態を `updateDataModel` メッセージとして返します:

```python
@app.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, Any]
) -> types.CallToolResult:
    if name == "get_recipe_a2ui":
        # ユーザー引数から選択されたレシピを解決
        style_list = arguments.get("cookingStyle", ["Baked"])
        protein_list = arguments.get("protein", ["Salmon"])
        style = style_list[0] if style_list else "Baked"
        protein = protein_list[0] if protein_list else "Salmon"
        recipe = RECIPES.get((style, protein))

        # 軽量な updateDataModel ペイロードを生成
        data_model_update = [
            {
                "version": "v0.9",
                "updateDataModel": {
                    "surfaceId": "recipe-card",
                    "path": "/",
                    "value": {
                        "title": recipe["title"],
                        "rating": recipe["rating"],
                        "reviews": recipe["reviews"],
                        "cookTime": recipe["cookTime"],
                        "prepTime": recipe["prepTime"],
                        "servings": recipe["servings"],
                        "image": recipe["image"],
                    },
                },
            }
        ]

        return types.CallToolResult(
            content=[
                types.TextContent(
                    type="text",
                    text=f"Generated recipe: {recipe['title']}",
                ),
                types.EmbeddedResource(
                    type="resource",
                    resource=types.TextResourceContents(
                        uri="a2ui://recipe-card/data",
                        mimeType="application/a2ui+json",
                        text=json.dumps(data_model_update),
                    ),
                ),
            ],
            _meta={
                "ui": {
                    "resourceUri": "a2ui://recipe-card",
                    "mimeType": "application/a2ui+json",
                }
            },
        )
```

## カタログネゴシエーション

サーバーがクライアントに A2UI を送信する前に、双方はどのカタログが利用可能かを合意する必要があります。システムのアーキテクチャに応じて、このネゴシエーションは 2 つの方法のいずれかで行われます。

### オプション A: MCP 初期化時 (推奨)

MCP はステートフルなセッションプロトコルであるため、最も効率的な方法は接続確立時に一度だけ機能を宣言することです。クライアントは `capabilities` のもとで A2UI サポートを宣言します:

```json
{
  "jsonrpc": "2.0",
  "method": "initialize",
  "id": "init-123",
  "params": {
    "protocolVersion": "2025-11-25",
    "clientInfo": {
      "name": "a2ui-enabled-client",
      "version": "1.0.0"
    },
    "capabilities": {
      "a2ui": {
        "clientCapabilities": {
          "v0.9": {
            "supportedCatalogIds": [
              "https://a2ui.org/specification/v0_9/basic_catalog.json"
            ]
          }
        }
      }
    }
  }
}
```

サーバーはセッションの間、この状態を保持します。

### オプション B: メッセージごとのメタデータ (ステートレスサーバー用)

サーバーをステートレスに保つ必要がある場合、クライアントはすべてのツール呼び出しの `_meta` フィールドで A2UI 機能を渡すことができます:

```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "id": "id-123",
  "params": {
    "name": "generate_report",
    "arguments": {"date": "2026-03-01"},
    "_meta": {
      "a2ui": {
        "clientCapabilities": {
          "v0.9": {
            "supportedCatalogIds": [
              "https://a2ui.org/specification/v0_9/basic_catalog.json"
            ],
            "inlineCatalogs": []
          }
        }
      }
    }
  }
}
```

## ユーザーアクションの処理

`Button` などのインタラクティブなコンポーネントは、MCP ツール呼び出しとしてサーバーに送り返されるアクションをトリガーできます。

### 1. アクション付き Button の定義

A2UI JSON 内で、コンポーネントに `action` を追加します:

```json
{
  "id": "confirm-button",
  "component": {
    "Button": {
      "child": "confirm-button-text",
      "action": {
        "event": {
          "name": "confirm_booking",
          "context": {
            "start": "/dates/start",
            "end": "/dates/end"
          }
        }
      }
    }
  }
}
```

### 2. クライアントがアクションをツール呼び出しとして送信

ユーザーがボタンをクリックすると、クライアントはサーフェス状態に対してデータバインディング (`/dates/start` など) を解決し、必須のアクションフィールドを含むツール呼び出しを送信します:

```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "id": "id-456",
  "params": {
    "name": "a2ui_action",
    "arguments": {
      "name": "confirm_booking",
      "surfaceId": "booking-surface",
      "sourceComponentId": "confirm-button",
      "timestamp": "2026-03-20T12:00:00Z",
      "context": {
        "start": "2026-03-20",
        "end": "2026-03-25"
      }
    }
  }
}
```

### 3. サーバー側でのアクション処理

```python
@app.tool()
async def a2ui_action(
    name: str,
    surfaceId: str,
    sourceComponentId: str,
    timestamp: str,
    context: dict[str, Any],
) -> types.CallToolResult:
    """A2UI ユーザーアクションを処理します。"""
    if name == "confirm_booking":
        # 予約を処理し、確認 UI を返します
        return types.CallToolResult(content=[
            types.TextContent(
                type="text",
                text=f"Booking confirmed for {surfaceId}: {context['start']} to {context['end']}"
            )
        ])
    raise ValueError(f"Unknown action: {name}")
```

> [!NOTE]
> 5 つのアクションフィールド (`name`, `surfaceId`, `sourceComponentId`, `timestamp`, `context`) はすべて A2UI 仕様で必須です。ツールのパラメータに全フィールドを宣言することで、MCP SDK が `surfaceId` などを除外してサーフェスコンテキストが失われるのを防ぎます。

## エラーハンドリング

クライアントは、ツール呼び出しを介して A2UI のレンダリングおよびバリデーションエラーをサーバーに報告できます:

```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "id": "id-789",
  "params": {
    "name": "a2ui_error",
    "arguments": {
      "code": "VALIDATION_FAILED",
      "surfaceId": "booking-surface",
      "path": "/components/0/text",
      "message": "Failed to parse A2UI payload."
    }
  }
}
```

サーバー側で処理します:

```python
@app.tool()
async def a2ui_error(
    code: str,
    surfaceId: str,
    message: str,
    path: str | None = None,
) -> types.CallToolResult:
    """A2UI クライアントエラーを処理します。"""
    # エラーをログに記録し、再試行またはフォールバック UI を送信します
    return types.CallToolResult(content=[
        types.TextContent(
            type="text",
            text=f"Acknowledged error {code} on surface {surfaceId}: {message}"
        )
    ])
```

## バーバライゼーションと可視性制御

MCP **Resource Annotations** を使用して、後続のターンで LLM が A2UI ペイロードを「読む」ことができるかどうかを制御します:

```python
a2ui_resource = types.EmbeddedResource(
    type="resource",
    resource=types.TextResourceContents(
        uri="a2ui://training-plan-page",
        mimeType="application/a2ui+json",
        text=json.dumps(a2ui_payload)
    ),
    # ユーザーには UI を表示し、LLM には生の JSON を隠す
    annotations=types.Annotations(audience=["user"])
)
```

| 対象 (Audience) | 動作                                                       |
| --------------- | ---------------------------------------------------------- |
| _(空)_          | ユーザーと LLM の両方に表示されます                        |
| `["user"]`      | ユーザー向けにレンダリングされ、LLM コンテキストからは隠されます |
| `["assistant"]` | LLM が後続の推論に利用できますが、レンダリングはされません  |

## A2UI Agent SDK の使用

本番環境では、**A2UI Agent SDK** がスキーマ管理、バリデーション、プロンプト生成を自動化します:

```bash
pip install a2ui-agent-sdk
```

```python
from a2ui.strategies.schema import A2uiSchemaManager
from a2ui.basic_catalog.provider import BasicCatalog

# Basic Catalog を使ってスキーママネージャーを初期化
schema_manager = A2uiSchemaManager(
    catalogs=[BasicCatalog.get_config()],
)

# 送信前に A2UI 出力を検証
selected_catalog = schema_manager.get_selected_catalog()
selected_catalog.validator.validate(a2ui_payload)
```

スキーマ管理、動的カタログ、ストリーミングの詳細については、完全な [エージェント開発ガイド](agent-development.md) を参照してください。

## 次のステップ

- [A2UI 仕様](../specification/v0.9-a2ui.md) — 完全なプロトコルリファレンス
- [コンポーネントギャラリー](../reference/components.md) — 利用可能なコンポーネントの一覧
- [A2UI Surface 内の MCP Apps](mcp-apps-in-a2ui.md) — HTML ベースの MCP アプリを A2UI 内に埋め込む
- [クライアントのセットアップ](client-setup.md) — A2UI を表示するレンダーの構築
