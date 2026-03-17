# 🎬 Pin2Reel Studio

## 概要
Pinterestボード/画像URLを入力すると、**15〜30秒の縦動画（BGM・字幕・CTA付き）**を自動生成するAIクリエイティブツール。海外で伸びる「インスピレーションボード→動画化」体験を、日本向けにテンプレート化して提供する。

## 海外事例分析
- Pinterest公式のボード動画化機能（2024〜）で「静的ボードを短尺動画化」需要が顕在化
- Typito/FlexClip等のPinterest動画メーカーは存在するが、日本語テンプレとEC導線最適化は弱い
- 日本ではCanva/CapCutの汎用利用が中心で、**Pinterest起点の特化SaaSは未成熟**

## ターゲット
- D2C小規模ブランド
- ハンドメイド作家
- Instagram/TikTok運用代行の個人

## 料金
- Free: 月5本（透かしあり）
- Starter: $9/月（50本）
- Pro: $19/月（200本・ブランドキット）

## ユーザーフロー
1. ボードURL or 画像10枚を入力
2. トーン選択（Minimal / Pop / Luxury）
3. AIがシーン構成・字幕・BGMを生成
4. プレビュー微調整→MP4出力
5. Instagram/TikTokへワンクリック投稿下書き

## デザインコンセプト
- 「Canvaより速く、CapCutより迷わない」
- 暗色UI + ネオンアクセント、カード主体
- プレビュー1st（編集画面で最初に動画が見える）

## アーキテクチャ
- Next.js + Supabase Auth/DB
- 生成: Replicate系動画/画像補間 + ffmpeg
- ジョブ実行: AWS Lambda + SQS
- 配信: CloudFront + S3

## DB設計
- users(id, plan, created_at)
- projects(id, user_id, title, style, status)
- assets(id, project_id, type, source_url, s3_key)
- renders(id, project_id, duration_sec, cost_usd, output_url)
- subscriptions(id, user_id, stripe_customer_id, plan)

## コスト見積もり（月）
- Supabase: $0〜$25
- S3/CloudFront: $2
- 生成API: 1本$0.03想定、月300本で$9
- 合計: 約$12〜$36

## MVPスコープ
- URL/画像入力
- 3スタイルテンプレ
- 15秒動画出力
- Stripe課金

## マーケ計画
- Xで「1分で動画化」ビフォーアフター投稿
- Pinterest運用コミュニティへ無料枠配布
- ハンドメイド作家向けテンプレ配布

## 技術スタック
Next.js, TypeScript, Supabase, Stripe, ffmpeg, Replicate, AWS Lambda/S3/CloudFront

## リスク
- 生成品質のばらつき
- 版権画像混入
- 動画生成API単価変動

## 競合分析
- Canva/CapCut: 汎用すぎて工程が多い
- FlexClip: 日本語テンプレが弱い
- 差別化: **Pinterest特化 + 日本語EC CTA最適化**

## $20達成シナリオ
- Starter 3人で$27 MRR
- 生成原価（1人50本利用想定）約$1.5/人
- 3人で粗利約$22超

## ユニットエコノミクス
- ARPU: $9
- COGS/user: $2.1
- 粗利: $6.9（76%）
- 回収期間: 1ヶ月以内（獲得CPA $5想定）
