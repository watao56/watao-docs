# 🎨 PrismBoard Studio

## 概要
海外で伸びている「AIムードボード」文脈を、日本の個人クリエイター向けに**“映える短尺リールまで一気通貫”**へ落としたマイクロSaaS。画像3〜5枚と訴求文を入れるだけで、15秒縦動画・サムネ・投稿文を同時生成する。

## 海外事例分析
- **Kosmik / VSCO Canvas**: ムードボード体験は強いが、最終アウトプット(投稿素材)への接続が弱い。
- **Miro AI Moodboard**: チーム向け。個人の即投稿ニーズには重い。
- **示唆**: 日本市場では「考える」より「すぐ投稿」が刺さるため、生成後1クリックでSNS用パッケージ化。

## ターゲット
- Instagram/TikTok運用中の1人事業主
- ハンドメイド作家、サロン、飲食のSNS担当

## 料金
- Free: 月5生成
- Starter: $6/月（60生成）
- Pro: $12/月（200生成）

## ユーザーフロー
1. 参照画像アップロード
2. ブランドトーン選択（Cute/Minimal/Luxury）
3. AIがムードボード→15秒動画→投稿文を生成
4. そのままダウンロード/予約投稿

## デザインコンセプト
「Neon Craft」：暗背景＋高彩度アクセント。作業画面はミニマル、出力は派手。

## アーキテクチャ
- Next.js + Cloudflare R2
- 生成: Replicate(画像補完) + ffmpeg
- 認証: Clerk
- 決済: Stripe

## DB設計
- users(id, plan, monthly_quota)
- projects(id, user_id, style, status)
- assets(id, project_id, type, r2_key)
- generations(id, project_id, model_cost_usd, duration_sec)

## コスト見積もり（月）
- Vercel/Workers: $0〜5
- R2: $1
- AI推論: $8（有料5人想定）
- 合計: 約$14

## MVPスコープ
- 3テンプレート
- 縦動画15秒固定
- IG投稿文生成

## マーケ計画
- Xで「1分で作る販促リール」デモを毎日投稿
- Canvaテンプレ比較記事をSEO投入
- 初期10名に無料導入→作例を許諾付き公開

## 技術スタック
Next.js / Supabase / Cloudflare R2 / Replicate / Stripe

## リスク
- 生成品質ぶれ → テンプレ制約で担保
- AIコスト増 → 解像度段階制

## 競合分析
Canvaより「日本語訴求と短尺販売導線」に特化。CapCutより企画〜投稿文までワンストップ。

## $20達成シナリオ
Starter($6)×4人 = $24/月

## ユニットエコノミクス
- ARPU: $6.8
- 変動費/人: $1.4
- 粗利: $5.4（79%）
