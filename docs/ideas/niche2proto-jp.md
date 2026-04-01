# 🧪 Niche2Proto JP

## 概要
**Niche2Proto JP**は、海外で伸び始めた小粒プロダクトを毎日3件収集し、**日本向けLP文案・機能要件・MVP実装タスク**まで自動で1枚化するマイクロSaaS。  
「見つける」だけでなく「作れる状態」にする。

## 海外事例分析
- **Exploding Topics**: 初期トレンド検知
- **Trends.vc**: 文脈付きの市場理解
- **Indie Hackers**: 小粒課題を高速実装する文化
- 示唆: 日本では“翻訳情報”はあるが“実装直結”が不足

## ターゲット
- 個人開発者
- 受託エンジニアの新規事業枠
- マイクロSaaS運営者

## 料金
- Solo: **$7/month**（日次3件）
- Pro: **$12/month**（日次10件＋CSV）

## ユーザーフロー
1. 興味カテゴリを選択
2. 日次で海外事例カード配信
3. 「JP化する」を押す
4. LP/要件/実装タスク生成
5. GitHub Issueテンプレ出力

## デザインコンセプト
- リサーチノートではなく「発注書」UI
- 1カード1画面、可読性重視
- KPIは“読む数”でなく“着手数”

## アーキテクチャ
- Crawler: Scheduled Workers
- Enrichment: LLM summarization
- App: Next.js + Supabase
- Export: GitHub API連携

## DB設計
- trends(id, source_url, title, score, captured_at)
- jp_briefs(id, trend_id, icp, lp_copy, mvp_scope)
- users(id, email, plan)
- exports(id, user_id, brief_id, kind)

## コスト見積もり（月）
- Fetch/Worker: $6
- LLM要約: $9
- DB/Storage: $5
- 合計: **$20**

## MVPスコープ
- 3ソース収集
- 日次3件配信
- JP化1枚
- GitHub Issue出力

## マーケ計画
- 「海外→日本MVP化」実例を毎週公開
- Indie Hackers日本語圏へ配布
- 1週間無料トライアル

## 技術スタック
Cloudflare Workers / Next.js / Supabase / OpenAI / GitHub API

## リスク
- 情報ソース品質
- 要約の浅さ
- 既存トレンドサービスとの差別化

## 競合分析
- Exploding Topics: 検知中心
- Trends.vc: 分析中心
- 本案: **実装タスク化中心**

## $20達成シナリオ
- Solo 3人 = $21/月
- Pro 2人 = $24/月

## ユニットエコノミクス
- Solo粗利: 約65%
- Pro粗利: 約75%
- LTV改善はテンプレ資産化で対応
