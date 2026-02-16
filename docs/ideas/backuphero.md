# 💾 BackupHero — GitHub/GitLabリポジトリ自動バックアップSaaS

## 1. 概要・解決する課題

**なぜ金を払うか（1行）:** GitHubアカウント凍結・誤削除・障害でコードを全喪失するリスクを、自動バックアップで防ぐ。

### 課題の深刻度
- GitHub アカウント凍結: ToS違反の誤判定、支払い問題、国際制裁等で突然アクセス不能に
  - 2019年: イラン・クリミアの開発者がアカウント凍結、全リポジトリアクセス不能
  - 2023年: 著作権クレームで有名OSSリポジトリが一時削除
- GitHub障害: 2023年に複数回の大規模障害（数時間アクセス不能）
- 誤操作: `git push --force` で履歴破壊、リポジトリ誤削除
- **リポジトリ = 資産**: 個人開発者のリポジトリは生計の源。失うと取り返しがつかない

### なぜ「ないと困る」か
「GitHubが絶対安全」は幻想。年1回の障害でも、それが自分に当たったら全損。月$5の保険は安い。

---

## 2. ターゲットユーザー

### プライマリペルソナ
**木村勇太（34歳）- フリーランスエンジニア**
- GitHubに50リポジトリ（うち20はクライアントワーク）
- バックアップは取っていない（「GitHubが壊れるわけない」）
- 同業者がアカウント凍結された話を聞いて不安になった
- 手動バックアップは面倒すぎて続かない

### セカンダリペルソナ
**渡辺修一（42歳）- 小規模開発会社CEO**
- 社員10人、GitHub Organizationで100リポジトリ
- クライアントとの契約でソースコードのバックアップ義務あり
- 現状: 月1回手動でclone → 外付けHDDに保存（形骸化）
- 「自動化してくれるなら$15/月は安い」

---

## 3. 料金プラン

| プラン | 月額 | 内容 |
|--------|------|------|
| Free | $0 | 3リポジトリ、週次バックアップ、7日間保持 |
| Pro | $5/月 | 30リポジトリ、日次バックアップ、90日間保持、Issues/Wiki含む |
| Team | $15/月 | 無制限リポジトリ、日次バックアップ、365日間保持、Organization対応、S3エクスポート |

---

## 4. ユーザーフロー

```
1. GitHub OAuthでサインアップ（30秒）
2. バックアップしたいリポジトリを選択（全選択も可）
3. バックアップ設定:
   - 頻度: 日次/週次
   - 含めるもの: コード、Issues、Wiki、Releases
4. 初回バックアップ実行（バックグラウンド）
5. ダッシュボードで状態確認:
   - 最終バックアップ日時
   - バックアップサイズ
   - 復元ボタン（zipダウンロード or 新リポジトリにpush）
6. バックアップ失敗時 → メール/Slack通知
```

---

## 5. システムアーキテクチャ

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│  EventBridge │────▶│  Scheduler   │────▶│  SQS Queue   │
│  (日次実行)   │     │  Lambda      │     │              │
└──────────────┘     └──────────────┘     └──────┬───────┘
                                                  │
                                           ┌──────▼───────┐
                                           │  Backup      │
                                           │  Worker      │
                                           │  (ECS Fargate│
                                           │   or Lambda) │
                                           └──────┬───────┘
                                                  │
                            ┌─────────────────────┼──────────────┐
                            │                     │              │
                     ┌──────▼──────┐  ┌───────────▼──┐  ┌───────▼──────┐
                     │  S3         │  │  DynamoDB    │  │  通知         │
                     │  (バックアップ│  │  (メタデータ)│  │  SES/Slack   │
                     │   保存)     │  │              │  │              │
                     └─────────────┘  └──────────────┘  └──────────────┘

┌──────────────┐
│  Next.js     │  ← ダッシュボード + 復元UI
│  (Vercel)    │
└──────────────┘
```

---

## 6. コンポーネント詳細

### 6.1 バックアップスケジューラー (Lambda)
- EventBridgeで日次/週次起動
- ユーザーごとのバックアップ対象リポジトリを取得
- 各リポジトリのバックアップジョブをSQSに投入

### 6.2 バックアップワーカー (Lambda / ECS Fargate)
- **git clone --mirror**: 全ブランチ、タグ、履歴を含むフルクローン
- **Issues/Wiki**: GitHub REST API で取得、JSON形式で保存
- **Releases**: アセットファイルも含めてダウンロード
- 結果をtar.gz圧縮 → S3に保存
- 増分バックアップ: 前回からの差分のみ保存（git bundle使用）
- **大規模リポジトリ対応**: Lambda 15分制限を超える場合はECS Fargateにフォールバック

### 6.3 復元マネージャー
- **zipダウンロード**: S3からpre-signed URLを生成
- **リポジトリ復元**: 新しいGitHubリポジトリを作成し、バックアップからpush
- **Issues復元**: GitHub API で再作成

### 6.4 ストレージ管理
- S3 Lifecycle Policy: 
  - Free: 7日後に削除
  - Pro: 90日後に Glacier移行 → 180日後に削除
  - Team: 365日後に Glacier移行 → 730日後に削除
- S3 Intelligent-Tiering: アクセスパターンに基づく自動最適化

---

## 7. データベース設計 (DynamoDB)

### Users テーブル
```
PK: USER#{github_user_id}
SK: PROFILE
Attributes:
  - github_username: string
  - email: string
  - plan: "free" | "pro" | "team"
  - total_storage_bytes: number
  - slack_webhook_url: string (optional)
  - stripe_customer_id: string
  - created_at: ISO8601
```

### Repositories テーブル
```
PK: USER#{github_user_id}
SK: REPO#{repo_full_name}
Attributes:
  - backup_frequency: "daily" | "weekly"
  - include_issues: boolean
  - include_wiki: boolean
  - include_releases: boolean
  - last_backup: ISO8601
  - last_backup_size_bytes: number
  - status: "active" | "paused" | "error"
  - error_message: string
```

### Backups テーブル
```
PK: REPO#{repo_full_name}
SK: BACKUP#{timestamp}
Attributes:
  - s3_key: string
  - size_bytes: number
  - type: "full" | "incremental"
  - includes: list["code", "issues", "wiki", "releases"]
  - duration_seconds: number

TTL: plan別の保持期間
```

---

## 8. インフラ+AIコスト見積もり

### 月間想定（Pro 20人 × 平均15リポジトリ = 300リポジトリ）

| 項目 | 計算根拠 | 月額コスト |
|------|---------|-----------|
| Lambda (スケジューラー) | 300実行/日 × 30日 = 9,000 | $0.01 |
| Lambda (ワーカー) | 300 × 30日、各5分512MB = 2,250,000 GB-秒 | $3.75 |
| S3 Standard | 300リポジトリ × 平均50MB = 15GB | $0.35 |
| S3 データ転送 | 15GB/月 (GitHub→Lambda→S3) | $0 (同リージョン) |
| S3 Glacier | 古いバックアップ ~30GB | $0.12 |
| DynamoDB | 5 WCU, 10 RCU | $0.50 |
| SES | ~500通 | $0.05 |
| SQS | ~9,000メッセージ | <$0.01 |
| GitHub API | 5,000 req/h/install → 十分 | $0 |
| Vercel | Hobby | $0 |
| ドメイン | 年$12 | $1.00 |
| **合計** | | **~$5.78/月** |

### AIコスト: **$0**（git操作+API呼び出しのみ）

### ⚠️ スケール時の注意
- S3コストはリポジトリサイズに比例。大規模monorepo（数GB）を持つユーザーが増えるとコスト増
- 対策: Team以外は1リポジトリ500MB上限、超過分は$0.05/GB/月の従量課金

---

## 9. MVPスコープ

### Phase 1: コアMVP（2週間）
- GitHub OAuth + リポジトリ選択
- git clone --mirror → S3保存（日次/週次）
- ダッシュボード（バックアップ一覧、状態表示）
- zipダウンロード（復元）
- メール通知（失敗時）
- **工数: 10日**

### Phase 2: 有料機能（1週間）
- Stripe決済
- Issues/Wiki/Releasesバックアップ
- Slack通知
- 増分バックアップ
- **工数: 5日**

### Phase 3: 拡張（2週間）
- GitLab対応
- Organization一括バックアップ
- S3エクスポート（ユーザーの自前S3へ）
- リポジトリ復元（新リポジトリ作成+push）
- **工数: 10日**

---

## 10. 周知・マーケティング計画

### Week 1-2: 恐怖マーケティング + 教育コンテンツ

**Zenn記事:**
- タイトル: 「GitHubアカウントが凍結されたら、あなたのコードは全て消える — バックアップの重要性」
- 内容: 実際の凍結事例紹介 → 手動バックアップの限界 → BackupHero紹介
- 想定PV: 3,000-8,000（センセーショナルな題材でバズ可能性あり）

**Qiita記事:**
- タイトル: 「git clone --mirrorで完全バックアップを取る方法（& 自動化サービスの紹介）」
- 技術的なHowToが入口 → サービス紹介

**Twitter/X:**
- 投稿例1: 「GitHubが100%安全だと思ってる？2023年だけで大規模障害3回。アカウント凍結の誤判定も。あなたの50リポジトリ、バックアップ取ってますか？ #エンジニア」
- 投稿例2: 「月$5であなたのGitHubリポジトリを毎日自動バックアップ。Issues、Wiki、Releasesも含めて。復元はワンクリック → [URL]」
- タイミング: 月・水・金 21:00 JST
- **特にGitHub障害発生時にリアルタイム投稿**: 「GitHubまた落ちてますね。BackupHeroがあれば、障害中もコードにアクセスできます」

### Week 3-4: 開発者コミュニティ
- **r/selfhosted**: 「Alternative to manual GitHub backups — automated, $5/mo」
- **Hacker News**: Show HN（GitHub障害のニュースが出た直後を狙う）
- **Dev.to**: 「Why Every Developer Needs GitHub Backup (And How I Automated It)」

### 継続施策
- GitHub障害のたびにTwitterで存在感 → 定期的な認知獲得
- 「GitHub Backup」のSEO（検索需要は障害のたびに急増）
- CLI版をOSSで公開 → SaaS版へのファネル

---

## 11. 技術スタック

| レイヤー | 技術 |
|---------|------|
| Frontend | Next.js 14 |
| Hosting | Vercel (Hobby) |
| Auth | NextAuth.js + GitHub OAuth |
| Backend | AWS Lambda + (ECS Fargate for large repos) |
| Git操作 | isomorphic-git / シェルコマンド (git) |
| Storage | Amazon S3 + S3 Glacier |
| Database | Amazon DynamoDB |
| Queue | Amazon SQS |
| 定期実行 | Amazon EventBridge |
| 通知 | Amazon SES + Slack Webhook |
| 決済 | Stripe |

---

## 12. リスクと対策

| リスク | 深刻度 | 対策 |
|--------|--------|------|
| Lambda 15分制限で大規模リポジトリに対応不可 | High | SQSでリポジトリサイズ判定 → 大規模はECS Fargate（最大4時間）にルーティング |
| S3コストの急増（大規模リポジトリ） | High | 1リポジトリ500MB上限（Team以外）、超過従量課金、Glacier活用 |
| GitHub APIレート制限 | Medium | GitHub App (5,000req/h/install)、git clone --mirrorはAPI消費しない |
| セキュリティ（ユーザーのコードを保存） | High | S3暗号化 (SSE-S3)、バケットポリシーでユーザー分離、SOC2準拠を目指す旨明記 |
| ユーザーが「GitHubが壊れるわけない」と思っている | Medium | 恐怖マーケティング + 実際の障害事例で啓蒙 |
| 競合 (BackHub等) | Medium | 価格差で対抗（BackHub $6/3repos vs BackupHero $5/30repos） |

---

## 13. 競合分析・差別化

| 機能 | BackupHero | BackHub ($2/repo/月) | Rewind ($9/月〜) | gitbackup (OSS) |
|------|-----------|---------------------|------------------|-----------------|
| 自動バックアップ | ✅ | ✅ | ✅ | ✅ (要設定) |
| Issues/Wiki | ✅ Pro | ✅ | ✅ | ❌ |
| 復元機能 | ✅ | ✅ | ✅ | ❌ |
| Slack通知 | ✅ | ❌ | ❌ | ❌ |
| Organization対応 | ✅ Team | ✅ | ✅ | ✅ |
| 30リポジトリの月額 | **$5** | **$60** | **$39** | **$0 (DIY)** |
| セットアップ | 30秒 | 2分 | 2分 | 30分+ |

### なぜ勝てるか
1. **圧倒的な価格**: BackHubは$2/repo/月 → 30リポジトリで$60。BackupHeroは$5で30リポジトリ
2. **Issues/Wiki/Releases含む**: コードだけでなくプロジェクト全体をバックアップ
3. **日本語UI**: 競合は全て英語
4. **Slack通知**: 失敗時にすぐ気づける

---

## 14. $20/月達成の現実的シナリオ

### 前提
- Free → Pro変換率: 15%（バックアップは継続性が重要 → Free7日間保持では不安）
- 月次解約率: 3%（保険型）

### 月次モデル
| 月 | 新規Free | 累計Free | Pro変換 | 累計Pro | MRR |
|----|---------|---------|---------|---------|-----|
| 1 | 40 | 40 | 6 | 6 | $30 ✅ |
| 2 | 35 | 69 | 5 | 11 | $55 |
| 3 | 50 | 110 | 8 | 18 | $90 |

### Free 40人/月の根拠
- Zenn「GitHub凍結」記事: バズ狙いで5,000PV → 40登録
- Twitter: GitHub障害時の投稿で10-20人
- Reddit/HN: 10人

### $20達成: **1ヶ月目**

---

## 15. ユニットエコノミクス

### Proユーザー1人あたり（15リポジトリ、月額）
| 項目 | コスト |
|------|--------|
| Lambda (バックアップ) | $0.19 |
| S3 Standard (750MB) | $0.02 |
| S3 Glacier (古い分) | $0.01 |
| DynamoDB | $0.03 |
| SES | $0.01 |
| **合計コスト** | **$0.26** |
| **収益** | **$5.00** |
| **粗利** | **$4.74 (94.8%)** |

### 注意: 大規模リポジトリ保有ユーザー
- 500MB超のリポジトリを多数持つユーザー: コスト$1-2/月
- 対策: 500MB/repo上限 + 超過従量課金で保護

### 損益分岐点
- 固定費: ~$2/月（ドメイン+基本インフラ）
- **1人のProユーザーで黒字**
