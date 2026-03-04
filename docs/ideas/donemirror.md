# ✅ DoneMirror

## 概要
ToDoではなく**Done（やったこと）**を蓄積し、1日の成果を自動で「見せられる1枚」に変換する生産性ツール。達成感を先に設計して継続率を上げる。

## 海外事例分析
- Done listトレンド（Zapier/Redditなど）: 未完了ストレス対策として拡大
- Sunsama/Notion: 計画側は強いが“完了可視化”は弱い
- BeReal文化: 日次の軽い投稿が継続を生む

## ターゲット
- タスク管理で疲弊しがちな個人
- 勉強/副業の継続勢
- ADHD傾向のセルフマネジメント層

## 料金
- Free: 1日5件記録
- Pro: $4/月（無制限、週次レポート）
- Buddy: $8/月（相互応援機能）

## ユーザーフロー
1. 完了した行動を1タップ記録
2. AIがカテゴリ自動整理
3. 夜に成果カードを自動生成
4. 壁紙/共有画像として保存

## デザインコンセプト
「**達成のスクラップブック**」。柔らかいグラデーションとステッカーUI。

## アーキテクチャ
- Next.js PWA
- API: AWS Lambda + API Gateway
- DB: DynamoDB
- 画像生成: Satori + Resvg（AI非依存）
- 要約: OpenAI mini

## DB設計
- users(id, timezone, plan)
- done_logs(id, user_id, text, category, done_at)
- daily_cards(id, user_id, date, image_url, summary)
- buddies(id, user_id, peer_user_id, status)

## コスト見積もり（月）
- AWS Free Tier中心: $0〜$4
- AI要約: $1〜$3
- 合計: **$1〜$7**

## MVPスコープ
- Done入力
- 日次カード自動生成
- 週次サマリーメール

## マーケ計画
- #今日やったこと テンプレ配布
- 学習コミュニティとの提携
- 連続記録ステッカーのSNS配布

## 技術スタック
Next.js / AWS Lambda / DynamoDB / SES / OpenAI mini

## リスク
- 3日坊主 → buddy機能で継続誘導
- 記録の手間 → 音声入力とテンプレ短文化

## 競合分析
通常ToDoは「未完了の圧」が強い。DoneMirrorは**達成感の即時報酬**で差別化。

## $20達成シナリオ
Pro($4)×5人で$20達成。

## ユニットエコノミクス
- ARPU: $4
- 変動費: $0.25/人
- 粗利: 約94%
