# 🛡️ ClauseCue Lite

## 概要
フリーランス契約書の**危険条項（再委託禁止・無期限修正・著作権帰属）**を抽出し、更新期限と交渉テンプレを出すライト保険型SaaS。保険寄りは5案中この1件のみに限定。

## 海外事例分析
- **Juro / Ironclad**: 契約ワークフロー
- **IndieLaw系テンプレ文化**: 個人向け契約知識ニーズ
- 日本ギャップ: 小規模向けの超軽量チェックが少ない

## ターゲット
- デザイナー/動画編集フリーランス
- 小規模制作会社

## 料金
- Lite: $6/月（10契約）
- Pro: $12/月（無制限+通知）

## ユーザーフロー
1. 契約PDF投入
2. AIが条項タグ付け（赤/黄/緑）
3. 期限と交渉文テンプレ提示
4. 期限前に通知

## デザインコンセプト
- "Signal Light"
- 法務感を薄めた分かりやすい色設計

## アーキテクチャ
- Front: Next.js
- Backend: Python FastAPI
- OCR: AWS Textract (低頻度利用)
- AI: GPT-5.3 mini
- Notification: Resend + Discord webhook

## DB設計
- users(id, email, plan)
- contracts(id, user_id, file_url, signed_date)
- clauses(id, contract_id, type, risk_level, note)
- reminders(id, contract_id, remind_at, sent)
- templates(id, clause_type, negotiation_text)

## コスト見積もり（月）
- Hosting: $7
- Textract: $3
- AI: $5
- Email/通知: $1
- 合計: **$10〜18**

## MVPスコープ
- PDF読込
- 10種類条項判定
- 期限通知
- 交渉テンプレ3種

## マーケ計画
- フリーランス向け契約チェック無料キャンペーン
- デザインスクール卒業生コミュニティ導入

## 技術スタック
Next.js / FastAPI / AWS Textract / Postgres / Stripe

## リスク
- 法的助言との境界
- OCR精度不足

## 競合分析
- 契約管理SaaS: 企業向けで高価
- 弁護士相談: 単発で高コスト
- 差別化: 個人向け月額・通知・交渉テンプレ

## $20達成シナリオ
- Lite($6)×2 + Pro($12)×1 = $24

## ユニットエコノミクス
- ARPU: $7.9
- COGS/user: $1.5
- 粗利: 81%
