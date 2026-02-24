# 🌿 WidgetGarden

- カテゴリ: マイクロSaaS（埋め込みミニウィジェット）
- 目標: 月$20
- 想定評価: A

## 概要
ノーコードで「カウントダウン・投票・進捗バー・FAQ折りたたみ」など小さな埋め込みウィジェットを作れるマイクロSaaS。LP改善の即効性に特化。

## 海外事例分析
- Elfsight/Tally系のembedツール需要は継続。
- ただし多機能過多で重い。
- 日本の個人事業LPは「軽くて安い」需要が強い。

## ターゲット
- STUDIO/ペライチ/Wix利用者
- 個人SaaSのLP運営者
- マーケ代理店の小案件

## 料金
- Solo $5/月: 3ウィジェット
- Pro $9/月: 20ウィジェット
- Agency $19/月: 100ウィジェット

## ユーザーフロー
1) テンプレ選択
2) 文言/色を編集
3) 埋め込みコード取得
4) LPへ貼付
5) 表示回数とCVR確認

## デザインコンセプト
「Candy Blocks」。角丸大きめ、色はポップだがミニマル。30秒で設置できる体験を重視。

## アーキテクチャ
静的JS配信(CDN) + 管理画面。APIはFastify。トラッキングは軽量イベントのみ。

## DB設計
accounts(id,plan)
widgets(id,account_id,type,config_json,is_active)
events(id,widget_id,event_type,created_at)
sites(id,account_id,domain)

## コスト見積もり（月次）
- CDN/Worker: $0〜$5
- DB: $0〜$5
- 合計: $5〜$8

## MVPスコープ（14日）
- 4種ウィジェット
- テーマ編集
- 埋め込みコード
- 基本分析
- Stripe決済

## マーケ計画
- Before/After LP改善事例を毎週公開
- ノーコード制作者へアフィリエイト
- Product Hunt/JPコミュニティ投稿

## 技術スタック
Next.js, Fastify, PostgreSQL, Cloudflare CDN, Stripe

## リスク
無料競合 → 「日本語フォント最適化」「軽量表示速度」で勝つ。サポート負荷 → テンプレを絞る。

## 競合分析
Elfsightより安価・軽量。Canva埋め込みよりCTR計測が可能。

## $20達成シナリオ
Solo 4人で$20達成、またはPro 3人で$27。導入難易度が低く初速が出やすい。

## ユニットエコノミクス
ARPU $6.5、変動費$0.2、粗利96.9%、LTV/CAC>4を狙える。
