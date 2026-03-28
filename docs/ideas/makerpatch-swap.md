# 🧩 MakerPatch Swap

## 概要
海外のbuild-in-public文化を日本向けに再設計した、**週1回の「ビフォー/アフター交換会」**コミュニティ。作業前30秒宣言→作業後30秒成果を交換し、相互フィードバックを得る。

## 海外事例分析
- **Focusmate**: 同時作業の習慣化需要は大きい
- **ADPList**: ランダムな相互支援の価値が定着
- **Indie Hackers build logs**: 公開進捗が継続率を押し上げる
- 日本ギャップ: 「軽く見せる」場が少なく、発信ハードルが高い

## ターゲット
- 副業クリエイター
- 個人開発者
- デザイン学習者

## 料金
- Free: 週1マッチ
- Pro: **$6/mo**（週5マッチ＋履歴分析）
- Team: $19/mo（5人）

## ユーザーフロー
1. 作業テーマ登録
2. 15分セッションに参加
3. 前後スクショを投稿
4. 相手がテンプレFB
5. 週次レポート受領

## デザインコンセプト
"**tiny stage, big momentum**"。ネオンの進捗バー、カセットUI、投稿が並ぶギャラリー感。

## アーキテクチャ
- Front: Next.js
- Realtime: Supabase Realtime
- Match logic: Cloud Functions cron
- Media: Cloudflare R2
- Optional AI: OpenAI miniモデルでFB下書き

## DB設計
- users(id, profile, timezone, plan)
- sessions(id, slot_at, status)
- matches(id, session_id, user_a, user_b)
- posts(id, session_id, user_id, before_url, after_url, note)
- feedback(id, post_id, from_user, text, ai_assist)

## コスト見積もり（月）
- Supabase Pro: $25
- R2: $2
- AI補助: $4
- 合計: **$31**（初期はSupabase Freeで$6程度）

## MVPスコープ
- 15分セッション
- 2人マッチ固定
- before/after投稿
- 週次振り返りメール

## マーケ計画
- Discordコミュニティ連携
- 「週3回で何が進んだか」実例投稿
- 初月無料コード配布

## 技術スタック
Next.js / Supabase / Cloudflare R2 / Resend / Stripe

## リスク
- 参加者密度不足
- マッチ品質偏り
- 荒らし対策コスト

## 競合分析
- Focusmateは汎用集中、MakerPatchは**成果交換**に特化
- Discordは運営コストが高く、データ蓄積が弱い

## $20達成シナリオ
- Pro 4ユーザーで$24
- 低コスト運用時（Free tier中心）で粗利$20超

## ユニットエコノミクス
- ARPU: $6
- 変動費/有料ユーザー: $0.7
- 粗利率: 88%+
