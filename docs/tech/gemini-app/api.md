# API活用ガイド

Gemini APIを活用することで、自社システムやアプリケーションに高性能AIを組み込むことができます。Google AI StudioとVertex AIの2つの方法があり、それぞれ異なる特徴と価格体系を持ちます。

## Google AI Studio 概要

!!! info "Google AI Studio とは"
    Google AI Studioは、Gemini APIを簡単にテスト・プロトタイプできるWeb開発環境です。コーディング不要でプロンプトをテストし、そのままAPI化できます。

**主な特徴**:
- **無料で利用開始可能**
- **ブラウザベースの開発環境**
- **プロンプト設計支援**
- **リアルタイムテスト**
- **自動コード生成**（Python, Node.js等）

### アクセス方法

1. [aistudio.google.com](https://aistudio.google.com) にアクセス
2. Googleアカウントでサインイン
3. 利用規約に同意
4. 無料枠内で即座に利用開始

## Gemini API 仕様

### 利用可能モデル（2026年2月現在）

| モデル | 特徴 | 用途 | コンテキスト長 |
|--------|------|------|---------------|
| **Gemini 3 Pro** | 最高性能 | 複雑なタスク | 200万トークン |
| **Gemini 2.5 Pro** | 高性能・マルチモーダル | 汎用的な業務 | 100万トークン |
| **Gemini 2.5 Flash** | 高速・効率的 | リアルタイム処理 | 100万トークン |
| **Gemini 2.5 Flash Lite** | 軽量・低コスト | 大量処理 | 100万トークン |

### 料金体系（Google AI Studio）

!!! note "無料枠"
    Google AI Studioでは、以下の無料枠があります：
    
    **Gemini 2.5 Flash**:
    - 入力: 150万トークン/月
    - 出力: 150万トークン/月
    
    **Gemini 2.5 Pro（制限付き）**:
    - 1日50プロンプト
    - 3時間に15プロンプト（バースト制限）

**有料料金（超過分）**:

```
Gemini 2.5 Flash:
- 入力: $0.075 / 100万トークン
- 出力: $0.30 / 100万トークン

Gemini 2.5 Pro:
- 入力: $1.25 / 100万トークン
- 出力: $5.00 / 100万トークン

Gemini 3 Pro:
- 入力: $2.50 / 100万トークン
- 出力: $10.00 / 100万トークン
```

### レート制限

| モデル | プロンプト/分 | トークン/分 |
|--------|--------------|------------|
| Flash | 1,000 | 400万 |
| Pro | 360 | 120万 |
| 3 Pro | 180 | 80万 |

## Vertex AI での利用

!!! info "Vertex AI とは"
    Google CloudのエンタープライズAIプラットフォーム。企業レベルのセキュリティ、スケーラビリティ、SLA を提供します。

**Google AI Studio との違い**:

| 項目 | Google AI Studio | Vertex AI |
|------|-----------------|-----------|
| **対象** | 開発者・個人 | 企業・本格運用 |
| **セキュリティ** | 標準 | エンタープライズ級 |
| **SLA** | なし | 99.5%〜99.9% |
| **データ保持** | Google管理 | 顧客管理 |
| **料金** | 従量課金のみ | 従量+固定プラン |
| **サポート** | コミュニティ | 企業サポート |

**Vertex AI 料金例**（一部）:
```
Gemini 2.5 Pro（Vertex AI）:
- 入力: $1.25 / 100万トークン
- 出力: $5.00 / 100万トークン
- 最小利用料金: 月額$0（従量課金）

エンタープライズ機能:
- Private Service Connect: $0.01/時間
- Customer Managed Encryption Keys (CMEK): 追加料金なし
```

## マルチモーダル API

### 画像処理

**対応形式**: JPEG, PNG, GIF, WebP
**最大サイズ**: 20MB
**最大解像度**: 3072 x 3072

```python
import google.generativeai as genai

genai.configure(api_key="YOUR_API_KEY")

model = genai.GenerativeModel('gemini-2.5-pro')

# 画像をアップロード
with open('image.jpg', 'rb') as f:
    image_data = f.read()

response = model.generate_content([
    "この画像の内容を詳しく説明してください。",
    {"mime_type": "image/jpeg", "data": image_data}
])

print(response.text)
```

### 動画処理

**対応形式**: MP4, MOV, AVI, FLV, MKV, WebM
**最大サイズ**: 2GB
**最大長**: 10時間

```python
# 動画ファイルのアップロード
video_file = genai.upload_file(path="meeting.mp4")

# 処理待ち
import time
while video_file.state.name == "PROCESSING":
    print("動画を処理中...")
    time.sleep(5)
    video_file = genai.get_file(video_file.name)

# 動画分析
response = model.generate_content([
    "この会議動画から主要な決定事項とアクションアイテムを抽出してください。",
    video_file
])

print(response.text)
```

### PDF・文書処理

```python
# PDFファイルの処理
pdf_file = genai.upload_file(path="report.pdf")

response = model.generate_content([
    "このレポートの要点を3つのポイントでまとめてください。",
    pdf_file
])

print(response.text)
```

## Function Calling

!!! tip "Function Calling とは"
    Gemini APIが外部関数を呼び出すことで、リアルタイムデータの取得や外部システムとの連携が可能になる機能です。

### 基本的な実装

```python
def get_weather(location: str) -> dict:
    """指定した地域の天気情報を取得"""
    # 実際のAPI呼び出し（例：OpenWeatherMap）
    return {
        "location": location,
        "temperature": "22°C",
        "condition": "晴れ",
        "humidity": "65%"
    }

# 関数の定義
weather_tool = genai.protos.Tool(
    function_declarations=[
        genai.protos.FunctionDeclaration(
            name="get_weather",
            description="指定した地域の現在の天気情報を取得します",
            parameters=genai.protos.Schema(
                type=genai.protos.Type.OBJECT,
                properties={
                    "location": genai.protos.Schema(
                        type=genai.protos.Type.STRING,
                        description="天気を調べたい地域名"
                    )
                },
                required=["location"]
            )
        )
    ]
)

# モデル設定
model = genai.GenerativeModel(
    'gemini-2.5-pro',
    tools=[weather_tool]
)

# 実行
chat = model.start_chat()
response = chat.send_message("東京の今日の天気はどうですか？")

# Function Callingの処理
for part in response.parts:
    if fn := part.function_call:
        args = dict(fn.args)
        result = get_weather(**args)
        
        # 結果を返送
        response = chat.send_message(
            genai.protos.Part(
                function_response=genai.protos.FunctionResponse(
                    name=fn.name,
                    response={"result": result}
                )
            )
        )
        
print(response.text)
```

### 高度な例：データベース連携

```python
import sqlite3

def search_products(category: str, price_max: int) -> list:
    """商品データベースから商品を検索"""
    conn = sqlite3.connect('products.db')
    cursor = conn.cursor()
    
    query = """
    SELECT name, price, description 
    FROM products 
    WHERE category = ? AND price <= ?
    ORDER BY price DESC
    LIMIT 5
    """
    
    cursor.execute(query, (category, price_max))
    results = cursor.fetchall()
    conn.close()
    
    return [
        {"name": row[0], "price": row[1], "description": row[2]}
        for row in results
    ]

# ツール定義
search_tool = genai.protos.Tool(
    function_declarations=[
        genai.protos.FunctionDeclaration(
            name="search_products",
            description="指定した条件で商品データベースを検索",
            parameters=genai.protos.Schema(
                type=genai.protos.Type.OBJECT,
                properties={
                    "category": genai.protos.Schema(
                        type=genai.protos.Type.STRING,
                        description="商品カテゴリ（electronics, books, clothing等）"
                    ),
                    "price_max": genai.protos.Schema(
                        type=genai.protos.Type.INTEGER,
                        description="最大予算（円）"
                    )
                },
                required=["category", "price_max"]
            )
        )
    ]
)
```

## コンテキストキャッシング

!!! info "コンテキストキャッシングとは"
    長い文書やデータを事前にキャッシュすることで、繰り返し使用時のコストと応答時間を削減する機能です。

**利用シーン**:
- 大きなマニュアル・仕様書の分析
- 長時間の会議録音の処理
- 大量のログファイル分析

```python
# 大きなファイルをキャッシュ
document = genai.upload_file(path="large_manual.pdf")

# キャッシュ作成
cache = genai.caching.CachedContent.create(
    model='gemini-2.5-pro',
    contents=[document],
    ttl=datetime.timedelta(hours=1),  # 1時間キャッシュ
)

# キャッシュを使用してモデル作成
model = genai.GenerativeModel.from_cached_content(cached_content=cache)

# 高速な応答（キャッシュ済みデータを利用）
response = model.generate_content("このマニュアルの安全に関する章の要点は？")
```

**コスト削減効果**:
```
通常の処理: 1M トークン × $1.25 = $1.25
キャッシュ利用: キャッシュ作成 $0.50 + 継続利用 $0.25/時間
→ 5回以上使用で元が取れる
```

## 実践的なコード例

### Python による REST API 実装

```python
import os
import requests
from flask import Flask, request, jsonify

app = Flask(__name__)

GEMINI_API_KEY = os.getenv('GEMINI_API_KEY')
GEMINI_API_URL = 'https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-pro:generateContent'

@app.route('/api/analyze', methods=['POST'])
def analyze_text():
    data = request.json
    text = data.get('text', '')
    task = data.get('task', 'summarize')
    
    # プロンプトのカスタマイズ
    prompts = {
        'summarize': f'以下のテキストを3つのポイントで要約してください:\n{text}',
        'translate': f'以下のテキストを自然な日本語に翻訳してください:\n{text}',
        'sentiment': f'以下のテキストの感情分析を行い、ポジティブ/ネガティブ/ニュートラルで分類してください:\n{text}'
    }
    
    payload = {
        'contents': [{
            'parts': [{'text': prompts.get(task, text)}]
        }]
    }
    
    headers = {
        'Content-Type': 'application/json',
        'x-goog-api-key': GEMINI_API_KEY
    }
    
    try:
        response = requests.post(GEMINI_API_URL, json=payload, headers=headers)
        response.raise_for_status()
        
        result = response.json()
        generated_text = result['candidates'][0]['content']['parts'][0]['text']
        
        return jsonify({
            'success': True,
            'result': generated_text,
            'task': task
        })
        
    except Exception as e:
        return jsonify({
            'success': False,
            'error': str(e)
        }), 500

if __name__ == '__main__':
    app.run(debug=True)
```

### Node.js によるストリーミング実装

```javascript
const { GoogleGenerativeAI } = require('@google/generative-ai');

const genAI = new GoogleGenerativeAI(process.env.GEMINI_API_KEY);

async function streamingChat(userMessage, conversationHistory = []) {
    const model = genAI.getGenerativeModel({ model: 'gemini-2.5-pro' });
    
    // 会話履歴を含むプロンプト構築
    const fullPrompt = [
        ...conversationHistory.map(msg => ({
            role: msg.role,
            parts: [{ text: msg.content }]
        })),
        {
            role: 'user',
            parts: [{ text: userMessage }]
        }
    ];
    
    const chat = model.startChat({
        history: fullPrompt.slice(0, -1)
    });
    
    const result = await chat.sendMessageStream(userMessage);
    
    // ストリーミングレスポンス
    let fullResponse = '';
    for await (const chunk of result.stream) {
        const chunkText = chunk.text();
        fullResponse += chunkText;
        
        // WebSocketやSSEでリアルタイム送信
        console.log('Chunk:', chunkText);
    }
    
    return fullResponse;
}

// 使用例
(async () => {
    const conversation = [
        { role: 'user', content: 'Python の基本を教えて' },
        { role: 'assistant', content: 'Pythonは...' }
    ];
    
    const response = await streamingChat(
        '具体的なコード例も含めて説明して',
        conversation
    );
    
    console.log('Full response:', response);
})();
```

## OpenAI API との比較

### 主な違い

| 項目 | Gemini API | OpenAI API |
|------|------------|------------|
| **モデル種類** | Pro/Flash/Lite系列 | GPT-4o/4o-mini系列 |
| **コンテキスト長** | 最大200万トークン | 最大128Kトークン |
| **マルチモーダル** | ✅ 画像/動画/PDF/音声 | ✅ 画像/音声のみ |
| **無料枠** | ✅ 月150万トークン | ❌ なし |
| **料金** | Flash: $0.075/1M | 4o-mini: $0.15/1M |
| **Function Calling** | ✅ 対応 | ✅ 対応 |
| **ストリーミング** | ✅ 対応 | ✅ 対応 |
| **ファインチューニング** | 🔄 開発中 | ✅ 対応 |

### 移行のポイント

**OpenAI → Gemini 移行時の考慮事項**:

```python
# OpenAI形式
openai_messages = [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "Hello!"}
]

# Gemini形式への変換
def convert_to_gemini_format(openai_messages):
    gemini_contents = []
    system_instruction = None
    
    for msg in openai_messages:
        if msg["role"] == "system":
            system_instruction = msg["content"]
        else:
            gemini_contents.append({
                "role": "user" if msg["role"] == "user" else "model",
                "parts": [{"text": msg["content"]}]
            })
    
    return gemini_contents, system_instruction

# 使用例
contents, system = convert_to_gemini_format(openai_messages)
model = genai.GenerativeModel(
    'gemini-2.5-pro',
    system_instruction=system
)
```

## セキュリティとベストプラクティス

### API キーの管理

```bash
# 環境変数での管理
export GEMINI_API_KEY="your-api-key-here"

# .env ファイル使用
echo "GEMINI_API_KEY=your-api-key-here" >> .env

# クラウドシークレット管理（Google Secret Manager）
gcloud secrets create gemini-api-key --data-file=api-key.txt
```

### レート制限対応

```python
import time
from typing import Callable

def with_retry(max_retries=3, delay=1):
    def decorator(func: Callable):
        def wrapper(*args, **kwargs):
            for attempt in range(max_retries):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if "rate limit" in str(e).lower() and attempt < max_retries - 1:
                        time.sleep(delay * (2 ** attempt))  # Exponential backoff
                        continue
                    raise e
            return None
        return wrapper
    return decorator

@with_retry(max_retries=3)
def call_gemini_api(prompt):
    # API呼び出し
    pass
```

### エラーハンドリング

```python
def safe_generate_content(model, prompt):
    try:
        response = model.generate_content(prompt)
        
        # コンテンツフィルタリングチェック
        if response.prompt_feedback.block_reason:
            return {
                'success': False,
                'error': 'Content was blocked',
                'reason': response.prompt_feedback.block_reason
            }
        
        return {
            'success': True,
            'content': response.text,
            'usage': response.usage_metadata
        }
        
    except Exception as e:
        return {
            'success': False,
            'error': str(e),
            'error_type': type(e).__name__
        }
```

## 次のステップ

- [Workspace連携](workspace.md) - GoogleサービスとAPI連携
- [vs-Claude](vs-claude.md) - API選択の判断基準
- [Gems](gems.md) - API機能をGems で活用