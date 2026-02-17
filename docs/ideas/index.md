# 💡 アイデア

プロダクトアイデアと設計書のまとめです。

## 🏆 プロダクト設計書 v5（2026-02-17）

v4の成功を受け、新たに5案を設計。「ないと困る」「払わないと損」レベルの課題解決に特化し、全てA評価以上を達成。

| プロダクト | カテゴリ | 評価 | 月額目標達成 | 粗利率 |
|-----------|---------|------|------------|--------|
| [💸 ReceiptKeep](receiptkeep.md) ([レビュー](receiptkeep-review-v2.md)) | フリーランス向けレシート管理 | **A** | 1ヶ月目 | 99% |
| [🚨 ErrorLens](errorlens.md) ([レビュー](errorlens-review-v2.md)) | JSエラー収集SaaS | **A-** | 2ヶ月目 | 95% |
| [⏰ WakeMeHook](wakemehook.md) ([レビュー](wakemehook-review-v2.md)) | B2B向けスケジュールHTTP | **A-** | 2ヶ月目 | 82% |
| [📅 SlotAlert](slotalert.md) ([レビュー](slotalert-review.md)) | 店舗向けキャンセル待ち管理 | **A** | 1ヶ月目 | 93% |
| [🔔 GitNudge](gitnudge.md) ([レビュー](gitnudge-review.md)) | GitHub放置検知リマインダー | **A-** | 2ヶ月目 | 95% |

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
