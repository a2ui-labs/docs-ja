# A2UI カタログ

## 概要

このガイドでは、A2UI カタログのアーキテクチャを定義し、実装に向けた道筋を示します。カタログスキーマの構造、あらかじめ用意された「Basic Catalog」を使う方法とアプリ固有のカタログを定義する方法、さらにカタログのネゴシエーション、バージョニング、実行時バリデーションの技術仕様を説明します。

## カタログ定義

カタログは、エージェントがサーバー主導 UI で A2UI サーフェスを定義するために使えるコンポーネント、関数、テーマをまとめた [JSON Schema ファイル](../../specification/v0_9/json/client_capabilities.json#L62C5-L95C6) です。エージェントから送られる A2UI JSON は、選択されたカタログに対して検証されます。

以下は [Catalog JSON Schema](../../specification/v0_9/json/client_capabilities.json#L62C5-L95C6) の一部です。

```json
{
  "Catalog": {
    "type": "object",
    "description": "A collection of component and function definitions.",
    "properties": {
      "catalogId": {
        "type": "string",
        "description": "Unique identifier for this catalog."
      },
      "components": {
        "type": "object",
        "description": "Definitions for UI components supported by this catalog.",
        "additionalProperties": {
          "$ref": "https://json-schema.org/draft/2020-12/schema"
        }
      },
      "functions": {
        "type": "array",
        "description": "Definitions for functions supported by this catalog.",
        "items": {
          "$ref": "#/$defs/FunctionDefinition"
        }
      },
      "theme": {
        "title": "A2UI Theme",
        "description": "A schema that defines a catalog of A2UI theme properties.",
        "type": "object",
        "additionalProperties": {
          "$ref": "https://json-schema.org/draft/2020-12/schema"
        }
      }
    },
    "required": [
      "catalogId"
    ],
    "additionalProperties": false
  }
}
```

## カタログ戦略

すべての A2UI サーフェスはカタログによって駆動されます。カタログは、エージェントが利用できるコンポーネント、関数、テーマを示す単なる JSON Schema ファイルです。

試作段階でも本番アプリでも、必要条件は同じです。UI を表現するために使うカタログ定義を用意しなければなりません。

### Basic Catalog

素早く始められるように、A2UI チームは [Basic Catalog](../../specification/v0_9/catalogs/basic/catalog.json) を維持しています。

これは、汎用コンポーネント（Button、Input、Card など）と関数の標準セットを含む、事前定義済みのカタログです。特別な「カタログ種別」ではなく、すでに実装済みでオープンソースのレンダラーが存在するカタログの 1 つの版にすぎません。

Basic Catalog は、独自のスキーマを最初から書かなくてもアプリを立ち上げたり、A2UI の概念を検証したりするのに役立ちます。レンダラーごとの実装を容易に保つため、意図的に最小限にしています。

A2UI は LLM が設計時または実行時に UI を生成する前提で作られているため、複数クライアント間で標準化されたカタログが必須だとは考えていません。LLM はクライアントごとのカタログを読み取れます。

[A2UI v0.9 Basic Catalog を見る](../../specification/v0_9/catalogs/basic/catalog.json)

### 独自カタログの定義

Basic Catalog は開始点として便利ですが、本番アプリの多くは、自身のデザインシステムに合わせた独自カタログを定義します。

独自カタログを定義すると、エージェントはアプリ内に存在するコンポーネントとビジュアル言語だけを使うようになります。汎用の入力欄やボタンに制限されません。カタログは最初から作ってもよいですし、時間短縮のために Basic Catalog の定義を取り込んでも構いません（たとえば、Basic の Text 定義だけを使い、自前の Card コンポーネントを定義するなど）。

単純さを優先するなら、アダプター経由で Basic Catalog をマッピングするよりも、クライアントのデザインシステムを直接反映するカタログを作ることを推奨します。A2UI は GenUI を前提にしているため、LLM はクライアントごとに異なるカタログを解釈できると想定しています。

[Rizzcharts のカタログ例を見る](../../samples/agent/adk/rizzcharts/catalog_schemas/0.9/rizzcharts_catalog_definition.json)

### 推奨

| ユースケース | 推奨 | 工数 |
| :--- | :--- | :--- |
| 既存の成熟したフロントエンドに A2UI を追加する | 既存のデザインシステムに合わせたカタログを定義する | 中 |
| 新規アプリ / グリーンフィールドに A2UI を追加する | まず Basic Catalog で始め、アプリの成長に合わせて独自カタログへ発展させる | 低（レンダラーが存在する前提） |

## カタログの作成

カタログは、サーフェスを構築する際にエージェントが利用できるコンポーネント、テーマ、関数を定義する [Catalog スキーマ](../../specification/v0_9/json/client_capabilities.json#L62C5-L95C6) に準拠した JSON Schema ファイルです。

### 例: 最小カタログ

単一コンポーネントを定義するシンプルなカタログです。

```json
{
  "$id": "https://github.com/.../hello_world/v1/catalog.json",
  "components": {
    "HelloWorldBanner": {
      "type": "object",
      "description": "A simple banner greeting.",
      "properties": {
        "message": {
          "type": "string",
          "description": "The banner text."
        },
        "backgroundColor": {
          "type": "string",
          "default": "#f0f0f0"
        }
      },
      "required": [
        "message"
      ]
    }
  }
}
```

エージェントがこのカタログを使うと、次のような構造に厳密に従ったペイロードを生成します。

```json
[
  {
    "version": "v0.9",
    "createSurface": {
      "surfaceId": "hello-world-surface",
      "catalogId": "https://github.com/.../hello_world/v1/catalog.json"
    }
  },
  {
    "version": "v0.9",
    "updateComponents": {
      "surfaceId": "hello-world-surface",
      "components": [
        {
          "id": "root",
          "component": "HelloWorldBanner",
          "message": "Hello, world! Welcome to your first catalog.",
          "backgroundColor": "#4CAF50"
        }
      ]
    }
  }
]
```

### スタンドアロンなカタログ

A2UI のカタログは、LLM の推論と依存管理を簡単にするため、自己完結している必要があります。外部ファイル参照を含めない形です。

最終的なカタログはスタンドアロンである必要がありますが、ローカル開発中は JSON Schema の `$ref` を使って外部ドキュメントを参照しながらモジュール分割して作成できます。公開前には `tools/build_catalog/assemble_catalog.py` を実行し、外部参照を 1 つの独立した JSON Schema ファイルへまとめてください。

```bash
uv run tools/build_catalog/assemble_catalog.py [INPUTS ...] --output-name <OUTPUT_NAME> [--catalog-id <ID>] [--version <VERSION>] [--extend-basic-catalog] [--out-dir <DIR>] [--verbose]
```

引数の意味は次のとおりです。

- `inputs`: A2UI コンポーネントカタログ JSON のパスまたは URL
- `--output-name`: 結合後のカタログ名（必須）
- `--catalog-id`: 出力のカスタム `catalogId`
- `--version`: 使用する A2UI 仕様バージョン。`0.9` または `0.10`
- `--extend-basic-catalog`: `basic_catalog.json` をルート出力に自動的に含める
- `--out-dir`, `-o`: 生成物の保存先ディレクトリ
- `--verbose`, `-v`: 詳細ログを有効化する

### 構成とインポート

すべてをゼロから定義する必要はありません。既存の Basic Catalog や他のカタログのコンポーネントを再利用できますし、既存のレンダリングロジックを流用することもできます。

#### 例: Basic Catalog を拡張する

このカタログは Basic Catalog の要素をすべて取り込み、新しい `SuggestionChips` コンポーネントを追加します。

```json
{
  "$id": "https://github.com/.../hello_world_with_all_basic/v1/catalog.json",
  "components": {
    "allOf": [
      { "$ref": "basic_catalog_definition.json#/components" },
      {
        "SuggestionChips": {
          "type": "object",
          "description": "A list of suggested prompts",
          "properties": {
            "suggestions": {
              "type": "array",
              "description": "The suggested prompts."
            }
          },
          "required": [ "suggestions" ]
        }
      }
    ]
  }
}
```

**公開前には、外部 `$ref` を解決するために `tools/build_catalog/assemble_catalog.py` を必ず実行してください。**

#### 例: コンポーネントを部分的に取り込む

このカタログは Basic Catalog から `Text` だけを取り込み、シンプルな Popup サーフェスを構成します。

```json
{
  "$id": "https://github.com/.../hello_world_with_some_basic/v1/catalog.json",
  "components": {
    "allOf": [
      { "$ref": "basic_catalog.json#/components/Text" },
      {
        "Popup": {
          "type": "object",
          "description": "A modal overlay that displays an icon and text.",
          "properties": {
            "text": { "$ref": "common_types.json#/$defs/ComponentId" }
          },
          "required": [ "text" ]
        }
      }
    ]
  }
}
```

**公開前には、外部 `$ref` を解決するために `tools/build_catalog/assemble_catalog.py` を必ず実行してください。**

### レンダラーの実装

クライアントレンダラーは、スキーマ定義を実際のコードへマッピングすることでカタログを実装します。

`Hello World` カタログ向けの TypeScript レンダラーの例です。
