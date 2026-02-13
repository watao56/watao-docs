# ☁️ AWS（Amazon Web Services）

AWSの基本概念から実践的な活用方法までを体系的にまとめたガイドです。

## AWSとは

Amazon Web Services（AWS）は、Amazonが提供するクラウドコンピューティングプラットフォームです。2006年にサービスを開始し、現在では世界最大のクラウドプロバイダーとして200以上のサービスを提供しています。

### 特徴

- **従量課金制**: 使った分だけ支払い
- **スケーラビリティ**: 需要に応じて自動スケール
- **グローバル**: 世界中にデータセンターを展開
- **高可用性**: 99.9%以上のSLA
- **セキュリティ**: 企業レベルのセキュリティ機能

## 主要サービス一覧

### 🖥️ コンピューティング

| サービス | 概要 | 用途 |
|---------|-----|-----|
| **EC2** | 仮想サーバー | Webサーバー、アプリケーション実行 |
| **Lambda** | サーバーレス関数 | イベント処理、マイクロサービス |
| **ECS** | コンテナ管理 | Dockerアプリケーション |
| **EKS** | Kubernetes管理 | コンテナオーケストレーション |
| **Batch** | バッチ処理 | 大量データ処理 |

### 💾 ストレージ

| サービス | 概要 | 用途 |
|---------|-----|-----|
| **S3** | オブジェクトストレージ | ファイル保存、静的ウェブサイト |
| **EBS** | ブロックストレージ | EC2インスタンス用ディスク |
| **EFS** | ファイルシステム | 複数インスタンス共有ストレージ |
| **Glacier** | アーカイブストレージ | 長期保存、バックアップ |

### 🗄️ データベース

| サービス | 概要 | 用途 |
|---------|-----|-----|
| **RDS** | リレーショナルDB | MySQL、PostgreSQL等の管理 |
| **DynamoDB** | NoSQL DB | 高速なキーバリューストア |
| **Aurora** | 高性能DB | MySQL/PostgreSQL互換の高性能DB |
| **Redshift** | データウェアハウス | 大規模データ分析 |

### 🌐 ネットワーク・セキュリティ

| サービス | 概要 | 用途 |
|---------|-----|-----|
| **VPC** | 仮想プライベートクラウド | ネットワーク分離 |
| **CloudFront** | CDN | コンテンツ配信 |
| **Route 53** | DNS | ドメイン管理、ルーティング |
| **ALB/NLB** | ロードバランサー | 負荷分散 |
| **WAF** | Webアプリケーションファイアウォール | セキュリティ |
| **IAM** | アクセス管理 | ユーザー・権限管理 |

### 🔧 管理・運用

| サービス | 概要 | 用途 |
|---------|-----|-----|
| **CloudWatch** | モニタリング | メトリクス、ログ監視 |
| **CloudFormation** | インフラ管理 | コードによるインフラ構築 |
| **Systems Manager** | システム管理 | パッチ適用、設定管理 |
| **CloudTrail** | API監査 | 操作ログ記録 |

## 料金体系の基本

### 無料枠（Free Tier）

新規アカウントから12か月間利用可能：

- **EC2**: t2.micro（1vCPU、1GB RAM）を月750時間
- **S3**: 5GBのストレージ
- **RDS**: db.t2.micro（20GB）を月750時間
- **Lambda**: 月100万リクエスト
- **CloudFront**: 50GBのデータ転送

!!! tip "無料枠の活用"
    学習や小規模テストには無料枠で十分。本格運用前に必ず試用しましょう。

### 課金モデル

#### オンデマンド
- **特徴**: 前払い不要、柔軟性が高い
- **用途**: 開発・テスト、予測困難な負荷
- **料金**: 最も高い単価

#### リザーブドインスタンス
- **特徴**: 1～3年の契約で最大75%割引
- **用途**: 安定した負荷が見込める本番環境
- **支払い**: 前払い/一部前払い/後払い

#### Savings Plans
- **特徴**: 1～3年の使用量をコミット、最大72%割引
- **用途**: リザーブドより柔軟、インスタンスタイプ変更可
- **推奨**: 現在AWSが推奨している割引方法

#### スポットインスタンス
- **特徴**: 余剰キャパシティを最大90%割引
- **用途**: バッチ処理、中断可能な処理
- **リスク**: 需要増加時に中断される可能性

!!! warning "料金監視の重要性"
    意図しない高額請求を避けるため、CloudWatchやBillingアラートを必ず設定しましょう。

## アカウント作成・初期設定のベストプラクティス

### 1. アカウントセットアップ

```bash
# ルートユーザーの設定
1. 強固なパスワード設定
2. MFA（多要素認証）の有効化
3. アカウント情報の正確な入力
4. 請求アラートの設定
```

### 2. IAMユーザーの作成

!!! danger "ルートユーザーは使用禁止"
    ルートユーザーでの日常操作は厳禁。必ずIAMユーザーを作成して使用しましょう。

```bash
# IAMユーザー作成手順
1. IAMコンソールにアクセス
2. 「ユーザー」→「ユーザーの追加」
3. 管理者権限グループに追加
4. アクセスキーの発行（必要に応じて）
5. MFAの設定
```

### 3. 請求設定

```bash
# Billing & Cost Managementの設定
1. 請求アラートの有効化
2. コスト予算（Budget）の作成
3. コストエクスプローラーの活用
4. 請求レポートの設定
```

### 4. セキュリティ設定

```bash
# セキュリティの基本設定
1. CloudTrailの有効化（API監査）
2. Config Rulesの設定（コンプライアンス）
3. GuardDutyの有効化（脅威検知）
4. SecurityHubの設定（統合セキュリティ）
```

## AWS CLIの基本

### インストール

```bash
# macOS（Homebrew）
brew install awscli

# Ubuntu/Debian
sudo apt update
sudo apt install awscli

# Windows（Chocolatey）
choco install awscli

# pip（Python）
pip install awscli
```

### 認証設定

```bash
# AWS CLIの設定
aws configure

# 入力項目
AWS Access Key ID: [アクセスキーID]
AWS Secret Access Key: [シークレットアクセスキー]
Default region name: ap-northeast-1
Default output format: json
```

### プロファイル管理

```bash
# プロファイルを指定して設定
aws configure --profile my-profile

# プロファイルを使用
aws s3 ls --profile my-profile

# 環境変数で設定
export AWS_PROFILE=my-profile
export AWS_REGION=ap-northeast-1
```

### 基本コマンド

```bash
# 認証情報の確認
aws sts get-caller-identity

# リージョン一覧
aws ec2 describe-regions

# 利用可能なサービス一覧
aws help

# コマンドヘルプ
aws ec2 help
aws s3 help
```

!!! tip "AWS CLIのバージョン管理"
    AWS CLI v2の使用を推奨。v1は2024年7月にサポート終了予定です。

### よく使うコマンド例

```bash
# S3操作
aws s3 ls                          # バケット一覧
aws s3 mb s3://my-bucket          # バケット作成
aws s3 cp file.txt s3://my-bucket/ # ファイルアップロード

# EC2操作
aws ec2 describe-instances         # インスタンス一覧
aws ec2 run-instances             # インスタンス作成
aws ec2 stop-instances --instance-ids i-1234567890abcdef0

# RDS操作
aws rds describe-db-instances     # DBインスタンス一覧
aws rds create-db-instance        # DBインスタンス作成
```

## サービス別ガイド

### EC2（仮想サーバー）
- [EC2概要](ec2/index.md) - インスタンスタイプと料金モデル
- [セットアップ](ec2/setup.md) - インスタンス作成の詳細手順
- [活用事例](ec2/use-cases.md) - Webサーバー、Auto Scaling等
- [セキュリティ](ec2/security.md) - SG、IAM、監視のベストプラクティス

### S3（オブジェクトストレージ）
- [S3概要](s3/index.md) - ストレージクラスと料金
- [セットアップ](s3/setup.md) - バケット作成、アクセス制御、CLI操作
- [活用事例](s3/use-cases.md) - 静的ホスティング、バックアップ、データレイク

### CloudFront（CDN）
- [CloudFront概要](cloudfront/index.md) - CDNの仕組みと料金
- [セットアップ](cloudfront/setup.md) - ディストリビューション作成、SSL/TLS、キャッシュ
- [活用事例](cloudfront/use-cases.md) - 静的サイト配信、Lambda@Edge、WAF連携

!!! note "継続学習のヒント"
    AWSは常に新機能がリリースされます。[AWS News Blog](https://aws.amazon.com/jp/blogs/news/)や[AWS What's New](https://aws.amazon.com/jp/new/)で最新情報をキャッチアップしましょう。