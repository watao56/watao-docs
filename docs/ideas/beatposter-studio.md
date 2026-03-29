# 🎚️ BeatPoster Studio

## 概要
BeatPoster Studioは、商品の1行訴求（例:「朝3分で肌が整う」）を入力すると、**音楽同期したキネティックタイポ動画（9:16/1:1）**を20秒で生成するマイクロSaaS。海外で急成長している「Submagic / Captions / CapCutテンプレ文化」を、日本の小規模EC・個人クリエイター向けに「日本語フォント最適化」「縦動画向け余白設計」で再実装する。

## 海外事例分析
- **Captions / Submagic**: テロップ自動化需要が拡大。課題は「誰でも同じ見た目」になり差別化が弱い。
- **Canva Magic Design + CapCut Templates**: テンプレ高速化は強いが、訴求文の構造設計は弱い。
- 余地: 日本市場向けに「訴求コピー→動きの型」を強制することで、短時間でも“見せたくなる”品質を担保。

## ターゲット
- Shopify/BASE運用のD2C 1人チーム
- Instagram/TikTokで商品訴求する個人事業主
- 広告代理店のジュニア運用者（低予算案件）

## 料金
- Free: 月5本（透かし付き）
- Starter: $6/月（60本）
- Pro: $12/月（200本 + ブランドプリセット）

## ユーザーフロー
1. 商品名・訴求1行・トーン（クール/ポップ）入力
2. 3つのモーション案を提示
3. BGMプリセット選択（著作権フリー）
4. 生成→即プレビュー→MP4書き出し
5. 投稿後CTRメモを保存（次回提案に反映）

## デザインコンセプト
- **Neo-Editorial**: 黒背景+蛍光アクセント+太字明朝/ゴシック混在
- ノードエディタ風UIで「文字の跳ね」を可視化
- “作業ツール”でなく“作品ツール”に見えるUI

## アーキテクチャ
- Front: Next.js + Tailwind + Framer Motion
- API: FastAPI (Render free tier)
- Worker: AWS Lambda（Remotionレンダリング）
- Storage: Cloudflare R2（動画保存）
- Queue: Upstash Redis

## DB設計
- users(id, email, plan, created_at)
- projects(id, user_id, hook_text, tone, aspect, status)
- renders(id, project_id, template_id, duration_sec, cost_usd, output_url)
- metrics(id, project_id, posted_at, ctr, cvr_note)
- billing_events(id, user_id, provider, amount, created_at)

## コスト見積もり（月）
- Vercel Hobby: $0
- Render API: $0〜$7
- Lambda + S3転送: $6（200本想定）
- R2: $1
- OpenAI要約/提案: $4
- **合計: $11〜$18**

## MVPスコープ
- 3テンプレ固定（Bold/Minimal/Pop）
- 日本語フォント2種
- 9:16固定出力
- Stripeサブスク
- 生成履歴と再編集

## マーケ計画
- Xで「1商品3パターン比較」毎日投稿
- Shopify/BASEコミュニティへ無料枠案内
- 初月は5社に無料導入→事例化

## 技術スタック
Next.js, FastAPI, Remotion, Lambda, R2, Upstash, Stripe, Postgres(Supabase)

## リスク
- 生成時間遅延 → キュー優先制御
- 著作権誤解 → BGM利用範囲をUI明記
- テンプレ飽き → 月次テンプレ追加

## 競合分析
- CapCut: 無料だが業務再現性が低い
- Canva: 汎用で速いが“売るための動き”が弱い
- BeatPoster: 訴求文設計と日本語タイポに特化

## $20達成シナリオ
- Starter 4人（$24）で達成
- または Pro 2人（$24）

## ユニットエコノミクス
- ARPU: $6.8
- 1ユーザー月間変動費: $1.4
- 粗利: $5.4（粗利率79%）
- 回収: 無料流入中心でCACほぼ$0
