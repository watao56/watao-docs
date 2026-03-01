# 🎞️ Reel2Manga

## 概要
ショート動画をアップすると、主要シーンを抽出して**縦長マンガ風カルーセル**を自動生成。TikTok/Instagram向けに「見せたくなる」静止画コンテンツへ再編集するAIクリエイティブツール。

## 海外事例分析
- **Captions / OpusClip**: 動画再利用ニーズが強い
- **Remini / Lensa**: スタイル変換課金が成立
- 日本では「動画→漫画調カルーセル」の即時生成特化はまだニッチ。

## ターゲット
- SNS運用代行、個人クリエイター、講師

## 料金
- Starter: $4/月（30本）
- Pro: **$9/月**（120本、ブランドテンプレ保存）

## ユーザーフロー
1. 動画URL or アップロード
2. シーン自動抽出（5〜8コマ）
3. 漫画スタイル選択（少年/ミニマル/ネオン）
4. Canva互換PNG一括DL

## デザインコンセプト
- 黒背景 + 蛍光アクセント
- 「コマ送り」アニメーションで体験の楽しさを演出

## アーキテクチャ
- Front: Next.js
- Worker: Modal or RunPod（GPUスポット）
- Storage: Cloudflare R2
- DB: Supabase
- Queue: Upstash Redis

## DB設計
- users, projects, uploads, frames, styles, exports, billing_events

## コスト見積もり（月）
- Vercel: $0
- Supabase: $0
- R2: $1
- GPU推論: $8（Pro 15ユーザー想定）
- 合計: **$9**

## MVPスコープ
- 動画→5コマ抽出
- 2スタイル変換
- PNG出力
- Stripe課金

## マーケ計画
- 「同じ動画を漫画化」比較投稿（Before/After）
- TikTok運用代行向けアフィリエイト

## 技術スタック
Next.js, FFmpeg, Python worker, Diffusionモデル, Supabase, Stripe

## リスク
- 生成品質のブレ
- GPUコスト増
- 対策: 解像度段階課金、再生成回数制限

## 競合分析
- OpusClip: 動画編集強いが漫画化弱い
- Canva: 汎用で深い自動化がない
- 差別化: ワンクリック漫画カルーセル特化

## $20達成シナリオ
- Pro $9 x 3人 = $27
- または Starter $4 x 5人 = $20

## ユニットエコノミクス
- ARPU: $7
- 変動費: $1.2
- 粗利率: 82.8%
