# 🔁 Give2Get Booth

## 概要
**「他人に2件フィードバックすると自分の投稿が上位表示される」**交換ルールを自動化するコミュニティ運営SaaS。海外のgive-firstカルチャーを日本向けにゲーム化。

## 海外事例分析
- Indie Hackers/ADPListの相互レビュー文化
- DiscordのCritique Channel運用課題（偏在、未返信）
- Redditのkarma設計思想（貢献で可視性獲得）

## ターゲット
- デザイン/動画/コピーの学習コミュニティ管理者
- スクール運営者
- 個人運営の有料サロン

## 料金
- Starter: $0（1コミュニティ/週30投稿）
- Pro: $8/月（投稿上限緩和、スコア分析）
- Team: $15/月（複数スペース）

## ユーザーフロー
1. Discord招待 or Webコミュニティ接続
2. 投稿テンプレ（目的/困りごと）でWIP投稿
3. AIがコメント品質を採点
4. 貢献ポイントに応じて自投稿の露出が上がる
5. 週次レポート配信

## デザインコンセプト
- 「Arcade Scoreboard」
- ネオン掲示板風UI、貢献ランキングを軽量表示

## アーキテクチャ
- Front: Next.js
- Backend: Supabase Edge Functions
- Bot: Discord Interactions API
- AI採点: 小型LLM API（コスト上限付き）

## DB設計
- communities(id, owner_id, platform, plan)
- posts(id, community_id, author_id, content, created_at)
- feedback(id, post_id, reviewer_id, quality_score, created_at)
- reputation(id, community_id, user_id, points, tier)

## コスト見積もり（月）
- Supabase Pro相当: $0〜$25（初期は無料枠）
- AI採点: $4
- Bot運用: $2
- 合計: **$6〜$12**（小規模時）

## MVPスコープ
- 投稿/レビュー/ポイント
- 2件レビューでブースト
- Discord通知
- 週次CSVレポート

## マーケ計画
- 「過疎レビューを防ぐbot」としてDiscord運営者へ訴求
- 学習コミュニティ向け無料導入キャンペーン

## 技術スタック
Next.js, Supabase, Discord API, OpenAI/Claude小型モデル

## リスク
- 採点が不公平に感じられる → 手動補正機能
- ゲーム化疲れ → 非表示モード提供

## 競合分析
- 一般フォーラムツールは“交換ルール自動化”が弱い
- 本案は**コミュニティ活性KPI直結**で差別化

## $20達成シナリオ
- Pro $8 × 3コミュニティ = $24

## ユニットエコノミクス
- ARPPU: $8.5
- 変動費/有料コミュ: $1.2
- 粗利率: 約86%
