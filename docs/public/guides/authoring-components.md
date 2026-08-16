---
render_macros: false
---

# カスタムコンポーネントを作成する

`rizzcharts` サンプルを例に、A2UI でカスタムコンポーネントを定義、実装、登録する方法を説明します。このガイドは、Angular コードを中心にコンポーネントを作成することに焦点を当てています。

## 概要

新しいコンポーネントの作成には、主に 4 つの手順があります。

1.  **カタログスキーマを定義する**: JSON Schema にコンポーネントのプロパティと型を指定します。
2.  **コンポーネントを定義する(クライアント)**: 使用しているフレームワーク(例: Angular)で UI を実装します。
3.  **レンダラーに登録する(クライアント)**: コンポーネントをクライアント側カタログへ追加します。
4.  **エージェントから呼び出す**: `send_a2ui_json_to_client` を通じて、そのコンポーネントを使うようエージェントに指示します。

---

## 1. カタログスキーマを定義する

カタログスキーマは、カタログの API を定義します。利用可能なコンポーネントとそのプロパティを列挙し、エージェントはそれを使って UI payload を構築します。

**このスキーマは、クライアントとサーバー(エージェント)の間の契約として機能します。** レンダリングが機能するには、双方がこのスキーマに合意している必要があります。クライアントは対応しているカタログを通知し、サーバーは互換性のあるものを選択します。この handshake の仕組みについては [A2UI Catalog Negotiation](../concepts/catalogs.md#a2ui-catalog-negotiation) を参照してください。

[`rizzcharts`](../../../samples/community/agent/adk/rizzcharts/python/README.md) の例では、カタログスキーマは [`rizzcharts_catalog_definition.json`](../../../samples/community/agent/adk/rizzcharts/catalog_schemas/0.9/rizzcharts_catalog_definition.json) に定義されています。

以下は `Chart` コンポーネントのスキーマです。

```json
"Chart": {
  "type": "object",
  "description": "An interactive chart that uses a hierarchical list of objects for its data.",
  "properties": {
    "type": {
      "type": "string",
      "description": "The type of chart to render.",
      "enum": [
        "doughnut",
        "pie"
      ]
    },
    "title": {
      "type": "object",
      "description": "The title of the chart. Can be a literal string or a data model path.",
      "properties": {
        "literalString": {
          "type": "string"
        },
        "path": {
          "type": "string"
        }
      }
    },
    "chartData": {
      "type": "object",
      "description": "The data for the chart, provided as a list of items. Can be a literal array or a data model path.",
      "properties": {
        "literalArray": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "label": {
                "type": "string"
              },
              "value": {
                "type": "number"
              },
              "drillDown": {
                "type": "array",
                "description": "An optional list of items for the next level of data.",
                "items": {
                  "type": "object",
                  "properties": {
                    "label": {
                      "type": "string"
                    },
                    "value": {
                      "type": "number"
                    }
                  },
                  "required": [
                    "label",
                    "value"
                  ]
                }
              }
            },
            "required": [
              "label",
              "value"
            ]
          }
        },
        "path": {
          "type": "string"
        }
      }
    }
  },
  "required": [
    "type",
    "chartData"
  ]
}
```

---

## 2. コンポーネントを実装する(クライアント)

クライアント側フレームワークを使ってコンポーネントを実装します。Angular の場合、コンポーネントは `@a2ui/angular/v0_9` が提供する `CatalogComponent` を拡張する必要があります。

`rizzcharts` の例では、`Chart` コンポーネントは `chart.ts` に定義されています。

まず、コンポーネントの API を TypeScript で定義します。これは手順 1 で定義した JSON Schema と一致させる必要があります。

```typescript
// api.ts
import {ComponentApi} from '@a2ui/web_core/v0_9';
import {z} from 'zod';

export const ChartApi = {
  name: 'Chart',
  schema: z.object({
    type: z.enum(['doughnut', 'pie']),
    title: z.string().optional(),
    chartData: z.array(
      z.object({
        label: z.string(),
        value: z.number(),
        drillDown: z.array(
          z.object({
            label: z.string(),
            value: z.number(),
          })
        ).optional(),
      })
    ),
  }).strict(),
} satisfies ComponentApi;
```

次に、Angular コンポーネントを実装します。

```typescript
import {CatalogComponent} from '@a2ui/angular/v0_9';
import {Component, computed} from '@angular/core';
import {BaseChartDirective} from 'ng2-charts';
import {ChartApi} from './api';

@Component({
  selector: 'a2ui-chart',
  imports: [BaseChartDirective],
  template: `
    <div>
      <h2>{{ title() }}</h2>
      <canvas baseChart [data]="chartData()" [type]="chartType()"></canvas>
    </div>
  `,
})
export class Chart extends CatalogComponent<typeof ChartApi> {
  protected readonly chartType = computed(() => this.props()['type']?.value() || 'pie');
  protected readonly title = computed(() => this.props()['title']?.value() || '');
  protected readonly chartData = computed(() => {
    const rawData = this.props()['chartData']?.value() || [];
    return {
      labels: rawData.map(item => item.label),
      datasets: [
        {
          data: rawData.map(item => item.value),
        },
      ],
    };
  });
}
```

コンポーネントを実装するときは、次の重要点を意識してください。

- **`CatalogComponent` を拡張する**: 型安全な `props` シグナル入力にアクセスできます。
- **`props()` シグナルを使う**: `this.props()['propertyName']?.value()` を通じて解決済みプロパティへリアクティブにアクセスします。データバインディングや式の解決はフレームワークが自動的に処理します。

---

## 3. レンダラーに登録する(クライアント)

コンポーネントを実装したら、クライアントカタログに登録します。これにより、エージェントが使用するコンポーネント名が実装クラスにマッピングされます。

カタログの定義には `AngularCatalog` クラスを使用します。

```typescript
import {AngularCatalog, BASIC_COMPONENTS, BASIC_FUNCTIONS} from '@a2ui/angular/v0_9';
import {Chart} from './chart';
import {ChartApi} from './api';

const customChartComponent = {
  ...ChartApi,
  component: Chart
};

export const RIZZ_CHARTS_CATALOG = new AngularCatalog(
  'https://github.com/.../rizzcharts_catalog_definition.json',
  [...BASIC_COMPONENTS, customChartComponent],
  BASIC_FUNCTIONS
);
```

登録時の重要点は次のとおりです。

- **Eager Registration**: コンポーネントクラスはカタログ定義に直接登録されます。

---

## 4. エージェントから呼び出す

カスタムコンポーネントを使用するには、自分のカタログを理解する A2UI SDK のツールでエージェントを初期化します。SDK はカタログの解決と、モデルへの例の提供を処理します。

流れは次のようにつながります。

### 4.1 セッション準備(Executor)

実行レイヤー(例: `RizzchartsAgentExecutor`)は受信メッセージをインターセプトし、A2UI が有効かどうか、クライアントがどのカタログをサポートしているかを検出します。カタログを解決し、セッション状態に保存します。

```python
# In agent_executor.py

use_ui = try_activate_a2ui_extension(context)
if use_ui:
    # Resolve catalog based on client capabilities
    a2ui_catalog = self.schema_manager.get_selected_catalog(
        client_ui_capabilities=capabilities
    )
    examples = self.schema_manager.load_examples(a2ui_catalog, validate=True)

    # Save to session (Event contains state_delta)
    await runner.session_service.append_event(
        session,
        Event(
            actions=EventActions(
                state_delta={
                    _A2UI_ENABLED_KEY: True,
                    _A2UI_CATALOG_KEY: a2ui_catalog,
                    _A2UI_EXAMPLES_KEY: examples,
                }
            ),
        ),
    )
```

### 4.2 エージェントツール設定

Agent は [SendA2uiToClientToolset](../../../agent_sdks/python/a2ui_agent/src/a2ui/adk/send_a2ui_to_client_toolset.py) を使って、A2UI をクライアントへ送るためのツールをエージェントに提供します。

```python
from a2ui.adk.send_a2ui_to_client_toolset import SendA2uiToClientToolset

a2ui_catalog = self.schema_manager.get_selected_catalog(
    client_ui_capabilities=capabilities
)
agent.tools = [
    SendA2uiToClientToolset(
        a2ui_catalog=a2ui_catalog,
        a2ui_enabled=True,
    )
]
```

### 4.3 ツール実行

LLM による [SendA2uiToClientToolset](../../../agent_sdks/python/a2ui_agent/src/a2ui/adk/send_a2ui_to_client_toolset.py) のツール呼び出しは、A2A Agent Executor で [A2uiEventConverter](../../../agent_sdks/python/a2ui_agent/src/a2ui/adk/a2a/event_converter.py) によってインターセプトされます。これにより、ツール呼び出しは A2UI payload を持つ A2A Dataparts に自動変換されます。

```python
from a2ui.adk.a2a.event_converter import (
    A2uiEventConverter,
)

config = A2aAgentExecutorConfig(event_converter=A2uiEventConverter())
```
