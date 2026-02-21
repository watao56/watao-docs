# 💡 アイデア

プロダクトアイデアと設計書のまとめです。

## 🎯 多様性重視・5カテゴリ設計 v11（2026-02-21）

保険型/監視系から完全脱却。マーケティング/AI×クリエイティブ/パーソナル/チーム生産性/ライティングの5カテゴリで設計。全案海外実証済み、全案A評価。

| プロダクト | カテゴリ | 海外事例 | 評価 | 粗利率 |
|-----------|---------|---------|------|--------|
| [🏆 TestiWall](testiwall.md) ([レビュー](testiwall-review.md)) | テスティモニアル収集Widget | Senja ($50K MRR) | **A** | 99.8% |
| [🎨 MojiForge](mojiforge.md) ([レビュー](mojiforge-review.md)) | AI絵文字/スタンプパック生成 | LINE Creators (¥76億/年) | **A** | 95.5% |
| [🖼️ DreamPaper](dreampaper.md) ([レビュー](dreampaper-review.md)) | AI壁紙ジェネレーター | iOS壁紙アプリ群 | **A** | 89.6% |
| [🔄 RetroFlow](retroflow.md) ([レビュー](retroflow-review.md)) | AIファシリ付き振り返りボード | Retrium ($5M+ ARR) | **A** | 97.3% |
| [✍️ CopyTone](copytone.md) ([レビュー](copytone-review.md)) | AIブランドボイス分析 | Writer.com ($200M+) | **A** | 95.6% |

### v11の特徴
- **5カテゴリ完全分散**: マーケ/クリエイティブ/パーソナル/チーム/ライティング
- **保険型ゼロ**: 全てが「作る・表現する・改善する」系
- **バイラル性重視**: MojiForge/DreamPaperはSNS拡散力が高い
- **B2B〜B2Cミックス**: RetroFlow(B2B) / DreamPaper(B2C) / TestiWall(B2B2C)
- **AI費用$0〜$11/月**: TestiWallはAI不使用、他もGPT-4o-mini/Flux Schnellで低コスト

---

## 🌏 海外発・画期的プロダクト v10（2026-02-20）

保険型/監視系を脱却し、**海外で流行っているが日本未展開**のプロダクトを日本市場向けにローカライズ。デザイン性・インパクト重視。全案A評価。

| プロダクト | カテゴリ | 海外事例 | 評価 | 粗利率 |
|-----------|---------|---------|------|--------|
| [🎙️ KoeLink](koelink.md) ([レビュー](koelink-review.md)) | AI音声メモ→構造化ノート | AudioPen ($50K MRR) | **A** | 80%+ |
| [🎨 BrandKit](brandkit.md) ([レビュー](brandkit-review.md)) | AIブランドアイデンティティ生成 | Looka ($10M+ ARR) | **A** | 99% |
| [🌱 NiwaLog](niwalog.md) ([レビュー](niwalog-review.md)) | ソーシャル・デジタルガーデン | Obsidian Publish / Are.na | **A** | 99%+ |
| [🚀 ShipFolio](shipfolio.md) ([レビュー](shipfolio-review.md)) | ビルダー・ポートフォリオ | Peerlist / Read.cv | **A** | 99%+ |
| [🖼️ OGP Lab](ogplab.md) ([レビュー](ogplab-review.md)) | ダイナミックOGP画像生成API | Bannerbear ($40K MRR) | **A** | 99%+ |

### v10の特徴
- **全案が海外実証済み**: 各プロダクトに$10K〜$10M規模の海外成功事例あり
- **デザインドリブン**: 見せたくなる、使いたくなるUI/UX設計
- **カテゴリ多様性**: AI×クリエイティブ / デザインツール / ソーシャル / ポートフォリオ / 開発者API
- **日本語特化で差別化**: 海外勢が手薄な日本語フォント・文化対応で勝負
- **粗利80〜99%**: KoeLink以外はAI費用ゼロ構造

---

## 🏆 プロダクト設計書 v9（2026-02-20）

v8までの45+案と重複しない新領域を開拓。Webhook配信監視・Webアクセシビリティ・SEOファイル保全・セキュリティヘッダー・構造化データの5案。全てAI不使用、外部API費$0、粗利99%超。

| プロダクト | カテゴリ | 評価 | 月額目標達成 | 粗利率 |
|-----------|---------|------|------------|--------|
| [🔔 WebhookGuard](webhookguard.md) ([レビュー](webhookguard-review.md)) | Webhook配信監視 | **A** | 2ヶ月目（Pro 4人） | 99.2% |
| [♿ A11yPing](a11yping.md) ([レビュー](a11yping-review.md)) | Webアクセシビリティ自動監視 | **A-** | 2ヶ月目（Pro 4人） | 99.9% |
| [🤖 RoboGuard](roboguard.md) ([レビュー](roboguard-review.md)) | robots.txt/sitemap.xml破損検知 | **A** | 2ヶ月目（Pro 5人） | 99.9% |
| [🛡️ HeaderShield](headershield.md) ([レビュー](headershield-review.md)) | HTTPセキュリティヘッダー監視 | **A-** | 2ヶ月目（Pro 4人） | 99.9% |
| [📐 SchemaLint](schemalint.md) ([レビュー](schemalint-review.md)) | 構造化データ破損検知 | **A-** | 2ヶ月目（Pro 4人） | 99.9% |

### v9の特徴
- **全案A評価以上**: WebhookGuard・RoboGuardがA、A11yPing・HeaderShield・SchemaLintがA-
- **全てAI不使用、外部API費$0**: axe-core(OSS)、cheerio(OSS)、テキスト差分のみ
- **「売上損失・法的リスク・SEO壊滅」の実害防止**: 解約=即リスク復活の保険型
- **Web保全スイート構想**: RoboGuard+SchemaLint+HeaderShieldでWebサイト保全を一気通貫
- **新領域**: Webhook監視(WebhookGuard)、アクセシビリティ定期チェック(A11yPing)はブルーオーシャン
- **インフラコスト$0〜$0.05/月**: 50ユーザー時でも事実上ゼロコスト

---

## 🏆 プロダクト設計書 v8（2026-02-19）

v7までの40+案と重複しない新領域を開拓。事業許認可・法人届出・SSL証明書・技術EOL・フォームリード管理の5案。全てAI不使用、外部API費$0、粗利98-99%超。

| プロダクト | カテゴリ | 評価 | 月額目標達成 | 粗利率 |
|-----------|---------|------|------------|--------|
| [📋 PermitPing](permitping.md) ([レビュー](permitping-review.md)) | 事業許認可更新期限管理 | **A** | 3ヶ月目（Pro 4人+Biz 1人） | 99.5% |
| [🏢 BizNotify](biznotify.md) ([レビュー](biznotify-review.md)) | 法人届出・報告書期限管理 | **A** | 3ヶ月目（Pro 3人+Advisor 1人） | 99.6% |
| [🔒 SSLWatcher](sslwatcher.md) ([レビュー](sslwatcher-review.md)) | マルチドメインSSL証明書一括監視 | **A-** | 2ヶ月目（Agency 1社+Pro 2人） | 99.5% |
| [⏳ EOLTracker](eoltracker.md) ([レビュー](eoltracker-review.md)) | 技術スタックEOL監視 | **A-** | 3ヶ月目（Pro 3人+Team 1人） | 99.4% |
| [📬 FormLeadPing](formleadping.md) ([レビュー](formleadping-review.md)) | フォーム問い合わせ対応漏れ防止 | **A-** | 3ヶ月目（Pro 4人+Biz 1人） | 98.5% |

### v8の特徴
- **全案A評価以上**: PermitPing・BizNotifyがA、SSLWatcher・EOLTracker・FormLeadPingがA-
- **全てAI不使用、外部API費$0**: endoflife.date API（無料）、Node.js tlsモジュール（ネイティブ）のみ
- **「罰金・営業停止・情報漏洩」の恐怖訴求**: 解約=即リスク復活の保険型
- **士業・Web制作会社パートナーチャネル**: 1人の導入で10-50社に波及
- **新領域**: 事業許認可（PermitPing）、法人届出横断管理（BizNotify）は市場にブルーオーシャン
- **既存プロダクトとの補完**: FormShield×FormLeadPing、CertRemind×PermitPing、TaxCalendar×BizNotify

### ❌ ボツ案（B+以下→破棄）
v8設計過程で以下5案を設計・レビューし、全てB+以下で破棄:

- **RefundRadar** (B+): SaaS SLA違反検知。WTP弱い、返金額が小さい
- **ReplyGuard** (B+): Gmail未返信アラート。OAuth審査リスク、無料代替多い
- **GigProtect** (B+): 契約書AI検知。非弁行為リスク、利用頻度低い
- **StockPing** (B+): EC在庫アラート。プラットフォーム標準機能と競合
- **RankDrop** (B): SEO順位追跡。SerpApiコストがスケールしない（計算ミスで$900/月）

---

## 🏆 プロダクト設計書 v7（2026-02-18）

「払わないと損」レベルの実害防止に特化。クラウド請求爆発・API破壊的変更・UI崩壊・税金延滞・ドメインハイジャック — 全て見落とすと金銭的損害が発生する課題。全5案A評価以上。

| プロダクト | カテゴリ | 評価 | 月額目標達成 | 粗利率 |
|-----------|---------|------|------------|--------|
| [💸 QuotaGuard](quotaguard.md) ([レビュー](quotaguard-review.md)) | クラウド請求爆発防止 | **A-** | 3ヶ月目（Pro 4人） | 96.4% |
| [🔍 ChangelogSpy](changelogspy.md) ([レビュー](changelogspy-review.md)) | 依存SaaS Breaking Change検知 | **A** | 3ヶ月目（Pro 4人） | 99.3% |
| [📸 PixelProof](pixelproof.md) ([レビュー](pixelproof-review.md)) | デプロイ後ビジュアルリグレッション検知 | **A** | 3ヶ月目（Pro 3人） | 97.1% |
| [📆 TaxCalendar](taxcalendar.md) ([レビュー](taxcalendar-review.md)) | フリーランス税務期限リマインダー | **A** | 3ヶ月目（Pro 6人） | 99.4% |
| [🛡️ DnsShield](dnsshield.md) ([レビュー](dnsshield-review.md)) | DNS/WHOIS変更監視 | **A** | 3ヶ月目（Pro 4人） | 97.8% |

### v7の特徴
- **全案A評価以上**: ChangelogSpy・PixelProof・TaxCalendar・DnsShieldがA、QuotaGuardがA-
- **「見落とし=金銭損害」の保険型**: 全て解約するとリスクが復活するモデル
- **粗利96〜99%**: インフラコスト$0.20〜$2.00/月で圧倒的に低い
- **ターゲット多様化**: 開発者（QuotaGuard/ChangelogSpy）、Web制作者（PixelProof/DnsShield）、フリーランス全般（TaxCalendar）
- **3-6人で$20達成**: 全案とも少数有料ユーザーで目標クリア

---

## 🏆 プロダクト設計書 v6（2026-02-17 追加分）

v5の成功を受け、サブエージェント完遂作業として3案を新規設計・改善。WakeMeHookをB→A-にブラッシュアップし、AkiAlert・GitNudgeを新規設計。

| プロダクト | カテゴリ | 評価 | 月額目標達成 | 粗利率 |
|-----------|---------|------|------------|--------|
| [⏰ WakeMeHook](wakemehook.md) ([レビューv2](wakemehook-review-v2.md)) | B2B向けスケジュールHTTP | **A-** | 即座（6人で$20） | 100%→73% |
| [🚨 AkiAlert](akialert.md) ([レビュー](akialert-review.md)) | 人気施設キャンセル待ち通知 | **A-** | 即座（4人で$20） | 94% |
| [🔔 GitNudge](gitnudge.md) ([レビュー](gitnudge-review.md)) | GitHub放置検知リマインダー | **A-** | 即座（2人で$20） | 92% |

### v6の特徴
- **WakeMeHookブラッシュアップ**: B評価→A-評価、初期コスト$40→$0に削減
- **超低ハードル**: 2-6人で$20達成、全て無料期間で利益出し可能
- **明確な実需要**: 予約困難、PR放置、B2B自動化の普遍的課題
- **技術的確実性**: 2-3週間でMVP、リスク軽微

## 🏆 プロダクト設計書 v5（2026-02-17 前半）

v4の成功を受け、新たに5案を設計。「ないと困る」「払わないと損」レベルの課題解決に特化し、全てA評価以上を達成。

| プロダクト | カテゴリ | 評価 | 月額目標達成 | 粗利率 |
|-----------|---------|------|------------|--------|
| [💸 ReceiptKeep](receiptkeep.md) ([レビュー](receiptkeep-review-v2.md)) | フリーランス向けレシート管理 | **A** | 1ヶ月目 | 99% |
| [🚨 ErrorLens](errorlens.md) ([レビュー](errorlens-review-v2.md)) | JSエラー収集SaaS | **A-** | 2ヶ月目 | 95% |
| [📅 SlotAlert](slotalert.md) ([レビュー](slotalert-review.md)) | 店舗向けキャンセル待ち管理 | **A** | 1ヶ月目 | 93% |

### v5の共通特徴
- **全てA評価以上**: 厳格なレビューを全て突破
- **必要顧客数の少なさ**: 3-15人で$20達成（現実的）
- **高粗利率**: 82-99%で持続可能性が高い
- **明確な課題解決**: 「ないと困る」レベルの価値提供
- **技術的実現性**: 2-3週間でMVP実装可能

## 🏆 プロダクト設計書 v4（2026-02-16）

v3までの反省を活かし、**「お金を失う・機会を逃す」を防ぐ監視/リマインド系**に特化。全てAI不使用で粗利94%以上、全プロダクトA評価。

| プロダクト | カテゴリ | 評価 | 月額目標達成 | 粗利率 |
|-----------|---------|------|------------|--------|
| [🛎️ FormShield](formshield.md) ([レビュー](formshield-review.md)) | フォーム死活監視 | **A** | 3ヶ月目 | 94.4% |
| [💰 InvoiceTrack](invoicetrack.md) ([レビュー](invoicetrack-review.md)) | 請求書入金リマインダー | **A** | 2ヶ月目 | 97.0% |
| [⭐ ReviewPing](reviewping.md) ([レビュー](reviewping-review.md)) | Googleレビュー通知 | **A** | 3ヶ月目 | 98.0% |
| [⚡ SpeedLoss](speedloss.md) ([レビュー](speedloss-review.md)) | ページ速度低下アラート | **A** | 3ヶ月目 | 97.0% |
| [📜 CertRemind](certremind.md) ([レビュー](certremind-review.md)) | 資格・免許更新リマインダー | **A** | 3ヶ月目 | 99.3% |

### v4の共通特徴
- **全てAI不使用** → コスト予測が安定、外部AI API依存なし
- **「失う前に防ぐ」保険型** → 解約=リスク復活。継続率が高い
- **月$0.02〜$1.11のインフラコスト** → 1人の有料ユーザーで即黒字
- **1〜2週間でMVP** → CRUD+バッチ+通知のシンプル構成
- **BtoB/BtoCハイブリッド** → Web制作者・フリーランス・個人まで幅広くカバー

---

## 🏆 プロダクト設計書 v3（ライト層向け・全A評価）

v2は開発者/技術者向けが多かったため、v3は**非エンジニア・一般ユーザー・ライト層**をターゲットに設計。全て総合評価A以上。

| プロダクト | カテゴリ | 評価 | 月額目標達成 | 粗利率 |
|-----------|---------|------|------------|--------|
| [💳 SubsAlert](subsalert.md) ([レビュー](subsalert-review.md)) | サブスク解約忘れ防止 | **A** | 3ヶ月目 | 99.0% |
| [🛡️ HoshoNote](hosho-note.md) ([レビュー](hosho-note-review.md)) | 保証期間管理 | **A** | 2-3ヶ月目 | 99.0% |
| [📱 ShopPost](shoppost.md) ([レビュー](shoppost-review.md)) | 店舗向けSNS投稿AI生成 | **A** | 2ヶ月目 | 95.8% |
| [👴 MimamoriPing](mimamori-ping.md) ([レビュー](mimamori-ping-review.md)) | 高齢親の安否確認 | **A** | 2ヶ月目 | 98.3% |
| [📅 YoyakuRemind](yoyaku-remind.md) ([レビュー](yoyaku-remind-review.md)) | 予約キャンセル料防止 | **A** | 2ヶ月目 | 99.0% |

### v3の共通特徴
- **ターゲット: 非エンジニア** → ITリテラシー不要、登録10秒、操作3タップ
- **「お金を失う」を防ぐ** → サブスク解約忘れ、保証切れ修理費、キャンセル料、SNS集客
- **4/5がAI不使用** → コスト予測が安定。ShopPostのみAI使用（粗利95%超維持）
- **月$1-3のインフラコスト** → 少数の有料ユーザーで黒字
- **2週間でMVP** → 全て技術的にシンプル

---

## 🏆 プロダクト設計書 v2（全A評価）

前回の反省を活かし、「ないと困る」レベルの課題を解決する保険型SaaSを5つ設計。全て総合評価A以上。

| プロダクト | カテゴリ | 評価 | 月額目標達成 | 粗利率 |
|-----------|---------|------|------------|--------|
| [🔐 EnvLeakGuard](envleakguard.md) ([レビュー](envleakguard-review.md)) | セキュリティ（秘密情報漏洩検知） | **A** | 1ヶ月目 | 98.2% |
| [📧 MailRepWatch](mailrepwatch.md) ([レビュー](mailrepwatch-review.md)) | メール到達性モニタリング | **A** | 1ヶ月目 | 99.2% |
| [👁️ PageDiff](pagediff.md) ([レビュー](pagediff-review.md)) | Webページ変更検知 | **A** | 1ヶ月目 | 95.4% |
| [🛡️ DepsFence](depsfence.md) ([レビュー](depsfence-review.md)) | 依存パッケージ脆弱性・ライセンス | **A** | 1ヶ月目 | 99.0% |
| [💾 BackupHero](backuphero.md) ([レビュー](backuphero-review.md)) | GitHubリポジトリ自動バックアップ | **A** | 1ヶ月目 | 94.8% |

### v2の共通特徴
- **全てAI不使用** → コスト予測が安定、外部API依存なし
- **保険型モデル** → 解約=リスク復活。継続率が高い
- **月$2-6のインフラコスト** → 1人の有料ユーザーで黒字
- **2週間でMVP** → AIエージェントで高速実装可能

---

## プロダクト設計書 v1（レビュー済み）

### ⭐ 最有望
| プロダクト | カテゴリ | 評価 | 月額目標達成 |
|-----------|---------|------|------------|
| [🏷️ Namer](namer.md) ([レビュー](namer-review.md)) | AI活用（ライト層向け） | **A-** | 初月〜1ヶ月 |

### 🟢 実行推奨
| プロダクト | カテゴリ | 評価 | 月額目標達成 |
|-----------|---------|------|------------|
| [⏰ CronFailsafe](cron-failsafe.md) ([レビュー](cron-failsafe-review.md)) | 開発者向けツール | **B+** | 2-3ヶ月 |
| [📋 ContractAlert](contract-alert.md) ([レビュー](contract-alert-review.md)) | スモールビジネス向け | **B+** | 3ヶ月 |
| [🖼️ OGP Gen](ogp-gen.md) ([レビュー](ogp-gen-review.md)) | コンテンツクリエイター向け | **B+** | 3ヶ月 |
| [🐕 DomainWatchdog](domain-watchdog.md) ([レビュー](domain-watchdog-review.md)) | 自動化/監視系 | **B+** | 3ヶ月 |

## 過去のアイデア

- [DeadLink — リンク切れ監視SaaS](deadlink.md)
- [ナイトーク — AIと寝る前に5分だけ雑談するサービス](nightalk.md) ([レビュー](nightalk-review.md))

## 共通制約条件

- 目標: 月$20の収益
- インフラ: AWS or 無料サービス（月$5以下のランニングコスト）
- 開発: AIエージェントが全て実装
- 品質基準: シニアエンジニアレビューでA以上（v2）/ B+以上（v1）
