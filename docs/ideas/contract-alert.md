# 📋 ContractAlert — 契約更新・解約期限リマインダー

## 1. 概要・解決する課題

**ContractAlert**は、SaaSサブスクリプション、保険、賃貸契約、ドメインなどの**契約更新日・解約期限を一元管理**し、最適なタイミングでリマインダーを送るサービス。

### なぜ金を払うか

- 不要なSaaSの自動更新で**毎月数千円〜数万円が無駄に引き落とされている**
- 保険の更新時期を逃し、より安いプランへの切り替えチャンスを失っている
- ドメインの更新忘れで高額な取り戻し費用が発生
- **「何もしないと損する」**を防ぐサービス = 払う理由が明確
- 月$3で年間数万円の無駄を防げる

## 2. ターゲットユーザー

### ペルソナA: フリーランスエンジニア（佐藤、35歳）
- 業務で10以上のSaaSを契約（AWS、GitHub、Figma、Adobe、Notion等）
- 個人でもサブスク多数（Netflix、Spotify、ジム等）
- 年に2-3回、使っていないサービスの更新に気づかず課金されている
- スプレッドシートで管理しようとしたが続かない

### ペルソナB: 個人事業主・小規模経営者（山田、42歳）
- 店舗の賃貸契約、保険、リース、各種SaaS
- 税理士から「もっと契約管理をちゃんとして」と言われている
- 契約更新日をカレンダーに入れているが、埋もれて見逃す

## 3. 料金プラン

| プラン | 月額 | 契約数 | 通知 | 機能 |
|--------|------|--------|------|------|
| Free | $0 | 5 | Email | 基本リマインダー |
| Pro | $3/月 | 50 | Email/Slack/LINE | 節約レポート、カテゴリ管理 |
| Business | $8/月 | 200 | 上記+Webhook | チーム共有、CSV出力 |

年払い: 20%OFF（Pro $29/年、Business $77/年）

## 4. ユーザーフロー

```
1. サインアップ（Google OAuth / Email）
2. 契約を登録（サービス名、月額/年額、更新日、解約期限）
3. リマインダー設定（更新の30日前、7日前、1日前等）
4. LINEまたはSlackを接続（任意）
5. 期限が近づくと通知が届く
6. 「要対応」「更新済」「解約済」でステータス管理
7. 月次/年次の契約コストサマリーレポート
```

## 5. システムアーキテクチャ

```
[CloudFront + S3: SPA]
        |
[API Gateway → Lambda: CRUD API]
        |
[DynamoDB: 契約データ]
        |
[EventBridge Scheduler: 日次チェック]
        |
[Lambda: リマインダー送信]
        |
[SES / LINE Messaging API / Slack API]
```

## 6. コンポーネント詳細

### 6.1 契約管理API
- CRUD: 契約の作成・読取・更新・削除
- カテゴリ分類（SaaS、保険、賃貸、その他）
- 月額換算自動計算（年額→月額）

### 6.2 リマインダーエンジン
- 日次バッチ（EventBridge → Lambda）
- 各契約のリマインダー設定に基づき通知判定
- 通知済みフラグで重複防止

### 6.3 通知システム
- **Email**: SES
- **LINE**: LINE Messaging API（無料枠: 200通/月）
- **Slack**: Incoming Webhook
- 通知テンプレート: 「[サービス名]の更新日まであと7日です。月額¥1,980。継続しますか？」

### 6.4 レポート機能
- 月次コストサマリー（合計支出、カテゴリ別内訳）
- 「3ヶ月以上ステータス未確認」の契約をハイライト

### 6.5 フロントエンド
- React SPA
- ダッシュボード: 直近の期限一覧、月額合計、カテゴリ別グラフ
- モバイルレスポンシブ必須

## 7. データベース設計（DynamoDB）

### Users テーブル
```
PK: USER#{userId}
Fields: email, plan, lineUserId, slackWebhookUrl, createdAt
```

### Contracts テーブル
```
PK: USER#{userId}
SK: CONTRACT#{contractId}
Fields:
  - name: string (サービス名)
  - category: enum (saas|insurance|rental|domain|other)
  - amount: number (金額)
  - currency: string (JPY|USD)
  - billingCycle: enum (monthly|yearly|other)
  - renewalDate: string (ISO date)
  - cancellationDeadline: string (ISO date, nullable)
  - autoRenew: boolean
  - reminders: number[] ([30, 7, 1] = 30日前、7日前、1日前)
  - status: enum (active|renewed|cancelled|pending)
  - notes: string
  - createdAt, updatedAt
GSI: renewal-date-index (PK: USER#{userId}, SK: renewalDate) — 期限順ソート
```

### Notifications テーブル
```
PK: CONTRACT#{contractId}
SK: NOTIFY#{date}#{type}
Fields: channel, sentAt, status
TTL: 90日
```

## 8. インフラ+AIコスト見積もり

### 想定規模（ローンチ3ヶ月後）
- ユーザー: 80人（Free 65人、Pro 12人、Business 3人）
- 契約数: 800
- 通知/日: 50通

### AWS コスト
| サービス | 月額見積もり |
|----------|-------------|
| Lambda | $0.20 |
| DynamoDB（オンデマンド） | $0.50 |
| API Gateway | $0.10 |
| S3 + CloudFront | $0.50 |
| SES | $0.10 |
| EventBridge | $0.05 |
| Cognito | $0.00 |
| Route 53 | $0.50 |
| CloudWatch Logs | $0.30 |
| **合計** | **$2.25/月** |

### 外部APIコスト
- LINE Messaging API: 無料（200通/月以内）
- ドメイン: $12/年 = $1/月

### AIコスト
- 不要（AI機能なし）
- 開発時: Claude Code $20-30

**総ランニングコスト: 約$3.25/月**

## 9. MVPスコープ

### MVP（2週間）
- Google OAuth認証
- 契約CRUD（名前、金額、更新日、リマインダー日数）
- **SaaSテンプレート50個**（AWS、GitHub、Netflix、Spotify等の主要サービスをワンタップ登録）
- Email通知
- 最小限ダッシュボード

### v1.1（+1週間）
- LINE通知連携
- Slack通知連携
- カテゴリ管理

### v1.2（+1週間）
- 月次コストサマリーレポート
- CSV出力

## 10. 周知・マーケティング計画

### Phase 1: ローンチ前
1. LP作成: 「あなたは月にいくらサブスクに払っていますか？」
2. Twitter/Xで「サブスク地獄」共感ツイート
3. note記事「サブスク管理をサボったら年間5万円損していた話」

### Phase 2: ローンチ
1. Product Hunt投稿
2. Zenn/Qiita「フリーランスの契約管理術」記事
3. フリーランスコミュニティ（Lancers、CrowdWorks周辺）で告知
4. 個人事業主向けFacebookグループに投稿

### Phase 3: 継続
1. SEO: 「サブスク 管理」「契約更新 忘れ」「SaaS 解約忘れ」
2. フリーランス向けメディアへの寄稿
3. App Store風の「サブスク管理」比較記事を自作

### 特に注力するチャネル
- **日本語SEO**: 「サブスク 管理 アプリ」は月間検索ボリューム1,000+
- **フリーランスコミュニティ**: 直接的なペイン訴求

## 11. 技術スタック

- **Frontend**: React 18 + Vite + TailwindCSS
- **Backend**: Node.js (Lambda)
- **DB**: DynamoDB
- **Auth**: Amazon Cognito + Google OAuth
- **Infra**: AWS CDK
- **CI/CD**: GitHub Actions

## 12. リスクと対策

| リスク | 影響 | 対策 |
|--------|------|------|
| スプレッドシートやカレンダーで十分 | ユーザーが有料にしない | 「3ヶ月未確認」ハイライト等、能動的でなくても機能する仕組み |
| 契約データの手動入力が面倒 | 登録されない | メール転送で自動登録（v2）、テンプレートで入力簡略化 |
| LINE Messaging APIの無料枠超過 | コスト増 | Pro以上のみLINE通知にし、無料枠内に収める |
| 競合（マネーフォワード等）の機能追加 | 差別化消失 | 「契約の解約期限」に特化した深掘り |
| 個人情報（契約情報）の管理責任 | 法的リスク | 金額・サービス名のみ。個人情報は最小限 |

## 13. 競合分析・差別化

| サービス | 価格 | 特徴 | 弱点 |
|----------|------|------|------|
| マネーフォワード | 無料/¥500〜 | 家計簿全般 | 契約管理は付属機能で弱い |
| Subscriptions (iOS) | 無料/¥400 | iOSアプリ | iOS限定、Web版なし |
| Truebill/Rocket Money | $3-12/月 | 米国向け | 日本未対応 |
| スプレッドシート | 無料 | 自由度高い | リマインダーなし、続かない |

### ContractAlertの差別化
- **「解約期限」に特化** — 更新日だけでなく、「解約するなら何日前まで」を管理
- **LINE通知** — 日本ユーザーに最適なチャネル
- **Web版** — PC/スマホ問わずアクセス可能
- **$3/月** — ランチ1回未満の価格で年間数万円の節約

## 14. $20/月達成の現実的シナリオ

### シナリオ
- **月1**: ローンチ、note/Zenn記事 → Free 30人
- **月2**: SEO効果、フリーランスコミュニティ → Free 50人、Pro 4人 = **$12/月**
- **月3**: LINE通知追加、口コミ → Free 70人、Pro 7人 = **$21/月** ✅
- **月6**: Free 120人、Pro 15人、Business 2人 = **$61/月**

### 根拠
- 「サブスク 管理」の月間検索量は1,000+（日本語）
- フリーランス人口は日本で約460万人（2023年）
- 全員がサブスクに月平均¥5,000+払っている
- 「年間1万円以上節約できる」と実感すれば$3/月は安い
- Free→Pro転換率8%で計算

**$20/月達成見込み: ローンチ後3ヶ月**
