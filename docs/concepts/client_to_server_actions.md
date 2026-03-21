# A2UI のクライアントからサーバーへのアクション

A2UI の対話性は、双方向の通信ループに依存します。エージェントがコンポーネント更新とデータ更新をストリーミングして UI を駆動する一方で、クライアントは **アクション** と **データモデル同期** を通じてユーザーの意図をエージェントへ返します。

## アクションのアーキテクチャ

アクションは UI コンポーネントが動作を起こすための仕組みです。`Action` スキーマは [`common_types.json`](../specification/v0_9/json/common_types.json) に定義されており、主に 2 種類あります。

1. **サーバーイベント**: 処理のためにエージェントへ送られる
2. **ローカル関数呼び出し**: クライアント側だけで実行される

### スキーマ上でのアクション配線

`Button` のようなコンポーネントは `action` プロパティを持ちます。サーバーイベントの例です。

```json
{
  "id": "submit-btn",
  "component": "Button",
  "child": "btn-text",
  "action": {
    "event": {
      "name": "submit_reservation",
      "context": {
        "time": { "path": "/reservationTime" },
        "size": { "path": "/partySize" }
      }
    }
  }
}
```

- **`name`**: エージェント側で分岐するための安定した識別子
- **`context`**: キーと値のマップ。値はリテラルでもよく、`path` を使ってデータモデルの現在値を参照してもよい

> [!NOTE]
> **Context とデータモデルの違い**: データモデルはサーフェス全体の状態ツリーを表しますが、アクション内の `context` はその状態から手で選んだ **「ビュー」** またはサブセットです。これにより、エージェントは巨大なデータモデル全体をたどらなくても、特定イベントに必要な値だけを受け取れます。

### ローカルアクションとサーバーイベント

サーバーイベントがエージェントとの主な対話手段ですが、**ローカルアクション** を使うと、ネットワーク往復なしで即時にクライアント側の動作を行えます。反応速度が重要な UI パターンでは特に有効です。

```json
{
  "id": "help-btn",
  "component": "Button",
  "child": "help-text",
  "action": {
    "functionCall": {
      "call": "openUrl",
      "args": { "url": "https://a2ui.org/help" }
    }
  }
}
```

ローカルアクションの典型例:

- **検証**: サーバーへ送る前にフォーム入力を検証する
- **整形**: `formatString` を使ってローカル表示値を整える

### Basic Catalog のアクション検証（Checks）

Basic Catalog には、クライアント上で実行できる限定的なチェックが定義されています。対話的コンポーネントは `checks` の一覧を定義できます（[`common_types.json`](../specification/v0_9/json/common_types.json) の `Checkable` スキーマを使用）。`Button` では、いずれかのチェックが失敗すると、クライアント側で自動的に無効化されます。

- **UX 重視**: アクションチェックは、無効な操作を起こる前に防ぐことで **UI 状態** を管理するためのものです。サーバー側で必ず行うべき **データ整合性** チェックの代わりにはなりません。

これにより、ユーザーが送信しようとする前に、空欄禁止のような要件を UI 側で強制できます。

```json
{
  "id": "submit-button",
  "component": "Button",
  "child": "submit-text",
  "checks": [
    {
      "condition": {
        "call": "required",
        "args": { "value": { "path": "/partySize" } }
      },
      "message": "Party size is required"
    }
  ],
  "action": { "event": { "name": "submit_booking" } }
}
```

## ローカル状態の更新と「Write」契約

アクションが送出される前から、クライアントは UI の状態をローカルで管理しています。A2UI は `TextField`、`CheckBox`、`Slider` などの入力コンポーネントに対して **Read/Write 契約** を定義します。

1. **Read (Model → View)**: コンポーネントが描画されるとき、バインドされた `path` から値を読み取る
2. **Write (View → Model)**: ユーザーが操作した瞬間に、クライアントが新しい値をローカルのデータモデルへ書き込む

つまり、ローカルモデルが常に UI の現在状態の真実になります。View-to-Model の同期はクライアント内だけで完結します。ユーザーアクション（ボタン押下など）が起きたときだけ、その状態がサーバーへ同期されます。

> [!IMPORTANT]
> **同期更新**: ローカルモデルの更新は **同期的** です。これにより、アクションが `context` の `path` を解決したり `DataModelSync` ペイロードを組み立てたりする前に、データモデルが完全に更新された状態であることが保証されます。入力とクリックの間に競合状態はありません。`Write` は常に先に確定します。

このローカル優先の設計には大きな **性能上の利点** があります。同期が即時かつローカルで行われるため、開発者はネットワークのデバウンスや、ユーザーが `TextField` に入力する際の遅延揺れを気にしなくて済みます。ユーザーが正式なアクションを送る準備ができるまで、ネットワークは個々のキー入力のような「UI ノイズ」から完全に守られます。

### フォーム送信パターン

この分離により、堅牢なフォーム送信パターンを実現できます。

- **Binding**: `TextField` を `/reservationTime` にバインドする
- **Interaction**: ユーザーが `7:00 PM` と入力すると、`/reservationTime` のローカルモデルが即座に更新される
- **Submission**: ユーザーが "Book" ボタンを押すと、ボタンのアクションがローカルモデルから `path: "/reservationTime"` を解決し、現在値をサーバーへ送る

## ユーザー操作フロー

ユーザーがコンポーネントを操作したとき（たとえばボタンを押したとき）:

1. **Resolve**: クライアントが `context` 内のすべての `path` 参照をローカルの **Data Model** に対して解決する
2. **Construct**: クライアントが [`client_to_server.json`](../specification/v0_9/json/client_to_server.json) に準拠した `action` ペイロードを組み立てる
3. **Dispatch**: ペイロードを選択したトランスポート（A2A、WebSocket など）で送る

### 例: アクションペイロード（v0.9）

ユーザーが上のボタンを押し、データモデルに `{"reservationTime": "7:00 PM", "partySize": 4}` がある場合、クライアントは `action` キーを使って次のメッセージを送ります。

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

> [!IMPORTANT]
> **バージョニングの注意（v0.8 と v0.9）**: v0.8 では、トップレベルのペイロードキーは `userAction` でした（例: `{"userAction": {...}}`）。v0.9 では、よりシンプルな `action` キーに変わっています。標準のプロトコルパーサーは、ペイロード内で宣言されたバージョンに対応するキーを期待します。

## エージェント側の処理

エージェント（またはオーケストレーター）はこのイベントを受け取り、処理します。エージェント的なシステムでは、通常このイベントを LLM 向けの隠れたユーザー問い合わせに変換します。

**エージェント処理の例（Python）:**

```python
if action_name == "submit_reservation":
    time = context.get("time")
    size = context.get("size")
    # これを LLM に渡す
    query = f"User submitted a reservation for {size} people at {time}."
    response = await llm.generate(query)
```

## クライアントからサーバーへのエラー報告

ユーザーアクションに加えて、クライアントは [`client_to_server.json`](../specification/v0_9/json/client_to_server.json) で定義された `error` ペイロードを使って、システムレベルのエラーをサーバーへ報告できます。

### バリデーション失敗

エージェントがカタログスキーマやプロトコルルールに違反する A2UI JSON を送った場合、クライアントは `VALIDATION_FAILED` エラーを送ります。これはエージェント的システムにとって重要なフィードバックループです。

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

エージェントはこのエラーを受け取り、謝罪するか内部で自己修正して、修正版の UI を再送できます。

## データモデル同期（v0.9）

A2UI v0.9 では、強力な「ステートレス」同期機能を導入しました。クライアントがサーバーへ送るすべてのメッセージのメタデータに、サーフェスの **完全なデータモデル** を自動的に含められます。

### 同期を有効にする

同期はサーフェス初期化時にエージェントから要求されます。`createSurface` メッセージで `sendDataModel: true` を設定すると、クライアントは同期ループを開始します。

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

### ワイヤ上での同期

同期が有効な場合、クライアントはデータモデルを別メッセージとして送信しません。代わりに、送信するトランスポートのエンベロープ（たとえば A2A メッセージ）の **メタデータ** に付与します。

A2A（Agent-to-Agent）のバインディングでは、データモデルはエンベロープの `metadata` フィールド内の `a2uiClientDataModel` オブジェクトに格納されます。

**同期付き A2A エンベロープの例:**

```json
{
  "parts": [{ "text": "Submit the reservation" }],
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

### なぜデータモデル同期を使うのか？

- **配線がシンプル**: すべての入力項目をボタンの `context` に手動でマッピングする必要がなくなる
- **ステートレスなエージェント**: ユーザーセッションごとにローカル状態を保持しなくてよい。毎回、完全な現在コンテキストを受け取れる
- **音声やテキストのショートカット**: ユーザーが特定のボタンを押さなくても、「submit して」のような操作をトリガーできる。更新済みのデータモデルをテキストメッセージと一緒に受け取るため、すぐに処理できる

## クライアントのメタデータと機能

エージェントが安全に UI を送るには、クライアントがどのコンポーネントカタログをサポートしているかを広告する必要があります。これは `a2uiClientCapabilities` オブジェクトで行います。

### 機能の広告

クライアントは、サーバーへ送るメッセージの **メタデータ** に `a2uiClientCapabilities` オブジェクトを含めます（たとえば A2A エンベロープの `metadata` フィールド）。

```json
{
  "v0.9": {
    "supportedCatalogIds": [
      "https://a2ui.org/specification/v0_10/basic_catalog.json",
      "https://my-company.com/catalogs/v1/custom.json"
    ],
    "inlineCatalogs": []
  }
}
```

- **`supportedCatalogIds`**: クライアントが描画できるカタログ URI の配列
- **`inlineCatalogs`**: （任意）開発環境や特殊な環境向けに、完全なカタログスキーマを inline で送るための仕組み

このハンドシェイクがなければ、エージェントはレンダラーが特定のコンポーネントを扱えるか確信できません。

## トランスポートとエンコーディング

A2UI 自体は transport-agnostic ですが、実際には **A2A (Agent-to-Agent)** や WebSockets 経由で使われることが多いです。ペイロードがどのように包まれるかを理解することは実装上重要です。

### A2A エンコーディング

標準的な A2A バインディングでは、A2UI メッセージは A2A の **DataPart** としてエンコードされます。A2UI ペイロードだと識別できるように、特定のメタデータで包む必要があります。

- **mimeType**: `application/json+a2ui`

`DataPart` の `data` フィールドには、A2UI メッセージの **リスト** を入れます。これにより、`createSurface` の後に `updateComponents` を続けるような複数更新を 1 つのネットワークパケットで送れます。

> [!NOTE]
> **A2A のバージョニング**: `data` フィールドに **リスト** を使う仕様は **A2A v1.0** で導入されました。以前の A2A プロトコルでは、`data` フィールドに単一の JSON オブジェクトを入れることが期待されます。

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

## セキュリティ上の考慮

A2UI は、安全でサンドボックス化された通信を中核原則として設計されています。ユーザー状態や対話トリガーをネットワーク越しに渡すプロトコルである以上、可視性と実行の境界は厳密に保つ必要があります。

### サンドボックス化された実行

A2UI の大きな価値は、制限による安全性です。エージェントが生の JavaScript を注入するような任意コード実行を禁止することで、A2UI はエージェントが事前登録されたクライアント側の動作だけを起こせるようにします。`functionCall` メカニズムは、悪意あるスクリプトを露出させずにクライアント環境へ安全に作用する手段です。

### データモデルの隔離とオーケストレーターのルーティング

`sendDataModel: true` を有効にすると、クライアントはサーフェスの完全なデータモデルを outgoing message に含めます。このデータの可視性は次のように理解する必要があります。

- **ポイントツーポイントの可視性**: そのトランスポートエンベロープを受け取るバックエンド（サーフェスを作ったエージェント、または中間のオーケストレーター）だけがこのペイロードを読める
- **オーケストレーターの責務**: マルチエージェント構成では、中央のオーケストレーターがユーザー意図を専門サブエージェントへルーティングすることが多い。オーケストレーターは **データ隔離** を強制しなければならない。`a2uiClientDataModel` を解析し、`surfaceId` を特定し、そのデータモデルがその surface を所有するサブエージェントにだけ渡るようにする必要があります。あるエージェントの surface のデータが別のエージェントへ漏れてはいけません。

## オーケストレーションとルーティング

マルチエージェントシステムでは、中央の **オーケストレーター** が、ユーザーと複数の専門サブエージェントのやり取りを管理することがよくあります。重要な課題は、クライアントから送られる `action` メッセージを、UI surface を生成したサブエージェントへ正しく返すことです。

### Surface Ownership パターン

これを扱うために、オーケストレーターは `surfaceId` と所有サブエージェントの対応表を保持する必要があります。通常は **Session State** に保存します。

#### 1. 所有権を記録する

サブエージェントが `createSurface` メッセージを出したら、オーケストレーターがそれを横取りして所有権を記録します。

```python
# 簡略化したオーケストレーターのロジック: 所有権を記録する
def on_surface_created(surface_id, agent_name, session):
    # オーケストレーターのセッション状態に対応表を保存
    session.state.update({f"owner_of_{surface_id}": agent_name})
```

#### 2. ユーザーアクションをルーティングする

クライアントが `action` をオーケストレーターへ返すと、オーケストレーターは `surfaceId` を参照して、正しいサブエージェントへリクエストを転送します。

```python
# 簡略化したオーケストレーターのロジック: アクションをルーティングする
async def handle_incoming_action(payload, session):
    action = payload.get("action")
    surface_id = action.get("surfaceId")

    # 所有エージェントを検索
    target_agent = session.state.get(f"owner_of_{surface_id}")

    if target_agent:
        # リクエストをプログラム的にサブエージェントへ転送
        return transfer_to(target_agent)
```

このパターンにより、複雑なマルチエージェント環境でも、双方向通信ループは surface ごとに stateful なまま維持されます。

### メタデータの剥離によるデータ漏えい防止

マルチエージェント環境では、`a2uiClientDataModel` に複数の surface の状態が含まれることがあります。機微データの漏えいを防ぐには、オーケストレーターがメタデータから **不要な surface を削除** し、呼び出し対象のサブエージェントが所有する surface だけを残す必要があります。

これは outgoing metadata interceptor で実装するのが最適です。

```python
# 簡略化したオーケストレーターのインターセプター: データモデルを剥離する
async def intercept(self, request_payload, target_agent, session):
    message = request_payload["params"]["message"]
    data_model = message.get("metadata", {}).get("a2uiClientDataModel")

    if data_model:
        # target_agent が所有する surface だけに絞り込む
        filtered_surfaces = {
            surface_id: state for surface_id, state in data_model["surfaces"].items()
            if session.state.get(f"owner_of_{surface_id}") == target_agent.name
        }

        # 剥離後のデータモデルで置き換える
        message["metadata"]["a2uiClientDataModel"]["surfaces"] = filtered_surfaces

    return request_payload
```

メタデータを剥離することで、オーケストレーターはサブエージェントに認可された範囲のデータだけを渡せます。

> [!CAUTION]
> **セキュリティリスク: 状態スクレイピング**: オーケストレーターが `a2uiClientDataModel` を剥離し損ねると、悪意ある、あるいは侵害されたサブエージェントが他のアクティブ surface の状態を「スクレイプ」できる可能性があります。たとえば、天気予報サブエージェントが、オーケストレーターの漏えいによって銀行用 surface の機微データを読むことが起こり得ます。剥離はマルチエージェントシステムの必須セキュリティ要件です。

---

## 包括的な例

### 1. ボタン送信（明示的なコンテキスト）

この例では、ボタンが送信に必要なデータを明示的に集めます。

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
        "partySize": { "path": "/partySize" },
        "reservationTime": { "path": "/reservationTime" }
      }
    }
  }
}
```

**結果のアクションペイロード:**
エージェントは `partySize` と `reservationTime` を `context` フィールドから直接受け取ります。

### 2. 音声での送信（データモデル同期）

このシナリオでは、ユーザーはボタンを押しません。代わりに「OK、フォームを送信して」と言います。

**初期化:**
エージェントは `sendDataModel: true` で surface を作成しています。

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

**クライアント送信:**
クライアントはユーザーのテキストとデータモデルをメタデータに含む A2A メッセージを送ります。

```json
{
  "parts": [{ "text": "Okay, submit the form" }],
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

**エージェントのアクション:**
エージェントはユーザー意図（"submit"）を読み取り、`metadata` から `partySize` と `reservationTime` の現在値を取得します。これで追加の確認なしに処理できます。
