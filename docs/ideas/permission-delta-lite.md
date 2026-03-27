# 🛡️ Permission Delta Lite

## 概要
Chrome拡張機能の権限変更（例: `tabs` 追加）を検知し、**危険度付きで通知**するライト監視SaaS。保険型は本案のみ。

## 海外事例分析
- **Extension管理の企業セキュリティ需要**は増加
- **Manifest V3移行期**で権限変更が頻発
- 個人向けに「危険度を日本語で説明する」製品はまだ少ない

## ターゲット
- セキュリティ感度の高い個人
- 小規模開発チーム
- ノーコード運用者

## 料金
- Free: 5拡張まで
- Pro: $6/月（50拡張）

## ユーザーフロー
1. 監視対象拡張を登録
2. 日次で権限差分を取得
3. 変更理由テンプレ付きで通知
4. 必要なら無効化手順を表示

## デザインコンセプト
- 「交通信号UI」: 緑/黄/赤で危険度表示
- セキュリティ感より“分かりやすさ”重視

## アーキテクチャ
- Cloudflare Workers cron
- 拡張情報収集 + 差分算出
- 通知: Discord/Email

## DB設計
- users(id, email, plan)
- watched_extensions(id, user_id, extension_id, name)
- permission_snapshots(id, extension_id, perms_json, fetched_at)
- deltas(id, extension_id, level, summary, created_at)

## コスト見積もり（月）
- Workers + D1: $0〜$5
- 通知: $0〜$2
- 合計: **$1〜$7**

## MVPスコープ
- 権限差分検知
- 危険度分類
- 通知
- 変更履歴画面

## マーケ計画
- セキュリティTips系の短文投稿を継続
- 「今週の危険変更ランキング」を無料公開

## 技術スタック
Cloudflare Workers, D1, Hono, Next.js, Stripe

## リスク
- データソース仕様変更
  - 対策: 取得層を抽象化し、fallback APIを用意

## 競合分析
- 企業向け管理製品は高価格
- 個人向けは情報が断片的で、継続監視が弱い

## $20達成シナリオ
- Pro 4人（$24）で達成

## ユニットエコノミクス
- ARPU $6
- 原価 $0.5/人
- 粗利率 **91%**
