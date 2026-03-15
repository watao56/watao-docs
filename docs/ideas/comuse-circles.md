# 🧩 CoMuse Circles

## 概要
毎週1テーマで作品を持ち寄り、AIが相性の良い3人を組んで**非同期リミックス交換**を回すコミュニティSaaS。海外のBuild in Public文化・Duet文化を、少人数で継続しやすい日本語UXに落とし込む。

## 海外事例分析
- Geneva / Circle: コミュニティ基盤はあるが創作フローは汎用
- ADPList: マッチング体験は強いが創作継続導線は弱い
- TikTok Duet文化: “相互創作”の熱量が高い
- 日本ギャップ: 小さな創作コミュニティの運営工数が高い

## ターゲット
- イラスト/デザイン/動画の個人クリエイター
- 副業でポートフォリオを増やしたい人
- コミュニティ運営者

## 料金
- Free: 参加のみ、月2テーマ
- Member: $5/月（毎週参加、AIマッチング）
- Host: $15/月（自分のサークル運営、分析ダッシュボード）

## ユーザーフロー
1. 自分の作風タグを登録
2. 週テーマに作品アップ
3. AIが3人サークルを編成
4. 互いに24h以内でリミックス返却
5. 週末にベスト作品をカード化して共有

## デザインコンセプト
- **“Club Poster”**
- ネオン色＋粗い紙質の背景
- 作品サムネを壁に貼るようなUI

## アーキテクチャ
- Next.js + Supabase Realtime
- マッチングロジック: Python API（embedding + ルール）
- 通知: Discord Webhook / Email
- 画像保存: S3またはR2

## DB設計
- users(id, name, email, plan)
- circles(id, host_user_id, title, visibility)
- themes(id, circle_id, title, week_start)
- submissions(id, theme_id, user_id, asset_url, tags)
- matches(id, theme_id, user_a, user_b, user_c, status)
- remixes(id, match_id, from_user_id, to_user_id, asset_url)
- subscriptions(id, user_id, plan, status)

## コスト見積もり（月）
- Supabase: $0
- Hosting: $0〜5
- 通知/メール: $1
- embedding/API: $2
- ストレージ: $1
- 合計: **約$4〜9/月**

## MVPスコープ
- 1コミュニティ内運用
- 週次テーマ固定
- 3人マッチングのみ
- Discord通知のみ

## マーケ計画
- 「今週のリミックス」公開ギャラリー
- クリエイター向けDiscordへの体験配布
- Host向け初月無料で運営者獲得

## 技術スタック
Next.js / Supabase / Python(FastAPI) / pgvector / Stripe / Discord Webhook

## リスク
- 初期アクティブ不足でネットワーク効果が出ない
- 作品の権利ルール整備不足

## 競合分析
- Circle: 運営機能中心 → **作品交換ワークフロー特化**
- Discord: 無料だが運用負荷高い → **自動マッチで運営工数削減**

## $20達成シナリオ
- Member 4人（$20 MRR）
- または Host 2人（$30 MRR）

## ユニットエコノミクス
- ARPU: $6.2
- 変動費/人: $0.35
- 粗利/人: $5.85
- 粗利率: **94.3%**
