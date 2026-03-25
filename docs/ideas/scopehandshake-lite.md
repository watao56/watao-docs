# 🤝 ScopeHandshake Lite

## 概要
（保険型 1/5）
フリーランス案件の「追加作業」をチャット上で即合意し、**1クリックで変更履歴を残す**ミニSaaS。請求漏れ・認識齟齬を軽く防ぐ。

## 海外事例分析
- **Bonsai/HelloBonsai**: 契約・請求は強いが重い
- **Notion+手動運用**: 軽いが証跡が散らばる
- ギャップ: 日本の小規模案件向けの“超軽量変更合意”特化が少ない

## ターゲット
- 1〜3人の制作チーム
- Web制作・デザイン受託フリーランス

## 料金
- Free: 月5件
- Pro: $5/月（無制限、PDF出力）

## ユーザーフロー
1. 依頼内容を貼る
2. AIが変更差分を3行要約
3. 相手が承認リンクをクリック
4. タイムスタンプ付き履歴保存

## デザインコンセプト
「**握手カード**」。合意前/後が視覚的に一目で分かる。

## アーキテクチャ
- Next.js
- Supabase
- Cloudflare Workers（署名リンク）
- OpenAI API（差分要約）

## DB設計
- users(id, plan)
- projects(id, user_id, client_name)
- changes(id, project_id, before_text, after_text, summary)
- approvals(id, change_id, approver_email, approved_at, signature_hash)

## コスト見積もり（月）
- Hosting: $0
- Supabase: $0
- LLM: $2
- Email: $1
- 合計: **$3**

## MVPスコープ
- 変更要約
- 承認リンク
- PDFエクスポート

## マーケ計画
- フリーランス向けコミュニティでテンプレ配布
- 「追加作業の断り方」記事経由で流入

## 技術スタック
Next.js, Supabase, Cloudflare Workers, OpenAI API, Stripe

## リスク
- 法的効力の誤解 → 利用規約で「補助記録」と明記

## 競合分析
- Bonsai: 機能過多で月額高め
- 電子契約SaaS: 導入ハードル高い

## $20達成シナリオ
- Pro 4ユーザー × $5 = $20/月

## ユニットエコノミクス
- ARPU: $5
- 変動費/人: $0.5
- 粗利率: 90%
