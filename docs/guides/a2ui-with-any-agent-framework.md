# 任意のエージェントフレームワークで A2UI を使う(AG-UI 使用)

A2UI は宣言的 UI 形式です。[AG-UI](https://ag-ui.com/) は、エージェントとブラウザの間で A2UI メッセージを運ぶトランスポートです。CopilotKit の AG-UI 実装は、今日 A2UI をユーザーの前に出す最短経路です。CopilotKit が対応している任意のエージェントフレームワーク(ADK、LangGraph、CrewAI、Mastra、カスタム Python/TS サービスなど)は、トランスポートの glue を追加せずに A2UI を出力し、React アプリでレンダリングできます。

!!! info "信頼できる情報源"

    このガイドは、CopilotKit の [ADK + A2UI docs](https://docs.copilotkit.ai/adk/generative-ui/a2ui) の主要な手順を反映しています。最新の API surface については CopilotKit のドキュメントを参照してください。

## 1. CopilotKit を設定する

選択したフレームワーク(ADK、LangGraph、CrewAI、Mastra など)とともに、React/Next.js アプリに CopilotKit をインストールします。

```bash
npx copilotkit@latest init
```

または [CopilotKit quickstart](https://docs.copilotkit.ai/quickstart) に従って既存プロジェクトへ接続します。これは標準的な CopilotKit セットアップであり、A2UI 専用 scaffold ではありません。

## 2. A2UI を有効化する

### Backend

`CopilotRuntime` で A2UI を有効化し、`render_a2ui` ツールを注入して、エージェントが A2UI surface を生成できるようにします。

```ts title="app/api/copilotkit/route.ts"
import {CopilotRuntime} from '@copilotkit/runtime';

const runtime = new CopilotRuntime({
  agents: {default: myAgent},
  a2ui: {injectA2UITool: true},
});
```

特定のエージェントに限定するには、`a2ui: { injectA2UITool: true, agents: ["my-agent"] }` を使います。

### Frontend

A2UI レンダラーは自動的に有効化されます。必要に応じてテーマを渡せます。

{% raw %}

```tsx
import {CopilotKitProvider} from '@copilotkit/react-core/v2';
import '@copilotkit/react-core/v2/styles.css';
import {myCustomTheme} from '@copilotkit/a2ui-renderer';

<CopilotKitProvider runtimeUrl="/api/copilotkit" a2ui={{theme: myCustomTheme}}>
  {children}
</CopilotKitProvider>;
```

{% endraw %}

### カスタムコンポーネント(BYOC)

A2UI には、すぐに動作する surface を作れる組み込みカタログ(Text、Image、Card など)が付属しています。本当の力は、そこに _あなたの_ React コンポーネントを拡張できることです。つまり、あなたのデザインシステムとデータ形状を使い、すでに信頼している primitive からエージェントがインターフェースを構成できます。カタログは 3 つの要素で構成されます。

1. **定義** — Zod スキーマと自然言語の説明です。エージェントがシステムプロンプトで見る内容です。
2. **レンダラー** — 定義ごとに 1 つある型付き React コンポーネントです。ユーザーが見る内容です。
3. **登録** — provider を通してカタログを渡し、A2UI レンダラーがコンポーネントの描画方法を知るようにします。

#### 1. コンポーネントスキーマを定義する

Zod でプラットフォーム非依存の定義を作成します。`description` フィールドはエージェントのプロンプトに注入され、LLM が各コンポーネントをいつ使うべきか判断できるようにします。スキーマはエージェントが送る props を検証します。

```ts title="lib/a2ui/definitions.ts"
import {z} from 'zod';

export const myDefinitions = {
  StatusBadge: {
    description: 'A colored status badge.',
    props: z.object({
      text: z.string(),
      variant: z.enum(['success', 'warning', 'error']).optional(),
    }),
  },
  Metric: {
    description: 'A key metric with label and value.',
    props: z.object({
      label: z.string(),
      value: z.string(),
      trend: z.enum(['up', 'down']).optional(),
    }),
  },
};

export type MyDefinitions = typeof myDefinitions;
```

#### 2. React レンダラーを作成する

各定義を React コンポーネントにマッピングします。`createCatalog` は定義型に対して generic なので、レンダラーが受け取る props は Zod スキーマに対して型チェックされます。`props.text` のタイプミスはコンパイルエラーになります。

{% raw %}

```tsx title="lib/a2ui/renderers.tsx"
'use client';

import {createCatalog, type CatalogRenderers} from '@copilotkit/a2ui-renderer';
import {myDefinitions, type MyDefinitions} from './definitions';

const myRenderers: CatalogRenderers<MyDefinitions> = {
  StatusBadge: ({props}) => {
    const colors = {
      success: {bg: '#dcfce7', text: '#166534'},
      warning: {bg: '#fef3c7', text: '#92400e'},
      error: {bg: '#fee2e2', text: '#991b1b'},
    };
    const c = colors[props.variant ?? 'success'];
    return (
      <span
        style={{
          padding: '2px 8px',
          borderRadius: 9999,
          fontSize: '0.75rem',
          background: c.bg,
          color: c.text,
        }}
      >
        {props.text}
      </span>
    );
  },

  Metric: ({props}) => (
    <div>
      <div style={{fontSize: '0.75rem', color: '#6b7280'}}>{props.label}</div>
      <div style={{fontSize: '1.5rem', fontWeight: 700}}>
        {props.value} {props.trend === 'up' ? '↑' : props.trend === 'down' ? '↓' : ''}
      </div>
    </div>
  ),
};

export const myCatalog = createCatalog(myDefinitions, myRenderers, {
  catalogId: 'my-app-catalog',
  includeBasicCatalog: true, // merges with built-in components
});
```

{% endraw %}

`catalogId` は、エージェントがこのカタログを対象にするときに使う安定したハンドルです。`includeBasicCatalog: true` にすると、独自コンポーネントと並んで組み込みコンポーネントも利用できます。省略すると _独自コンポーネントだけ_ をレンダリングします。

#### 3. CopilotKit にカタログを渡す

{% raw %}

```tsx title="app/layout.tsx"
'use client';

import {CopilotKitProvider} from '@copilotkit/react-core/v2';
import '@copilotkit/react-core/v2/styles.css';
import {myCatalog} from '@/lib/a2ui/renderers';

export default function Layout({children}: {children: React.ReactNode}) {
  return (
    <CopilotKitProvider runtimeUrl="/api/copilotkit" a2ui={{catalog: myCatalog}}>
      {children}
    </CopilotKitProvider>
  );
}
```

{% endraw %}

これでエージェントは、組み込みコンポーネントと並んであなたのカスタムコンポーネントも参照でき、自分が出力する任意の A2UI surface でそれらを使えるようになります。

複数カタログ、テーマ hook、高度なパターンを含む完全な BYOC リファレンスについては、CopilotKit の [Custom Components (BYOC) section](https://docs.copilotkit.ai/adk/generative-ui/a2ui#custom-components-byoc) を参照してください。

## 3. 高度な使い方

カスタムカタログ、細かな制御、高度なパターンなど、A2UI 統合 surface 全体については CopilotKit の [A2UI docs](https://docs.copilotkit.ai/generative-ui/a2ui) を参照してください。

## 次のステップ

- **[A2UI Composer](https://a2ui-composer.ag-ui.com/)** — ウィジェットを視覚的に構築します。
- **[概念 › トランスポート](../concepts/transports.md)** — A2UI が AG-UI にどのようにマッピングされるかを説明します。
- **[v0.9 specification](../specification/v0.9-a2ui.md)** — 基盤となるプロトコルです。
