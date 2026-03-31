# 🎬 GlowBoard Cinema

## 概要
画像3〜5枚と一言を入れると、**4秒ループの“映画予告風”縦動画＋サムネ**を同時生成するAIクリエイティブツール。Canva Magic MediaやCapCut Templatesの需要を、日本語クリエイター向けに「最短30秒で投稿素材化」へ特化。

## 海外事例分析
- **Canva Magic Studio**: テキスト→デザイン/動画生成の大衆化を牽引
- **CapCut Templates**: テンプレ主導の短尺制作UXが定着
- **Captions**: “編集しない編集”の需要が高い
- 日本では「テンプレはあるが、ブランド統一+時短」がまだ弱い

## ターゲット
- Instagram/TikTok運用中の個人事業主
- EC小規模運営者
- ポートフォリオ更新頻度を上げたいデザイナー

## 料金
- Free: 月10本（透かしあり）
- Starter: $6/月（100本、透かしなし）
- One-shot: $5/30本追加

## ユーザーフロー
1. 画像投入（3〜5枚）
2. トーン選択（cinematic / neon / minimal）
3. 自動生成（動画+サムネ）
4. ワンクリックDL or 共有

## デザインコンセプト
「暗背景+発光グラデ+大型タイポ」。SNSで“見せたくなる”質感重視。

## アーキテクチャ
- Next.js (App Router)
- Cloudflare R2（素材保存）
- Replicate/RunPodの動画生成モデル
- Supabase（Auth/DB）
- Vercel cronでキュー処理

## DB設計
- users(id, plan, credits)
- projects(id, user_id, style, status)
- assets(id, project_id, type, url)
- generations(id, project_id, model, cost_usd, latency_ms)

## コスト見積もり（月）
- Vercel: $0〜5
- Supabase: $0
- R2: $1
- 生成API: $6（100〜150本想定）
- 合計: **$7〜12**

## MVPスコープ
- 3スタイル固定
- 4秒動画固定
- クレジット課金
- Stripe決済

## マーケ計画
- X/Discordで「生成前後比較」を毎日投稿
- 初回50人に無料クレジット配布
- 制作代行者向けアフィリエイト20%

## 技術スタック
Next.js / TypeScript / Supabase / Stripe / R2 / Replicate

## リスク
- 生成品質のブレ
- 著作権混入画像
- API単価変動

## 競合分析
- Canva: 汎用強いが細かいSNS運用導線が薄い
- CapCut: 編集強いがブランド一貫性弱い
- 本案: 日本語運用者向け最短導線に集中

## $20達成シナリオ
- Starter 4人（$24）で達成

## ユニットエコノミクス
- ARPU: $6
- 変動費/ユーザー: $1.2
- 粗利/ユーザー: $4.8
- 粗利率: 80%
