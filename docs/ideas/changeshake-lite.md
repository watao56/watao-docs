# 🤝 ChangeShake Lite

## 概要
**ChangeShake Lite**は、制作・開発の追加依頼を「1画面で見積り/納期/影響範囲」に変換し、クライアントのワンクリック同意を取得するミニ保険型SaaS。  
請求漏れ・認識齟齬を減らし、関係性を壊さず守る。

## 海外事例分析
- **StopScopeCreep**: フリーランス向け変更要求管理
- **Clickwrap系（SpotDraft等）**: 軽量同意ログの実務浸透
- 示唆: 日本では契約前提より、**案件進行中の追加合意**がニーズ大

## ターゲット
- Web制作フリーランス
- 小規模受託チーム
- ノーコード制作者

## 料金
- Lite: **$4/month**（月20件の変更依頼）
- Pro: **$9/month**（無制限＋PDF出力）

## ユーザーフロー
1. 追加依頼を入力
2. AIが影響範囲を整形
3. 金額/納期を提示
4. URL共有→相手が承認
5. 履歴を案件別に保存

## デザインコンセプト
- 見積書っぽさを排除し、チャットカード型
- 「拒否しにくい圧」を避けるニュートラル配色
- 1案件1タイムライン

## アーキテクチャ
- Next.js + Supabase
- 署名代替: Click + IP + Timestamp
- PDF: serverless renderer

## DB設計
- projects(id, owner_id, client_name)
- change_requests(id, project_id, summary, delta_price, delta_days, status)
- approvals(id, change_request_id, approved_by, approved_at, ip_hash)
- invoices(id, project_id, total_delta)

## コスト見積もり（月）
- Supabase: $5〜$25
- PDF生成: $2
- AI整形: $3
- 合計: **$10〜$30**

## MVPスコープ
- 変更依頼テンプレ
- 同意リンク
- 履歴一覧
- CSV/PDF出力

## マーケ計画
- フリーランス向け「スコープ崩壊回避」事例発信
- 制作コミュニティでテンプレ配布
- 既存見積書ツールからの乗換導線

## 技術スタック
Next.js / Supabase / Stripe / OpenAI mini

## リスク
- 法的有効性の期待値管理
- クライアント拒否時の運用
- 似たツールの参入

## 競合分析
- 汎用請求ツール: 変更管理が弱い
- 法務SaaS: 中小には重い
- 本案: **追加依頼だけに特化**

## $20達成シナリオ
- Pro 3人 = $27/月
- または Lite 5人 = $20/月

## ユニットエコノミクス
- 粗利率: 90%前後
- 変動費低く、少人数でも黒字化しやすい
