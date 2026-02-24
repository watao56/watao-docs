# 🔥 ChallengePods

- カテゴリ: ソーシャル/コミュニティ（7日チャレンジ運営）
- 目標: 月$20
- 想定評価: A

## 概要
インフルエンサーや講師が「7日チャレンジ」を作り、参加者が毎日成果を投稿。AIが毎日お題・フィードバックを自動生成する小規模コミュニティ運営SaaS。

## 海外事例分析
- Geneva/Discordの有料コミュニティで短期チャレンジが高エンゲージ。
- Circle.so上でも「cohort型」課金が伸長。
- 日本は運営工数が重く、継続困難なコミュニティが多い。

## ターゲット
- Xで発信する個人クリエイター
- オンライン講師/コーチ
- 小規模サロン運営者

## 料金
- Host $8/月: チャレンジ3本
- Pro Host $15/月: 無制限+参加分析
- 参加者課金はStripe Connectで任意

## ユーザーフロー
1) 主催者がテーマ作成
2) AIが7日分のお題生成
3) 参加者が投稿
4) AIが称賛/改善コメント
5) 完走バッジ配布

## デザインコンセプト
「Campfire UI」。温かいオレンジ系、日次進捗リング、完走時の演出アニメ。

## アーキテクチャ
Next.js + Supabase Realtime。投稿解析はGPT-4o-mini。通知はDiscord webhook。

## DB設計
hosts(id,plan)
challenges(id,host_id,title,start_date,price)
participants(id,challenge_id,user_id,status)
posts(id,participant_id,day,content,ai_feedback)

## コスト見積もり（月次）
- Vercel/Render無料枠 + Supabase $0〜$5
- AI推論: $3
- 合計: $8以下

## MVPスコープ（14日）
- 主催者作成
- 7日テンプレ
- 投稿/リアクション
- AIフィードバック
- 完走証明画像

## マーケ計画
- 「7日で1作品作る」無料チャレンジを自社開催
- 主催者向けテンプレ配布
- 完走証明のSNSシェア導線

## 技術スタック
Next.js, Supabase, OpenAI mini, Stripe, Discord API

## リスク
荒らし投稿 → 招待制/NGワード。AIコメント品質ぶれ → 固定プロンプト+手動修正UI。

## 競合分析
Discord単体運用は管理が重い。ChallengePodsは課金/進捗/完走演出がワンセット。

## $20達成シナリオ
Hostプラン3人で$24達成。コミュニティ運営者はLTV高く継続率も高い。

## ユニットエコノミクス
ARPU $8、変動費$0.5、粗利93.7%、チャーン10%でも黒字維持。
