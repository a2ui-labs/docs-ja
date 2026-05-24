# ユーザーアクションの処理

このガイドでは、A2UI がユーザー操作をどのように処理するかを説明します。コンポーネントは `action` プロパティを使って、ローカル **Function**(レンダラーで実行)または **Event**(エージェントへ dispatch)をトリガーします。さらに **Data Model Synchronization** により、エージェントは常に UI の完全な状態へアクセスでき、音声コマンドのようなシームレスなマルチモーダル操作が可能になります。この設計により、安全で制限された環境を維持しながら、高い応答性を持つインターフェースを実現できます。

## Action アーキテクチャ

Action は、UI コンポーネントが `common_types.json` の [`Action`](../../specification/v0_9/json/common_types.json#L271-L313) スキーマで定義された動作をトリガーできるようにします。Action は次のものをトリガーできます。

1.  **Event**: 処理のために Agent へ dispatch されます(Agent 上で実行、例: "Submit" のクリック)。
2.  **Function**: [`FunctionCall`](../../specification/v0_9/json/common_types.json#L200-L242) を使ってレンダラー上で完全に実行されます(Renderer 上で実行、例: URL を開く)。

### 1. Function(ローカル)

Function は、ネットワーク往復なしでレンダラー上の即時動作を実行します。エージェントにはローカル関数呼び出しは通知されません。Function は `functionCall` キーワードを使います。

```json
{
  "id": "help-btn",
  "component": "Button",
  "child": "help-text",
  "action": {
    "functionCall": {
      "call": "openUrl",
      "args": {"url": "https://a2ui.org/help"}
    }
  }
}
```

Function の一般的な用途は次のとおりです。

- **Navigation**: URL を開く、またはタブを切り替える。
- **Validation**: 送信前に入力を確認する(下の Checks を参照)。

### 2. Event(エージェント)

Event は処理のためにデータをエージェントへ送ります。Event は `event` キーワードを使います。

`Button` のようなコンポーネントは `action` プロパティを公開します。Event を接続する方法は次のとおりです。

```json
{
  "id": "submit-btn",
  "component": "Button",
  "child": "btn-text",
  "action": {
    "event": {
      "name": "submit_reservation",
      "context": {
        "time": {"path": "/reservationTime"},
        "size": {"path": "/partySize"}
      }
    }
  }
}
```

- **`name`**: エージェントが分岐処理に使える安定した識別子です。
- **`context`**: key-value ペアのマップです。値は literal にすることも、データモデルの現在状態から取り出すために `path` を使うこともできます。

NOTE: **Context と Data Model**: Data Model は surface 全体の状態ツリーを表しますが、action の `context` は実質的に、その状態から選び出した **"view"** または部分集合です。これにより Agent は、大きく複雑になり得るデータモデルを探索せずに、特定の event に必要な値だけを受け取れるため、作業が単純になります。

### Basic Catalog の関数検証(Checks)

basic catalog は、レンダラー上で実行できる限定的な check の集合を定義しています。インタラクティブコンポーネントは、`common_types.json` の [`Checkable`](../../specification/v0_9/json/common_types.json#L258-L270) スキーマを使って `checks` のリストを定義できます。`Button` の場合、いずれかの check が失敗すると、そのボタンはレンダラー上で**自動的に無効化**されます。

- **UX 重視**: 検証 check は、不正な操作が発生する前に防ぐことで **UI 状態(ユーザー体験)** を管理するよう設計されています。ただし、エージェント側で実行すべき **Data Integrity** check の代替ではありません。

これにより、ユーザーが送信を試みる前に、UI が必須フィールドなどの要件を強制できます。

```json
{
  "id": "submit-button",
  "component": "Button",
  "child": "submit-text",
  "checks": [
    {
      "condition": {
        "call": "required",
        "args": {"value": {"path": "/partySize"}}
      },
      "message": "Party size is required"
    }
  ],
  "action": {"event": {"name": "submit_booking"}}
}
```

## ローカル状態更新と "Write" 契約

Event が dispatch される前から、レンダラーは UI の状態をローカルで管理しています。A2UI は `TextField`、`CheckBox`、`Slider` のようなすべての入力コンポーネントに対して **Read/Write Contract** を定義します。

1.  **Read (Model → View)**: コンポーネントがレンダリングされるとき、Data Model のバインドされた `path` から値を取得します。
2.  **Write (View → Model)**: ユーザーが操作した瞬間(例: 文字入力やチェックボックスのクリック)、レンダラーは新しい値をローカル Data Model に**即座に**書き込みます。

つまりローカルモデルは、UI の現在状態に対する**常に** source of truth です。この "View-to-Model" 同期はレンダラー上だけで行われます。データモデルは、ボタンクリックのような event が発生したときだけエージェントへ送信されます。

IMPORTANT: **同期的な更新**: ローカルモデル更新は**同期的**です。これにより、Event が `context` path を解決したり `DataModelSync` payload をパッケージしたりする前に、Data Model が完全に更新済みであることが保証されます。入力とクリックの間に race condition はありません。"Write" は常に先に commit されます。

この local-first アプローチには大きな**性能上の利点**があります。同期が即時かつローカルで行われるため、開発者はユーザーが `TextField` に入力している間のネットワーク debounce を実装したり、レイテンシの揺らぎを心配したりする必要がありません。ユーザーが正式な Event を dispatch する準備ができるまで、ネットワークは個々のキー入力のような "UI noise" から完全に保護されます。

### フォーム送信パターン

この分離により、堅牢なフォーム送信パターンが可能になります。

- **Binding**: `TextField` が `/reservationTime` にバインドされます。
- **Interaction**: ユーザーが "7:00 PM" と入力します。`/reservationTime` のローカルモデルが即座に更新されます。
- **Submission**: ユーザーが "Book" ボタンをクリックします。ボタンの Event はローカルモデルから `path: "/reservationTime"` を解決し、現在値をエージェントへ送ります。

## ユーザー操作フロー

ユーザーがコンポーネントと操作するとき(例: ボタンをクリック)、次が起こります。

1.  **Resolve**: レンダラーは `context` 内のすべての `path` 参照をローカル **Data Model** に対して解決します。
2.  **Construct**: レンダラーは [`client_to_server.json`](../../specification/v0_9/json/client_to_server.json) に準拠した `action` payload を構築します。
3.  **Dispatch**: 選択されたトランスポート(例: A2A、WebSockets)を介して payload が送信されます。

### 例: Action Payload(v0.9)

データモデルに `{"reservationTime": "7:00 PM", "partySize": 4}` がある状態でユーザーが上記のボタンをクリックすると、レンダラーは `action` キーを使って次のメッセージを送信します。

```json
{
  "version": "v0.9",
  "action": {
    "name": "submit_reservation",
    "surfaceId": "booking-surface",
    "sourceComponentId": "submit-btn",
    "timestamp": "2026-02-25T10:40:00Z",
    "context": {
      "time": "7:00 PM",
      "size": 4
    }
  }
}
```

IMPORTANT: **バージョン管理に関する注記(v0.8 vs v0.9)**: v0.8 では、トップレベル payload キーは `userAction` でした(例: `{"userAction": {...}}`)。v0.9 では、上記のよりシンプルな `action` キーへ移行しました。標準プロトコル parser は、payload で宣言されたバージョンに対応するキーを期待します。

## エージェント処理

Agent(または Orchestrator)はこの event を受け取り、処理します。エージェントシステムでは、エージェントは通常 event を LLM 向けの隠れたユーザー query に変換します。

**エージェント処理の例(Python):**

```python
if action_name == "submit_reservation":
    time = context.get("time")
    size = context.get("size")
    # Feed this to the LLM
    query = f"User submitted a reservation for {size} people at {time}."
    response = await llm.generate(query)
```

## レンダラーからエージェントへのエラー報告

ユーザーがトリガーした Event に加えて、レンダラーは [`client_to_server.json`](../../specification/v0_9/json/client_to_server.json) に定義された `error` payload を使って、システムレベルのエラーをエージェントへ報告できます。

### 検証失敗

エージェントが送信した A2UI JSON がカタログスキーマやプロトコルルールに違反している場合、レンダラーは `VALIDATION_FAILED` エラーを送信します。これはエージェントシステムにとって重要な feedback loop です。

```json
{
  "version": "v0.9",
  "error": {
    "code": "VALIDATION_FAILED",
    "surfaceId": "booking-surface",
    "path": "/components/0/children",
    "message": "Expected array of strings, got null."
  }
}
```

エージェントはこのエラーを受け取り、謝罪するか(または内部で self-correct して)、修正済み UI を再送できます。

## Data Model Sync(v0.9)

A2UI v0.9 では、強力な "stateless" 同期機能が導入されました。これにより、レンダラーはエージェントへ送るすべてのメッセージの metadata に、surface の**完全なデータモデル**を自動的に含められます。

### Sync の有効化

同期は surface 初期化時にエージェントから要求されます。`createSurface` メッセージで `sendDataModel: true` を設定すると、エージェントはレンダラーに sync loop の開始を指示します。

```json
{
  "version": "v0.9",
  "createSurface": {
    "surfaceId": "booking-surface",
    "catalogId": "https://a2ui.org/catalogs/v1/basic.json",
    "sendDataModel": true
  }
}
```

### Wire 上の Sync

sync が有効な場合、レンダラーはデータモデルを別メッセージとして送信しません。代わりに、送信する transport envelope(例: A2A メッセージ)へ **metadata** として添付します。

A2A(Agent-to-Agent) binding では、データモデルは envelope の `metadata` フィールド内の `a2uiClientDataModel` オブジェクトに配置されます。

**Sync を含む A2A Envelope の例:**

```json
{
  "parts": [{"text": "Submit the reservation"}],
  "metadata": {
    "a2uiClientDataModel": {
      "version": "v0.9",
      "surfaces": {
        "booking-surface": {
          "reservationTime": "7:00 PM",
          "partySize": 4,
          "notes": "Window seat preferred"
        }
      }
    }
  }
}
```

### Data Model Sync を使う理由

- **より単純な配線**: すべての入力フィールドをボタンの `context` プロパティへ手動でマッピングする必要がありません。エージェントは metadata を見るだけで、すべてのフィールドの現在状態を確認できます。
- **Stateless Agents**: エージェントはユーザーセッションごとのローカル状態を維持する必要がありません。すべての操作ごとに完全な現在 context を受け取れます。
- **音声ショートカット**: ユーザーは特定のボタンをクリックしなくても、音声やテキスト(例: "okay submit")で Event をトリガーできます。エージェントはテキストメッセージとともに更新済みデータモデルを受け取るため、リクエストをすぐに処理できます。

## レンダラー Metadata と Capabilities

エージェントが安全に UI を送るには、まずレンダラーがどのコンポーネントカタログをサポートしているかを通知する必要があります。これは `a2uiClientCapabilities` オブジェクトで処理されます。

### Capabilities の通知

レンダラーは、エージェントへ送るメッセージの **metadata** に `a2uiClientCapabilities` オブジェクトを含めます(例: A2A envelope の `metadata` フィールド)。

```json
{
  "v0.9": {
    "supportedCatalogIds": [
      "https://a2ui.org/specification/v0_9/catalogs/basic/catalog.json",
      "https://my-company.com/catalogs/v1/custom.json"
    ],
    "inlineCatalogs": []
  }
}
```

- **`supportedCatalogIds`**: レンダラーがレンダリングできる catalog URI の配列です。
- **`inlineCatalogs`**: (任意) 開発環境や特殊な環境で、完全な catalog schema を inline で送信できるようにします。

この handshake がないと、エージェントは送信する特定のコンポーネントをレンダラーが扱えるか確信できません。

## Transport と Encoding

A2UI は transport-agnostic ですが、もっとも一般的には **A2A(Agent-to-Agent)** または WebSockets 上で使われます。実装では payload がどのようにラップされるかを理解することが重要です。

### A2A Encoding

標準 A2A binding では、A2UI メッセージは A2A **DataPart** としてエンコードされます。A2UI payload として識別するために、その part は特定の metadata でラップされる必要があります。

- **mimeType**: `application/json+a2ui`

`DataPart` の `data` フィールドには、A2UI メッセージの **list** が含まれます。これにより、複数の更新(例: `createSurface` に続く `updateComponents`)を 1 つのネットワークパケットで送信できます。

NOTE: **A2A バージョン管理**: `data` フィールドで **list** を使う方式は **A2A v1.0** で導入されました。以前の A2A プロトコルでは、`data` フィールドに単一の JSON オブジェクトが含まれることを期待します。

```json
{
  "kind": "data",
  "metadata": {
    "mimeType": "application/json+a2ui"
  },
  "data": [
    {
      "version": "v0.9",
      "action": { ... }
    }
  ]
}
```

## セキュリティ上の考慮事項

A2UI は、安全で sandboxed な通信を中心原則として設計されています。プロトコルはユーザー状態と操作トリガーをネットワーク上で渡すため、データの可視性と実行に厳格な境界を適用します。

### Sandboxed 実行

A2UI の中核的な価値は、制限によるセキュリティです。エージェントからの任意コード実行(raw JavaScript の注入など)を禁止することで、A2UI はエージェントが事前登録済みの動作だけをトリガーできるようにします。`functionCall` メカニズムは、ユーザーを悪意あるスクリプトにさらさずに、エージェントがレンダラー環境とやり取りできる安全で sandboxed な方法として機能します。

### Data Model の分離と Orchestrator Routing

`sendDataModel: true` が有効な場合、レンダラーは surface 全体のデータモデルを送信メッセージに含めます。開発者はこのデータの可視性を理解する必要があります。

- **Point-to-Point Visibility**: この payload を読めるのは、transport envelope を受け取るバックエンド、つまり surface を作成した Agent または中間 Orchestrator だけです。
- **Orchestrator の責任**: multi-agent アーキテクチャでは、中央 Orchestrator がユーザーの意図を専門 sub-agent にルーティングすることがよくあります。Orchestrator は**データ分離**を強制しなければなりません。`a2uiClientDataModel` をパースし、`surfaceId` を特定し、その surface を所有する特定の sub-agent にだけデータモデルを渡す責任があります。あるエージェントの surface のデータが別のエージェントへ漏れてはなりません。

## Orchestration と Routing

multi-agent システムでは、中央の **Orchestrator** がユーザーと複数の専門 sub-agent の間の相互作用を管理することがよくあります。重要な課題は、レンダラーからの `action` メッセージが、その UI surface を生成した特定の sub-agent に戻るようルーティングすることです。

### Surface Ownership パターン

これを処理するには、orchestrator が `surfaceId` と所有 sub-agent のマッピングを維持する必要があります。通常、このマッピングは **Session State** に保存されます。

#### 1. Ownership のマッピング

sub-agent が `createSurface` メッセージを出すと、orchestrator がそれをインターセプトして ownership を記録します。

```python
# Simplified Orchestrator Logic: Record Ownership
def on_surface_created(surface_id, agent_name, session):
    # Store the mapping in the orchestrator's session state
    session.state.update({f"owner_of_{surface_id}": agent_name})
```

#### 2. Event のルーティング

レンダラーが orchestrator へ `action` を送ると、orchestrator は `surfaceId` を調べ、リクエストを正しい sub-agent へ転送します。

```python
# Simplified Orchestrator Logic: Route Event
async def handle_incoming_action(payload, session):
    action = payload.get("action")
    surface_id = action.get("surfaceId")

    # Lookup the owning agent
    target_agent = session.state.get(f"owner_of_{surface_id}")

    if target_agent:
        # Programmatically route the request to the sub-agent
        return transfer_to(target_agent)
```

このパターンにより、複雑な multi-agent 環境でも、双方向通信ループが各機能領域ごとに完全で stateful なまま維持されます。

### Metadata Stripping によるデータ漏えい防止

multi-agent 環境では、`a2uiClientDataModel` に、異なるエージェントが所有する複数 surface の状態が含まれることがあります。機密データ漏えいを防ぐには、orchestrator が metadata から不要な部分を **strip** し、呼び出し対象 sub-agent が所有する surface だけを含める必要があります。

これは outbound metadata interceptor に実装するのが最適です。

```python
# Simplified Orchestrator Interceptor: Strip Data Model
async def intercept(self, request_payload, target_agent, session):
    message = request_payload["params"]["message"]
    data_model = message.get("metadata", {}).get("a2uiClientDataModel")

    if data_model:
        # Filter surfaces to only those owned by the target_agent
        filtered_surfaces = {
            surface_id: state for surface_id, state in data_model["surfaces"].items()
            if session.state.get(f"owner_of_{surface_id}") == target_agent.name
        }

        # Replace with the stripped data model
        message["metadata"]["a2uiClientDataModel"]["surfaces"] = filtered_surfaces

    return request_payload
```

metadata を strip することで、orchestrator は sub-agent が閲覧を許可されたデータモデル部分だけを受け取るよう保証します。

CAUTION: **セキュリティリスク: 状態スクレイピング**: Orchestrator が `a2uiClientDataModel` を strip しない場合、悪意ある、または侵害された sub-agent が他の active surface の状態を「scrape」できる可能性があります。たとえば orchestrator が multi-surface データモデル全体を漏らすと、天気 sub-agent が銀行 surface の機密データを読めてしまう可能性があります。multi-agent システムでは stripping は必須のセキュリティ要件です。

---

## 総合例

### 1. ボタン送信(明示的 Context)

この例は、送信に必要なデータを明示的に集めるボタンを示します。

**コンポーネント定義:**

```json
{
  "id": "submit-button",
  "component": "Button",
  "child": "submit-text",
  "action": {
    "event": {
      "name": "submit_booking",
      "context": {
        "partySize": {"path": "/partySize"},
        "reservationTime": {"path": "/reservationTime"}
      }
    }
  }
}
```

**結果の Action Payload:**
エージェントは、`context` フィールドに `partySize` と `reservationTime` が直接入った `action` オブジェクトを受け取ります。

### 2. 音声で送信(Data Model Sync)

このシナリオでは、ユーザーはボタンをクリックしません。代わりに "Okay, submit the form." と言います。

**初期化:**
エージェントは `sendDataModel: true` で surface を作成しました。

```json
{
  "version": "v0.9",
  "createSurface": {
    "surfaceId": "booking-surface",
    "catalogId": "...",
    "sendDataModel": true
  }
}
```

**レンダラー送信:**
レンダラーは、ユーザーのテキストとデータモデルを metadata に含む A2A メッセージを送信します。

```json
{
  "parts": [{"text": "Okay, submit the form"}],
  "metadata": {
    "a2uiClientDataModel": {
      "version": "v0.9",
      "surfaces": {
        "booking-surface": {
          "partySize": 4,
          "reservationTime": "7:00 PM"
        }
      }
    }
  }
}
```

**エージェント処理:**
エージェントはユーザーの意図("submit")を見て、`metadata` から `partySize` と `reservationTime` の現在値を取得し、追加確認なしでタスクを完了できます。
