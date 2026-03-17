# 🍽️ DinnerSprint Club

## 概要
「同じテーマで挑戦する3〜5人」を自動マッチし、**月1回の小規模ディナー/オンライン雑談会**を運営できるコミュニティ運営SaaS。Partiful/Luma的なイベント体験を、実行コミュニティ向けに最適化。

## 海外事例分析
- Partiful/Lumaはイベント招待体験で急成長
- Geneva/Discordコミュニティは活発だが、オフライン接続導線が弱い
- 日本ではイベント作成はあるが、**継続マッチング運営特化**は少ない

## ターゲット
- 小規模コミュニティ運営者
- コワーキング運営
- 副業コミュニティ主催者

## 料金
- Free: 1グループ・月1イベント
- Host: $8/月（5グループ）
- Community: $18/月（無制限 + CSV分析）

## ユーザーフロー
1. コミュニティ作成
2. 参加者が興味タグ・予算・場所を登録
3. AIが毎週マッチ候補を提示
4. 主催者が承認→招待リンク配信
5. 開催後に満足度回収・次回改善

## デザインコンセプト
- 「行きたくなるイベント台帳」
- カレンダーよりカード重視
- warm color + 写真大きめで感情導線

## アーキテクチャ
- Next.js + Supabase
- マッチング: OpenAI embeddings + pgvector
- 通知: Discord Webhook / Email
- 画像: Cloudinary free tier

## DB設計
- communities(id, owner_id, name)
- members(id, community_id, profile, tags)
- events(id, community_id, theme, date, mode)
- matches(id, event_id, member_ids, score)
- feedback(id, event_id, member_id, rating, note)

## コスト見積もり（月）
- Supabase: $0〜$25
- AIマッチング: $3
- メール/通知: $2
- 合計: 約$5〜$30

## MVPスコープ
- タグ登録
- 自動マッチ候補
- 招待ページ
- フィードバック回収

## マーケ計画
- 「月1ミニ会」テンプレを無料配布
- 主催者向けNotion運営テンプレ連携
- X/Discordで成功事例を連投

## 技術スタック
Next.js, Supabase, pgvector, Resend, Discord webhook, Stripe

## リスク
- 初期コミュニティ母数不足
- オフライン会の運営責任範囲

## 競合分析
- Partiful: イベント単発向け
- Luma: 広範だが継続運営分析が浅い
- 差別化: **少人数継続コミュニティの反復運営**

## $20達成シナリオ
- Host 3人で$24
- 変動費低く粗利90%超

## ユニットエコノミクス
- ARPU: $8
- COGS/user: $0.6
- 粗利: $7.4（92.5%）
- LTV/CAC: 4.0以上を目標
