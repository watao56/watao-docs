# 🧩 PromptGuild

## 概要
毎日1テーマで「AIプロンプト対戦」をする小規模コミュニティSaaS。投票とリミックス機能で学習と交流を同時に回す。海外のPrompt marketplace文化を、**少人数ギルド制**で日本向けに最適化。

## 海外事例分析
- PromptBase: 売買は強いがコミュニティ継続が弱い
- Midjourney Discord文化: リミックス駆動の学習が伸びる
- Partiful的イベントUX: 参加導線の軽さが重要

## ターゲット
- 生成AIを学びたい副業層
- デザイナー/マーケの小チーム
- AI勉強会運営者

## 料金
- Free: 1ギルド参加
- Guild Pro: **$6/月**（私設ギルド3つ、投票分析）
- Community Pack: $12/月（管理者向け）

## ユーザーフロー
1. テーマ配信（例: 春のLPヒーロー）
2. プロンプト投稿
3. リミックス or 投票
4. 週次でベスト集を自動カード化

## デザインコンセプト
- カードバトルUI（勝敗が視覚的）
- ダーク＋アクセント1色で集中
- スコア推移を“ゲーム感覚”で表示

## アーキテクチャ
Next.js + Supabase Realtime + Cloudflare Images + OpenAI mini(要約のみ)

## DB設計
- guilds(id, owner_id, name)
- challenges(id, guild_id, topic, closes_at)
- submissions(id, challenge_id, user_id, prompt_text, output_url)
- votes(id, submission_id, user_id)
- plans(id, user_id, tier)

## コスト見積もり（月）
- Supabase: $0〜$25（初期はFree）
- Hosting: $0
- AI要約: $3
- 合計: **$6〜$10**

## MVPスコープ
- ギルド作成
- デイリーテーマ
- 投稿/投票
- 週次ハイライト生成

## マーケ計画
- Discordコミュニティへ無料ギルド配布
- 「今日のお題」画像をXへ自動投稿
- AI勉強会運営者と提携

## 技術スタック
Next.js, Supabase, TypeScript, Stripe, Cloudflare

## リスク
- 過疎化: 自動テーマ供給と週次ランキングで維持
- 低品質投稿: 最低文字数/タグ制限

## 競合分析
- PromptBase: 売買中心
- Discord: 情報流速が速すぎる
- 差別化: **継続学習×対戦UX**

## $20達成シナリオ
- Guild Pro($6)×4人 = $24

## ユニットエコノミクス
- ARPU: $6
- 変動費/人: $0.6
- 粗利率: 90%
