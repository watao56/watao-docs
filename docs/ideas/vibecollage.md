# ✨ VibeCollage

## 概要
テキストや写真3枚から、**Instagram/Threads向けの“雑誌っぽいコラージュ”**を自動生成するAIデザインツール。海外ではCanvaテンプレ＋CapCut編集が一般化しているが、日本語フォント最適化・縦長比率最適化に特化した小粒SaaSは空白がある。

## 海外事例分析
- Canva/Adobe Express: 汎用で強いが、作業ステップが多い
- Captions/CapCut: 動画寄り、静止画カルーセルは補助的
- 需要示唆: “post-ready creative”に対する支払い意欲は高い
- 日本ギャップ: 和文組版（禁則処理/縦横混在）に弱い

## ターゲット
- 副業クリエイター
- 小規模D2C運営者
- SNS運用代行の個人

## 料金
- Free: 月10枚、透かしあり
- Pro: $6/月（透かしなし、月300枚）
- Studio: $12/月（ブランドプリセット、共同編集）

## ユーザーフロー
1. 投稿テーマ入力（例: 春の新作）
2. 参考画像3枚アップロード
3. AIが5案生成
4. 色/フォント微調整
5. PNG/WebP書き出し＋予約投稿連携

## デザインコンセプト
- 「ZINE + Neon」
- 余白大きめ、強いタイポ、レイヤー感
- テンプレ一覧を“ギャラリー体験”にして見せたくなるUI

## アーキテクチャ
- Next.js (App Router) + Cloudflare R2
- 生成: OpenAI Images or Replicate(FLUX) 切替
- ワーカー: AWS Lambda (生成ジョブ)
- Queue: SQS

## DB設計
- users(id, email, plan, created_at)
- projects(id, user_id, prompt, style, status)
- assets(id, project_id, type, url, meta_json)
- usage_daily(id, user_id, date, renders)
- subscriptions(id, user_id, stripe_sub_id, status)

## コスト見積もり（月）
- Vercel Hobby: $0
- R2: $1
- AI画像生成: $6（初期想定）
- 合計: **約$7/月**

## MVPスコープ
- 3スタイル固定
- 正方形/縦長の2比率
- Stripe決済
- 履歴保存

## マーケ計画
- Xで「1投稿5分で完成」動画
- デザイン比較（手作業30分 vs 3分）
- 初期20人に無料枠拡大でUGC獲得

## 技術スタック
Next.js / TypeScript / Supabase / Stripe / SQS / Lambda / R2

## リスク
- 生成品質のばらつき
- 著作権類似リスク（入力画像依存）

## 競合分析
- Canva: 汎用、速度勝負で差別化
- Adobe Express: 価格と複雑さで棲み分け

## $20達成シナリオ
- Pro 4人で $24 MRR
- もしくは Studio 2人で $24 MRR

## ユニットエコノミクス
- ARPU: $6.8
- 変動費/人: $0.9
- 粗利/人: $5.9（粗利率86%）
