# 📦 Repo2Teaser

## 概要
📦 Repo2Teaserは、マイクロSaaS（Git更新→告知カード）の月$20到達を狙う小粒プロダクト。

## 海外事例分析
- Typefullyの投稿最適化 / Bannerbearの自動画像生成API市場
- 海外で成立している行動（短尺投稿/コミュニティ連鎖/軽量自動化）を日本語UXに再設計する。

## ターゲット
OSSメンテナ、個人SaaS開発者、受託開発チーム

## 料金
Free（月5カード） / Indie $7/月（月100） / Studio $19/月（チーム）

## ユーザーフロー
1) GitHub連携 2) release/prを検知 3) AIが1行価値訴求を生成 4) OGP・X投稿文を出力

## デザインコンセプト
ターミナル×ポスターのハイブリッド。dark base + accent 1色でブランド維持。

## アーキテクチャ
Webhook ingest (AWS Lambda) + Queue + renderer + Next.js dashboard

## DB設計
- users, repos, events, teaser_cards, templates, publish_logs

## コスト見積もり
Webhook主体で常時監視不要。画像生成はHTMLレンダでAPI費ゼロ。AI費は月$2〜6。

## MVPスコープ
GitHub App連携、PR/Releaseトリガー、カード生成、X/LinkedInコピペ出力

## マーケ計画
Product Huntローンチ準備勢へ「更新してるのに伝わらない」を解消訴求。

## 技術スタック
Node.js, AWS Lambda, SQS, DynamoDB or Supabase, Satori, Stripe

## リスク
GitHub権限懸念→最小read権限。スパム投稿→手動承認フロー。

## 競合分析
Typefullyは手入力中心。Bufferは開発イベント起点が弱い。

## $20達成シナリオ
$7プラン3人で$21達成。ターゲット明確で到達が早い。

## ユニットエコノミクス
ARPU $8.1, 変動費/ユーザー$0.6, 粗利約92%。
