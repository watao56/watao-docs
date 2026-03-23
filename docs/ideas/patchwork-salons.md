# 🧵 Patchwork Salons

## 概要
「未完成の作品」だけを持ち寄る**週1の小部屋コミュニティ運営SaaS**。提出→相互フィードバック→次週の改善宣言までをテンプレ化。

## 海外事例分析
- **Geneva / Circle**: 小規模コミュニティの継続率が高い
- **ADPList**: 短時間メンタリング需要
- **Indie Hackers build-in-public**: 未完成公開が拡散導線になる

## ターゲット
- デザイナー/動画編集者/個人開発者
- Discord運営者（既存コミュニティ管理を楽にしたい人）

## 料金
- Host Lite: $8/月（30人まで）
- Host Pro: $15/月（100人まで）

## ユーザーフロー
1. ルーム作成（週次テーマ設定）
2. 参加者が未完成物を投稿
3. 自動で3人にレビュー配布
4. 翌週の改善TODOを自動発行

## デザインコンセプト
「**Craft Notebook**」: 紙ノート風カードUI、投稿を“作品帳”として残す。

## アーキテクチャ
- Next.js + Supabase
- Discord OAuth連携
- OpenAI miniで要約/アクション抽出
- Cloudflare R2に添付保存

## DB設計
- hosts(id, user_id, plan)
- salons(id, host_id, title, schedule)
- posts(id, salon_id, author_id, artifact_url)
- feedback(id, post_id, from_user_id, score, text)
- actions(id, user_id, week, next_step)

## コスト見積もり（月）
- Supabase: $0〜$5
- AI: $1.5
- 合計: **$6.5以下**

## MVPスコープ
- 週次テーマ運用
- 自動レビュー割当
- 次アクション自動要約

## マーケ計画
- 「未完成晒し会」テンプレを無料配布
- Discordコミュニティ管理者へDM営業
- Xスペースで週次発表会

## 技術スタック
Next.js, Supabase, Discord API, OpenAI mini

## リスク
- 初期参加率不足 → テンプレサロンを最初から同梱
- 荒れ対策 → ルール違反自動検知

## 競合分析
- Circle: 汎用的で運営負荷高い
- Discord単体: ワークフロー自動化が弱い

## $20達成シナリオ
- Host Lite 3人 = $24

## ユニットエコノミクス
- ARPU: $8
- 変動費: $0.4/host
- 粗利率: **95%**
