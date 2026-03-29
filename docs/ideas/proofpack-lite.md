# 🛡️ ProofPack Lite

## 概要
ProofPack Liteは、フリーランスが納品前に「要件合意・修正回数・著作権範囲」を1ページで確認・保存できる**軽量リスクヘッジSaaS**。保険型は1案のみに留めつつ、実務で即課金される領域に絞る。

## 海外事例分析
- **Bonsai / HoneyBook**: 契約管理は強いが小規模案件には重い。
- **Notionテンプレ運用**: 安いが証跡の整合性が弱い。
- 余地: 「納品直前チェック」一点特化で簡潔に。

## ターゲット
- 動画/デザイン制作の副業層
- 月5件未満の小規模受託者

## 料金
- Free: 月2案件
- Solo: $4/月（20案件）
- Pro: $9/月（無制限 + PDF自動生成）

## ユーザーフロー
1. 案件名・合意事項をフォーム入力
2. 相手に確認リンク送付
3. 相手がチェックボックス承認
4. タイムスタンプ付きPDF保管
5. 納品時に証跡を添付

## デザインコンセプト
- **Legal Calm**: 明るい余白+信頼色（ブルーグレー）
- “怖い法務”ではなく“安心チェックリスト”

## アーキテクチャ
- Front: SvelteKit
- Backend: Supabase + Edge Functions
- PDF: Cloudflare Worker + Playwright
- E-sign lightweight: magic link approval

## DB設計
- users(id, email, plan)
- projects(id, user_id, client_name, title, status)
- checkpoints(id, project_id, item_key, item_text, required)
- approvals(id, project_id, approver_email, approved_at, ip_hash)
- artifacts(id, project_id, pdf_url, checksum, created_at)

## コスト見積もり（月）
- Supabase無料枠: $0
- Cloudflare Worker: $0〜$5
- Email送信: $1
- AI（文面整形）: $2
- **合計: $3〜$8**

## MVPスコープ
- 合意チェックテンプレ3種
- 承認リンク
- PDF出力
- 案件一覧

## マーケ計画
- ココナラ/ランサーズ界隈へ実例投稿
- 「納品前30秒チェック」無料テンプレ配布
- 税理士・士業アカウントと提携投稿

## 技術スタック
SvelteKit, Supabase, Cloudflare Workers, Resend, Stripe

## リスク
- 法的効力の誤認 → 利用規約で範囲明記
- 入力手間 → テンプレ化と自動補完

## 競合分析
- 電子契約SaaS: 高機能高単価
- Notionテンプレ: 安価だが証跡弱い
- ProofPack Lite: 低価格・最小証跡に特化

## $20達成シナリオ
- Solo 5人（$20）
- Pro 3人（$27）

## ユニットエコノミクス
- ARPU: $4.8
- 変動費: $0.7
- 粗利率: 85%
