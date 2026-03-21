# Model Context Protocol (MCP) 上での A2UI

このガイドでは、**Model Context Protocol (MCP)** の Tools と Resources を使って、A2UI の宣言的構文でリッチな対話 UI を構築する方法を説明します。

サンプルは [MCP Samples](../../samples/agent/mcp) を参照してください。

## カタログのネゴシエーション

サーバーがクライアントへ A2UI を送るには、まずプロトコルの相互サポートを確認し、どのカタログが利用できるかを決める必要があります。システム構成によって、この能力のネゴシエーションは 2 通りの方法で行えます。接続ハンドシェイク時に 1 回だけ行うか、メッセージごとに行うかです。

### オプション A: MCP 初期化時のカタログハンドシェイク

MCP は状態を持つセッションプロトコルなので、最も効率がよいのは接続確立時に機能を 1 回だけ宣言する方法です。クライアントは、標準 `initialize` リクエストの capabilities オブジェクト（多くは experimental か custom のキー）に A2UI サポートを宣言します。サーバーはその状態をセッション中保持します。

**Initialize リクエストの例:**

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
          "v0.10": {
            "supportedCatalogIds": [
              "https://a2ui.org/specification/v0_10/basic_catalog.json"
            ]
          }
        }
      }
    }
  }
}
```

### オプション B: 各 MCP メッセージでカタログを伝える（ステートレスサーバー向け）

MCP サーバーを完全にステートレスにしたい場合、クライアントは各 tool call リクエストの `_meta` フィールドで A2UI バージョンとカタログサポートを送れます。サーバーはそのメタデータを見て、応答 UI に使うカタログを即座に判断します。

**Call リクエストのメタデータ例:**

```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "id": "id-123",
  "params": {
    "name": "generate_report",
    "arguments": { "date": "2026-03-01" },
    "_meta": {
      "a2ui": {
        "clientCapabilities": {
          "v0.10": {
            "supportedCatalogIds": [
              "https://a2ui.org/specification/v0_10/basic_catalog.json"
            ],
            "inlineCatalogs": []
          }
        }
      }
    }
  }
}
```

## 埋め込みリソースとして A2UI コンテンツを返す

Embedded Resource を使うと、Tool はサーバー側に保存や追跡を必要とせず、その応答に直接紐づく UI レイアウトを返せます。

- **URI**: `a2ui://` プレフィックスと、分かりやすい識別子を使う必要があります（例: `a2ui://training-plan-page`）
- **MIME Type**: `application/json+a2ui` を使います。これにより MCP クライアントは生 JSON ではなく A2UI レンダラーへペイロードを渡します

#### Python 実装例

```python
import mcp.types as types

@self.tool()
def get_hello_world_ui():
    a2ui_payload = [
        {
            "version": "v0.10",
            "createSurface": {
                "surfaceId": "default",
                "catalogId": "https://a2ui.org/specification/v0_10/basic_catalog.json"
            }
        },
        {
            "version": "v0.10",
            "updateComponents": {
                "surfaceId": "default",
                "components": [
                    {
                        "id": "root",
                        "component": "Text",
                        "text": "Hello World!"
                    }
                ]
            }
        }
    ]

    # A2UI を Embedded Resource として包む
    a2ui_resource = types.EmbeddedResource(
        type="resource",
        resource=types.TextResourceContents(
            uri="a2ui://training-plan-page",
            mimeType="application/json+a2ui",
            text=json.dumps(a2ui_payload),
        )
    )

    text_content = types.TextContent(
        type="text",
        text="Here is your generated training plan summary..."
    )

    return types.CallToolResult(content=[text_content, a2ui_resource])
```

## ユーザーアクションの扱い

`Button` のような対話的コンポーネントは、`actions` をサーバーへ送れます。

#### 1. アクション付きの A2UI JSON

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

#### 2. A2UI アクションの MCP ペイロード

ボタンが押されると、クライアントは `/dates/start` や `/dates/end` のような絶対・相対パスをサーフェスのバインディング状態に対して解決し、その内容を MCP tool call の引数へ変換します。

```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "id": "id-456",
  "params": {
    "name": "action",
    "arguments": {
      "name": "confirm_booking",
      "context": {
        "start": "2026-03-20",
        "end": "2026-03-25"
      }
    }
  }
}
```

#### 3. MCP サーバー側のアクションハンドラー

MCP サーバーは tool call を受け取り、対応するハンドラーを実行します。

```python
@self.tool()
async def action(action_payload: Dict[str, Any]) -> Dict[str, Any]:
    if action_payload["name"] == "confirm_booking":
        return {"response": f"Booking confirmed for {action_payload['context']['start']} to {action_payload['context']['end']}."}
    raise ValueError(f"Unknown action: {action_payload['name']}")
```

## エラー処理

ユーザー操作と同様に、MCP サーバーはクライアントからのエラーも受け取れます。

#### 1. A2UI エラーの MCP ペイロード

クライアントが A2UI ペイロードの処理に失敗した場合、エラー MCP ペイロードをサーバーへ送れます。

```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "id": "id-789",
  "params": {
    "name": "error",
    "arguments": {
      "code": "INVALID_JSON",
      "message": "Failed to parse A2UI payload.",
      "surfaceId": "default",
    }
  }
}
```

#### 2. エラーハンドラーの MCP サーバーツール

MCP サーバーは tool call を受け取り、対応するハンドラーを実行します。

```python
@self.tool()
async def error(error_payload: Dict[str, Any]) -> Dict[str, Any]:
    return {"response": f"Received A2UI error: {error_payload['error']}."}
```

## 音声化と可視性の制御

MCP の **Resource Annotations** を使うと、その後の assistant がバックエンドのペイロードを「読む」かどうかを制御できます。

```python
a2ui_resource = types.EmbeddedResource(
    type="resource",
    resource=types.TextResourceContents(
        uri="a2ui://training-plan-page",
        mimeType="application/json+a2ui",
        text=json.dumps(a2ui_payload)
    ),
    # 生 JSON は LLM から隠し、UI だけをユーザーへ表示する
    annotations=types.Annotations(audience=["user"])
)
```

- **空の audience**: 要素はユーザーと LLM の両方に見える
- **audience `user`**: ビュー画面へアイテムを表示するために必要
