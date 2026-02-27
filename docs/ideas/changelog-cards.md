# 📣 Changelog Cards

## 概要
📣 Changelog Cardsは、マイクロSaaS（Indie Hacker向け配布/広報）に特化した月$20達成向けマイクロプロダクト。AIエージェントのみで2週間MVP実装を前提に、低コスト運用で黒字化を狙う。

## 海外事例分析
- 参照トレンド: Loom/Linearの更新共有文化、SNSでのリリース告知カード需要
- 示唆: 海外では「汎用ツール」か「高価格SaaS」が主流。日本向けには**狭い用途+日本語UX最適化**が刺さる。
- 日本でのギャップ: 使い方が難しい/英語前提/日本文化の導線不足。

## ターゲット
個人開発者、OSSメンテナ、小規模SaaS運営者

## 料金
Free / Maker $4 / Team $10

## ユーザーフロー
①GitHub連携→②コミット/PRを要約→③SNSカード自動生成→④X/LinkedInへワンクリ投稿

## デザインコンセプト
Apple風グラデ背景+角丸大。1枚で「更新が伝わる」ミニマルUI。

## アーキテクチャ
GitHub App + Queue(Upstash) + Node worker + S3互換ストレージ

## DB設計
主要テーブル: users, repos, releases, card_templates, generated_cards, publish_logs

## コスト見積もり
10有料ユーザーで月$1.9（ほぼ画像生成/保存のみ）

## MVPスコープ
GitHub同期、更新要約、カード生成、予約投稿（Xのみ）

## マーケ計画
#buildinpublic の毎週投稿テンプレ提供、GitHub READMEバッジ配布

## 技術スタック
Node.js, Next.js, Supabase, Upstash, Cloudflare R2

## リスク
X API制約。初期はダウンロード方式を主にして依存低減。

## 競合分析
Canva手動作成より圧倒的に速い。更新データ連動が核。

## $20達成シナリオ
Maker $4×5人 = $20/月（最短到達）

## ユニットエコノミクス
ARPU $4.8、変動費/人 $0.09、粗利率98%
