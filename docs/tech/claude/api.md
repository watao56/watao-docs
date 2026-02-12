# API活用ガイド

> Claude API を使ったアプリケーション開発の完全ガイド。
> 料金体系から実装例まで、効率的な活用法を詳しく解説します。

## Anthropic API 概要

**Anthropic API** は Claude の機能をプログラムから利用できる REST API です。

### 🚀 API の特徴

- **高性能モデル**: Opus 4.6、Sonnet 4.5 等の最新モデル
- **大容量コンテキスト**: 200K〜1M トークン
- **豊富な機能**: Tool Use、Vision、Extended Thinking
- **開発者フレンドリー**: 明確なドキュメント、SDKサポート

### 🔗 基本情報

| 項目 | 詳細 |
|------|------|
| **ベースURL** | `https://platform.claude.com/api` |
| **認証方式** | API Key (Bearer Token) |
| **レート制限** | プランと使用量による |
| **SDK** | Python、TypeScript、Go |

## 料金体系（2026年最新）

### 💰 モデル別料金

| モデル | 入力（100万トークン） | 出力（100万トークン） | 特徴 |
|--------|---------------------|---------------------|------|
| **Opus 4.6** | $15.00 | $75.00 | 最高性能・長時間推論 |
| **Sonnet 4.5** | $3.00 | $15.00 | バランス型・汎用 |
| **Haiku 4.5** | $0.25 | $1.25 | 高速・軽量 |

!!! note "長文コンテキスト料金"
    200K トークンを超える場合、追加料金が適用されます（1.1倍）

### 📊 使用量目安

| 用途 | 月額予算目安 | 推奨モデル |
|------|-------------|-----------|
| **個人開発** | $10-50 | Sonnet 4.5 |
| **小規模SaaS** | $100-500 | Sonnet 4.5 + Haiku 4.5 |
| **企業アプリ** | $1000+ | Opus 4.6 + Sonnet 4.5 |

### 💡 コスト最適化戦略

#### 1. **モデル使い分け**
```python
def select_model(task_complexity):
    if task_complexity == "simple":
        return "claude-haiku-4-5"
    elif task_complexity == "medium":
        return "claude-sonnet-4-5"
    else:
        return "claude-opus-4-6"
```

#### 2. **Prompt Caching 活用**
```python
# 頻繁に使用する長文プロンプトはキャッシュ化
system_prompt = """
長い System Prompt...
（キャッシュされ、コスト削減）
"""
```

#### 3. **ストリーミング活用**
```python
# レスポンス速度向上 + UX改善
response = anthropic.messages.create(
    model="claude-sonnet-4-5",
    messages=[...],
    stream=True  # ストリーミング有効
)
```

## Messages API

### 🔥 基本的な使用法

#### Python SDK
```python
import anthropic

client = anthropic.Anthropic(
    api_key="your-api-key"
)

response = client.messages.create(
    model="claude-sonnet-4-5",
    max_tokens=1000,
    messages=[
        {
            "role": "user",
            "content": "Pythonで簡単なWebAPIを作る方法を教えて"
        }
    ]
)

print(response.content[0].text)
```

#### TypeScript SDK
```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
});

const response = await anthropic.messages.create({
    model: 'claude-sonnet-4-5',
    max_tokens: 1000,
    messages: [{
        role: 'user',
        content: 'React で TODO アプリを作る方法は？'
    }]
});

console.log(response.content[0].text);
```

### ⚙️ 高度な設定

#### System Message 使用
```python
response = client.messages.create(
    model="claude-sonnet-4-5",
    max_tokens=2000,
    system="""あなたは経験豊富なPythonエンジニアです。
    常にPEP8準拠で、セキュアなコードを提供してください。""",
    messages=[
        {
            "role": "user", 
            "content": "Flask で認証付きAPIを作って"
        }
    ]
)
```

#### 複数メッセージの会話
```python
conversation = [
    {"role": "user", "content": "Pythonで機械学習を始めたい"},
    {"role": "assistant", "content": "scikit-learn から始めることをお勧めします..."},
    {"role": "user", "content": "具体的なコード例を見せて"}
]

response = client.messages.create(
    model="claude-sonnet-4-5",
    max_tokens=2000,
    messages=conversation
)
```

## Extended Thinking API

**Extended Thinking** は複雑な推論タスクに特化した機能です。

### 🧠 基本実装

```python
response = client.messages.create(
    model="claude-opus-4-6",  # Extended Thinking はOpus推奨
    max_tokens=4000,
    messages=[
        {
            "role": "user",
            "content": """複雑な経営判断について深く分析してください。
            
            <situation>
            SaaS企業で、新機能開発 vs 既存機能改善の
            リソース配分を決める必要がある
            </situation>
            
            <constraints>
            - 開発チーム10名
            - 予算制約あり
            - 競合他社の動向を考慮
            - ROI の最大化が必要
            </constraints>
            
            段階的に分析して最適解を提案してください。"""
        }
    ],
    # Extended Thinking 有効化
    beta_headers={"anthropic-version": "extended-thinking-2025-11-20"}
)
```

### 💭 思考プロセスの取得

```python
# Extended Thinking の思考プロセスも取得
for content_block in response.content:
    if content_block.type == "thinking":
        print("思考プロセス:", content_block.text)
    elif content_block.type == "text":
        print("最終回答:", content_block.text)
```

## Tool Use（Function Calling）

**Tool Use** は外部システムとの連携機能です。

### 🔧 基本的なツール定義

```python
tools = [
    {
        "name": "get_weather",
        "description": "指定された都市の現在の天気を取得",
        "input_schema": {
            "type": "object",
            "properties": {
                "city": {
                    "type": "string",
                    "description": "都市名"
                }
            },
            "required": ["city"]
        }
    }
]

response = client.messages.create(
    model="claude-sonnet-4-5",
    max_tokens=1000,
    tools=tools,
    messages=[
        {
            "role": "user",
            "content": "東京の天気を調べて"
        }
    ]
)
```

### 🛠️ ツール実行の処理

```python
def execute_tool(tool_name, tool_input):
    if tool_name == "get_weather":
        # 実際の天気API呼び出し
        return f"{tool_input['city']}の天気は晴れです"
    
def handle_tool_use(response):
    if response.stop_reason == "tool_use":
        tool_call = response.content[-1]
        
        # ツールを実行
        result = execute_tool(
            tool_call.name, 
            tool_call.input
        )
        
        # 結果をClaudeに送信
        follow_up = client.messages.create(
            model="claude-sonnet-4-5",
            max_tokens=1000,
            tools=tools,
            messages=[
                {"role": "user", "content": "東京の天気を調べて"},
                {"role": "assistant", "content": response.content},
                {
                    "role": "user",
                    "content": [
                        {
                            "type": "tool_result",
                            "tool_use_id": tool_call.id,
                            "content": result
                        }
                    ]
                }
            ]
        )
        
        return follow_up.content[0].text
```

### 🚀 高度なツール例

#### データベース連携ツール
```python
database_tools = [
    {
        "name": "execute_sql",
        "description": "SQLクエリを実行してデータを取得",
        "input_schema": {
            "type": "object",
            "properties": {
                "query": {"type": "string", "description": "SQLクエリ"},
                "parameters": {"type": "array", "description": "パラメータ"}
            },
            "required": ["query"]
        }
    },
    {
        "name": "analyze_data",
        "description": "データの統計分析を実行",
        "input_schema": {
            "type": "object",
            "properties": {
                "data": {"type": "array", "description": "分析対象データ"},
                "analysis_type": {
                    "type": "string", 
                    "enum": ["correlation", "regression", "clustering"]
                }
            },
            "required": ["data", "analysis_type"]
        }
    }
]
```

## Vision API

**Vision API** は画像解析機能を提供します。

### 👁️ 基本的な画像分析

```python
import base64

# 画像をBase64エンコード
def encode_image(image_path):
    with open(image_path, "rb") as image_file:
        return base64.b64encode(image_file.read()).decode('utf-8')

image_data = encode_image("chart.png")

response = client.messages.create(
    model="claude-sonnet-4-5",
    max_tokens=1500,
    messages=[
        {
            "role": "user",
            "content": [
                {
                    "type": "text",
                    "text": "この図表を分析して、データを表形式で抽出してください"
                },
                {
                    "type": "image",
                    "source": {
                        "type": "base64",
                        "media_type": "image/png",
                        "data": image_data
                    }
                }
            ]
        }
    ]
)
```

### 📊 UI スクリーンショット分析

```python
def analyze_ui_screenshot(image_path, analysis_type):
    image_data = encode_image(image_path)
    
    prompts = {
        "usability": "このUIの使いやすさを分析し、改善提案を提供してください",
        "accessibility": "アクセシビリティの観点から問題点を指摘してください",
        "design": "デザインの一貫性とビジュアル階層を評価してください"
    }
    
    response = client.messages.create(
        model="claude-sonnet-4-5",
        max_tokens=2000,
        messages=[
            {
                "role": "user",
                "content": [
                    {"type": "text", "text": prompts[analysis_type]},
                    {
                        "type": "image",
                        "source": {
                            "type": "base64",
                            "media_type": "image/png",
                            "data": image_data
                        }
                    }
                ]
            }
        ]
    )
    
    return response.content[0].text
```

## Streaming

**ストリーミング** でリアルタイム応答を実現します。

### ⚡ 基本的なストリーミング

```python
def stream_response(prompt):
    stream = client.messages.create(
        model="claude-sonnet-4-5",
        max_tokens=1000,
        messages=[{"role": "user", "content": prompt}],
        stream=True
    )
    
    for event in stream:
        if event.type == "content_block_delta":
            print(event.delta.text, end="", flush=True)
        elif event.type == "message_stop":
            print("\n[Complete]")

# 使用例
stream_response("Pythonでアプリを作る手順を教えて")
```

### 🔄 非同期ストリーミング

```python
import asyncio
from anthropic import AsyncAnthropic

async def async_stream_chat(messages):
    client = AsyncAnthropic(api_key="your-key")
    
    async with client.messages.stream(
        model="claude-sonnet-4-5",
        max_tokens=1000,
        messages=messages
    ) as stream:
        async for event in stream:
            if event.type == "content_block_delta":
                yield event.delta.text

# Webアプリでの使用例（FastAPI）
from fastapi import FastAPI
from fastapi.responses import StreamingResponse

app = FastAPI()

@app.post("/chat/stream")
async def stream_chat(request):
    return StreamingResponse(
        async_stream_chat(request.messages),
        media_type="text/plain"
    )
```

## Batches API

**Batches API** で大量処理を効率化します。

### 📦 バッチ処理の実装

```python
# 大量のタスクを一括処理
batch_requests = [
    {
        "custom_id": f"request_{i}",
        "method": "POST",
        "url": "/v1/messages",
        "body": {
            "model": "claude-haiku-4-5",
            "max_tokens": 500,
            "messages": [
                {"role": "user", "content": f"商品{i}のレビューを要約して"}
            ]
        }
    }
    for i in range(1000)  # 1000件の処理
]

# バッチ作成
batch = client.batches.create(
    input_file_id=upload_batch_file(batch_requests),
    endpoint="/v1/messages",
    completion_window="24h"
)

# 結果取得
results = client.batches.retrieve(batch.id)
```

## Prompt Caching

**Prompt Caching** でコストを大幅削減します。

### 💾 キャッシュ活用

```python
# 長いシステムプロンプトをキャッシュ化
long_system_prompt = """
長い技術仕様書やガイドライン...
（数千文字の内容）
"""

response = client.messages.create(
    model="claude-sonnet-4-5",
    max_tokens=1000,
    system=[
        {
            "type": "text",
            "text": long_system_prompt,
            "cache_control": {"type": "ephemeral"}  # キャッシュ指定
        }
    ],
    messages=[
        {"role": "user", "content": "この仕様に基づいてコードを作成して"}
    ]
)

# 以降の同じシステムプロンプトを使った呼び出しは
# キャッシュから取得されコストが削減される
```

## OpenAI API との比較・移行ガイド

### 🔄 API 構造比較

| 項目 | OpenAI | Anthropic |
|------|--------|-----------|
| **認証** | `Authorization: Bearer` | `x-api-key: ` |
| **モデル指定** | `gpt-4o` | `claude-sonnet-4-5` |
| **システムメッセージ** | messages配列内 | 専用systemパラメータ |
| **ストリーミング** | `stream: true` | `stream: true` |

### 📝 移行例

#### OpenAI → Anthropic

**OpenAI版**:
```python
import openai

response = openai.ChatCompletion.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": "あなたはPythonの専門家です"},
        {"role": "user", "content": "FastAPIの使い方を教えて"}
    ],
    max_tokens=1000,
    stream=True
)
```

**Anthropic版**:
```python
import anthropic

client = anthropic.Anthropic(api_key="your-key")

response = client.messages.create(
    model="claude-sonnet-4-5",
    system="あなたはPythonの専門家です",
    messages=[
        {"role": "user", "content": "FastAPIの使い方を教えて"}
    ],
    max_tokens=1000,
    stream=True
)
```

### 🚀 実践的な移行戦略

1. **段階的移行**: 新機能から Claude API を使用
2. **A/B テスト**: 品質と性能を比較
3. **コスト分析**: 使用量ベースでコスト比較
4. **機能活用**: Tool Use、Extended Thinking 等の独自機能を活用

---

!!! tip "API活用のコツ"
    **適切なモデル選択** + **Caching活用** + **ストリーミング** = 高品質・低コスト・高速

!!! success "Claude API の優位性"
    長文コンテキスト、Tool Use、Extended Thinking など、
    ChatGPT API にはない独自機能で差別化可能です。