# 🎡 SpinShelf Studio

## 概要
スマホ写真8枚をアップするだけで、**360°風プロダクト回転動画 + 商品ページ用静止画セット**を生成するAIクリエイティブツール。Shopify/BASEの商品ページにそのまま貼れる。

## 海外事例分析
- **Luma AI**: スマホ3Dキャプチャの一般化（高品質3D表現の敷居を下げた）
- **Photoroom**: EC画像の自動加工UXが定着
- **Canva Magic Media**: 非デザイナーの「すぐ使える生成体験」が主流

示唆: 日本EC個人店向けに「3Dっぽく見える見栄え」を低コストで提供すれば刺さる。

## ターゲット
- BASE/Shopifyの個人EC（ハンドメイド、コスメ、小物）
- 月10〜200商品を扱う小規模D2C

## 料金
- Free: 月10クレジット（透かしあり）
- Lite: $9/月（120クレジット）
- Pro: $19/月（400クレジット）

## ユーザーフロー
1. 商品写真8枚アップロード
2. 背景/回転速度テンプレ選択
3. 45秒以内に動画+静止画を生成
4. Shopify/BASE向けサイズで書き出し

## デザインコンセプト
「**Turntable Neon**」: 黒背景×ネオングラデ。生成結果を“見せたくなる”UI。

## アーキテクチャ
- Front: Next.js + Tailwind
- API: FastAPI
- AI: Replicate(画像補完) + FFmpeg
- Storage: S3
- Queue: SQS
- Deploy: AWS Fargate (最小)

## DB設計
- users(id, email, plan, credits)
- projects(id, user_id, status, style, created_at)
- assets(id, project_id, type, s3_key, duration_sec)
- billing_events(id, user_id, amount, provider, created_at)

## コスト見積もり（月）
- AWS: $4.5
- AI推論: $6.0（有料10人想定）
- 合計: **$10.5**

## MVPスコープ
- 8枚入力→回転動画生成
- 4テンプレ書き出し
- クレジット課金
- 公開ギャラリー（任意）

## マーケ計画
- Xで「ビフォー/アフター」短尺投稿
- Shopifyコミュニティへ導入事例投稿
- 初月無料コード配布

## 技術スタック
Next.js, FastAPI, PostgreSQL, S3, SQS, Stripe, Replicate

## リスク
- 生成失敗率 → 再試行キュー + 自動返却
- ECの季節波動 → クレジット繰越で解約抑制

## 競合分析
- Photoroom: 静止画強いが360風動画弱い
- CapCut: 汎用すぎてEC最適化が弱い

## $20達成シナリオ
- Lite 3人 = $27
- 変動費約$6 → 粗利約$21

## ユニットエコノミクス
- ARPU: $9
- 1ユーザー変動費: $1.8
- 粗利率: **80%**
