# 🧩 UGCSlot Lite

## 概要
EC運営者が商品情報を入れるだけで、TikTok/リール向けの15秒UGC台本を10本自動生成。フック・構成・CTAをテンプレ化。

## 海外事例分析
- 参考サービス: **Arcads / Foreplay / Motion AI ad scripts**
- 海外では「短時間で見栄えの良い成果物を作る」需要が強い。日本市場では日本語UI・日本語テンプレ不足が参入余地。

## ターゲット
Shopify小規模店舗、D2C個人ブランド、運用代行の副業者

## 料金
Starter $8/月（200台本） / Pro $16/月（1000台本+ブランドトーン）

## ユーザーフロー
(1)商品URL貼付→(2)AIが特徴抽出→(3)台本候補表示→(4)編集→(5)CSV/Notionへ出力

## デザインコンセプト
ガチャUIのように「次の1本」を引ける体験。

## アーキテクチャ
Next.js + Firecrawl(or jsdom fetch) + OpenAI mini + Supabase + Stripe

## DB設計（MVP）
- users, products, script_generations, script_templates, exports, subscriptions

## コスト見積もり（月次）
1台本あたり約$0.0008。Starter上限200本でも$0.16。

## MVPスコープ（2週間）
URL解析、台本10本生成、トーン切替、エクスポート、課金

## マーケ計画
Shopifyコミュニティ/国内D2C Discordへ「無料50本」配布。

## 技術スタック
TypeScript, Next.js, Supabase, OpenAI mini, Stripe

## リスク
生成が凡庸→業種別テンプレを先に手作業で20種作成し品質安定。

## 競合分析
汎用AIより「UGC広告フォーマット特化」で時短価値を出す。

## $20達成シナリオ
Starter 3人で$24達成。

## ユニットエコノミクス
ARPU $9.5、粗利96%。サポート工数少。
