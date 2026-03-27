# 🧭 Focus Drift HUD

## 概要
「今の集中度」と「次の5分アクション」を常時表示する、デスクトップ用パーソナル生産性HUD。タスク管理より**再始動の速さ**に特化。

## 海外事例分析
- **Rise / Sunsama**: 計画型生産性の需要
- **Lofi + flowtime apps**: 集中の雰囲気演出が定着
- **Goblin Tools**: 実行可能な最小行動への分解ニーズ
- 日本では「HUD型で最小再開」を主軸にした製品は希少。

## ターゲット
- 在宅ワーカー
- ADHD傾向のある知的労働者
- 複業でタスク切替が多い人

## 料金
- Free: 2ボード
- Pro: $4/月（無制限・統計）

## ユーザーフロー
1. いまの作業を選択
2. 中断時に「戻る一手」を自動生成
3. HUDで常時表示
4. 完了でドリフト時間を可視化

## デザインコンセプト
- SFコックピット風の半透明UI
- 1画面1アクションで認知負荷を削減

## アーキテクチャ
- Tauriデスクトップアプリ
- ローカルSQLite
- AI補助はOpenAI miniモデル（任意）

## DB設計
- tasks(id, title, state, priority)
- drift_logs(id, task_id, pause_at, resume_at, drift_sec)
- next_actions(id, task_id, text, source)
- settings(id, key, value)

## コスト見積もり（月）
- デスクトップ中心でサーバー不要
- AI補助使用者のみ従量（$0〜$5）
- 固定費: **ほぼ$0**

## MVPスコープ
- HUD表示
- 中断/再開ログ
- 次の一手生成（テンプレ+任意AI）
- CSVエクスポート

## マーケ計画
- 「再開時間が何分短縮したか」をSNSカード化
- Product Hunt / Reddit r/productivityで配布

## 技術スタック
Tauri, React, SQLite, Rust, optional OpenAI API

## リスク
- 常駐アプリは導入障壁がある → Web版ミニモードを用意

## 競合分析
- Sunsama等は計画重視、再始動HUDには未特化
- ToDoアプリとの差別化は“中断復帰の即効性”

## $20達成シナリオ
- Pro 5人（$20）で達成

## ユニットエコノミクス
- ARPU $4
- 原価 $0.2（AI未使用時）
- 粗利率 **95%+**
