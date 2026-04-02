# 🛡️ PromiseStamp Lite

## 概要
（保険型1案）
納品前に「要件・修正回数・支払条件」を1画面で相互確認し、**スタンプ付き合意ログ**を残す超軽量ツール。契約書ほど重くないが、揉め事を下げる。

## 海外事例分析
- Bonsai/HoneyBookの事前合意UX
- Clickwrapの軽量同意トレンド
- フリーランス市場で「チャット合意のみ」の事故が継続

## ターゲット
- 動画編集/デザイン/実装の個人受託
- 小規模制作チーム

## 料金
- Free: 月3案件
- Pro: $5/月（無制限、PDF出力）
- One-shot: $2/案件（単発）

## ユーザーフロー
1. 案件テンプレ入力
2. クライアントへURL送付
3. 相手が確認・同意
4. タイムスタンプ付き要約を自動保存

## デザインコンセプト
- 「Stamp & Receipt」
- 紙の受領印をモチーフに安心感を演出

## アーキテクチャ
- Front/API: Next.js Route Handlers
- DB: Supabase
- PDF: Browserless + template
- 通知: Resend

## DB設計
- deals(id, owner_id, client_email, scope_text, revisions, fee, due_date)
- approvals(id, deal_id, approver, approved_at, ip_hash)
- snapshots(id, deal_id, summary_json, pdf_url)
- users(id, email, plan)

## コスト見積もり（月）
- Hosting/DB: $8
- メール/PDF: $3
- 合計: **$11**

## MVPスコープ
- 合意フォーム
- タイムスタンプ保存
- PDFエクスポート
- Stripe決済

## マーケ計画
- フリーランス向けチェックリスト配布
- 「契約書まで不要な案件向け」訴求

## 技術スタック
Next.js, Supabase, Stripe, Resend

## リスク
- 法的効力への誤解 → 免責と利用用途を明記
- 既存ツールとの差別化 → 1画面完結・即共有を徹底

## 競合分析
- 電子契約は重い/高い
- 本案は**ライト案件向けの最小同意**に特化

## $20達成シナリオ
- Pro $5 × 4 = $20
- または One-shot $2 × 10 = $20

## ユニットエコノミクス
- ARPPU: $5.4
- 変動費/有料ユーザー: $0.7
- 粗利率: 約87%
