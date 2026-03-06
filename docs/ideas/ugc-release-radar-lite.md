# 🛡️ UGC Release Radar Lite

## 概要
🛡️ UGC Release Radar Liteは、月$20達成を最短で狙う小粒プロダクト。海外で伸びる体験を日本語UXに寄せ、**低固定費・高粗利**で回す。

## 海外事例分析
- 参照: DocuSign / Air.inc / Creator contracts
- 示唆:
  - 「映える出力」や「共有したくなる成果物」が継続率を押し上げる
  - 日本語UIとテンプレ不足が国内ギャップ
  - 小規模課金（$4〜$8）でも十分成立

## ターゲット
- 初期: 個人クリエイター/副業ビルダー/小規模チーム（1〜5名）
- 課題: 作る・見せる・続けるの導線が弱く、毎回手作業になる

## 料金
- Free: 月3回まで
- Starter: $5/月（主要機能解放）
- Pro: $9/月（チーム/履歴/追加容量）

## ユーザーフロー
1. 登録（Google/メール）
2. 入力（画像・URL・メモ等）
3. AI/テンプレ処理
4. 出力（シェアリンク/埋め込み/ダウンロード）
5. リアクション計測→改善提案

## デザインコンセプト
- Dark + Neonアクセント、1画面1目的
- 生成結果を「カード化」してSNS共有しやすく
- 日本語フォント最適化（BIZ UDPGothic + Inter）

## アーキテクチャ
- Front: Next.js (App Router) + Tailwind
- API: Next.js Route Handler
- Job: Cloudflare Workers Cron または GitHub Actions
- Storage: Supabase (Postgres + Storage)
- Auth: Supabase Auth
- Analytics: Plausible（軽量）

## DB設計
- users(id, email, plan, created_at)
- projects(id, user_id, type, source_url, status, created_at)
- outputs(id, project_id, payload_json, share_slug, created_at)
- subscriptions(id, user_id, stripe_customer_id, plan, status)
- usage_events(id, user_id, event_type, tokens, created_at)

## コスト見積もり（月）
- Supabase Free〜$25（MVPは無料枠内想定）
- Vercel/Cloudflare: $0〜$5
- AI API: $3〜$12（キャッシュ前提）
- 合計: **$3〜$17**

## MVPスコープ
- 登録/課金
- コア変換1本
- 共有URL
- 使用量制限
- 最低限の管理画面

## マーケ計画
- Xで制作過程を公開（Before/After）
- Product Hunt / note / Zennで事例投稿
- 初期10ユーザーへ手動オンボード

## 技術スタック
- TypeScript / Next.js / Supabase / Stripe / OpenAI mini系

## リスク
- 海外強豪との機能競争
- AIコスト増
- 初期流入不足

## 競合分析
- 海外: 高機能だが日本語文脈が弱い
- 国内: 同カテゴリ専用の軽量ツールが少ない
- 勝ち筋: **日本語テンプレ + 共有体験 + 低価格**

## $20達成シナリオ
- Starter($5) x 4人 = $20
- または Pro($9) x 3人 = $27
- KPI: 無料→有料転換 4%以上

## ユニットエコノミクス
- ARPU: $6.2
- 変動費/人: $0.7〜$1.8
- 粗利率: 71〜89%
- 回収期間: 1か月以内（有機流入中心）

---

## この案の具体仕様（差別化）
- 入力: クリエイター案件の同意書/使用期間
- 処理: 期限・媒体条件を自動抽出し、超過利用リスクを色分け
- 出力: ブランド提出用の証跡レポート
- 海外トレンド適合: UGC案件増加に伴う契約運用ニーズを軽量SaaS化（本バッチ唯一の保険型）
