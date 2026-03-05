# 🧰 FAQForge Widget

## 概要
問い合わせログ（Gmail/フォーム）を読み、**24時間でFAQウィジェットを自動更新**するマイクロSaaS。海外のsupport deflection文脈を、ノーコード導入に寄せた小粒SaaS。

## 海外事例分析
- Intercom Fin: 高機能だが高価格
- HelpScout Docs: 手動更新コストが高い
- Crisp FAQ: 導入は簡単だが自動最適化が弱い

## ターゲット
- 1〜10人のEC/個人SaaS
- CS専任がいない事業者

## 料金
- Starter: **$8/月**（1サイト、月300問い合わせまで）
- Plus: $15/月（3サイト）

## ユーザーフロー
1. Gmailまたはフォーム接続
2. FAQ候補を自動生成
3. 承認ワンクリック
4. JSタグで埋め込み

## デザインコンセプト
- 管理画面は“受信トレイ風”で直感的
- 公開側はミニマル（角丸カード）

## アーキテクチャ
Next.js API + Supabase + OpenAI mini + Cloudflare Workers配信

## DB設計
- workspaces(id, name, plan)
- sources(id, workspace_id, type, config_json)
- faq_items(id, workspace_id, q, a, status, score)
- widgets(id, workspace_id, embed_key, theme)

## コスト見積もり（月）
- Hosting/DB: $8
- AI要約生成: $4
- 合計: **$12**

## MVPスコープ
- Gmail連携
- FAQ候補生成
- 公開ウィジェット
- クリック計測

## マーケ計画
- 「問い合わせ削減率」実績投稿
- Shopify小規模店向けDM営業
- Product Hunt mini launch

## 技術スタック
Next.js, Supabase, OpenAI, Cloudflare Workers, Stripe

## リスク
- 誤回答: 承認フロー必須
- 初期データ不足: テンプレFAQ提供

## 競合分析
- Intercom系は高価格
- 差別化: **低価格・導入10分・承認型AI**

## $20達成シナリオ
- Starter($8)×3ユーザー = $24

## ユニットエコノミクス
- ARPU: $8
- 変動費/人: $1.1
- 粗利率: 86%
