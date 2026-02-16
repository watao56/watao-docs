# 🛡️ DepsFence — 依存パッケージ脆弱性・ライセンス違反スキャナー

## 1. 概要・解決する課題

**なぜ金を払うか（1行）:** 依存パッケージの脆弱性放置で情報漏洩、ライセンス違反で訴訟リスク — 両方を$5/月で防ぐ。

### 課題の深刻度
- Log4Shell (2021): 依存ライブラリの脆弱性で世界中のサービスに影響
- GPL混入: 知らずにGPLライブラリを商用プロダクトに組み込み、訴訟リスク
- Dependabot: 脆弱性アラートは出すがライセンスはノーチェック。アラート疲れで無視される
- Snyk: 高機能だが$25/developer/月〜。個人開発者・小規模チームに高すぎる

### なぜ「ないと困る」か
- 脆弱性放置 → 顧客データ漏洩 → 法的責任
- ライセンス違反 → 訴訟 → 損害賠償
- 月$5で法的リスクを低減できるなら、払わない理由がない

---

## 2. ターゲットユーザー

### プライマリペルソナ
**中村翔太（29歳）- スタートアップエンジニア**
- 5人チームでNode.js + Reactのプロダクト開発
- package.jsonに200+の依存パッケージ
- Dependabotのアラートは来るが、量が多くて対応しきれない
- ライセンスは一度も確認したことがない（「たぶん大丈夫」）
- 資金調達時にセキュリティ監査を求められ、慌てている

### セカンダリペルソナ
**松本さゆり（38歳）- 受託開発会社のリーダー**
- クライアントにセキュリティレポートの提出を求められることが増えた
- 手動でライセンスチェックしていたが、毎回2時間かかる
- 「自動でレポート出してくれるツールがほしい」

---

## 3. 料金プラン

| プラン | 月額 | 内容 |
|--------|------|------|
| Free | $0 | 1リポジトリ、週次スキャン、脆弱性チェックのみ |
| Pro | $5/月 | 10リポジトリ、push時スキャン、脆弱性+ライセンスチェック、Slack通知、PDFレポート |
| Team | $15/月 | 50リポジトリ、チーム管理、監査ログ、コンプライアンスダッシュボード |

---

## 4. ユーザーフロー

```
1. GitHub OAuthでサインアップ（30秒）
2. リポジトリ選択 → 対応ファイル自動検出
   （package.json, requirements.txt, Gemfile, go.mod, pom.xml等）
3. 初回フルスキャン実行（1-2分）
4. 結果ダッシュボード:
   - 脆弱性: Critical 2, High 5, Medium 12
   - ライセンス: GPL混入 1件 ⚠️, MIT 180件 ✅
   - 修正推奨: 「lodash 4.17.20 → 4.17.21 で CVE-2021-23337 修正」
5. 以降、pushごとに差分スキャン
6. 月次PDFレポート自動生成（Pro）→ クライアント提出用
```

---

## 5. システムアーキテクチャ

```
┌─────────┐     ┌──────────────┐     ┌──────────────┐
│  GitHub  │────▶│  Webhook     │────▶│  SQS Queue   │
│  Push    │     │  API Gateway │     │              │
└─────────┘     └──────────────┘     └──────┬───────┘
                                            │
                                     ┌──────▼───────┐
                                     │  Scanner     │
                                     │  Lambda      │
                                     └──────┬───────┘
                                            │
                          ┌─────────────────┼──────────────────┐
                          │                 │                  │
                   ┌──────▼──────┐  ┌──────▼───────┐  ┌──────▼──────┐
                   │  OSV API    │  │  License DB  │  │  DynamoDB   │
                   │  (脆弱性DB) │  │  (SPDX)      │  │  (結果保存)  │
                   └─────────────┘  └──────────────┘  └─────────────┘
                                                             │
                                                      ┌──────▼───────┐
                                                      │  通知/レポート │
                                                      │  SES/Slack/S3│
                                                      └──────────────┘
```

---

## 6. コンポーネント詳細

### 6.1 パッケージ解析 (Lambda)
- GitHub APIで依存ファイルを取得
- 対応パッケージマネージャー:
  - **npm**: package.json + package-lock.json
  - **Python**: requirements.txt, Pipfile, pyproject.toml
  - **Ruby**: Gemfile, Gemfile.lock
  - **Go**: go.mod, go.sum
  - **Java**: pom.xml, build.gradle
- lockfileがあれば正確なバージョン解決、なければ最新バージョン仮定

### 6.2 脆弱性スキャン
- **OSV.dev API** (Google運営、無料): CVE/GHSA情報の問い合わせ
- パッケージ名+バージョン → 既知脆弱性のリスト
- 深刻度: CVSS スコアに基づくCritical/High/Medium/Low分類
- 修正バージョン情報も取得

### 6.3 ライセンススキャン
- **SPDX License List** + npmレジストリ/PyPI API等からライセンス情報取得
- ライセンス分類:
  - ✅ Permissive: MIT, Apache-2.0, BSD, ISC
  - ⚠️ Copyleft: GPL, LGPL, AGPL, MPL
  - ❌ Unknown: ライセンス未指定（要確認）
- ポリシー設定: ユーザーがNGライセンスを指定可能

### 6.4 レポートジェネレーター
- PDFレポート: 脆弱性+ライセンス一覧、対応推奨、経時変化
- クライアント提出用のフォーマット

---

## 7. データベース設計 (DynamoDB)

### Users テーブル
```
PK: USER#{github_user_id}
SK: PROFILE
Attributes: github_username, email, plan, slack_webhook_url, 
            stripe_customer_id, license_policy (object), created_at
```

### Repositories テーブル
```
PK: USER#{github_user_id}
SK: REPO#{repo_full_name}
Attributes:
  - webhook_id: string
  - package_files: list[string]
  - last_scanned_commit: string
  - vulnerability_summary: {critical, high, medium, low}
  - license_summary: {permissive, copyleft, unknown}
  - last_scan: ISO8601
```

### ScanResults テーブル
```
PK: REPO#{repo_full_name}
SK: SCAN#{timestamp}
Attributes:
  - vulnerabilities: list[{package, version, cve_id, severity, fix_version}]
  - licenses: list[{package, version, license, category}]
  - report_s3_key: string (PDFレポートのS3パス)

TTL: 180日（Pro）/ 30日（Free）
```

---

## 8. インフラ+AIコスト見積もり

### 月間想定（Pro 20人、平均5リポジトリ = 100リポジトリ）

| 項目 | 計算根拠 | 月額コスト |
|------|---------|-----------|
| Lambda (スキャン) | 100リポジトリ × 30push/月 = 3,000実行、各5秒512MB | $0.13 |
| API Gateway | 3,000 + ダッシュボード5,000 = 8,000リクエスト | $0.03 |
| DynamoDB | 5 WCU, 10 RCU | $0.50 |
| S3 (PDFレポート) | 100 × 12 = 1,200ファイル × 100KB | <$0.01 |
| SES | ~500通 | $0.05 |
| SQS | ~6,000メッセージ | <$0.01 |
| OSV API | 無料 | $0 |
| Vercel | Hobby | $0 |
| ドメイン | 年$12 | $1.00 |
| **合計** | | **~$1.72/月** |

### AIコスト: **$0**
- OSV.dev API（無料）+ SPDXデータベース + レジストリAPIで完結

---

## 9. MVPスコープ

### Phase 1: コアMVP（2週間）
- GitHub OAuth + リポジトリ選択
- npm (package.json/lock) パーサー
- OSV.dev API連携で脆弱性チェック
- npmレジストリからライセンス取得
- ダッシュボード（脆弱性+ライセンス一覧）
- メール通知
- **工数: 10日**

### Phase 2: 有料機能（1週間）
- Stripe決済
- Slack通知
- PDFレポート生成（pdfkit）
- Webhook（push時スキャン）
- ライセンスポリシー設定
- **工数: 5日**

### Phase 3: 拡張（2週間）
- Python/Ruby/Go/Javaパーサー追加
- チーム管理機能
- 経時変化グラフ
- **工数: 7日**

---

## 10. 周知・マーケティング計画

### Week 1-2: 開発者向けコンテンツ

**Zenn記事:**
- タイトル: 「あなたのpackage.json、GPL混入してませんか？1分でチェックする方法」
- 内容: ライセンス違反のリスク解説 → 手動チェックの大変さ → DepsFence紹介
- 想定PV: 3,000-5,000（「npm ライセンス チェック」は検索需要あり）

**Qiita記事:**
- タイトル: 「Dependabotだけで大丈夫？依存パッケージの脆弱性+ライセンスを一括チェック」
- 既存ツールとの比較を含める

**Twitter/X:**
- 投稿例: 「Dependabotは脆弱性だけ。ライセンスチェックはしてくれない。GPL混入に気づかず商用リリースしたら訴訟リスク。両方チェックするサービス作りました → [URL] #個人開発」
- タイミング: 火・木 21:00 JST

### Week 3-4: スタートアップ向け

- **IndieHackers/r/startups**: 「VCデューデリジェンスでセキュリティ監査を求められたら、このツールで一発レポート」
- **スタートアップ系Slack/Discord**: 日本のスタートアップコミュニティで紹介
- **Product Hunt**: ローンチ

### 継続施策
- GitHub Actionsとしても提供（Freeでpublic repo対応）→ 有料版ファネル
- SEO: 「npm 脆弱性 チェック」「ライセンス 確認 ツール」

---

## 11. 技術スタック

| レイヤー | 技術 |
|---------|------|
| Frontend | Next.js 14 |
| Hosting | Vercel (Hobby) |
| Auth | NextAuth.js + GitHub OAuth |
| Backend | AWS Lambda (Node.js 20) |
| 脆弱性DB | OSV.dev API (無料) |
| ライセンスDB | SPDX + npm/PyPI API |
| Database | Amazon DynamoDB |
| PDF生成 | pdfkit (Node.js) |
| 通知 | Amazon SES + Slack Webhook |
| 決済 | Stripe |

---

## 12. リスクと対策

| リスク | 深刻度 | 対策 |
|--------|--------|------|
| Dependabot（無料）との差別化不足 | High | ライセンスチェック+PDFレポートが差別化。Dependabotにない機能を前面に |
| OSV.dev APIの信頼性 | Medium | ローカルキャッシュ（日次同期）、フォールバックとしてGitHub Advisory Database API |
| lockfileなし時の精度低下 | Medium | lockfileなしの場合は警告表示、最新バージョン仮定と明記 |
| 多言語パーサーの実装工数 | Medium | MVP: npm only。Phase3で順次追加。各パーサーは1-2日で実装可能 |
| Snykの無料プラン拡大 | Medium | Snyk Freeは5プロジェクト/200テスト。DepsFenceは10リポジトリ+ライセンスで差別化 |

---

## 13. 競合分析・差別化

| 機能 | DepsFence | Dependabot (Free) | Snyk Free | Snyk Team ($25/dev/月) | FOSSA ($0-$$) |
|------|-----------|-------------------|-----------|----------------------|---------------|
| 脆弱性スキャン | ✅ | ✅ | ✅ | ✅ | ✅ |
| ライセンスチェック | ✅ Pro | ❌ | ❌ Free | ✅ | ✅ （高額） |
| PDFレポート | ✅ Pro | ❌ | ❌ | ✅ | ✅ |
| Slack通知 | ✅ Pro | ❌ | ✅ | ✅ | ✅ |
| カスタムライセンスポリシー | ✅ | ❌ | ❌ | ✅ | ✅ |
| 最低月額 | **$0-$5** | **$0** | **$0** | **$25/dev** | **要見積** |
| プロジェクト上限 | 10 (Pro) | 無制限 | 5 | 無制限 | - |

### なぜ勝てるか
1. **脆弱性+ライセンスのワンストップ**: Dependabotはライセンス非対応。Snyk Freeもライセンス非対応
2. **PDFレポート**: クライアント・VCへの提出用。Dependabotにはない
3. **価格**: Snyk Teamの1/5、FOSSAの1/10以下
4. **シンプルさ**: Snykは多機能すぎて設定が複雑。DepsFenceは30秒セットアップ

---

## 14. $20/月達成の現実的シナリオ

### 前提
- Free → Pro変換率: 12%
- 月次解約率: 5%

### 月次モデル
| 月 | 新規Free | 累計Free | Pro変換 | 累計Pro | MRR |
|----|---------|---------|---------|---------|-----|
| 1 | 50 | 50 | 6 | 6 | $30 ✅ |
| 2 | 40 | 84 | 5 | 11 | $55 |
| 3 | 60 | 134 | 7 | 17 | $85 |

### Free 50人/月の根拠
- Zenn/Qiita記事: 3,000PV → CTR5% = 150クリック → 30登録
- Twitter: 10人
- GitHub Actions経由: 10人

### $20達成: **1ヶ月目**

---

## 15. ユニットエコノミクス

### Proユーザー1人あたり（5リポジトリ、月額）
| 項目 | コスト |
|------|--------|
| Lambda | $0.01 |
| DynamoDB | $0.03 |
| OSV API | $0 |
| SES/S3 | $0.01 |
| **合計コスト** | **$0.05** |
| **収益** | **$5.00** |
| **粗利** | **$4.95 (99.0%)** |

### 損益分岐点
- 固定費: $1.72/月
- **1人のProユーザーで黒字**
