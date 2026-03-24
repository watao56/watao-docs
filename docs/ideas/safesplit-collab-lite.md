# 🛡️ SafeSplit Collab Lite

## 概要
（保険型枠 1/5）
コラボ案件で揉めやすい「成果物の権利範囲・再利用可否」を、テンプレ質問に答えるだけで1枚契約サマリ化。監視ではなく**事前合意デザイン**に寄せた軽量ツール。

## 海外事例分析
- DocuSign/契約管理は重く高価。
- クリエイター向けの「ライトな合意UI」は海外で増加。
- 日本は法務以前の口頭合意が多く、導入障壁が低い。

## ターゲット
- デザイナー×編集者
- インフルエンサー案件の小規模チーム

## 料金
- Pay-as-you-go: **$1/合意書**
- Mini Plan: **$4/月（10件）**

## ユーザーフロー
1. テンプレ選択
2. 5問回答
3. 合意サマリ生成
4. 共有リンクで確認
5. PDF出力

## デザインコンセプト
- 法務感を薄めたカードUX
- 「怖くない契約」デザイン

## アーキテクチャ
- Front: Next.js
- API: Supabase Edge
- PDF: Puppeteer
- Storage: R2

## DB設計
- users(id, plan)
- agreements(id, owner_id, title, status)
- clauses(id, agreement_id, type, value)
- signatures(id, agreement_id, signer_email, agreed_at)

## コスト見積もり（月）
- Hosting/DB: $0〜$5
- AI整形: $3
- 合計: **$8**

## MVPスコープ
- 3テンプレ
- 合意サマリ
- 共有確認
- PDF出力・課金

## マーケ計画
- 制作コミュニティへの配布
- 案件トラブル事例コンテンツで訴求

## 技術スタック
Next.js / Supabase / Stripe / Puppeteer

## リスク
- 法的有効性の誤解
- 免責文言の整備不足

## 競合分析
- 契約SaaS: 重厚
- 本案: **事前すり合わせ専用の超軽量**

## $20達成シナリオ
- $4プラン × 3 + 単発$1 × 8 = $20

## ユニットエコノミクス
- ARPPU: $4相当
- 変動費: $0.4
- 粗利率: 85%
