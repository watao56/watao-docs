# 🛡️ TrialPocket Lite

## 概要
TrialPocket Liteは、保険型（無料トライアルの解約忘れ防止）に特化した月$20到達を狙うマイクロプロダクト。

## 海外事例分析
- 参照潮流: Rocket Money / Trim のサブスク管理ニーズ
- 日本でのギャップ: 英語圏前提・UIが重い・共有導線が弱い
- 勝ち筋: 日本語UI + 3分以内で価値体験 + 共有導線最適化

## ターゲット
SaaSを試しまくる個人開発者・マーケ担当

## 料金
- プラン: Lite $4/月
- 無料→有料転換導線: 初回の成功体験後に使用回数上限を提示

## ユーザーフロー
1. 試したサービス名と終了日を登録
2. 終了3日前/当日に通知
3. ワンクリックで解約ページメモへ遷移
4. 月末にムダ課金レポートを受け取る

## デザインコンセプト
「期限が見える」タイムラインUI。赤黄緑の単純信号。

## アーキテクチャ
Next.js + Supabase + cron + webhook notifier

## DB設計
- 主要テーブル: users, trials, reminders, cancel_links, reports
- 監査/運用: created_at, updated_at, soft_deleteを全テーブルに付与

## コスト見積もり（月次）
- インフラ: $0〜$8（無料枠中心、超過時のみ従量）
- AIコスト: ほぼゼロ（AIは通知文面最適化のみ）
- その他: Stripe手数料のみ

## MVPスコープ
登録、通知、解約リンク保存、月次レポ、課金

## マーケ計画
Product Hunt系「free trial整理」層に刺す

## 技術スタック
- Frontend: Next.js / React
- Backend: Serverless Functions
- DB: Supabase or DynamoDB
- Billing: Stripe
- Analytics: PostHog

## リスク
既存競合あり→日本語UIとDiscord通知で差別化。

## 競合分析
一般家計簿よりSaaSトライアル特化で高速。

## $20達成シナリオ
Lite 5人($20)で達成

## ユニットエコノミクス
ARPU $4 / 変動費 $0.05 / 粗利99%
