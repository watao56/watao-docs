# 🏠 RoomLoop AI

## 概要
店舗オーナーや小規模不動産向けに、1枚の室内写真から「3つの世界観（和モダン/北欧/韓国カフェ風など）」を自動生成し、SNS投稿用カルーセルまで作るツール。

## 海外事例分析
- 参考サービス: **RoomGPT / ReimagineHome / Pinterest Collages**
- 海外では「短時間で見栄えの良い成果物を作る」需要が強い。日本市場では日本語UI・日本語テンプレ不足が参入余地。

## ターゲット
カフェ・サロン・民泊オーナー、賃貸仲介の個人営業

## 料金
Free: 月5生成 / Pro $7/月（100生成+ブランドカラー） / Agency $19/月（3アカウント）

## ユーザーフロー
(1)写真アップロード→(2)スタイル選択→(3)AI生成→(4)比較スライダー確認→(5)Instagram/X向け書き出し

## デザインコンセプト
「Before/Afterを見せたくなる」重視。暗背景+ネオン枠で変化を強調。

## アーキテクチャ
Next.js + Cloudflare R2 + Replicate(FLUX-schnell) + Supabase + Cloudflare Workers Cron

## DB設計（MVP）
- users, projects, renders, style_presets, exports, subscriptions

## コスト見積もり（月次）
固定費: Supabase無料枠 + Vercel無料枠。変動費: 1生成$0.004想定。Pro1人100生成で$0.4。

## MVPスコープ（2週間）
写真1枚→3スタイル生成、比較UI、PNG書き出し、Stripe課金

## マーケ計画
Instagramで「1枚で内装妄想」ショート動画。地域の店舗改装アカウントへDM。

## 技術スタック
TypeScript, Next.js, Tailwind, Supabase, Stripe, Replicate

## リスク
生成品質ブレ→プリセットを絞る。著作権懸念→利用規約で入力画像権利を明記。

## 競合分析
Canvaは汎用、RoomLoopは「空間特化+日本語導線+SNS書き出し」で差別化。

## $20達成シナリオ
Pro 3人で$21達成。初月は無料配布30件→3-5%転換を狙う。

## ユニットエコノミクス
ARPU $7、粗利94%。LTV 6ヶ月で$42、CACはDM+SNS運用で$5以下目標。
