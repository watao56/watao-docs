# 🎞️ HoloGrid Reel

## 概要
**HoloGrid Reel**は、1枚の商品画像や作品画像から「海外風グリッチ/ホログラム演出の6秒ループ動画」を生成し、Instagram Reels/TikTok用に即出力するAIクリエイティブツール。  
「静止画しかない個人ショップ・クリエイター」に特化し、**1分以内に“見せたくなる”短尺素材**を作る。

## 海外事例分析
- **CapCut Templates**: テンプレ文化で短尺量産が進んだ
- **Canva Magic Studio**: 非デザイナーの生成編集を一般化
- **Runway / Pika系**: 映像生成自体は強いが、SNS投稿導線の即効性はまだ重い
- 示唆: 日本向けは「高機能」より**テンプレ化＋投稿即用**が刺さる

## ターゲット
- ハンドメイド作家
- 小規模D2C（1〜3人）
- SNS運用代行の副業者

## 料金
- Free: 月10本（透かしあり）
- Starter: **$6/month**（月120本）
- Drop Pack: **$4/50本**（買い切り）

## ユーザーフロー
1. 画像アップロード
2. テンプレ選択（Holo/Neon/Glass）
3. AIが6秒ループ生成
4. キャプション候補を1行提案
5. mp4書き出し

## デザインコンセプト
- 「クラブのVJ卓」風UI
- 黒背景＋シアン/マゼンタの発光
- 触りたくなるノブUI（実際はプリセット）

## アーキテクチャ
- Front: Next.js + Tailwind
- API: FastAPI
- Queue: SQS
- Worker: AWS Fargate Spot
- Storage: S3 + CloudFront
- Auth/Billing: Clerk + Stripe

## DB設計
- users(id, email, plan, credits, created_at)
- projects(id, user_id, source_image_url, template, status)
- renders(id, project_id, duration_sec, cost_usd, output_url)
- payments(id, user_id, stripe_session_id, amount)

## コスト見積もり（月）
- S3/CloudFront: $3
- Fargate Spot: $8
- AI推論API: $10
- 合計: **$21**（1000本想定）
- 1本あたり原価: 約$0.021

## MVPスコープ
- 3テンプレ
- 画像1枚→6秒動画
- Stripe決済
- 投稿サイズ（9:16）固定

## マーケ計画
- Xで「静止画→6秒動画」ビフォーアフターを毎日投稿
- Etsy/BASE系コミュニティへ無料コード配布
- 初月はテンプレ投票で共同制作感を作る

## 技術スタック
Next.js / FastAPI / AWS S3 / Fargate / Stripe / Postgres(Supabase)

## リスク
- 生成品質ブレ
- 競合のテンプレ追従
- 著作権画像の扱い

## 競合分析
- CapCut: 強いがブランド向け最適化が弱い
- Canva: 多機能で逆に重い
- 本案: **「商品静止画専用」**に絞る

## $20達成シナリオ
- Starter 4人 = $24/月
- または Drop Pack 5件 = $20/月

## ユニットエコノミクス
- Starter ARPU: $6
- 粗利率: 約72%
- 回収期間: 1か月未満
