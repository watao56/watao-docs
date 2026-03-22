# 🫂 NeighborBench Club

## 概要
海外の「co-working social」「stranger meetup productivity」潮流（Geneva/Partiful/Luma）を、日本向けに**近所15分作業会**へ落とし込むコミュニティSaaS。主催者が“作業ベンチ”を作ると、半径圏内ユーザーが参加できる。

## 海外事例分析
- **Partiful**: 軽量イベント集客
- **Luma**: 小規模イベント管理
- **Geneva**: コミュニティ会話と継続率
- 日本は「重いイベント」より短時間・低心理負担形式が刺さる

## ターゲット
- フリーランス
- 副業勢
- リモートワーカー

## 料金
- Host Free: 月2イベント
- Host Pro: $7/月（無制限）
- Member課金なし

## ユーザーフロー
1. 主催者が15分ベンチ作成
2. 参加者がワンタップ参加
3. 終了後に成果1行投稿
4. 連続参加でバッジ獲得

## デザインコンセプト
- 「カフェの掲示板」風
- 暖色グラデ+角丸カード
- 参加ボタンを大きく、心理負担を下げる

## アーキテクチャ
- Next.js + Supabase Realtime
- 地域候補は手動入力（GPS常時利用なし）
- Discord連携通知（Webhook）

## DB設計
- users(id, name, role, city, created_at)
- benches(id, host_id, title, city, starts_at, duration_min, cap)
- bench_members(id, bench_id, user_id, status)
- bench_logs(id, bench_id, user_id, done_text)

## コスト見積もり（月）
- Supabase Pro未満運用: $0〜$5
- Vercel Hobby: $0
- 合計: $5以下

## MVPスコープ
- ベンチ作成/参加
- 参加ログ
- 連続参加バッジ

## マーケ計画
- Xで「15分作業会」タグ運用
- Discordコミュニティ3箇所へテスト導入

## 技術スタック
- Next.js / Supabase / Tailwind
- Discord Webhook

## リスク
- 初期参加者不足
- 地域偏在

## 競合分析
- Focusmateは1on1色が強い
- NeighborBenchは**ローカルゆるコミュニティ**に特化

## $20達成シナリオ
- Host Pro 3人 = $21/月

## ユニットエコノミクス
- ARPU: $7
- 変動費: $0.4/host
- 粗利率: 94%
