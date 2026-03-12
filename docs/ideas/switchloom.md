# 🧠 SwitchLoom

## 概要
SwitchLoomは、パーソナルツール/生産性（気分スイッチ別タスク実行）に特化した月$20到達を狙うマイクロプロダクト。

## 海外事例分析
- 参照潮流: Goblin Tools / Sunsama / ADHD向けmicro-planningトレンド
- 日本でのギャップ: 英語圏前提・UIが重い・共有導線が弱い
- 勝ち筋: 日本語UI + 3分以内で価値体験 + 共有導線最適化

## ターゲット
集中の波がある人、在宅ワーカー

## 料金
- プラン: Free / Plus $5/月
- 無料→有料転換導線: 初回の成功体験後に使用回数上限を提示

## ユーザーフロー
1. 今の状態を「重い/普通/軽い」で選ぶ
2. AIが5分・15分・45分の実行メニュー提示
3. 完了時に自動で次の最小タスクを提案
4. 日次で「できたログ」を1画面表示

## デザインコンセプト
トグル中心のミニマルUI。色は3状態のみで判断疲れを減らす。

## アーキテクチャ
React + Firebase + OpenAI mini model + scheduled summary

## DB設計
- 主要テーブル: users, tasks, state_logs, execution_logs, nudges
- 監査/運用: created_at, updated_at, soft_deleteを全テーブルに付与

## コスト見積もり（月次）
- インフラ: $0〜$8（無料枠中心、超過時のみ従量）
- AIコスト: 約$0.03/ユーザー/月（1日数回の軽量提案）
- その他: Stripe手数料のみ

## MVPスコープ
状態入力、3段メニュー、doneログ、週次レポ、課金

## マーケ計画
ADHD/生産性コミュニティに「5分モード」訴求

## 技術スタック
- Frontend: Next.js / React
- Backend: Serverless Functions
- DB: Supabase or DynamoDB
- Billing: Stripe
- Analytics: PostHog

## リスク
提案が外れる→ユーザー調整学習。継続率低下→連続達成バッジ。

## 競合分析
ToDoアプリより実行寄り、日記アプリより行動に直結。

## $20達成シナリオ
Plus 4人($20)で達成

## ユニットエコノミクス
ARPU $5 / 変動費 $0.2 / 粗利96%
