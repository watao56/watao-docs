# 🎨 RisoJam Studio

## 概要
テキストや写真1枚から、**リソグラフ風の2色ポスター**とSNS用アニメーションを同時生成するAIクリエイティブツール。海外の「raw/retroデザイン回帰」を日本語UIで即使える形にする。

## 海外事例分析
- **Kittl / Canva Magic Design**: ノンデザイナー向けテンプレ生成が伸長。
- **Are.na / Cosmos系のムードボード文化**: 完成度より“雰囲気”重視のクリエイティブ消費。
- **Risograph revival**: Etsy/InstagramでZINE・ポスター需要が継続。
- 日本では「リソ風テンプレを即量産できるSaaS」がまだ薄い。

## ターゲット
- 小規模カフェ/イベント主催者
- ZINE制作者、インディー音楽アカウント
- デザイナーではないが告知物を量産したい個人

## 料金
- Free: 月5枚
- Starter: $6/月（120枚）
- One-shot: $3/30枚パック

## ユーザーフロー
1. 写真 or テキストを入力
2. 「紙質」「インクにじみ」「2色パレット」を選択
3. AI生成（ポスター+縦動画）
4. PNG/MP4を書き出し、SNS投稿

## デザインコンセプト
- 「印刷工房UI」: 粒状ノイズ、蛍光アクセント、太字タイポ
- before/afterをスプリット表示して“見せたくなる”体験

## アーキテクチャ
- Next.js + Cloudflare R2
- 画像生成: Replicate（FLUX系）
- 加工: Cloud Run上のFFmpeg
- 課金: Stripe

## DB設計
- users(id, email, plan, credits, created_at)
- projects(id, user_id, prompt, palette, status, created_at)
- assets(id, project_id, type, url, cost_usd)
- billing_events(id, user_id, kind, amount, created_at)

## コスト見積もり（月）
- Vercel/Cloudflare無料枠: $0
- Replicate推定: $0.03/生成 × 300 = $9
- R2 + egress: $2
- 合計: **約$11**

## MVPスコープ
- 2色ポスター生成
- 3テンプレ（ライブ告知/セール/募集）
- MP4自動書き出し
- Stripe課金

## マーケ計画
- Xで「同じ写真→5種ポスター」毎日投稿
- BOOTH/ZINEコミュニティへの導線
- 初月はテンプレ配布でUGC獲得

## 技術スタック
Next.js, TypeScript, Supabase, Replicate API, FFmpeg, Stripe, Cloudflare R2

## リスク
- 生成品質のブレ → テンプレ固定+後処理で安定化
- 著作権画像の持ち込み → 利用規約と通報導線

## 競合分析
- Canva: 汎用的で強いが、リソグラフ特化の体験は弱い
- Kittl: デザイン強いが日本語導線と低価格パック訴求の余地

## $20達成シナリオ
- Starter 4人（$24）で達成
- もしくはOne-shot 7本（$21）

## ユニットエコノミクス
- Starter ARPU: $6
- 1ユーザー月間生成60枚想定、原価$1.8
- 粗利: $4.2（**70%**）
