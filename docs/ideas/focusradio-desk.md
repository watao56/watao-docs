# 📻 FocusRadio Desk

## 概要
タスク内容を入れると、**作業BGMと作業ブロック（25/50分）を自動編成**するパーソナル生産性ツール。Suno以降のAI音楽体験を「BGM生成」ではなく「実行導線」に転換。

## 海外事例分析
- **Suno/Udio**: AI音楽生成の受容が進行
- **Sunsama**: 計画と実行の橋渡し需要
- **Endel**: 状態連動サウンドの習慣化

## ターゲット
- 在宅ワーカー
- 勉強習慣を作りたい学生
- ADHD傾向のあるユーザー

## 料金
- Free: 1日2セッション
- Plus: $4/月（無制限）
- Team mini: $12/月（5名）

## ユーザーフロー
1. 今日のタスクを3件入力
2. 集中タイプ選択（deep/light/admin）
3. 自動でセッション編成
4. 完了後、実績カード表示

## デザインコンセプト
レトロラジオUI。ノブ操作で集中モード切替。

## アーキテクチャ
- PWA (Next.js)
- Web Audio API
- 既存ループ音源 + 軽量推薦ロジック
- Supabase（履歴）

## DB設計
- users(id, plan)
- tasks(id, user_id, text, energy)
- sessions(id, user_id, duration, sound_pack, completed)
- stats(id, user_id, focus_minutes, streak)

## コスト見積もり（月）
- Vercel: $0〜5
- Supabase: $0
- CDN音源: $1
- 合計: **$1〜6**

## MVPスコープ
- セッション自動提案
- 音源3種類
- 完了ログ・連続日数

## マーケ計画
- 「集中前後」の1枚カードをSNS共有
- 勉強配信コミュニティへの配布
- Product Hunt mini launch

## 技術スタック
Next.js / TypeScript / Supabase / Web Audio API

## リスク
- 音源の飽き
- 差別化不足（ポモドーロ汎用競合）

## 競合分析
- ポモドーロ系: 時間管理はあるが音体験弱い
- Endel: 音は強いがタスク導線は薄い
- 本案: タスク入力→音→完了まで一気通貫

## $20達成シナリオ
- Plus 5人（$20）で達成

## ユニットエコノミクス
- ARPU: $4
- 変動費/ユーザー: $0.3
- 粗利/ユーザー: $3.7
- 粗利率: 92.5%
