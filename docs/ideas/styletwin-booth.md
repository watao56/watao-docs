# 👗 StyleTwin Booth

## 概要
写真1枚から、**同一人物・同一商品テイストを保ったまま**「EC用ルックブック画像」を自動生成するAIクリエイティブSaaS。Mercari/BASE/Shopify小規模販売者向け。日本ではまだ手作業コラージュが多く、海外の"AI lookbook"文脈をローカライズ。

## 海外事例分析
- **Photoroom**: 背景除去と商品見せ方最適化
- **Pebblely**: 商品画像のシーン生成
- **Canva Magic Media**: ノンデザイナー向け高速生成
- 日本ギャップ: 「出品画像の世界観統一」をワンクリックでやる特化サービスが薄い

## ターゲット
- 副業D2C/ハンドメイド販売者
- Instagram経由で販売する個人
- 月5〜30商品を回す小規模ショップ

## 料金
- Free: 月10枚
- Starter: $7/月（200枚）
- Pro: $15/月（1000枚）

## ユーザーフロー
1. 商品画像と参考スタイルをアップロード
2. 「韓国ミニマル」「和モダン」などテンプレ選択
3. 4枚セット（正方形/縦長/バナー/サムネ）生成
4. そのままダウンロード or SNS投稿用に書き出し

## デザインコンセプト
- "Magazine Bento"
- 白余白＋大胆タイポ＋アクセント1色
- 共有したくなる統一感重視

## アーキテクチャ
- Front: Next.js + Tailwind
- API: FastAPI
- Gen: Replicate(FLUX系) + Cloudinary変換
- Storage: S3
- Auth/Billing: Supabase Auth + Stripe
- Queue: Upstash Redis + QStash

## DB設計
- users(id, email, plan, created_at)
- projects(id, user_id, style_preset, created_at)
- assets(id, project_id, input_url, output_url, ratio, cost_usd)
- subscriptions(id, user_id, stripe_sub_id, status)
- usage_daily(id, user_id, date, gen_count)

## コスト見積もり（月）
- Vercel/Render: $0〜7
- Supabase: $0
- S3/Cloudinary: $2
- 生成API: $8（約800枚）
- 合計: **$10〜17**

## MVPスコープ
- 3スタイル固定
- 3アスペクト比出力
- 画像一括ZIP
- Stripe課金

## マーケ計画
- Xで「出品画像ビフォーアフター」投稿
- メルカリ物販コミュニティへ無料枠配布
- BASEテンプレ制作者との提携

## 技術スタック
Next.js / FastAPI / Supabase / Stripe / Replicate / S3

## リスク
- 生成品質ぶれ
- 商標・著作権画像の扱い
- 生成APIコスト上振れ

## 競合分析
- Canva: 汎用すぎる
- Photoroom: 商品切り抜きは強いが日本語販売導線は弱い
- 差別化: 日本EC向けテンプレ＋複数比率同時生成

## $20達成シナリオ
- Starter($7)×3人 = $21

## ユニットエコノミクス
- ARPU: $7.8
- COGS/user: $1.4
- 粗利: 82%
- 回収期間: 1か月未満
