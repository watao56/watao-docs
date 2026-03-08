# 🛡️ RefundAlly Lite

## 概要
海外SaaSの返金条件・トライアル期限を1画面に集約し、返金可能期間を逃さないよう通知する軽量ツール。

## 海外事例分析
- 参考サービス: **Rocket Money cancellation awareness / Chargeflow dispute ops**
- 海外では「短時間で見栄えの良い成果物を作る」需要が強い。日本市場では日本語UI・日本語テンプレ不足が参入余地。

## ターゲット
複数SaaSを試す個人開発者・マーケ担当

## 料金
Free: 5サービス / Pro $4/月（無制限+Slack/Discord通知）

## ユーザーフロー
(1)契約メール転送→(2)AIで返金条件抽出→(3)期限カレンダー表示→(4)期限前通知

## デザインコンセプト
危機管理感を出しすぎず、カレンダー上の「回収可能額」を可視化。

## アーキテクチャ
Cloudflare Email Routing + Workers + Supabase + Cron + Discord webhook

## DB設計（MVP）
- users, subscriptions_detected, refund_policies, deadlines, notifications, plans

## コスト見積もり（月次）
AI抽出は初回のみ。1契約あたり$0.002程度。月50契約でも$0.1。

## MVPスコープ（2週間）
メール取込、条件抽出、期限通知、簡易ダッシュボード

## マーケ計画
「SaaSお試し難民」向けにXで実損事例をフックに訴求。

## 技術スタック
Workers, Supabase, OpenAI mini, Resend/Discord

## リスク
誤抽出→「要確認」ラベルと原文リンクを強制表示。

## 競合分析
家計簿系よりB2B SaaS契約に特化。

## $20達成シナリオ
Pro 5人で$20。

## ユニットエコノミクス
ARPU $4、粗利93%。解約率は高め想定でオンボーディングを短く。
