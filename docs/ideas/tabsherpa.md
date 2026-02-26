# 🧭 TabSherpa

## 概要
「開きすぎたブラウザタブ」をAIで自動整理し、**仕事モード別のタブセット**を1クリック復元する個人生産性ツール。海外ではtab manager系拡張が人気だが、日本語要約・復帰導線が弱い。

## 海外事例分析
- OneTab/Workona: 保存は強いが内容要約が弱い
- Arc Spaces: 体験は良いがブラウザ依存
- 日本ギャップ: “何のためのタブか”が残らない

## ターゲット
- 情報過多のナレッジワーカー
- PM/デザイナー/エンジニア

## 料金
- Free: 保存3セットまで
- Pro: $4/月（無制限、AI要約、週次レポート）

## ユーザーフロー
1. 拡張機能を導入
2. タブ群を「案件名」で保存
3. AIが内容を3行要約
4. 翌日ワンクリック復元

## デザインコンセプト
- 「コックピット」UI
- 各セットに色とアイコン、迷わない階層

## アーキテクチャ
- Chrome Extension + Next.js API
- 要約: gpt-4o-mini
- メタデータ: Supabase
- 同期: Edge Function

## DB設計
- users(id, email, plan)
- tab_sets(id, user_id, title, color, created_at)
- tabs(id, set_id, url, title, snapshot)
- summaries(id, set_id, summary_text, token_usage)
- weekly_reports(id, user_id, payload_json)

## コスト見積もり（月）
- Supabase: $0
- AI要約: $1
- Extension配布: $0
- 合計: **約$1**

## MVPスコープ
- 保存/復元
- セット名自動提案
- 3行要約

## マーケ計画
- 「タブ100枚→3セット」比較動画
- Product Hunt / X開発者層
- 無料配布テンプレ（職種別セット）

## 技術スタック
TypeScript / Chrome Extension API / Next.js / Supabase / OpenAI mini

## リスク
- ブラウザ権限への心理的抵抗
- 拡張機能ストア審査

## 競合分析
- OneTabより“再開時の文脈復元”で優位
- Workonaより低価格で個人特化

## $20達成シナリオ
- Pro 5人で $20 MRR

## ユニットエコノミクス
- ARPU: $4
- 変動費/人: $0.15
- 粗利率: 96.3%
