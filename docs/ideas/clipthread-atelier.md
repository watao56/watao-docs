# 🎞️ ClipThread Atelier

## 概要
海外で伸びている「長尺→短尺再編集」需要を、日本向けに**“スレッド映えするカルーセル+短尺動画”**へ特化したAI制作ツール。YouTube URLか原稿を入れると、X/Instagram向けの8枚カルーセルと15秒縦動画を同時生成。

## 海外事例分析
- **OpusClip / Captions**: 長尺動画の切り出し最適化が急伸。
- **Canva Magic Design**: デザインテンプレをAIで即生成する体験が一般化。
- 日本では「動画だけ」か「画像だけ」ツールが多く、**投稿フォーマット横断の同時生成**はまだニッチ。

## ターゲット
- 1人運営の発信者、マイクロインフルエンサー、個人開発者
- 毎日投稿したいが編集工数を抑えたい層

## 料金
- Free: 月5生成（透かしあり）
- Starter: $8/月（100生成、透かしなし）
- Solo Pro: $15/月（300生成、ブランドキット）

## ユーザーフロー
1. URL/テキスト入力
2. トーン選択（Sharp / Warm / Bold）
3. カルーセル8枚＋縦動画1本を自動生成
4. 1クリックでX/IG向け書き出し

## デザインコンセプト
「編集画面より“展示室”」。暗色背景にネオンアクセント、生成物をポスターのように並べる。

## アーキテクチャ
- Next.js + Supabase Auth/DB
- AWS Lambdaで生成ジョブ制御
- Replicate（画像/動画補助）+ OpenAI API（台本/コピー）
- S3保存、CloudFront配信

## DB設計
- users(id, plan, created_at)
- projects(id, user_id, source_type, source_url, tone, status)
- assets(id, project_id, type, s3_key, duration_sec)
- generations(id, user_id, tokens_in, tokens_out, model_cost)
- subscriptions(id, user_id, stripe_customer_id, plan, renew_at)

## コスト見積もり（月）
- Supabase: $0〜$25（初期は無料枠）
- AWS(S3/Lambda/CF): $3
- OpenAI/Replicate: $8（30〜50ユーザー想定）
- 合計: 約$11

## MVPスコープ
- URL/テキスト入力
- カルーセル8枚生成
- 15秒縦動画生成
- Stripe決済
- ダウンロード

## マーケ計画
- Xで「同一素材→3媒体変換」動画を毎日投稿
- Product Hunt Mini Launch
- 日本語コミュニティ（個人開発/運用代行）で事例配布

## 技術スタック
Next.js, TypeScript, Supabase, AWS Lambda/S3/CloudFront, Stripe, OpenAI API

## リスク
- 生成品質のブレ → テンプレ固定+再生成1回無料
- APIコスト増 → 解像度制御/秒数上限

## 競合分析
- Canva: 汎用すぎる
- OpusClip: 動画寄り
- 差別化: **カルーセル+短尺の同時最適化**

## $20達成シナリオ
- Starter $8 × 3人 = $24/月

## ユニットエコノミクス
- ARPU: $8
- 変動費/ユーザー: $1.6
- 粗利: $6.4 (80%)
