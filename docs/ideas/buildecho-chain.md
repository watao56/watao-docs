# 🔗 BuildEcho Chain

## 概要
🔗 BuildEcho Chainは、ソーシャル/コミュニティ（連鎖型ビルド報告）の月$20到達を狙う小粒プロダクト。

## 海外事例分析
- Genevaのクローズドコミュニティ運用 / Locketの軽量投稿体験
- 海外で成立している行動（短尺投稿/コミュニティ連鎖/軽量自動化）を日本語UXに再設計する。

## ターゲット
build in public層、勉強会主催者、Discordコミュニティ運営者

## 料金
Free（2チェーン参加） / Member $5/月（10チェーン） / Host $15/月（主催機能）

## ユーザーフロー
1) 1日1回15秒報告を投稿 2) 次の1人を指名 3) 連鎖が24h続くとChain Badge発行 4) Discordへ自動共有

## デザインコンセプト
ポップなチェーン可視化（ノードが光る）。達成時にカード生成してSNSに貼れる。

## アーキテクチャ
Next.js + Supabase Realtime + Discord Bot + cron workers

## DB設計
- users, chains, chain_members, posts, mentions, badges, webhooks

## コスト見積もり
無料構成中心。画像生成はSatori+Resvgで自前。10有料ユーザー時でも$1未満。

## MVPスコープ
チェーン作成、日次投稿、指名通知、バッジ発行、Discord投稿

## マーケ計画
海外のbuild-in-public文化を日本語導線化。既存Discordコミュニティへbot導入で拡散。

## 技術スタック
Next.js, Supabase, Upstash Redis, Discord API, Cloudflare Workers

## リスク
投稿疲れ→15秒制限で負担低減。荒らし→招待制+モデレーション。

## 競合分析
Discord単体は習慣化設計が弱い。Focusmateは同期前提で重い。

## $20達成シナリオ
$5プラン4人で$20到達。ホスト1人契約なら単月即達成。

## ユニットエコノミクス
ARPU $7.2, 変動費/ユーザー$0.1, 粗利約98%。
