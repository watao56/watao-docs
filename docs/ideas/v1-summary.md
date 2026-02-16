# 🏆 V1 プロダクトまとめ

初期に設計・レビューした5つのプロダクト群。「月$20の収益」を目標に、個人開発者・フリーランス向けのSaaSを中心に設計。

---

## 一覧

| # | プロダクト | カテゴリ | 評価 | 月額 | 月$20達成 | 技術スタック |
|---|-----------|---------|------|------|----------|-------------|
| 1 | [⏰ CronFailsafe](cron-failsafe.md) | cron死活監視 | **B+** | $4/月 | 4ヶ月 | Lambda + DynamoDB + EventBridge |
| 2 | [📋 ContractAlert](contract-alert.md) | 契約更新リマインダー | **B+** | $3/月 | 3ヶ月 | Lambda + DynamoDB + SES/LINE |
| 3 | [🏷️ Namer](namer.md) | AI命名+ドメイン空きチェック | **A-** | $2/回 | 初月〜1ヶ月 | GPT-4o-mini + RDAP + Stripe |
| 4 | [🖼️ OGP Gen](ogp-gen.md) | OGP画像生成API | **B+** | $3/月 | 3ヶ月 | Satori + Resvg-js + CloudFront |
| 5 | [🐕 DomainWatchdog](domain-watchdog.md) | ドメイン/SSL/DNS監視 | **B+** | $4/月 | 3ヶ月 | Lambda + RDAP/TLS/DNS |

---

## 各プロダクト概要

### ⏰ CronFailsafe — cronジョブ死活監視

cronやバッチ処理が「実行されなかった」ことを検知・通知するSaaS。`curl` 一行でジョブに組み込み、期待時間内にpingが来なければSlack/Emailで通知。

- **ペイン**: cronが数日〜数週間止まっていても気づかない
- **強み**: 導入が極めて簡単、低コスト運用
- **課題**: Healthchecks.io（OSS・無料20ジョブ）との差別化
- **対策**: Free枠拡大、日本語UI、cron以外の定期実行もカバー
- 📄 [設計書](cron-failsafe.md) / [レビュー](cron-failsafe-review.md)

### 📋 ContractAlert — 契約更新・解約期限リマインダー

SaaS・保険・賃貸等の契約更新日・解約期限を一元管理し、最適タイミングでリマインダーを送るサービス。

- **ペイン**: 不要SaaSの自動更新で毎月数千円〜数万円が無駄に
- **強み**: ROI訴求しやすい（月$3で年間数万円節約）、日本語ニッチ
- **課題**: 手動入力の面倒さによる離脱リスク
- **対策**: MVPからSaaSテンプレート（AWS, GitHub, Netflix等50個）を用意
- 📄 [設計書](contract-alert.md) / [レビュー](contract-alert-review.md)

### 🏷️ Namer — AI命名+ドメイン/SNS空き確認 ⭐最有望

サービス名・会社名をAIで100案生成し、ドメイン空き・SNSアカウント空きをリアルタイムチェック。都度課金モデル。

- **ペイン**: 名前を考えるのに何日も悩む → ドメインが空いていない → 無限ループ
- **強み**: 低い課金ハードル（$2/回）、SEOポテンシャル大、ドメインアフィリエイト副収入
- **課題**: Stripe小額決済の手数料（$2の19%）
- **対策**: 5回パック$8を推奨表示、最低$3に引き上げ検討
- 📄 [設計書](namer.md) / [レビュー](namer-review.md)

### 🖼️ OGP Gen — ノーコードOGP画像生成API

ブログ記事のOGP画像をURL一つで自動生成するAPIサービス。Satori + Resvg-jsでSVG→PNG変換。

- **ペイン**: 記事ごとにCanvaでOGP画像を作るのが面倒 → SNSのCTR低下
- **強み**: URL一行で完結するUX、技術的にシンプル、低コスト
- **課題**: Vercel og-imageとの差別化
- **対策**: Hugo/WordPress連携ガイドを早期公開
- 📄 [設計書](ogp-gen.md) / [レビュー](ogp-gen-review.md)

### 🐕 DomainWatchdog — ドメイン/SSL/DNS一括監視

保有ドメインの期限切れ、SSL証明書有効期限、DNS変更を一括監視してアラートを送るサービス。

- **ペイン**: ドメイン失効で数十万円の取り戻しコスト、SSL期限切れで売上直撃
- **強み**: ドメイン期限+SSL+DNSの3統合監視はユニーク、CronFailsafeとシナジーあり
- **課題**: ニッチすぎてマーケットが小さい可能性
- **対策**: CDN利用ドメインの誤検知除外リスト、Web制作会社へのアプローチ
- 📄 [設計書](domain-watchdog.md) / [レビュー](domain-watchdog-review.md)

---

## V1の共通特徴

- **ターゲット**: 主に個人開発者・フリーランスエンジニア
- **アーキテクチャ**: AWS サーバーレス（Lambda + DynamoDB + EventBridge）
- **運用コスト**: 月$3〜5程度
- **MVP期間**: 2週間
- **収益モデル**: フリーミアム（Free→Pro転換）またはペイパーユース

## V1からの学び → V2・V3へ

| 学び | V2以降での改善 |
|------|---------------|
| B+評価が多く「良いが突出しない」 | V2は全A評価を目標に設計 |
| 競合との差別化が「安い」だけになりがち | V2は「ないと困る」保険型に特化 |
| エンジニア向けに偏りすぎ | V3は非エンジニア・ライト層をターゲットに |
| AI依存のコスト不確実性（Namer） | V2はAI不使用で設計、V3も4/5がAI不使用 |
