# 🫂 BuildPair Lobby

## 概要
毎日15分だけ、同じ進捗温度のビルダー同士を自動ペアリングする**超短時間コミュニティSaaS**。音声会議不要、テキスト進捗交換だけで成立。

## 海外事例分析
- **Focusmate**: body doubling需要が強い
- **Lunchclub**: 軽量マッチング体験が定着
- **Indie Hackers build-in-public**: 毎日小さく報告する文化
- 日本では「短時間・非通話・開発者向け」ペアリングが少ない。

## ターゲット
- 個人開発者
- 副業クリエイター
- 小規模スタートアップの初期メンバー

## 料金
- Free: 週3マッチ
- Pro: $5/月（無制限）
- Team: $12/月（3席）

## ユーザーフロー
1. 毎朝「今日やる1タスク」を入力
2. 近いテーマの相手が自動マッチ
3. 15分後、成果スクショを交換
4. 連続達成バッジを獲得

## デザインコンセプト
- ロビー掲示板風UI
- ネオン色の進捗バッジでゲーム性

## アーキテクチャ
- Next.js + Supabase Realtime
- マッチング: cron + スコアリング関数
- 通知: Discord webhook / email

## DB設計
- users(id, name, timezone, plan)
- intents(id, user_id, text, tags, target_date)
- matches(id, user_a, user_b, status, created_at)
- checkins(id, match_id, user_id, proof_url, done)

## コスト見積もり（月）
- Supabase Pro: $25未満（初期はFree）
- Vercel: $0
- 通知: $0〜$3
- 合計: **$3〜$10**（初期は無料枠運用可）

## MVPスコープ
- 1日1回マッチング
- 進捗投稿
- 連続日数バッジ
- Stripe課金

## マーケ計画
- Xで「#15minbuild」ハッシュタグ連携
- Discordコミュニティに無料枠配布
- ランキング画像をシェア導線化

## 技術スタック
Next.js, Supabase, Upstash Redis, Stripe, Resend

## リスク
- マッチ品質不足 → タグ重み付けと手動スキップ
- 離脱率 → streak/称号で継続動機を設計

## 競合分析
- Focusmateは通話中心、開発者向け非通話特化に余地
- Discord運用は無料だが、記録/可視化が弱い

## $20達成シナリオ
- Pro 4人で達成
- またはTeam 2チームで達成

## ユニットエコノミクス
- Pro ARPU $5
- 原価（通知/DB）$0.4/人
- 粗利率 **92%**
