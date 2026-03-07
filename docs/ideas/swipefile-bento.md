# 📦 SwipeFile Bento

## 概要
広告・LP・SNS投稿のURLを投げるだけで、**フック/構成/CTAを分解してカード化**するマーケ向けマイクロSaaS。海外の"swipe file"文化を日本語UIで提供。

## 海外事例分析
- **Milled / Foreplay**: 広告収集
- **Taplio**: 投稿パターン活用
- 日本ギャップ: 保存はできても、再利用しやすい分解が弱い

## ターゲット
- 1人マーケ担当
- 受託でLP/広告運用するフリーランス
- SNS運用代行

## 料金
- Solo: $8/月
- Studio: $16/月（共有ワークスペース）

## ユーザーフロー
1. URL貼り付け
2. AIがHook/Body/CTA/Visualを抽出
3. "Bento Board"に自動配置
4. 類似案を1クリック生成

## デザインコンセプト
- "Bento Grid"
- 情報を小箱で管理、ドラッグで再構成
- Notion風だがより視覚重視

## アーキテクチャ
- Front: Next.js
- Backend: FastAPI
- Scraping: Browserless + readability
- AI: GPT-5.3 mini
- DB: PostgreSQL (Neon)

## DB設計
- users(id, email, plan)
- boards(id, user_id, title)
- cards(id, board_id, url, hook, body, cta, visual_note)
- generations(id, card_id, variant_text, created_at)
- team_members(id, board_id, user_id, role)

## コスト見積もり（月）
- Vercel: $0〜10
- Neon: $0
- Browserless: $5
- AI API: $4
- 合計: **$9〜19**

## MVPスコープ
- URL解析
- 4分解カード
- 類似案3本生成
- CSVエクスポート

## マーケ計画
- Xで"勝ち広告分解"を毎日投稿
- マーケDiscordでテンプレ配布
- 既存受託クライアントに導入

## 技術スタック
Next.js / FastAPI / Neon / OpenAI mini / Stripe

## リスク
- スクレイピング失敗率
- 利用規約順守（robots, terms）

## 競合分析
- Notion: 手作業整理
- Foreplay: 日本語分解と再生成が弱い
- 差別化: 日本語コピー分解＋即再利用

## $20達成シナリオ
- Solo($8)×1 + Studio($16)×1 = $24

## ユニットエコノミクス
- ARPU: $10.6
- COGS/user: $1.7
- 粗利: 84%
