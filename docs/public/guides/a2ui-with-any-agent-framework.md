# 任意のエージェントフレームワークとハーネスで A2UI を使う

A2UI は宣言的な UI 形式です。[AG-UI](https://ag-ui.com/) は、エージェントとアプリの間で A2UI メッセージを運ぶトランスポートです。このガイドでは、ADK、LangGraph、Mastra、Strands、CrewAI、Google Chat、Slack など、AG-UI に対応する任意のエージェントフレームワークやサービスに支えられた AG-UI アプリまたはハーネスに A2UI を追加する方法を説明します。

<style>
  .agui-demo-video {
    border-radius: 8px;
    display: block;
    margin: 24px auto;
    max-width: 100%;
    width: 75%;
  }

  @media (max-width: 700px) {
    .agui-demo-video {
      width: 100%;
    }
  }
</style>

<video class="agui-demo-video" controls playsinline preload="metadata">
  <source src="https://cdn.copilotkit.ai/docs/a2ui/ag-ui-a2ui-demo.mp4" type="video/mp4">
  ブラウザがビデオタグをサポートしていません。
</video>

以下の例では、AG-UI 互換のランタイムツールを使うことで、レンダラーの有効化、エージェントへのカタログの提供、UI 更新のユーザーへのストリーミングといった A2UI 部分に集中できるようにしています。プロトコルレベルのセットアップや概念については、[AG-UI のドキュメント](https://docs.ag-ui.com/)を参照してください。

## エージェントスキル

コーディングエージェントを使ってこれを組み込む場合は、アプリを変更する前に [AG-UI の `ag-ui-a2ui-integration` スキル](https://github.com/ag-ui-protocol/ag-ui/tree/main/skills/ag-ui-a2ui-integration)を読み込ませてください。このスキルは、AG-UI のフレームワークアダプター、対応している `create-ag-ui-app` のフラグ、トランスポートのセットアップ、A2UI ランタイムとレンダラーの配線、そして AG-UI + A2UI アプリのエンドツーエンド検証をカバーしています。

アプリが A2UI のレンダリングに CopilotKit を使用している場合は、CopilotKit v2 のランタイム、プロバイダー、テーマ、カタログの規約について、[CopilotKit の `a2ui-renderer` スキル](https://github.com/CopilotKit/CopilotKit/blob/main/skills/a2ui-renderer/SKILL.md)もあわせて読み込ませてください。

## 1. AG-UI をセットアップする

すでに使っているエージェントフレームワークから始め、エージェントとアプリの間に AG-UI ランタイムの接続を追加します。このランタイムは、A2UI メッセージを含むエージェントイベントをクライアント側のサーフェスにストリーミングします。

AG-UI の CLI を使って、好みのクライアントとエージェントフレームワークで AG-UI アプリの雛形を生成します。

```bash
npx create-ag-ui-app@latest
```

サポートされているフレームワークのテンプレートから直接始めることもできます。

```bash
npx create-ag-ui-app@latest --adk
npx create-ag-ui-app@latest --langgraph-py
npx create-ag-ui-app@latest --langgraph-js
```

Strands にはまだ雛形生成用のフラグがありません。既存の Strands エージェントをラップしてください(下記の Strands のパネルを参照)。

重要なのはトランスポートの契約です。アプリは AG-UI イベントを受け取り、A2UI のペイロードを A2UI レンダラーへルーティングします。一部の雛形生成パスは、内部で Next.js を使った [CopilotKit の A2UI ランタイム](https://docs.copilotkit.ai/generative-ui/a2ui)を利用していますが、セットアップの中心はあくまで AG-UI です。

## 2. エージェントまたはハーネスをセットアップする

A2UI の手順はどのフレームワークでも同じです。エージェントを AG-UI に接続し、A2UI ペイロードを有効化し、そのペイロードをアプリ内でレンダリングします。すでに使っているフレームワークやハーネスから始めてください。以下のスニペットは、対応する AG-UI インテグレーションから抜粋したもので、AG-UI がラップするフレームワークネイティブなエージェントの形を示しています。

=== "ADK"

    ADK は、エージェントがすでに Google の Agent Development Kit 上で動作している場合に使用します。AG-UI の ADK ミドルウェアは、エージェントを AG-UI イベントストリームとして公開します。

    ```python
    from fastapi import FastAPI
    from ag_ui_adk import ADKAgent, AGUIToolset, add_adk_fastapi_endpoint
    from google.adk.agents import Agent

    my_agent = Agent(
        name="assistant",
        instruction="You are a helpful assistant.",
        tools=[
            AGUIToolset(),  # AG-UI クライアントが提供するツールを追加します。
        ],
    )

    agent = ADKAgent(
        adk_agent=my_agent,
        app_name="my_app",
        user_id="user123",
    )

    app = FastAPI()
    add_adk_fastapi_endpoint(app, agent, path="/chat")
    ```

    詳しくは [AG-UI ADK middleware](https://github.com/ag-ui-protocol/ag-ui/tree/main/integrations/adk-middleware/python) を参照してください。

=== "LangGraph (Python)"

    LangGraph は、エージェントのワークフローがステートフルなノードのグラフである場合に使用します。普段どおりの LangGraph エージェントから始めてください。A2UI はグラフ側に追加のツール配線を必要としません。

    ```python
    from copilotkit import CopilotKitMiddleware
    from langchain.agents import create_agent
    from langchain_google_genai import ChatGoogleGenerativeAI

    gemini = ChatGoogleGenerativeAI(
        model="gemini-2.5-pro",
        thinking_budget=1024,
    )

    # 普通の LangGraph エージェントです。グラフ側には A2UI 用のツール配線はありません。
    # CopilotKit ランタイムがフロントエンドのカタログを転送し、`generate_a2ui` ツールを
    # 注入します。A2UI 機能を得るには CopilotKitMiddleware を含めてください。
    graph = create_agent(
        model=gemini,
        tools=[],
        middleware=[CopilotKitMiddleware()],
        system_prompt="You are a helpful assistant.",
    )
    ```

    LangGraph の A2UI ツールは CopilotKit のミドルウェア層で動作するため、A2UI 機能を得るには
    `CopilotKitMiddleware` を含めます。CopilotKit ランタイムがカタログを転送し、`generate_a2ui`
    を自動的に注入します。この例では、LangChain の Google GenAI インテグレーション経由で
    Gemini を使用しています。

    詳しくは [AG-UI LangGraph integration](https://github.com/ag-ui-protocol/ag-ui/tree/main/integrations/langgraph/python)
    と [ChatGoogleGenerativeAI integration](https://docs.langchain.com/oss/python/integrations/chat/google_generative_ai)
    を参照してください。

=== "LangGraph (FastAPI)"

    同じ LangGraph グラフを FastAPI アプリの背後で提供する場合は、FastAPI バリアントを使用します。
    エージェントの形は同一です。同じ `graph` をエクスポートし、AG-UI LangGraph エンドポイント経由
    で提供します。

    ```python
    from copilotkit import CopilotKitMiddleware
    from langchain.agents import create_agent
    from langchain_google_genai import ChatGoogleGenerativeAI

    gemini = ChatGoogleGenerativeAI(
        model="gemini-2.5-pro",
        thinking_budget=1024,
    )

    graph = create_agent(
        model=gemini,
        tools=[],
        middleware=[CopilotKitMiddleware()],
        system_prompt="You are a helpful assistant.",
    )
    ```

    詳しくは [AG-UI LangGraph integration](https://github.com/ag-ui-protocol/ag-ui/tree/main/integrations/langgraph/python) を参照してください。

=== "LangGraph (TypeScript)"

    LangGraph エージェントが TypeScript で書かれている場合は、TypeScript バリアントを使用します。
    その形は Python 版のエージェントと同じで、素の graph に CopilotKit ミドルウェアを加えたもの
    です。

    ```ts
    import { createAgent } from "langchain";
    import { ChatOpenAI } from "@langchain/openai";
    import { copilotkitMiddleware } from "@copilotkit/sdk-js/langgraph";

    export const graph = createAgent({
      model: new ChatOpenAI({ model: "gpt-4o" }),
      tools: [],
      middleware: [copilotkitMiddleware],
      systemPrompt: "You are a helpful assistant.",
    });
    ```

    詳しくは [AG-UI LangGraph TypeScript integration](https://github.com/ag-ui-protocol/ag-ui/tree/main/integrations/langgraph/typescript) を参照してください。

=== "Strands (Python)"

    Strands は、エージェントのオーケストレーションが AWS Strands 上に構築されている場合に使用
    します。素の Strands エージェントを AG-UI の Strands アダプターでラップします。

    ```python
    from strands import Agent
    from ag_ui_strands import StrandsAgent

    strands_agent = Agent(
        system_prompt="You are a helpful assistant.",
    )

    agent = StrandsAgent(
        agent=strands_agent,
        name="my-agent",
        description="A Strands agent exposed via AG-UI",
    )
    ```

    詳しくは [AG-UI AWS Strands integration](https://github.com/ag-ui-protocol/ag-ui/tree/main/integrations/aws-strands/python) を参照してください。

=== "Strands (TypeScript)"

    Strands エージェントが TypeScript で書かれている場合は、TypeScript バリアントを使用します。
    AG-UI の Strands アダプターが、AG-UI クライアント向けに Strands エージェントをラップします。

    ```ts
    import { Agent } from "@strands-agents/sdk";
    import { StrandsAgent } from "@ag-ui/aws-strands";
    import { createStrandsApp } from "@ag-ui/aws-strands/server";

    const strandsAgent = new Agent({
      systemPrompt: "You are a helpful assistant.",
      tools: [],
    });

    const aguiAgent = new StrandsAgent({
      agent: strandsAgent,
      name: "MyAgent",
      description: "A Strands agent exposed via AG-UI",
    });

    const app = await createStrandsApp(aguiAgent, { path: "/invocations" });
    app.listen(8000);
    ```

    詳しくは [AG-UI AWS Strands integration](https://github.com/ag-ui-protocol/ag-ui/tree/main/integrations/aws-strands/typescript) を参照してください。

=== "Slack"

    Slack は、ユーザー体験が Slack アプリ内にある場合に使用します。Slack のスレッドを同じ AG-UI
    エージェントエンドポイントにルーティングします。同じ AG-UI イベントストリームが Slack
    ハーネスに供給され、そのサーフェスのクライアントブリッジを通じて A2UI をレンダリングでき
    ます。

    <video class="agui-demo-video" controls playsinline preload="metadata">
      <source src="https://cdn.copilotkit.ai/docs/a2ui/ag-ui-slack-demo.mp4" type="video/mp4">
      ブラウザがビデオタグをサポートしていません。
    </video>

    CopilotKit の Slack アダプターは、すでにこのパターンを実装しています。

    ```ts
    import { createBot } from "@copilotkit/bot";
    import {
      slack,
      SanitizingHttpAgent,
      defaultSlackTools,
      defaultSlackContext,
    } from "@copilotkit/bot-slack";

    const bot = createBot({
      adapters: [
        slack({
          botToken: process.env.SLACK_BOT_TOKEN!,
          appToken: process.env.SLACK_APP_TOKEN!,
        }),
      ],
      agent: (threadId) => {
        const agent = new SanitizingHttpAgent({
          url: process.env.AGENT_URL!,
        });
        agent.threadId = threadId;
        return agent;
      },
      tools: [...defaultSlackTools],
      context: [...defaultSlackContext],
    });

    bot.onMention(async ({ thread }) => {
      await thread.runAgent();
    });

    await bot.start();
    ```

これらのスニペットは AG-UI サーバー接続を確立するものです。Slack も、独自のハーネスとクライアントブリッジを介して同じ AG-UI/A2UI の契約を使用します。次のセクションでは、アプリのサーフェス内で A2UI のレンダリング、カタログ、コンポーネント定義を有効にします。

## 3. A2UI を有効化する

望ましい開発者体験から始めます。エージェントが参照できるカタログ定義を定義し、各定義をレンダラーにマッピングし、カタログを作成して、そのカタログを CopilotKit に渡します。フロントエンドのカタログ設定こそが、A2UI を有効化するための対象サーフェスです。

{% raw %}

```tsx
import {CopilotKit, CopilotChat} from '@copilotkit/react-core/v2';
import {
  createCatalog,
  type CatalogDefinitions,
  type CatalogRenderers,
} from '@copilotkit/a2ui-renderer';
import {z} from 'zod';

// カタログ定義 — 構成要素となるコンポーネントをエージェントに説明します
export const catalogDefinitions = {
  Card: {
    description: 'A titled card container.',
    props: z.object({title: z.string(), subtitle: z.string().optional()}),
  },
  PrimaryButton: {
    description: 'A styled primary button.',
    props: z.object({label: z.string(), action: z.any().optional()}),
  },
} satisfies CatalogDefinitions;

// カタログレンダラー — 各 primitive を DOM でどう描画するか(この例では React)
export const catalogRenderers = {
  Card: MyCard,
  PrimaryButton: MyPrimaryButton,
} satisfies CatalogRenderers<typeof catalogDefinitions>;

// definitions と renderers を組み合わせて、カタログの宣言を定義します
const catalog = createCatalog(catalogDefinitions, catalogRenderers, {
  catalogId: 'my-catalog',
  includeBasicCatalog: true,
});

<CopilotKit runtimeUrl="/api/copilotkit" a2ui={{catalog}}>
  <CopilotChat />
</CopilotKit>;
```

{% endraw %}

プロバイダーにカタログを渡すと A2UI が自動的に有効化され、`generate_a2ui` ツールが注入されるため、エージェントは追加のランタイム設定なしにサーフェスを生成できます(CopilotKit ≥ 1.61.2)。オプトアウトしたい場合や、カタログなしで手動でオプトインしたい場合は、ランタイムを直接設定します。

```ts title="app/api/copilotkit/route.ts"
import {CopilotRuntime} from '@copilotkit/runtime';

const runtime = new CopilotRuntime({
  agents: {default: myAgent},
  a2ui: {injectA2UITool: true},
});
```

特定のエージェントに限定するには `a2ui: { injectA2UITool: true, agents: ["my-agent"] }` を使います。エージェントがすでに `a2ui_operations` を返す固定スキーマのフローでは、`a2ui: true` または `a2ui: {}` だけで十分です。

### カスタムコンポーネント(BYOC)

A2UI には、すぐに動作するサーフェスを作れる組み込みカタログ(Text、Image、Card など)が付属しています。以下に示す拡張版の BYOC フローでは、実際のアプリを想定して、同じカタログパターンを複数ファイルに分割した形で示します。

1. **定義**：Zod スキーマと自然言語による説明です。これはエージェントがシステムプロンプトで見る内容です。なお、クライアント側関数については、クライアントが実行時にアクティブなカタログ定義から設定を読み取ることで、その関数の実行境界(clientOnly ステータスなど)を判定します。
2. **レンダラー**：定義ごとに 1 つ用意する、型付きの React コンポーネントです。ユーザーが実際に目にする内容です。
3. **登録**：プロバイダーを通してカタログを渡し、A2UI レンダラーがコンポーネントの描画方法を認識できるようにします。

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

各定義を React コンポーネントにマッピングします。`createCatalog` は定義の型に対して generic であるため、レンダラーが受け取る props は Zod スキーマに対して型チェックされます。そのため、`props.text` のタイプミスはコンパイルエラーになります。

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
  includeBasicCatalog: true, // 組み込みコンポーネントとマージします
});
```

{% endraw %}

`catalogId` は、エージェントがこのカタログを対象にする際に使う安定したハンドルです。`includeBasicCatalog: true` にすると、独自コンポーネントと並んで組み込みコンポーネントも利用できます(省略すると _独自コンポーネントだけ_ をレンダリングします)。

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

これで、エージェントは組み込みコンポーネントと並んであなたのカスタムコンポーネントを認識し、自身が出力する任意の A2UI サーフェスでそれらを使用できるようになります。

複数カタログ、テーマフック、高度なパターンを含む完全な BYOC リファレンスについては、CopilotKit の [Custom Components (BYOC) section](https://docs.copilotkit.ai/generative-ui/a2ui) を参照してください。

## 4. 高度な使い方

カスタムカタログ、細かな制御、高度なパターンなど、A2UI 統合の全体像については CopilotKit の [A2UI docs](https://docs.copilotkit.ai/generative-ui/a2ui) を参照してください。

## 次のステップ

- **[A2UI Composer](https://a2ui-composer.ag-ui.com/)**：ウィジェットを視覚的に構築します。
- **[概念 › トランスポート](../concepts/transports.md)**：A2UI が AG-UI にどのようにマッピングされるかを説明します。
- **[v0.9 specification](../specification/v0.9-a2ui.md)**：基盤となるプロトコルです。
