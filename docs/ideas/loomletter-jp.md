# 📨 LoomLetter JP

## 概要
英語圏で伸びる「ニュースレター要約 + アクション化」体験を、日本語UIで再設計したパーソナル生産性ツール。受信したニュースレターを**3行サマリー + 今日やる1アクション**に変換し、Notion/Google Calendarへ1クリック反映する。

## 海外事例分析
- **Artifact / Particle系ニュース要約体験**: 情報過多を圧縮するUXが定着
- **Readwise Reader**: ハイライト→行動導線が強い
- **Superhuman**: Inbox処理の時間短縮に課金が成立
- 日本では「英語ニュース要約」はあるが、**日本語ニュースレター + 行動化連携**はまだ薄い。

## ターゲット
- 個人開発者、マーケ担当、情報収集量が多い会社員
- メルマガ購読10本以上の人

## 料金
- Free: 月30通まで要約
- Pro: **$5/月**（無制限、カレンダー連携、週次レポート）

## ユーザーフロー
1. 専用転送アドレスにメルマガ転送
2. 30秒以内に「3行要約 + 次アクション」生成
3. 「今日やる」ボタンでカレンダー登録
4. 週末に「今週の重要トピック」自動配信

## デザインコンセプト
- 「情報ダイエット」テーマ
- 白ベース + 彩度低めのカードUI
- サマリーを名刺サイズカードで共有可能（SNS映え）

## アーキテクチャ
- Next.js (App Router) + Vercel
- API: AWS Lambda (要約ジョブ)
- Queue: SQS
- DB: Supabase Postgres
- Auth: Clerk
- AI: GPT-4o-mini 相当（低トークン）

## DB設計
- users(id, email, plan, created_at)
- inbox_items(id, user_id, source, raw_text, received_at)
- summaries(id, inbox_item_id, short_summary, action_item, confidence)
- exports(id, user_id, target_type, target_id, created_at)
- usage_daily(id, user_id, date, items_processed, token_in, token_out)

## コスト見積もり（月）
- Vercel Hobby: $0
- Supabase Free: $0
- Lambda + SQS: $1
- AI推論: $4（Pro 20人規模）
- 合計: **$5**

## MVPスコープ
- メール転送受信
- 3行要約
- 1アクション抽出
- Google Calendar連携
- Stripe課金

## マーケ計画
- Xで「今週の要約カード」を毎日投稿
- Product Hunt / Indie Hackersで海外文脈を引用
- 日本語メルマガ発行者と相互紹介

## 技術スタック
Next.js / TypeScript / Supabase / Lambda / SQS / Stripe / OpenAI API

## リスク
- メールパース精度
- 要約のハルシネーション
- 対策: ソース引用行を必須表示、再生成ボタン

## 競合分析
- Readwise: 強いが日本語導線が弱い
- メールクライアントAI: 行動化連携が浅い
- 差別化: 日本語最適化 + カレンダー即連携

## $20達成シナリオ
- Pro $5 x 4人 = **$20/月**
- 初月目標: 30トライアル → 15%転換で達成

## ユニットエコノミクス
- ARPU: $5
- 変動費/ユーザー: $0.35
- 粗利: $4.65（93%）
- 回収期間: 初月
