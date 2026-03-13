# 🛡️ ContractSwitch Lite

## 概要
唯一の保険寄り枠。業務委託・サブスクの**自動更新/解約期限**だけを超軽量管理し、期限7日前に「その契約を続ける理由/切る理由」をAIで提示する。

## 海外事例分析
- Rocket Money: 解約忘れ防止の継続課金成立
- Dock.us契約管理系: 更新期限ワークフローの需要

## ターゲット
- フリーランス、小規模制作会社、個人事業主

## 料金
- Lite: $4/月（契約20件）
- Pro: $8/月（契約無制限）

## ユーザーフロー
1. 契約名・更新日登録
2. 7日前通知
3. AIが「継続/解約判断メモ」を生成
4. 決定を記録

## デザインコンセプト
事務ツール感を排除し、信号機カラーで判断を直感化。

## アーキテクチャ
Next.js + Supabase + cron通知（Discord/メール）。

## DB設計
- users(id, plan)
- contracts(id, user_id, vendor, renew_date, amount, status)
- reminders(id, contract_id, remind_at, sent)
- decisions(id, contract_id, action, memo)

## コスト見積もり（月）
- インフラ: $0
- AI: $1
- 合計: 約$1

## MVPスコープ
- 契約CRUD
- 通知
- 判断メモ生成

## マーケ計画
- 「今月切れたムダ契約」匿名テンプレ配布
- 税理士/フリーランスコミュニティ提携

## 技術スタック
Next.js, Supabase, Resend, OpenAI mini

## リスク
- 既存家計簿アプリとの競合 → B2Bフリーランス契約に絞る

## 競合分析
家計簿系は個人支出中心。ContractSwitchは仕事契約の意思決定に特化。

## $20達成シナリオ
- Pro 3人で$24

## ユニットエコノミクス
- ARPU: $5.5
- 変動費/人: $0.1
- 粗利率: 98%
