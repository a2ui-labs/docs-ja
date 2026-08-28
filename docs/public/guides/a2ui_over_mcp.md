# Model Context Protocol (MCP) 上での A2UI

このガイドでは、**Tools** と **Embedded Resources** を使って **MCP サーバー** から **リッチで対話的な A2UI インターフェース** を配信する方法を説明します。最後まで進めると、あらゆる MCP 互換クライアントに A2UI コンポーネントを返す、実際に動作する MCP サーバーを構築できます。

<video width="100%" height="auto" controls playsinline style="display: block; aspect-ratio: 16/9; object-fit: cover; border-radius: 8px; margin-bottom: 24px;">
  <source src="https://raw.githubusercontent.com/a2ui-project/a2ui/main/docs/public/assets/guides-a2ui-over-mcp-tour.mp4" type="video/mp4">
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
cd a2ui/samples/mcp/a2ui-over-mcp-recipe

# MCP サーバーを起動 (ポート 8000 で SSE トランスポート)
uv run .
```

### オプション A: MCP Inspector を使った対話

別のターミナルで [MCP Inspector](https://github.com/modelcontextprotocol/inspector) を起動し、サーバーと対話します。

```bash
npx @modelcontextprotocol/inspector
```

Inspector 内で以下の操作を行います:

1. **Transport Type** を `SSE` に設定します。
2. `http://localhost:8000/sse` に接続します。
3. **List Resources** をクリック → "Recipe Form" リソースが表示されます。
4. `a2ui://recipe-form` リソースを読み取ります → リソース内容はシンプルなフォームをレンダリングする A2UI JSON です。
5. **List Tools** をクリック → `get_recipe_a2ui` が表示されます。
6. ツールを実行します → レスポンスにレシピカードをレンダリングする A2UI JSON が含まれます。

> NOTE: 注記
>
> このサンプルは A2UI Agent SDK へのローカルパス参照を使用しています。ご自身のプロジェクトでは PyPI からインストールしてください:
>
> ```bash
> pip install a2ui-agent-sdk
> ```

### オプション B: レシピクライアント Web アプリの実行

A2UI over MCP を視覚的に確認できる完全な対話型体験を試すには、付属の Web アプリケーションを実行します。

> [!NOTE]
> **パッケージマネージャーの使用:** A2UI リポジトリ内の組み込みサンプルアプリケーションを実行するには、Corepack ワークスペースで設定されている Yarn (`yarn install` / `yarn dev`) が必要です。このリポジトリ外での通常の使用やスタンドアロンプロジェクトでは、お好みのパッケージマネージャー (npm, pnpm など) を使用してください。

1. 新しいターミナルウィンドウで、client ディレクトリに移動します:
    ```bash
    cd client
    ```
2. Node.js の依存関係をインストールします:
    ```bash
    yarn install
    ```
3. Vite 開発サーバーを起動します:
    ```bash
    yarn dev
    ```
4. ターミナルに表示された URL (通常は `http://localhost:5173`) をブラウザで開きます。

レスポンシブな 2 カラムインターフェースが表示されます。左側のカラムは MCP Resource (`a2ui://recipe-form`) から選択フォームをレンダリングします。オプションを選択して **「Get Recipe」** をクリックすると、MCP Tool (`get_recipe_a2ui`) が実行され、返されたカスタム A2UI レシピカードが右側のカラムに動的にレンダリングされます。

![選択フォームと動的なレシピカード生成を示す Dynamic Recipe Studio デモ](../assets/recipe_sample.gif)

すべてのサンプルは [`samples/community/mcp/`](https://github.com/a2ui-project/a2ui/tree/main/samples/community/mcp) で確認できます。

## 仕組み

MCP サーバーがクライアントに A2UI コンテンツを配信するには、主に 2 つの方法があります:

1. **リソースの読み取り経由 (`resources/read`)**: クライアントが MCP リソースを直接読み取ります (例: `a2ui://recipe-form`)。サーバーは A2UI JSON ペイロードを直接返します。
2. **ツールの呼び出し経由 (`tools/call`)**: クライアントが MCP ツールを呼び出します (例: `get_recipe_a2ui`)。サーバーはツールレスポンス内の **Embedded Resource** としてラップされた A2UI JSON ペイロードを返します。

どちらの場合も、クライアントは `application/a2ui+json` MIME タイプを検出し、ペイロードを A2UI レンダラーへルーティングします。

> [!IMPORTANT]
> **MIME タイプの一貫性**
> 配信チャネル (Resource として直接取得するか、Tool の `CallToolResult` 内で返されるか) に関係なく、A2UI JSON ペイロードは常に `application/a2ui+json` MIME タイプで識別されます。Tool レスポンスでは、ペイロードはこの MIME タイプを持つ `EmbeddedResource` 内にラップされている必要があります。この統一された識別により、クライアント側ミドルウェアが静的リソースと動的ツールレスポンスの両方をシームレスにインターセプトして A2UI へルーティングできます。

### 1. リソースベースの配信フロー (`resources/read`)

```
Client → resources/read → MCP Server
                             ↓
                 Retrieve A2UI JSON
                             ↓
Client ← ResourceContents ← MCP Server
          (application/a2ui+json)
   ↓
A2UI Renderer displays UI
```

### 2. ツールベースの配信フロー (`tools/call`)

```
Client → tools/call → MCP Server
                         ↓
              Generate A2UI JSON
                         ↓
         Wrap as EmbeddedResource
              (application/a2ui+json)
                         ↓
Client ← CallToolResult ← MCP Server
   ↓
A2UI Renderer displays UI
```

## Resources と Tools: 用途の分離

MCP 上で A2UI 統合を設計する場合、UI ペイロードが静的か動的かに応じて **Resources** と **Tools** のどちらかを選択します。

### 1. MCP Resources による静的 UI (`resources/read`)

ユーザーのプロンプト入力や会話履歴に依存しない、シンプルで静的なユーザーインターフェースには、A2UI を MCP Resource として直接配信します。

- **コンセプト**: クライアントは標準のリソース URI (例: `a2ui://recipe-form`) を使用して事前定義された A2UI リソースを読み取ります。
- **ユースケース**: 静的な設定フォーム、選択画面、設定ダッシュボード、固定レイアウトに最適です。
- **利点**: 実装が非常にシンプルでオーバーヘッドが少なく、LLM/エージェントが構造を取得するためにツール呼び出しを行う必要がありません。

**Python サーバーの例:**

```python
@app.list_resources()
async def list_resources() -> list[types.Resource]:
    return [
        types.Resource(
            uri="a2ui://recipe-form",
            name="Recipe Form",
            mimeType="application/a2ui+json",
            description="Static form allowing users to pick options.",
        )
    ]

@app.read_resource()
async def read_resource(uri: str) -> list[ReadResourceContents]:
    if uri == "a2ui://recipe-form":
        return [
            ReadResourceContents(
                content=json.dumps(recipe_form_json),
                mime_type="application/a2ui+json",
            )
        ]
    raise ValueError(f"Unknown resource: {uri}")
```

### 2. MCP Tools による動的 UI (`tools/call`)

会話のコンテキスト、ユーザーパラメータ、またはリアルタイムデータに基づいて動的に生成する必要があるユーザーインターフェースには、MCP Tool のレスポンス内で A2UI を配信します。

- **コンセプト**: クライアント/エージェントが特定の引数 (例: 選択した食材、好み) でツールを呼び出し、サーバーは `CallToolResult` 内の `EmbeddedResource` にラップされたカスタマイズされた A2UI JSON を返します。
- **ユースケース**: リアルタイムデータベースクエリ、過去の入力、対話型ステップバイステップウィザードの状態、パーソナライズされたおすすめ (例: カスタマイズされたレシピカード) に依存するコンテンツに最適です。
- **利点**: 柔軟性とコンテキスト認識を最大化し、高度に動的なフローをサポートします。
- **ベストプラクティス (フォールバックテキスト)**: `CallToolResult` 内で `EmbeddedResource` と一緒に必ず `TextContent` を含めてください。A2UI をサポートしていないクライアントは、代わりにこのテキストをユーザーに表示します。

**Python サーバーの例:**

```python
@app.call_tool()
async def handle_call_tool(name: str, arguments: dict[str, Any]) -> types.CallToolResult:
    if name == "get_recipe_a2ui":
        # Resolve dynamic selections from client parameters
        style = arguments.get("cookingStyle", "Baked")
        protein = arguments.get("protein", "Salmon")

        # Retrieve customized recipe database entry
        recipe_data = RECIPES.get((style, protein))

        # Customize base A2UI schema dynamically
        custom_recipe_json = copy.deepcopy(recipe_a2ui_json)
        custom_recipe_json[1]["updateComponents"]["components"][0]["text"] = recipe_data["title"]

        # Return customized recipe card as EmbeddedResource
        return types.CallToolResult(content=[
            types.EmbeddedResource(
                type="resource",
                resource=types.TextResourceContents(
                    uri="a2ui://recipe-card",
                    mimeType="application/a2ui+json",
                    text=json.dumps(custom_recipe_json),
                )
            )
        ])
```

## カタログのネゴシエーション

サーバーがクライアントに A2UI を送信する前に、双方はどのカタログが利用可能かを確立する必要があります。アーキテクチャに応じて、これは 2 つの方法のいずれかで行われます。

### オプション A: MCP 初期化時 (推奨)

MCP は状態を持つセッションプロトコルであるため、最も効率的な方法は接続セットアップ時に機能を 1 回だけ宣言することです。クライアントは `capabilities` 配下で A2UI サポートを宣言します:

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
              "https://a2ui.org/specification/v0_9/catalogs/basic/catalog.json"
            ]
          }
        }
      }
    }
  }
}
```

サーバーはこの状態をセッションの間保持します。

### オプション B: メッセージごとのメタデータ (ステートレスサーバー向け)

サーバーがステートレスである必要がある場合、クライアントはすべてのツール呼び出しの `_meta` フィールドで A2UI 機能を渡すことができます:

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
              "https://a2ui.org/specification/v0_9/catalogs/basic/catalog.json"
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

`Button` などの対話型コンポーネントは、MCP ツール呼び出しとしてサーバーに送り返されるアクションをトリガーできます。

### 1. アクションを含む Button の定義

A2UI JSON で、コンポーネントに `action` を追加します:

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

ユーザーがボタンをクリックすると、クライアントは surface の状態に対して (`/dates/start` などの) データバインディングを解決し、必須のアクションフィールドを含むツール呼び出しを送信します:

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

### 3. サーバーでアクションを処理

```python
@app.tool()
async def a2ui_action(
    name: str,
    surfaceId: str,
    sourceComponentId: str,
    timestamp: str,
    context: dict[str, Any],
) -> types.CallToolResult:
    """Handle A2UI user actions."""
    if name == "confirm_booking":
        # Process the booking, then return confirmation UI
        return types.CallToolResult(content=[
            types.TextContent(
                type="text",
                text=f"Booking confirmed for {surfaceId}: {context['start']} to {context['end']}"
            )
        ])
    raise ValueError(f"Unknown action: {name}")
```

> [!NOTE]
> 5つのアクションフィールドすべて (`name`、`surfaceId`、`sourceComponentId`、`timestamp`、`context`) は A2UI 仕様で必須とされています。ツールパラメーターにすべてのフィールドを宣言しておくことで、MCP SDK が `surfaceId` やその他のフィールドを除外してしまい、元の surface コンテキストが失われるのを防ぐことができます。

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

サーバーで処理します:

```python
@app.tool()
async def a2ui_error(
    code: str,
    surfaceId: str,
    message: str,
    path: str | None = None,
) -> types.CallToolResult:
    """Handle A2UI client errors."""
    # Log the error, retry, or send a fallback UI
    return types.CallToolResult(content=[
        types.TextContent(
            type="text",
            text=f"Acknowledged error {code} on surface {surfaceId}: {message}"
        )
    ])
```

## 発話と可視性の制御

MCP **Resource Annotations** を使用して、後続のターンで LLM が A2UI ペイロードを「読み取る」ことができるかどうかを制御します:

```python
a2ui_resource = types.EmbeddedResource(
    type="resource",
    resource=types.TextResourceContents(
        uri="a2ui://training-plan-page",
        mimeType="application/a2ui+json",
        text=json.dumps(a2ui_payload)
    ),
    # ユーザーには UI を表示し、LLM からは生 JSON を隠す
    annotations=types.Annotations(audience=["user"])
)
```

| Audience        | 動作                                                   |
| --------------- | ------------------------------------------------------ |
| _(空)_          | ユーザーと LLM の両方に表示されます                     |
| `["user"]`      | ユーザー向けにレンダリングされ、LLM コンテキストからは隠されます |
| `["assistant"]` | LLM が後続の推論に利用できますが、レンダリングはされません |

## A2UI Agent SDK の使用

本番環境では、**A2UI Agent SDK** がスキーマ管理、検証、プロンプト生成を自動で処理します:

```bash
pip install a2ui-agent-sdk
```

```python
from a2ui.strategies.schema import A2uiSchemaManager
from a2ui.basic_catalog.provider import BasicCatalog

# Initialize the schema manager with the basic catalog
schema_manager = A2uiSchemaManager(
    catalogs=[BasicCatalog.get_config()],
)

# Validate A2UI output before sending
selected_catalog = schema_manager.get_selected_catalog()
selected_catalog.validator.validate(a2ui_payload)
```

スキーマ管理、動的カタログ、ストリーミングの詳細については、[エージェント開発ガイド](agent-development.md) を参照してください。

## 次のステップ

- [A2UI 仕様](../specification/v0.9-a2ui.md) — 完全なプロトコルリファレンス
- [コンポーネントギャラリー](../reference/components.md) — 利用可能なコンポーネントを探索
- [A2UI Surface 内の MCP Apps](mcp-apps-in-a2ui.md) — HTML ベースの MCP アプリを A2UI 内に埋め込む
- [クライアントのセットアップ](client-setup.md) — A2UI を表示するレンダラーを構築する

