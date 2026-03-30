# 🛡️ ConsentScope Mini

## 概要
🛡️ ConsentScope Miniは、保険型（AI生成物の利用許諾ログ管理）を狙う月$20到達向けの超小型プロダクト。

## 海外事例分析
- 参照トレンド: DocuSign clickwrap / TermsFeed consent log / Gumroad license notes
- 示唆: 海外では「短時間で成果が出る」「見た目がシェアしやすい」体験が伸びている。
- 日本向け再設計: 導入ハードルを下げ、テンプレ主導で初回価値を10分以内に提供。

## ターゲット
- AI画像・音声を商用利用する個人クリエイター

## 料金
- $5/月（100ログまで）

## ユーザーフロー
- 作品アップロード→利用範囲テンプレ選択→相手が同意→署名URL保管→PDFエクスポート

## デザインコンセプト
- 法務感を薄めたクリーンUI。証跡をワンクリックPDF化。

## アーキテクチャ
- Next.js + Supabase + PDF generator + signed URL

## DB設計
- 主テーブル: users, assets, consent_requests, signatures, exports
- 最小ER方針: usersを親に、課金対象エンティティを1:Nで接続。監査用created_atを全テーブルに付与。

## コスト見積もり（AWS or 無料サービス想定）
- 固定費$0〜7、PDF生成$0.01/件
- AWS移行時: CloudFront + Lambda + DynamoDBでも月$5〜15で運用可能。

## MVPスコープ
- 同意リンク、記録一覧、PDF出力、期限リマインド

## マーケ計画
- AIクリエイター向けコミュニティで「無償テンプレ配布」から導線化
- 初月KPI: 無料登録30 / 有料転換3〜5。

## 技術スタック
- Next.js, Supabase, Stripe, React-PDF

## リスク
- 主リスク: 法務要件の誤解
- 対策: テンプレ改善を週次で回し、解約理由を5件単位で反映。

## 競合分析
- 大手電子契約は高価。ConsentScopeは個人向け最小機能で安価

## $20達成シナリオ
- 有料4人で$20、原価約$1.5

## ユニットエコノミクス
- ARPU $5, 変動費$0.4, 粗利率92%
- 目安LTV/CAC: 3以上（SNSオーガニック前提）。
