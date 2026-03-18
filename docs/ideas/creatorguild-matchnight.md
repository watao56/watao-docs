# 🫂 CreatorGuild Matchnight

## 概要
海外のLuma/Partiful文化を取り込み、**毎週30分の“相互フィードバック会”を自動マッチング**するコミュニティSaaS。Discord連携で、似た制作テーマ同士を3人組に編成。

## 海外事例分析
- **Luma / Partiful**: 軽量イベント運営UXが成長。
- **ADPList**: メンタリングマッチングの継続利用率が高い。
- 日本は「告知はできるが継続マッチが弱い」ため、週次固定導線に価値あり。

## ターゲット
- 創作系個人（動画/デザイン/文章）
- 小規模オンラインサロン運営者

## 料金
- Host Lite: $6/月（1コミュニティ・30名）
- Host Pro: $12/月（3コミュニティ・150名）

## ユーザーフロー
1. Discord接続
2. 参加者が得意領域と今週の課題を入力
3. 毎週自動で3人マッチ
4. セッション後に満足度入力→次週の組み合わせ最適化

## デザインコンセプト
「夜のラウンジ」。濃紺背景とガラスUI、カード中心。

## アーキテクチャ
- Next.js管理画面
- Discord Bot + Supabase
- マッチングロジックはCloudflare Workers
- 通知はDiscord Webhook

## DB設計
- communities(id, owner_id, name, discord_guild_id)
- members(id, community_id, user_id, skills, goals)
- weekly_intents(id, member_id, week_key, challenge)
- matches(id, community_id, week_key, member_a, member_b, member_c)
- feedback(id, match_id, score, memo)

## コスト見積もり（月）
- Supabase: $0〜$25
- Worker/cron: $0〜$5
- OpenAI(要約補助のみ): $2
- 合計: 約$7

## MVPスコープ
- Discord OAuth
- 週次マッチ
- フィードバック回収
- 管理者ダッシュボード

## マーケ計画
- 「週1で創作仲間が見つかる」訴求でX投稿
- Discordコミュニティ運営者に無料トライアル配布

## 技術スタック
Next.js, Supabase, Discord API, Cloudflare Workers, OpenAI API(minimal)

## リスク
- 初期参加者不足 → 3コミュニティ限定で密度を担保
- マッチ不満 → 手動シャッフル機能

## 競合分析
- Discord標準: マッチング機能なし
- Meetup: オフライン寄り
- 差別化: **創作テーマベースの週次自動編成**

## $20達成シナリオ
- Host Lite $6 × 4コミュニティ = $24

## ユニットエコノミクス
- ARPU: $6
- 変動費/顧客: $0.4
- 粗利: $5.6 (93%)
