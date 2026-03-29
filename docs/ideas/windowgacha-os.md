# 🪟 WindowGacha OS

## 概要
WindowGacha OSは、作業再開時に「今開くべき3ウィンドウ」をガチャ形式で提案するパーソナル生産性ツール。海外の“anti-overwhelm productivity”潮流（Goblin Tools, Sunsama）を、遊び感で実装。

## 海外事例分析
- **Sunsama / Akiflow**: 統合型は強いが設定が重い。
- **Goblin Tools**: 認知負荷軽減が支持される。
- 余地: 「計画」より「再開1クリック」を重視した軽量UX。

## ターゲット
- タブ/ウィンドウを大量に開きがちな開発者
- マルチ案件を抱えるフリーランス
- ADHD傾向で再開負荷が高いユーザー

## 料金
- 買い切り $9（Chrome拡張）
- Plus $3/月（再開履歴AI要約）

## ユーザーフロー
1. 仕事モード（開発/営業/学習）を選ぶ
2. 現在のウィンドウ群を自動分類
3. 「再開ガチャ」を押す
4. 優先3ウィンドウ+理由を表示
5. セッション後に“合ってたか”を評価

## デザインコンセプト
- **Arcade Utility**: スロット風UI+ミニマル配色
- 罪悪感でなく“再開の勢い”を作る

## アーキテクチャ
- Chrome Extension (MV3)
- Local first (IndexedDB)
- Optional cloud sync: Supabase
- AI要約: OpenAI mini model

## DB設計
- local_sessions(id, mode, started_at, ended_at)
- local_windows(id, session_id, title, domain, score)
- local_feedback(id, session_id, hit_rate, memo)
- cloud_profiles(user_id, plan, sync_enabled)
- cloud_events(id, user_id, event_type, payload)

## コスト見積もり（月）
- 基本ローカル実行: $0
- Plus利用者10名時AI: $3〜$5
- Supabase無料枠: $0
- **合計: $0〜$5**

## MVPスコープ
- ローカル分類
- 3ウィンドウ提案
- ワンクリック復元
- 買い切りライセンス検証

## マーケ計画
- Product Huntの“tab chaos”層向け英語LP
- 日本ではXで「作業再開30秒チャレンジ」投稿
- Chrome拡張まとめサイトへ掲載

## 技術スタック
TypeScript, Plasmo, IndexedDB, Supabase, Stripe/Paddle

## リスク
- 拡張権限への心理抵抗 → ローカル処理強調
- 継続利用の弱さ → 連続再開スコアで習慣化

## 競合分析
- OneTab: 保存はできるが再開提案が弱い
- Arc: 美しいが習慣化機能は限定
- WindowGacha: 再開意思決定に特化

## $20達成シナリオ
- 買い切り3本（$27）
- または Plus 7人（$21）

## ユニットエコノミクス
- ARPU（混合）: $6.2
- 変動費: $0.4
- 粗利率: 93%
