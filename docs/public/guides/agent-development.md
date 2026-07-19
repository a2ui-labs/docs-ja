# エージェント開発ガイド

A2UI インターフェースを生成する AI エージェントを構築します。このガイドでは、LLM から UI メッセージを生成し、ストリーミングする方法を扱います。

## クイックオーバービュー

A2UI エージェントを構築する流れ:

1. **ユーザーの意図を理解する** → 表示する UI を決定する
2. **A2UI JSON を生成する** → LLM の構造化出力またはプロンプトを使用する
3. **検証してストリーミングする** → スキーマを確認し、クライアントへ送信する
4. **アクションを処理する** → ユーザーの操作に応答する

## シンプルなエージェントから始める

このガイドでは ADK を使ってシンプルなエージェントを構築し、テキストのみの状態から A2UI へアップグレードしていきます。

手順の詳細は [ADK クイックスタート](https://google.github.io/adk-docs/get-started/python/) を参照してください。

```bash
pip install google-adk
adk create my_agent
```

> **TIP**: `uv` を使っていて、サンプルディレクトリ内(またはすでに `google-adk` に依存しているプロジェクト内)で作業している場合は、グローバルにインストールする代わりに `uv run adk` を使えます。
>
> ```bash
> uv run adk create my_agent
> ```

次に、`my_agent/agent.py` ファイルを、レストランのおすすめを提案する非常にシンプルなエージェントとして編集します。

```python
import json
from google.adk.agents.llm_agent import Agent
from google.adk.tools.tool_context import ToolContext

def get_restaurants(tool_context: ToolContext) -> str:
    """Call this tool to get a list of restaurants."""
    return json.dumps([
        {
            "name": "Xi'an Famous Foods",
            "detail": "Spicy and savory hand-pulled noodles.",
            "imageUrl": "http://localhost:10002/static/shrimpchowmein.jpeg",
            "rating": "★★★★☆",
            "infoLink": "[More Info](https://www.xianfoods.com/)",
            "address": "81 St Marks Pl, New York, NY 10003"
        },
        {
            "name": "Han Dynasty",
            "detail": "Authentic Szechuan cuisine.",
            "imageUrl": "http://localhost:10002/static/mapotofu.jpeg",
            "rating": "★★★★☆",
            "infoLink": "[More Info](https://www.handynasty.net/)",
            "address": "90 3rd Ave, New York, NY 10003"
        },
        {
            "name": "RedFarm",
            "detail": "Modern Chinese with a farm-to-table approach.",
            "imageUrl": "http://localhost:10002/static/beefbroccoli.jpeg",
            "rating": "★★★★☆",
            "infoLink": "[More Info](https://www.redfarmnyc.com/)",
            "address": "529 Hudson St, New York, NY 10014"
        },
    ])

AGENT_INSTRUCTION="""
You are a helpful restaurant finding assistant. Your goal is to help users find and book restaurants using a rich UI.

To achieve this, you MUST follow this logic:

1.  **For finding restaurants:**
    a. You MUST call the `get_restaurants` tool. Extract the cuisine, location, and a specific number (`count`) of restaurants from the user's query (e.g., for "top 5 chinese places", count is 5).
    b. After receiving the data, you MUST follow the instructions precisely to generate the final a2ui UI JSON, using the appropriate UI example from the `prompt_builder.py` based on the number of restaurants."""

root_agent = Agent(
    model='gemini-2.5-flash',
    name="restaurant_agent",
    description="An agent that finds restaurants and helps book tables.",
    instruction=AGENT_INSTRUCTION,
    tools=[get_restaurants],
)
```

この例を実行するには、`GOOGLE_API_KEY` 環境変数の設定を忘れないでください。

```bash
echo 'GOOGLE_API_KEY="YOUR_API_KEY"' > .env
```

ADK の Web インターフェースでこのエージェントを試すことができます。

```bash
adk web
```

リストから `my_agent` を選び、ニューヨークのレストランについて質問してみてください。UI にレストランの一覧がプレーンテキストとして表示されるはずです。

## A2UI メッセージの生成

LLM に A2UI メッセージを生成させるには、多少のプロンプトエンジニアリングが必要です。SDK が提供する `A2uiSchemaManager` を使うと、A2UI スキーマとコンポーネントカタログの例を含むシステムプロンプトを生成できます。

まず、`a2ui-agent-sdk` がインストールされていることを確認してください(サンプルには同梱されています)。

エージェントファイル(例: `agent.py`)内で、必要なクラスをインポートします。

```python
from a2ui.schema.constants import VERSION_0_8, VERSION_0_9
from a2ui.strategies.schema import A2uiSchemaManager
from a2ui.basic_catalog.provider import BasicCatalog
```

続いて、`A2uiSchemaManager` を使ってシステムプロンプトを生成します。これにより、スキーマと例が正しくフォーマットされ、常に最新の状態に保たれます。

```python
# Define your agent's role
ROLE_DESCRIPTION = (
    "You are a helpful restaurant finding assistant. Your final output MUST be a a2ui"
    " UI JSON response."
)

# Define rules for when to use which UI template
UI_DESCRIPTION = """
-   If the query is for a list of restaurants, use the restaurant data you have already received from the `get_restaurants` tool to populate the `dataModelUpdate.contents` (v0.8) or `updateDataModel.value` (v0.9+) object (e.g., for the "items" key).
-   If the number of restaurants is 5 or fewer, you MUST use the `SINGLE_COLUMN_LIST_EXAMPLE` template.
-   If the number of restaurants is more than 5, you MUST use the `TWO_COLUMN_LIST_EXAMPLE` template.
-   If the query is to book a restaurant (e.g., "USER_WANTS_TO_BOOK..."), you MUST use the `BOOKING_FORM_EXAMPLE` template.
-   If the query is a booking submission (e.g., "User submitted a booking..."), you MUST use the `CONFIRMATION_EXAMPLE` template.
"""

# Initialize the schema manager with the Basic Catalog
schema_manager = A2uiSchemaManager(
    version=VERSION_0_8, # Use VERSION_0_9 for newer protocol
    catalogs=[
        BasicCatalog.get_config(
            version=VERSION_0_8, examples_path="examples/0.8"
        )
    ],
)

# Generate the full system prompt
A2UI_AND_AGENT_INSTRUCTION = schema_manager.generate_system_prompt(
    role_description=ROLE_DESCRIPTION,
    ui_description=UI_DESCRIPTION,
    include_schema=True,
    include_examples=True,
    validate_examples=True,
)

root_agent = Agent(
    model='gemini-2.5-flash',
    name="restaurant_agent",
    description="An agent that finds restaurants and helps book tables.",
    instruction=A2UI_AND_AGENT_INSTRUCTION,
    tools=[get_restaurants],
)
```

## 出力の理解

エージェントはもはや厳密にはテキストのみを出力するわけではありません。代わりに、テキストと A2UI メッセージの **JSON リスト** を出力します。

インポートした `A2UI_SCHEMA` は、次のような有効な操作を定義する標準的な JSON スキーマです。

- `render`(UI を表示する)
- `update`(既存の UI 内のデータを変更する)

出力は構造化された JSON なので、クライアントへ送信する前にパースして検証できます。

```python
# 1. Parse the JSON
# Warning: Parsing the output as JSON is a fragile implementation useful for documentation.
# LLMs often put Markdown fences around JSON output, and can make other mistakes.
# Rely on frameworks to parse the JSON for you.
parsed_json_data = json.loads(json_string_cleaned)

# 2. Validate against A2UI_SCHEMA
# This ensures the LLM generated valid A2UI commands
jsonschema.validate(
    instance=parsed_json_data, schema=self.a2ui_schema_object
)
```

`A2UI_SCHEMA` に対して出力を検証することで、クライアントが不正な形式の UI 指示を受け取らないようにできます。

TODO: A2A 拡張を使わずに、クライアントレンダラーへ出力をパース・検証・送信する方法の例で、このガイドを続ける予定です。
