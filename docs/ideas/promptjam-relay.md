# 🎤 PromptJam Relay

## 概要
PromptJam Relayは、クリエイター同士が24時間で「お題→成果物→フィードバック」を回す**超軽量コミュニティSaaS**。海外のDiscordコミュニティ文化（daily challenge）を、課金可能な「運営テンプレ」付きで日本向けに商品化。

## 海外事例分析
- **Flow Club / Focusmate**: 同時接続型は強いが、成果物展示の導線が弱い。
- **Daily UI / Dribbble challenge**: 参加熱量は高いが継続運営が大変。
- 余地: 1日完結の“回す仕組み”をSaaS化して運営工数を削減。

## ターゲット
- デザイン/動画コミュニティの運営者
- オンラインサロンの小規模主催者
- 学習コミュニティ（生成AI勉強会）

## 料金
- Free: 1ルーム・10人まで
- Host: $8/月（3ルーム・自動集計）
- Guild: $15/月（無制限・成績ボード）

## ユーザーフロー
1. 主催者がお題テンプレ作成
2. 参加者が24h以内に投稿
3. AIが講評ドラフト生成
4. 相互投票でデイリーランキング
5. 次回お題を自動生成

## デザインコンセプト
- **Retro Stage**: ネオン看板+カード壁
- 成果物が“ギャラリー”として映えるレイアウト
- 参加したくなるゲーム的導線（連続参加バッジ）

## アーキテクチャ
- Front: Nuxt 3
- Backend: Supabase Edge Functions
- Realtime: Supabase Realtime
- AI: OpenAI（講評ドラフト）
- CDN: Cloudflare

## DB設計
- communities(id, owner_id, name, plan)
- rooms(id, community_id, title, cadence)
- prompts(id, room_id, prompt_text, open_at, close_at)
- submissions(id, prompt_id, user_id, artifact_url, note)
- votes(id, submission_id, voter_id, score)
- ai_feedback(id, submission_id, summary, tone)

## コスト見積もり（月）
- Supabase Pro不要（無料〜$25閾値まで）
- Cloudflare: $0
- AI講評: $5
- ストレージ: $2
- **合計: $7前後（初期）**

## MVPスコープ
- 24hチャレンジ固定
- 投稿・投票・ランキング
- AI講評（短文）
- Discordログイン

## マーケ計画
- 既存Discordコミュニティへ「1週間無料運営」提供
- Xで週次ベスト作品を紹介（UGC拡散）
- 主催者向けにテンプレ配布

## 技術スタック
Nuxt3, Supabase, Cloudflare R2, OpenAI, Discord OAuth

## リスク
- 荒らし投稿 → 招待制+通報
- 継続率低下 → 週次テーマ配信
- 競合Discord Botとの比較 → UI展示体験で差別化

## 競合分析
- Discord Bot: 安いが可視化が弱い
- Circle: 高機能だが重い
- PromptJam Relay: 「課題運営」一点特化で小規模に刺さる

## $20達成シナリオ
- Host 3件で$24
- Guild 2件で$30

## ユニットエコノミクス
- ARPU: $9.5
- 変動費/顧客: $1.1
- 粗利率: 88%
- LTV（6ヶ月）: $57想定
