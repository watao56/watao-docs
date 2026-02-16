# 💡 アイデア

プロダクトアイデアと設計書のまとめです。

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
