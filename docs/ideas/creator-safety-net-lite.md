# 🛡️ Creator Safety Net Lite

## 概要
案件の「証拠不足」トラブルを防ぐため、チャット・納品URL・修正履歴を自動タイムライン化して保全する保険型ミニSaaS。揉めた時に提出できる**時系列証跡パック**をワンクリック生成。

## 海外事例分析
- Bonsai: 契約/請求は強いが証跡収集は限定的
- Frame.io: クリエイティブレビューは強いが業務証憑化が弱い
- 示唆: 日本フリーランスは法務より“すぐ出せる証拠”価値が高い

## ターゲット
- 動画編集/デザインの受託フリーランス
- SNS運用代行

## 料金
- Lite: $5/月（3案件）
- Pro: $12/月（無制限）

## ユーザーフロー
1. 案件を作成
2. 納品URL/チャットログを貼る
3. 自動で時系列証跡化
4. PDFパックを生成して保存

## デザインコンセプト
「Forensic Clean」：白基調、証跡の信頼感を重視。

## アーキテクチャ
- Next.js
- Supabase Storage
- PDF generator (Playwright)

## DB設計
- clients(id, name, contact)
- projects(id, client_id, start_date, status)
- evidence(id, project_id, source_type, captured_at, hash)
- exports(id, project_id, pdf_url)

## コスト見積もり（月）
- Hosting: $5
- Storage: $2
- OCR/LLM補助: $1
- 合計: 約$8

## MVPスコープ
- 手動入力中心
- PDF証跡パック
- ハッシュ付与

## マーケ計画
- フリーランス向け「揉めない納品テンプレ」配布
- クラウドソーシング系コミュニティ投稿
- 失敗談→回避法の短尺発信

## 技術スタック
Next.js / Supabase / Playwright PDF / Stripe

## リスク
- 法的助言と誤解される → 免責表示を明確化
- 自動取得の難易度 → MVPは手動＋半自動

## 競合分析
契約管理ではなく「証拠提出の即応性」を売る。

## $20達成シナリオ
Lite($5)×4人 = $20/月

## ユニットエコノミクス
- ARPU: $6.2
- 変動費/人: $0.2
- 粗利: $6.0（96.7%）
