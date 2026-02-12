# API活用ガイド

OpenAI APIを使用することで、ChatGPTの機能をアプリケーションやサービスに統合できます。本ページでは、2026年2月現在の最新API仕様と実践的な活用方法を詳しく解説します。

!!! warning "重要な変更"
    2025年8月にAssistants APIの廃止が発表されました。2026年8月26日にサービス終了予定のため、新規開発ではResponses APIの利用を推奨します。

## OpenAI API概要

### 料金体系（2026年2月現在）

#### 主要モデルの料金

| モデル | 入力（1Mトークン） | 出力（1Mトークン） | 用途 |
|--------|-------------------|-------------------|------|
| GPT-5.2 | $15.00 | $45.00 | 最高品質・汎用 |
| GPT-4o | $2.50 | $10.00 | 高性能・マルチモーダル |
| GPT-4o mini | $0.15 | $0.60 | 軽量・高速 |
| o3 | $30.00 | $120.00 | 高度な推論 |
| o3 mini | $3.00 | $12.00 | 推論・低コスト |

#### 従量課金の仕組み
- **トークンベース**: 入力と出力のトークン数で課金
- **1トークン**: 約0.75単語（英語）、1文字（日本語目安）
- **最小課金**: $5から開始
- **プリペイド**: クレジット購入式

### レート制限

#### Tier別制限（月額利用額による）

| Tier | 条件 | RPM制限 | TPM制限 |
|------|------|---------|---------|
| Free | 無料枠 | 3 | 40K |
| Tier 1 | $5以上 | 3,500 | 10M |
| Tier 2 | $50以上 | 5,000 | 30M |
| Tier 3 | $500以上 | 7,000 | 60M |
| Tier 4 | $5,000以上 | 10,000 | 100M |

*RPM: Requests Per Minute, TPM: Tokens Per Minute*

## Chat Completions API

### 基本的な使用方法

最も基本的で重要なAPIエンドポイントです。

#### Python例

```python
import openai
import os

# APIキー設定
openai.api_key = os.getenv("OPENAI_API_KEY")

def simple_chat_completion():
    response = openai.chat.completions.create(
        model="gpt-4o",
        messages=[
            {"role": "system", "content": "あなたは親切なアシスタントです。"},
            {"role": "user", "content": "Pythonでファイルを読み書きする方法を教えて"}
        ],
        max_tokens=500,
        temperature=0.7
    )
    
    return response.choices[0].message.content

# 実行
result = simple_chat_completion()
print(result)
```

#### Node.js例

```javascript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function simpleChatCompletion() {
  try {
    const response = await openai.chat.completions.create({
      model: "gpt-4o",
      messages: [
        {"role": "system", "content": "あなたは親切なアシスタントです。"},
        {"role": "user", "content": "JavaScriptで配列操作をする方法を教えて"}
      ],
      max_tokens: 500,
      temperature: 0.7
    });
    
    return response.choices[0].message.content;
  } catch (error) {
    console.error('API Error:', error);
    throw error;
  }
}

// 実行
simpleChatCompletion()
  .then(result => console.log(result))
  .catch(error => console.error(error));
```

### 高度なパラメータ設定

```python
def advanced_chat_completion():
    response = openai.chat.completions.create(
        model="gpt-4o",
        messages=[
            {"role": "system", "content": "専門的なコードレビュアーとして振る舞ってください。"},
            {"role": "user", "content": "以下のPythonコードをレビューしてください：\n```python\ndef calc(x, y):\n    return x + y\n```"}
        ],
        # 創造性のコントロール
        temperature=0.3,  # 低め = より一貫した出力
        top_p=0.8,        # 確率分布の制限
        
        # 出力制御
        max_tokens=1000,
        presence_penalty=0.0,   # 新しいトピックの促進
        frequency_penalty=0.5,  # 繰り返し抑制
        
        # 応答形式
        response_format={"type": "text"},  # またはjson_object
        
        # 停止条件
        stop=["\n\n", "---"],
        
        # メタデータ
        user="user123"  # ユーザー識別
    )
    
    return response.choices[0].message.content
```

## Responses API（Assistants API後継）

### 基本概念

Responses APIは、より構造化された会話を管理するための新しいAPIです。

#### 主要概念
- **Conversation**: 継続的な会話セッション
- **Response**: 各応答とその設定
- **Tool Use**: 関数呼び出しや外部ツール利用

#### 基本実装

```python
import openai

class ConversationManager:
    def __init__(self, model="gpt-4o"):
        self.client = openai.OpenAI()
        self.model = model
        self.conversation_history = []
    
    def add_message(self, role, content):
        """メッセージを会話履歴に追加"""
        self.conversation_history.append({
            "role": role,
            "content": content
        })
    
    def get_response(self, user_message, tools=None):
        """応答を取得"""
        self.add_message("user", user_message)
        
        response = self.client.chat.completions.create(
            model=self.model,
            messages=self.conversation_history,
            tools=tools,
            tool_choice="auto" if tools else None
        )
        
        assistant_message = response.choices[0].message
        self.add_message("assistant", assistant_message.content)
        
        return assistant_message

# 使用例
manager = ConversationManager()
response = manager.get_response("こんにちは！プログラミングについて教えてください。")
print(response.content)
```

## Function Calling / Tool Use

### 基本的なFunction Calling

```python
def get_weather_tools():
    """天気予報取得用のツール定義"""
    return [
        {
            "type": "function",
            "function": {
                "name": "get_weather",
                "description": "指定した場所の現在の天気を取得します",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "location": {
                            "type": "string",
                            "description": "都市名（例：東京、大阪）"
                        },
                        "unit": {
                            "type": "string",
                            "enum": ["celsius", "fahrenheit"],
                            "description": "温度の単位"
                        }
                    },
                    "required": ["location"]
                }
            }
        }
    ]

def execute_function_call(name, arguments):
    """関数実行のシミュレーション"""
    if name == "get_weather":
        location = arguments["location"]
        unit = arguments.get("unit", "celsius")
        # 実際の実装では天気APIを呼び出し
        return f"{location}の天気は晴れ、気温は22度です。"
    return "関数が見つかりません"

def chat_with_function_calling():
    tools = get_weather_tools()
    
    response = openai.chat.completions.create(
        model="gpt-4o",
        messages=[
            {"role": "user", "content": "東京の天気を教えて"}
        ],
        tools=tools,
        tool_choice="auto"
    )
    
    message = response.choices[0].message
    
    # 関数呼び出しが要求された場合
    if message.tool_calls:
        for tool_call in message.tool_calls:
            function_name = tool_call.function.name
            function_args = json.loads(tool_call.function.arguments)
            
            # 関数実行
            function_result = execute_function_call(function_name, function_args)
            
            # 結果をChatGPTに返す
            follow_up = openai.chat.completions.create(
                model="gpt-4o",
                messages=[
                    {"role": "user", "content": "東京の天気を教えて"},
                    message,
                    {
                        "role": "tool",
                        "tool_call_id": tool_call.id,
                        "content": function_result
                    }
                ],
                tools=tools
            )
            
            return follow_up.choices[0].message.content
    
    return message.content
```

### 複数ツールの活用例

```python
def get_business_tools():
    """ビジネス用ツールセット"""
    return [
        {
            "type": "function",
            "function": {
                "name": "calculate_roi",
                "description": "投資収益率（ROI）を計算します",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "revenue": {"type": "number", "description": "収益"},
                        "cost": {"type": "number", "description": "費用"},
                        "period": {"type": "integer", "description": "期間（月）"}
                    },
                    "required": ["revenue", "cost"]
                }
            }
        },
        {
            "type": "function", 
            "function": {
                "name": "send_email",
                "description": "メールを送信します",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "to": {"type": "string", "description": "宛先メールアドレス"},
                        "subject": {"type": "string", "description": "件名"},
                        "body": {"type": "string", "description": "本文"}
                    },
                    "required": ["to", "subject", "body"]
                }
            }
        },
        {
            "type": "function",
            "function": {
                "name": "query_database",
                "description": "データベースからデータを取得します",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "query": {"type": "string", "description": "SQL クエリ"},
                        "table": {"type": "string", "description": "対象テーブル"}
                    },
                    "required": ["query", "table"]
                }
            }
        }
    ]

class BusinessAssistant:
    def __init__(self):
        self.client = openai.OpenAI()
        self.tools = get_business_tools()
    
    def execute_tool(self, name, arguments):
        """ツール実行"""
        if name == "calculate_roi":
            revenue = arguments["revenue"] 
            cost = arguments["cost"]
            period = arguments.get("period", 12)
            
            roi = ((revenue - cost) / cost) * 100
            monthly_roi = roi / period
            
            return {
                "roi_percent": round(roi, 2),
                "monthly_roi": round(monthly_roi, 2),
                "evaluation": "excellent" if roi > 200 else "good" if roi > 100 else "poor"
            }
        
        elif name == "send_email":
            # 実際の実装ではSMTP送信
            return f"メール送信完了: {arguments['to']}"
        
        elif name == "query_database":
            # 実際の実装ではDB接続
            return f"クエリ実行完了: {arguments['query']}"
        
        return "未対応のツールです"
    
    def process_request(self, user_message):
        """リクエスト処理"""
        response = self.client.chat.completions.create(
            model="gpt-4o",
            messages=[
                {"role": "system", "content": "あなたはビジネス分析の専門家です。必要に応じて利用可能なツールを使用してください。"},
                {"role": "user", "content": user_message}
            ],
            tools=self.tools,
            tool_choice="auto"
        )
        
        message = response.choices[0].message
        
        # ツール呼び出し処理
        if message.tool_calls:
            tool_results = []
            
            for tool_call in message.tool_calls:
                function_name = tool_call.function.name
                function_args = json.loads(tool_call.function.arguments)
                result = self.execute_tool(function_name, function_args)
                
                tool_results.append({
                    "role": "tool",
                    "tool_call_id": tool_call.id,
                    "content": json.dumps(result, ensure_ascii=False)
                })
            
            # 最終回答生成
            final_response = self.client.chat.completions.create(
                model="gpt-4o",
                messages=[
                    {"role": "system", "content": "あなたはビジネス分析の専門家です。"},
                    {"role": "user", "content": user_message},
                    message
                ] + tool_results
            )
            
            return final_response.choices[0].message.content
        
        return message.content

# 使用例
assistant = BusinessAssistant()
result = assistant.process_request(
    "売上1000万円、コスト600万円のプロジェクトのROIを計算して、結果をtakeshi@example.comに送信してください"
)
print(result)
```

## Vision API（画像入力）

### 画像分析の基本

```python
import base64
from pathlib import Path

def encode_image(image_path):
    """画像をbase64エンコード"""
    with open(image_path, "rb") as image_file:
        return base64.b64encode(image_file.read()).decode('utf-8')

def analyze_image(image_path, prompt="この画像について説明してください"):
    """画像分析"""
    base64_image = encode_image(image_path)
    
    response = openai.chat.completions.create(
        model="gpt-4o",  # Vision対応モデル
        messages=[
            {
                "role": "user",
                "content": [
                    {"type": "text", "text": prompt},
                    {
                        "type": "image_url",
                        "image_url": {
                            "url": f"data:image/jpeg;base64,{base64_image}",
                            "detail": "high"  # high, low, auto
                        }
                    }
                ]
            }
        ],
        max_tokens=1000
    )
    
    return response.choices[0].message.content

# 使用例
result = analyze_image(
    "chart.png", 
    "このグラフのトレンドと重要なポイントを分析してください"
)
print(result)
```

### 複数画像の比較分析

```python
def compare_images(image_paths, prompt):
    """複数画像の比較"""
    content = [{"type": "text", "text": prompt}]
    
    for i, path in enumerate(image_paths):
        base64_image = encode_image(path)
        content.append({
            "type": "image_url",
            "image_url": {
                "url": f"data:image/jpeg;base64,{base64_image}",
                "detail": "high"
            }
        })
    
    response = openai.chat.completions.create(
        model="gpt-4o",
        messages=[{"role": "user", "content": content}],
        max_tokens=1500
    )
    
    return response.choices[0].message.content

# 使用例
comparison = compare_images(
    ["before.jpg", "after.jpg"],
    "Before/Afterの画像を比較して、変化点と改善点を詳細に分析してください"
)
print(comparison)
```

## Whisper（音声→テキスト）/ TTS

### Whisper: 音声認識

```python
def transcribe_audio(audio_file_path, language="ja"):
    """音声をテキストに変換"""
    with open(audio_file_path, "rb") as audio_file:
        transcript = openai.audio.transcriptions.create(
            model="whisper-1",
            file=audio_file,
            language=language,  # 言語指定（オプション）
            response_format="verbose_json",  # text, json, verbose_json
            temperature=0.2
        )
    
    return {
        "text": transcript.text,
        "segments": transcript.segments,  # 詳細なタイミング情報
        "duration": transcript.duration
    }

def translate_audio(audio_file_path):
    """音声を英語に翻訳"""
    with open(audio_file_path, "rb") as audio_file:
        translation = openai.audio.translations.create(
            model="whisper-1",
            file=audio_file,
            response_format="text"
        )
    
    return translation

# 使用例
result = transcribe_audio("meeting_recording.mp3", language="ja")
print("Transcript:", result["text"])
print("Duration:", result["duration"])
```

### TTS: テキスト読み上げ

```python
from pathlib import Path

def text_to_speech(text, voice="nova", output_path="output.mp3"):
    """テキストを音声に変換"""
    response = openai.audio.speech.create(
        model="tts-1-hd",  # tts-1 (fast) or tts-1-hd (quality)
        voice=voice,       # alloy, echo, fable, nova, onyx, shimmer
        input=text,
        response_format="mp3",  # mp3, opus, aac, flac
        speed=1.0          # 0.25 ~ 4.0
    )
    
    response.stream_to_file(output_path)
    return output_path

# 使用例
audio_file = text_to_speech(
    "こんにちは。OpenAI APIを使用した音声合成のデモです。",
    voice="nova",
    output_path="demo.mp3"
)
print(f"Audio saved to: {audio_file}")
```

## Embeddings

### ベクトル埋め込みの活用

```python
import numpy as np
from sklearn.metrics.pairwise import cosine_similarity

def get_embedding(text, model="text-embedding-3-large"):
    """テキストのベクトル埋め込みを取得"""
    response = openai.embeddings.create(
        input=text,
        model=model  # text-embedding-3-large, text-embedding-3-small
    )
    return response.data[0].embedding

def calculate_similarity(text1, text2):
    """テキスト間の類似度計算"""
    embed1 = get_embedding(text1)
    embed2 = get_embedding(text2)
    
    similarity = cosine_similarity([embed1], [embed2])[0][0]
    return similarity

class DocumentSearcher:
    def __init__(self):
        self.documents = []
        self.embeddings = []
    
    def add_document(self, text, metadata=None):
        """文書を追加"""
        embedding = get_embedding(text)
        self.documents.append({
            "text": text,
            "metadata": metadata or {},
            "embedding": embedding
        })
    
    def search(self, query, top_k=5):
        """類似文書検索"""
        query_embedding = get_embedding(query)
        
        similarities = []
        for doc in self.documents:
            similarity = cosine_similarity(
                [query_embedding], 
                [doc["embedding"]]
            )[0][0]
            similarities.append(similarity)
        
        # Top-K取得
        top_indices = np.argsort(similarities)[-top_k:][::-1]
        
        results = []
        for idx in top_indices:
            results.append({
                "text": self.documents[idx]["text"],
                "metadata": self.documents[idx]["metadata"],
                "similarity": similarities[idx]
            })
        
        return results

# 使用例
searcher = DocumentSearcher()

# 文書を追加
documents = [
    "Pythonは機械学習に適したプログラミング言語です",
    "JavaScriptはWeb開発でよく使われます",
    "AIモデルの学習にはGPUが必要です",
    "データベースはSQLで操作します"
]

for doc in documents:
    searcher.add_document(doc)

# 検索
results = searcher.search("プログラミング言語について", top_k=2)
for result in results:
    print(f"Similarity: {result['similarity']:.3f} - {result['text']}")
```

## Fine-tuning概要

### Fine-tuningの基本概念

Fine-tuningにより、特定用途に特化したモデルを作成できます。

#### 対象モデル（2026年2月現在）
- **gpt-3.5-turbo**: 基本的なFine-tuning
- **gpt-4**: 高品質なFine-tuning（限定アクセス）
- **babbage-002**: 軽量・高速モデル

#### データ準備

```python
import json

def prepare_training_data():
    """学習データの準備例"""
    training_data = []
    
    # カスタマーサポート用のFine-tuning例
    examples = [
        {
            "messages": [
                {"role": "system", "content": "あなたは親切なカスタマーサポート担当者です。"},
                {"role": "user", "content": "注文をキャンセルしたいです"},
                {"role": "assistant", "content": "承知いたしました。注文番号を教えていただけますでしょうか？"}
            ]
        },
        {
            "messages": [
                {"role": "system", "content": "あなたは親切なカスタマーサポート担当者です。"},
                {"role": "user", "content": "商品が届かないです"},
                {"role": "assistant", "content": "ご心配をおかけして申し訳ございません。配送状況を確認いたします。"}
            ]
        }
    ]
    
    # JSONLファイルとして保存
    with open("training_data.jsonl", "w", encoding="utf-8") as f:
        for example in examples:
            f.write(json.dumps(example, ensure_ascii=False) + "\n")
    
    return "training_data.jsonl"

def create_fine_tuned_model():
    """Fine-tuningモデル作成"""
    # ファイルアップロード
    training_file = openai.files.create(
        file=open("training_data.jsonl", "rb"),
        purpose="fine-tune"
    )
    
    # Fine-tuningジョブ作成
    fine_tune_job = openai.fine_tuning.jobs.create(
        training_file=training_file.id,
        model="gpt-3.5-turbo",
        hyperparameters={
            "n_epochs": 3,
            "batch_size": 1,
            "learning_rate_multiplier": 2
        }
    )
    
    return fine_tune_job.id

# 使用例
prepare_training_data()
job_id = create_fine_tuned_model()
print(f"Fine-tuning job created: {job_id}")
```

## 実践的なコード例

### マルチモーダルChatBot

```python
class MultiModalChatBot:
    def __init__(self, model="gpt-4o"):
        self.client = openai.OpenAI()
        self.model = model
        self.conversation = []
    
    def add_text_message(self, role, content):
        """テキストメッセージ追加"""
        self.conversation.append({
            "role": role,
            "content": content
        })
    
    def add_image_message(self, image_path, prompt):
        """画像付きメッセージ追加"""
        base64_image = self.encode_image(image_path)
        
        self.conversation.append({
            "role": "user",
            "content": [
                {"type": "text", "text": prompt},
                {
                    "type": "image_url",
                    "image_url": {
                        "url": f"data:image/jpeg;base64,{base64_image}"
                    }
                }
            ]
        })
    
    def encode_image(self, image_path):
        """画像エンコード"""
        with open(image_path, "rb") as image_file:
            return base64.b64encode(image_file.read()).decode('utf-8')
    
    def get_response(self):
        """応答取得"""
        response = self.client.chat.completions.create(
            model=self.model,
            messages=self.conversation,
            max_tokens=1000
        )
        
        assistant_message = response.choices[0].message.content
        self.add_text_message("assistant", assistant_message)
        
        return assistant_message
    
    def reset_conversation(self):
        """会話リセット"""
        self.conversation = []

# 使用例
bot = MultiModalChatBot()
bot.add_text_message("system", "あなたは画像分析の専門家です。")
bot.add_text_message("user", "こんにちは！")

response1 = bot.get_response()
print("Bot:", response1)

bot.add_image_message("chart.png", "このグラフを分析してください")
response2 = bot.get_response()
print("Bot:", response2)
```

### APIエラーハンドリング

```python
import time
import random

class OpenAIAPIClient:
    def __init__(self, api_key, max_retries=3):
        self.client = openai.OpenAI(api_key=api_key)
        self.max_retries = max_retries
    
    def completion_with_retry(self, **kwargs):
        """リトライ機能付きAPI呼び出し"""
        for attempt in range(self.max_retries):
            try:
                response = self.client.chat.completions.create(**kwargs)
                return response
            
            except openai.RateLimitError as e:
                if attempt < self.max_retries - 1:
                    wait_time = (2 ** attempt) + random.uniform(0, 1)
                    print(f"Rate limit hit. Waiting {wait_time:.2f} seconds...")
                    time.sleep(wait_time)
                else:
                    raise e
            
            except openai.APITimeoutError as e:
                if attempt < self.max_retries - 1:
                    print(f"Timeout error. Retrying... (attempt {attempt + 1})")
                    time.sleep(1)
                else:
                    raise e
            
            except openai.APIError as e:
                print(f"API Error: {e}")
                raise e
            
            except Exception as e:
                print(f"Unexpected error: {e}")
                raise e
    
    def safe_completion(self, messages, model="gpt-4o", **kwargs):
        """安全なAPI呼び出し"""
        try:
            response = self.completion_with_retry(
                model=model,
                messages=messages,
                **kwargs
            )
            return {
                "success": True,
                "content": response.choices[0].message.content,
                "usage": response.usage
            }
        
        except Exception as e:
            return {
                "success": False,
                "error": str(e),
                "content": None
            }

# 使用例
client = OpenAIAPIClient(api_key=os.getenv("OPENAI_API_KEY"))

result = client.safe_completion(
    messages=[
        {"role": "user", "content": "Hello, how are you?"}
    ],
    temperature=0.7,
    max_tokens=500
)

if result["success"]:
    print("Response:", result["content"])
    print("Token usage:", result["usage"])
else:
    print("Error:", result["error"])
```

## まとめ

OpenAI APIは、強力なAI機能をアプリケーションに統合するための包括的なプラットフォームです。本ガイドで紹介した主要なポイント：

### 技術的ポイント
- **Chat Completions**: 基本的な対話機能
- **Responses API**: 新しい構造化対話管理
- **Function Calling**: 外部システムとの連携
- **マルチモーダル**: 画像・音声の統合処理
- **Embeddings**: 意味検索・類似度分析

### 実装のベストプラクティス
- **エラーハンドリング**: 適切なリトライ機能
- **料金管理**: 使用量の監視と制御
- **セキュリティ**: APIキーの安全な管理
- **パフォーマンス**: 効率的なプロンプト設計

### 今後の展開
- **Assistants API移行**: 2026年8月までにResponses APIへ
- **新モデル**: o系モデルの積極活用
- **マルチモーダル拡張**: より高度な統合機能

OpenAI APIを効果的に活用することで、従来では困難だった高度なAI機能を簡単にアプリケーションに組み込むことができます。適切な設計と実装により、ユーザー体験の大幅な向上が期待できるでしょう。

---

!!! note "関連ページ"
    - [プロンプト設計](prompts.md) - API用プロンプト最適化
    - [カスタムGPTs](custom-gpts.md) - Actions設定との連携
    - [モード活用](modes.md) - 適切なモデル選択指針