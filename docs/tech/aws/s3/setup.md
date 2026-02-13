# S3セットアップガイド

!!! info "前提条件"
    - AWS CLIがインストール・設定済み
    - 適切なIAM権限（S3FullAccessまたはカスタム権限）
    - AWS Management Consoleへのアクセス

## バケット作成

### AWSコンソールでの作成

1. **S3サービスにアクセス**
   - AWS Management Console → S3

2. **バケット作成**
   - 「バケットを作成」をクリック
   - バケット名を入力（グローバルで一意である必要があります）
   - リージョンを選択（東京: ap-northeast-1）

3. **基本設定**
   ```
   バケット名: my-app-storage-20250213
   リージョン: アジアパシフィック（東京）ap-northeast-1
   ```

!!! warning "バケット名の制限"
    - グローバルで一意
    - 3-63文字
    - 小文字、数字、ハイフン（-）のみ
    - IPアドレス形式は使用不可

### AWS CLIでの作成

```bash
# 基本的なバケット作成
aws s3 mb s3://my-app-storage-20250213 --region ap-northeast-1

# 作成確認
aws s3 ls
```

```bash
# より詳細な設定でバケット作成
aws s3api create-bucket \
    --bucket my-app-storage-20250213 \
    --region ap-northeast-1 \
    --create-bucket-configuration LocationConstraint=ap-northeast-1
```

## アクセス制御

### ブロックパブリックアクセス（推奨）

!!! danger "セキュリティ重要事項"
    デフォルトで全てのパブリックアクセスをブロックすることを強く推奨します。

```bash
# パブリックアクセスブロックの有効化
aws s3api put-public-access-block \
    --bucket my-app-storage-20250213 \
    --public-access-block-configuration \
    BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true
```

### バケットポリシー

**例1: 特定のIPからのみアクセス許可**

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "IPRestriction",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::my-app-storage-20250213/*",
            "Condition": {
                "IpAddress": {
                    "aws:SourceIp": "203.0.113.0/24"
                }
            }
        }
    ]
}
```

**例2: CloudFront OAC（Origin Access Control）用**

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "cloudfront.amazonaws.com"
            },
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::my-app-storage-20250213/*",
            "Condition": {
                "StringEquals": {
                    "AWS:SourceArn": "arn:aws:cloudfront::123456789012:distribution/EDFDVBD6EXAMPLE"
                }
            }
        }
    ]
}
```

```bash
# バケットポリシーの適用
aws s3api put-bucket-policy \
    --bucket my-app-storage-20250213 \
    --policy file://bucket-policy.json
```

### IAM権限設定

**開発者用のカスタムポリシー例**

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:PutObject",
                "s3:DeleteObject",
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::my-app-storage-20250213",
                "arn:aws:s3:::my-app-storage-20250213/*"
            ]
        }
    ]
}
```

## バージョニング設定

### バージョニングの有効化

```bash
# バージョニング有効化
aws s3api put-bucket-versioning \
    --bucket my-app-storage-20250213 \
    --versioning-configuration Status=Enabled

# MFA削除保護も有効化する場合
aws s3api put-bucket-versioning \
    --bucket my-app-storage-20250213 \
    --versioning-configuration Status=Enabled,MFADelete=Enabled \
    --mfa "arn:aws:iam::123456789012:mfa/username 123456"
```

### バージョン管理コマンド

```bash
# すべてのバージョンを表示
aws s3api list-object-versions --bucket my-app-storage-20250213

# 特定のオブジェクトのバージョン履歴
aws s3api list-object-versions \
    --bucket my-app-storage-20250213 \
    --prefix "documents/report.pdf"

# 特定のバージョンを取得
aws s3api get-object \
    --bucket my-app-storage-20250213 \
    --key documents/report.pdf \
    --version-id "3/L4kqtJlcpXroDTDmJ+rmSpXd3dIbrHY+MTRCxf3vjVBH40Nr8X8gdRQBpUMLUo" \
    report-v1.pdf
```

## ライフサイクルルール

### 基本的なライフサイクル設定

```json
{
    "Rules": [
        {
            "ID": "StandardToIA",
            "Status": "Enabled",
            "Filter": {
                "Prefix": "documents/"
            },
            "Transitions": [
                {
                    "Days": 30,
                    "StorageClass": "STANDARD_IA"
                },
                {
                    "Days": 90,
                    "StorageClass": "GLACIER_IR"
                },
                {
                    "Days": 365,
                    "StorageClass": "GLACIER"
                },
                {
                    "Days": 2555,
                    "StorageClass": "DEEP_ARCHIVE"
                }
            ]
        },
        {
            "ID": "LogsCleanup",
            "Status": "Enabled",
            "Filter": {
                "Prefix": "logs/"
            },
            "Expiration": {
                "Days": 90
            }
        },
        {
            "ID": "IncompleteMultipartCleanup",
            "Status": "Enabled",
            "Filter": {},
            "AbortIncompleteMultipartUpload": {
                "DaysAfterInitiation": 7
            }
        }
    ]
}
```

```bash
# ライフサイクル設定の適用
aws s3api put-bucket-lifecycle-configuration \
    --bucket my-app-storage-20250213 \
    --lifecycle-configuration file://lifecycle.json

# 設定確認
aws s3api get-bucket-lifecycle-configuration \
    --bucket my-app-storage-20250213
```

!!! tip "ライフサイクルルール設計のコツ"
    - アクセス頻度に基づいて段階的に移動
    - 小さなファイルは移動コストが高いため注意
    - ログファイルは定期削除を設定
    - 不完全なマルチパートアップロードの自動削除

## 暗号化設定

### S3マネージド暗号化（SSE-S3）

```bash
# デフォルト暗号化の設定（SSE-S3）
aws s3api put-bucket-encryption \
    --bucket my-app-storage-20250213 \
    --server-side-encryption-configuration '{
        "Rules": [
            {
                "ApplyServerSideEncryptionByDefault": {
                    "SSEAlgorithm": "AES256"
                }
            }
        ]
    }'
```

### KMSキー暗号化（SSE-KMS）

```bash
# KMSキーでの暗号化設定
aws s3api put-bucket-encryption \
    --bucket my-app-storage-20250213 \
    --server-side-encryption-configuration '{
        "Rules": [
            {
                "ApplyServerSideEncryptionByDefault": {
                    "SSEAlgorithm": "aws:kms",
                    "KMSMasterKeyID": "arn:aws:kms:ap-northeast-1:123456789012:key/12345678-1234-1234-1234-123456789012"
                },
                "BucketKeyEnabled": true
            }
        ]
    }'
```

!!! info "暗号化オプション"
    - **SSE-S3**: AWS管理の暗号化キー（デフォルト推奨）
    - **SSE-KMS**: AWS KMS管理の暗号化キー（監査ログが必要な場合）
    - **SSE-C**: 顧客提供の暗号化キー（特殊要件がある場合のみ）

### バケットキー設定

```bash
# バケットキーの有効化（KMS使用時のコスト削減）
aws s3api put-bucket-encryption \
    --bucket my-app-storage-20250213 \
    --server-side-encryption-configuration '{
        "Rules": [
            {
                "ApplyServerSideEncryptionByDefault": {
                    "SSEAlgorithm": "aws:kms",
                    "KMSMasterKeyID": "alias/aws/s3"
                },
                "BucketKeyEnabled": true
            }
        ]
    }'
```

## CORS設定

### ウェブアプリケーション用CORS

```json
{
    "CORSRules": [
        {
            "AllowedHeaders": ["*"],
            "AllowedMethods": ["GET", "PUT", "POST", "DELETE"],
            "AllowedOrigins": ["https://myapp.example.com"],
            "ExposeHeaders": ["ETag"],
            "MaxAgeSeconds": 3000
        }
    ]
}
```

```bash
# CORS設定の適用
aws s3api put-bucket-cors \
    --bucket my-app-storage-20250213 \
    --cors-configuration file://cors.json

# 設定確認
aws s3api get-bucket-cors --bucket my-app-storage-20250213
```

## AWS CLI操作

### 基本操作

```bash
# ファイルのアップロード
aws s3 cp file.txt s3://my-app-storage-20250213/

# フォルダの同期
aws s3 sync ./local-folder s3://my-app-storage-20250213/remote-folder/

# ファイルのダウンロード
aws s3 cp s3://my-app-storage-20250213/file.txt ./

# ファイルの削除
aws s3 rm s3://my-app-storage-20250213/file.txt

# フォルダの削除（再帰的）
aws s3 rm s3://my-app-storage-20250213/folder/ --recursive
```

### 高度な操作

```bash
# マルチパートアップロード（大きなファイル）
aws s3 cp large-file.zip s3://my-app-storage-20250213/ \
    --storage-class STANDARD_IA

# メタデータ付きアップロード
aws s3 cp document.pdf s3://my-app-storage-20250213/ \
    --metadata title="Important Document",author="John Doe" \
    --content-type application/pdf

# 暗号化指定アップロード
aws s3 cp sensitive.txt s3://my-app-storage-20250213/ \
    --sse aws:kms \
    --sse-kms-key-id alias/my-s3-key

# 条件付き同期（変更されたファイルのみ）
aws s3 sync ./website s3://my-app-storage-20250213/ \
    --delete \
    --exclude "*.tmp" \
    --include "*.html"
```

### バックアップスクリプト例

```bash
#!/bin/bash

BUCKET="my-app-storage-20250213"
PREFIX="backup/$(date +%Y-%m-%d)"

# データベースダンプを作成してアップロード
mysqldump -u user -p database > db_backup.sql
aws s3 cp db_backup.sql s3://$BUCKET/$PREFIX/ \
    --storage-class STANDARD_IA

# ログファイルを圧縮してアップロード
tar -czf logs_$(date +%Y%m%d).tar.gz /var/log/myapp/
aws s3 cp logs_$(date +%Y%m%d).tar.gz s3://$BUCKET/$PREFIX/

# 古いバックアップファイルをクリーンアップ
find /tmp/backup -name "*.sql" -mtime +7 -delete

echo "Backup completed: s3://$BUCKET/$PREFIX/"
```

## トラブルシューティング

### 一般的なエラー

**アクセス拒否エラー**
```bash
# 権限確認
aws s3api get-bucket-acl --bucket my-app-storage-20250213
aws iam list-attached-user-policies --user-name username

# バケットポリシー確認
aws s3api get-bucket-policy --bucket my-app-storage-20250213
```

**暗号化エラー**
```bash
# 暗号化設定確認
aws s3api get-bucket-encryption --bucket my-app-storage-20250213

# KMSキー権限確認
aws kms describe-key --key-id alias/my-s3-key
```

### パフォーマンス最適化

```bash
# マルチパート設定の調整
aws configure set default.s3.multipart_threshold 64MB
aws configure set default.s3.multipart_chunksize 16MB
aws configure set default.s3.max_concurrent_requests 10
aws configure set default.s3.max_bandwidth 100MB/s
```

!!! tip "パフォーマンス向上のコツ"
    - プレフィックスを工夫してホットスポットを避ける
    - Transfer Accelerationを有効化（グローバル転送時）
    - CloudFrontとの組み合わせで配信を高速化
    - 適切なマルチパート設定

## 次のステップ

- [ユースケース](use-cases.md): 具体的な活用事例とベストプラクティス
- [CloudFrontとの連携](../cloudfront/index.md): CDN経由でのコンテンツ配信