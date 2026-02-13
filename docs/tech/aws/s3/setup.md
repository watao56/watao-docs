# AWS S3セットアップガイド

このガイドでは、AWS S3の包括的なセットアップ方法を説明します。基本的なバケット作成から、セキュリティ設定、高度な機能まで実用的な例とともに解説します。

## 前提条件

!!! note "必要な準備"
    - AWSアカウントの作成済み
    - AWS CLI v2のインストール済み（`aws --version`で確認）
    - 適切なIAM権限の設定済み（S3の管理権限）

## 1. バケットの作成

### 1.1. AWS管理コンソールでの作成

1. **AWS管理コンソール**にログイン
2. **S3サービス**を選択
3. **「バケットを作成」**をクリック

**基本設定：**
- **バケット名**: グローバルで一意な名前（例: `my-company-data-2024`）
- **AWSリージョン**: 用途に応じて選択（例: `ap-northeast-1` 東京）

!!! warning "バケット命名規則"
    - 小文字、数字、ハイフンのみ使用可能
    - 3-63文字の長さ
    - IPアドレス形式は禁止
    - `xn--` で始まる名前は禁止

### 1.2. AWS CLIでの作成

```bash
# 基本的なバケット作成
aws s3 mb s3://my-bucket-name

# リージョンを指定してバケット作成
aws s3api create-bucket \
    --bucket my-bucket-name \
    --region ap-northeast-1 \
    --create-bucket-configuration LocationConstraint=ap-northeast-1

# バケット一覧の確認
aws s3 ls
```

!!! tip "リージョン指定の注意点"
    - `us-east-1`以外のリージョンでは`LocationConstraint`の指定が必要
    - `us-east-1`では`LocationConstraint`を指定しない

## 2. アクセス制御の設定

### 2.1. ブロックパブリックアクセス設定

!!! danger "セキュリティ重要"
    デフォルトですべてのパブリックアクセスがブロックされています。必要な場合のみ慎重に変更してください。

```bash
# 現在のブロックパブリックアクセス設定を確認
aws s3api get-public-access-block --bucket my-bucket-name

# すべてのパブリックアクセスをブロック（推奨）
aws s3api put-public-access-block \
    --bucket my-bucket-name \
    --public-access-block-configuration \
    BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true

# 静的Webサイト用に部分的に許可（慎重に実施）
aws s3api put-public-access-block \
    --bucket my-website-bucket \
    --public-access-block-configuration \
    BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=false,RestrictPublicBuckets=false
```

### 2.2. バケットポリシーの設定

**特定IAMユーザーにのみアクセスを許可:**

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::123456789012:user/username"
            },
            "Action": [
                "s3:GetObject",
                "s3:PutObject",
                "s3:DeleteObject"
            ],
            "Resource": "arn:aws:s3:::my-bucket-name/*"
        },
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::123456789012:user/username"
            },
            "Action": "s3:ListBucket",
            "Resource": "arn:aws:s3:::my-bucket-name"
        }
    ]
}
```

**暗号化を強制するポリシー:**

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Deny",
            "Principal": "*",
            "Action": "s3:PutObject",
            "Resource": "arn:aws:s3:::my-bucket-name/*",
            "Condition": {
                "StringNotEquals": {
                    "s3:x-amz-server-side-encryption": "AES256"
                }
            }
        }
    ]
}
```

**CLIでポリシーを適用:**

```bash
# ポリシーファイルを適用
aws s3api put-bucket-policy \
    --bucket my-bucket-name \
    --policy file://bucket-policy.json

# ポリシーを確認
aws s3api get-bucket-policy --bucket my-bucket-name

# ポリシーを削除
aws s3api delete-bucket-policy --bucket my-bucket-name
```

### 2.3. IAMポリシーの例

**S3フルアクセス（管理者用）:**

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "s3:*",
            "Resource": "*"
        }
    ]
}
```

**特定バケットへの読み取り専用アクセス:**

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::my-bucket-name",
                "arn:aws:s3:::my-bucket-name/*"
            ]
        }
    ]
}
```

## 3. バージョニング設定

!!! info "バージョニングとは"
    オブジェクトの複数バージョンを保持し、誤削除や変更からデータを保護する機能です。

```bash
# バージョニングを有効化
aws s3api put-bucket-versioning \
    --bucket my-bucket-name \
    --versioning-configuration Status=Enabled

# バージョニング状態を確認
aws s3api get-bucket-versioning --bucket my-bucket-name

# MFA削除保護を有効化（オプション）
aws s3api put-bucket-versioning \
    --bucket my-bucket-name \
    --versioning-configuration Status=Enabled,MFADelete=Enabled \
    --mfa "arn:aws:iam::123456789012:mfa/username 123456"

# すべてのバージョンを表示
aws s3api list-object-versions --bucket my-bucket-name

# 特定バージョンを取得
aws s3api get-object \
    --bucket my-bucket-name \
    --key myfile.txt \
    --version-id "3/L4kqtJlcpXroDTDmJ+rmSpXd3dIbrHY+MTRCxf3vjVBH40Nr8X8gdRQBpUMLUo" \
    myfile-version.txt
```

## 4. ライフサイクルルール

### 4.1. 基本的なライフサイクル設定

```json
{
    "Rules": [
        {
            "ID": "TransitionToIA",
            "Status": "Enabled",
            "Transitions": [
                {
                    "Days": 30,
                    "StorageClass": "STANDARD_IA"
                },
                {
                    "Days": 365,
                    "StorageClass": "GLACIER"
                },
                {
                    "Days": 2555,
                    "StorageClass": "DEEP_ARCHIVE"
                }
            ],
            "Expiration": {
                "Days": 3650
            }
        }
    ]
}
```

### 4.2. プレフィックス別ライフサイクル

```json
{
    "Rules": [
        {
            "ID": "LogsLifecycle",
            "Status": "Enabled",
            "Filter": {
                "Prefix": "logs/"
            },
            "Transitions": [
                {
                    "Days": 7,
                    "StorageClass": "STANDARD_IA"
                },
                {
                    "Days": 30,
                    "StorageClass": "GLACIER"
                }
            ],
            "Expiration": {
                "Days": 90
            }
        },
        {
            "ID": "TempFilesLifecycle",
            "Status": "Enabled",
            "Filter": {
                "Prefix": "temp/"
            },
            "Expiration": {
                "Days": 1
            }
        }
    ]
}
```

**CLIでライフサイクル設定:**

```bash
# ライフサイクル設定を適用
aws s3api put-bucket-lifecycle-configuration \
    --bucket my-bucket-name \
    --lifecycle-configuration file://lifecycle.json

# ライフサイクル設定を確認
aws s3api get-bucket-lifecycle-configuration --bucket my-bucket-name

# ライフサイクル設定を削除
aws s3api delete-bucket-lifecycle --bucket my-bucket-name
```

!!! tip "コスト最適化"
    - **Standard IA**: 30日後（アクセス頻度低）
    - **Glacier**: 365日後（アーカイブ）
    - **Deep Archive**: 7年後（長期保存）

## 5. 暗号化設定

### 5.1. SSE-S3（Amazon S3管理キー）

```bash
# デフォルト暗号化を設定（SSE-S3）
aws s3api put-bucket-encryption \
    --bucket my-bucket-name \
    --server-side-encryption-configuration '{
        "Rules": [
            {
                "ApplyServerSideEncryptionByDefault": {
                    "SSEAlgorithm": "AES256"
                },
                "BucketKeyEnabled": true
            }
        ]
    }'

# 暗号化設定を確認
aws s3api get-bucket-encryption --bucket my-bucket-name
```

### 5.2. SSE-KMS（AWS KMS管理キー）

```bash
# KMSキーでの暗号化設定
aws s3api put-bucket-encryption \
    --bucket my-bucket-name \
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

# カスタマーマスターキー（CMK）を作成
aws kms create-key \
    --description "S3 Bucket Encryption Key" \
    --key-usage ENCRYPT_DECRYPT

# キーエイリアスを作成
aws kms create-alias \
    --alias-name alias/s3-bucket-key \
    --target-key-id 12345678-1234-1234-1234-123456789012
```

### 5.3. アップロード時の暗号化指定

```bash
# SSE-S3で暗号化してアップロード
aws s3 cp file.txt s3://my-bucket-name/ \
    --server-side-encryption AES256

# SSE-KMSで暗号化してアップロード
aws s3 cp file.txt s3://my-bucket-name/ \
    --server-side-encryption aws:kms \
    --ssekms-key-id alias/s3-bucket-key

# クライアント側暗号化
aws s3 cp file.txt s3://my-bucket-name/ \
    --sse-c AES256 \
    --sse-c-key fileb://my-key.bin
```

!!! warning "KMS使用時の注意点"
    - KMSは暗号化・復号化の際に料金が発生
    - `BucketKeyEnabled=true`でコストを削減可能
    - クロスアカウントアクセス時は権限設定に注意

## 6. CORS設定

### 6.1. 基本的なCORS設定

```json
[
    {
        "AllowedHeaders": ["*"],
        "AllowedMethods": ["GET", "PUT", "POST", "DELETE", "HEAD"],
        "AllowedOrigins": ["*"],
        "ExposeHeaders": ["ETag", "x-amz-meta-custom-header"],
        "MaxAgeSeconds": 3600
    }
]
```

### 6.2. セキュアなCORS設定

```json
[
    {
        "AllowedHeaders": ["Authorization", "Content-Type"],
        "AllowedMethods": ["GET", "PUT", "POST"],
        "AllowedOrigins": [
            "https://example.com",
            "https://www.example.com",
            "https://app.example.com"
        ],
        "ExposeHeaders": ["ETag"],
        "MaxAgeSeconds": 1800
    }
]
```

**CLIでCORS設定:**

```bash
# CORS設定を適用
aws s3api put-bucket-cors \
    --bucket my-bucket-name \
    --cors-configuration file://cors.json

# CORS設定を確認
aws s3api get-bucket-cors --bucket my-bucket-name

# CORS設定を削除
aws s3api delete-bucket-cors --bucket my-bucket-name
```

!!! note "CORS使用例"
    - 静的Webサイトから別ドメインのAPIへのリクエスト
    - Webアプリケーションから直接S3へのファイルアップロード
    - SPA（Single Page Application）での画像・動画ストリーミング

## 7. AWS CLI操作

### 7.1. 基本操作

```bash
# ファイルをアップロード
aws s3 cp file.txt s3://my-bucket-name/

# フォルダを再帰的にアップロード
aws s3 cp ./local-folder s3://my-bucket-name/remote-folder/ --recursive

# ファイルをダウンロード
aws s3 cp s3://my-bucket-name/file.txt ./local-file.txt

# フォルダを再帰的にダウンロード
aws s3 cp s3://my-bucket-name/remote-folder/ ./local-folder/ --recursive

# ファイルを移動/名前変更
aws s3 mv s3://my-bucket-name/old-name.txt s3://my-bucket-name/new-name.txt

# ファイルを削除
aws s3 rm s3://my-bucket-name/file.txt

# フォルダを再帰的に削除
aws s3 rm s3://my-bucket-name/folder/ --recursive
```

### 7.2. 同期操作

```bash
# ローカルフォルダをS3に同期（差分のみアップロード）
aws s3 sync ./local-folder s3://my-bucket-name/backup/

# S3からローカルに同期（差分のみダウンロード）
aws s3 sync s3://my-bucket-name/backup/ ./local-folder/

# 削除も含めて同期（--delete オプション）
aws s3 sync ./local-folder s3://my-bucket-name/backup/ --delete

# 特定の拡張子のみ同期
aws s3 sync ./docs s3://my-bucket-name/docs/ \
    --include "*.md" --include "*.txt"

# 特定のファイルを除外
aws s3 sync ./project s3://my-bucket-name/project/ \
    --exclude "node_modules/*" --exclude "*.tmp"
```

### 7.3. オブジェクト一覧表示

```bash
# バケット一覧
aws s3 ls

# バケット内のオブジェクト一覧
aws s3 ls s3://my-bucket-name/

# プレフィックス指定で一覧表示
aws s3 ls s3://my-bucket-name/folder/

# 再帰的に全オブジェクト表示
aws s3 ls s3://my-bucket-name/ --recursive

# 人が読みやすい形式で表示
aws s3 ls s3://my-bucket-name/ --human-readable

# 合計サイズを表示
aws s3 ls s3://my-bucket-name/ --summarize --recursive
```

### 7.4. 署名付きURL（Presigned URL）

```bash
# 1時間有効な読み取り用URL生成
aws s3 presign s3://my-bucket-name/file.txt --expires-in 3600

# 24時間有効なURL生成
aws s3 presign s3://my-bucket-name/file.txt --expires-in 86400

# アップロード用の署名付きURL生成（POST）
aws s3api put-object \
    --bucket my-bucket-name \
    --key upload-file.txt \
    --expires $(date -d '+1 hour' +%s)
```

### 7.5. 高度なオプション

```bash
# マルチパートアップロード（大容量ファイル）
aws s3 cp large-file.zip s3://my-bucket-name/ \
    --storage-class STANDARD_IA

# 並列アップロード設定
aws configure set default.s3.max_concurrent_requests 20
aws configure set default.s3.multipart_threshold 64MB
aws configure set default.s3.multipart_chunksize 16MB

# メタデータ付きアップロード
aws s3 cp file.txt s3://my-bucket-name/ \
    --metadata author=John,project=Alpha \
    --content-type text/plain

# 条件付き操作
aws s3api put-object \
    --bucket my-bucket-name \
    --key file.txt \
    --body file.txt \
    --if-none-match "*"  # オブジェクトが存在しない場合のみ
```

!!! tip "パフォーマンス向上"
    - `--cli-read-timeout 0` で大容量ファイルのタイムアウト回避
    - `--cli-write-timeout 0` で書き込みタイムアウト回避
    - `--page-size 1000` でAPI呼び出し回数を調整

## 8. 静的サイトホスティング設定

### 8.1. 基本設定

```bash
# 静的Webサイトホスティングを有効化
aws s3api put-bucket-website \
    --bucket my-website-bucket \
    --website-configuration '{
        "IndexDocument": {
            "Suffix": "index.html"
        },
        "ErrorDocument": {
            "Key": "error.html"
        }
    }'

# ウェブサイト設定を確認
aws s3api get-bucket-website --bucket my-website-bucket
```

### 8.2. パブリック読み取りポリシー

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::my-website-bucket/*"
        }
    ]
}
```

```bash
# パブリック読み取りポリシーを適用
aws s3api put-bucket-policy \
    --bucket my-website-bucket \
    --policy file://website-policy.json

# ブロックパブリックアクセスを部分的に解除
aws s3api put-public-access-block \
    --bucket my-website-bucket \
    --public-access-block-configuration \
    BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=false,RestrictPublicBuckets=false
```

### 8.3. ファイルのアップロードとデプロイ

```bash
# Webサイトファイルをアップロード
aws s3 sync ./website/ s3://my-website-bucket/ \
    --delete \
    --cache-control "max-age=3600"

# HTMLファイルにContent-Typeを設定
aws s3 cp index.html s3://my-website-bucket/ \
    --content-type "text/html; charset=utf-8"

# CSSファイルにContent-Typeを設定
aws s3 cp styles.css s3://my-website-bucket/ \
    --content-type "text/css"

# JavaScriptファイルにContent-Typeを設定
aws s3 cp script.js s3://my-website-bucket/ \
    --content-type "application/javascript"
```

### 8.4. カスタムドメイン設定

```bash
# Route 53でホストゾーンを作成（ドメインを所有している場合）
aws route53 create-hosted-zone \
    --name example.com \
    --caller-reference $(date +%s)

# CNAMEレコードを作成してS3エンドポイントにマッピング
# バケット名は "www.example.com" 形式である必要がある
aws route53 change-resource-record-sets \
    --hosted-zone-id Z123456789 \
    --change-batch '{
        "Changes": [
            {
                "Action": "CREATE",
                "ResourceRecordSet": {
                    "Name": "www.example.com",
                    "Type": "CNAME",
                    "TTL": 300,
                    "ResourceRecords": [
                        {
                            "Value": "www.example.com.s3-website-ap-northeast-1.amazonaws.com"
                        }
                    ]
                }
            }
        ]
    }'
```

!!! note "静的サイトホスティングのURL形式"
    - **Website endpoint**: `http://bucket-name.s3-website-region.amazonaws.com`
    - **例**: `http://my-website-bucket.s3-website-ap-northeast-1.amazonaws.com`

## 9. イベント通知設定

### 9.1. SNSへの通知設定

```bash
# SNSトピックを作成
aws sns create-topic --name s3-notifications

# S3バケットからSNSへの通知設定
aws s3api put-bucket-notification-configuration \
    --bucket my-bucket-name \
    --notification-configuration '{
        "TopicConfigurations": [
            {
                "Id": "ObjectCreatedEvents",
                "TopicArn": "arn:aws:sns:ap-northeast-1:123456789012:s3-notifications",
                "Events": ["s3:ObjectCreated:*"],
                "Filter": {
                    "Key": {
                        "FilterRules": [
                            {
                                "Name": "prefix",
                                "Value": "uploads/"
                            },
                            {
                                "Name": "suffix",
                                "Value": ".jpg"
                            }
                        ]
                    }
                }
            }
        ]
    }'
```

### 9.2. SQSへの通知設定

```bash
# SQSキューを作成
aws sqs create-queue --queue-name s3-event-queue

# S3バケットからSQSへの通知設定
aws s3api put-bucket-notification-configuration \
    --bucket my-bucket-name \
    --notification-configuration '{
        "QueueConfigurations": [
            {
                "Id": "ObjectRemovedEvents",
                "QueueArn": "arn:aws:sqs:ap-northeast-1:123456789012:s3-event-queue",
                "Events": ["s3:ObjectRemoved:*"]
            }
        ]
    }'
```

### 9.3. Lambdaへの通知設定

```bash
# Lambda関数にS3の実行権限を付与
aws lambda add-permission \
    --function-name my-lambda-function \
    --principal s3.amazonaws.com \
    --statement-id s3-trigger \
    --action lambda:InvokeFunction \
    --source-arn arn:aws:s3:::my-bucket-name

# S3バケットからLambdaへの通知設定
aws s3api put-bucket-notification-configuration \
    --bucket my-bucket-name \
    --notification-configuration '{
        "LambdaConfigurations": [
            {
                "Id": "ProcessImages",
                "LambdaFunctionArn": "arn:aws:lambda:ap-northeast-1:123456789012:function:my-lambda-function",
                "Events": ["s3:ObjectCreated:Put"],
                "Filter": {
                    "Key": {
                        "FilterRules": [
                            {
                                "Name": "suffix",
                                "Value": ".png"
                            }
                        ]
                    }
                }
            }
        ]
    }'
```

### 9.4. 通知設定の確認と削除

```bash
# 現在の通知設定を確認
aws s3api get-bucket-notification-configuration --bucket my-bucket-name

# 通知設定を削除
aws s3api put-bucket-notification-configuration \
    --bucket my-bucket-name \
    --notification-configuration '{}'
```

### 9.5. EventBridge統合

```bash
# EventBridge（旧CloudWatch Events）を有効化
aws s3api put-bucket-notification-configuration \
    --bucket my-bucket-name \
    --notification-configuration '{
        "EventBridgeConfiguration": {}
    }'

# EventBridgeルール作成（例: 特定の拡張子のファイルのみ）
aws events put-rule \
    --name s3-pdf-uploads \
    --event-pattern '{
        "source": ["aws.s3"],
        "detail-type": ["Object Created"],
        "detail": {
            "bucket": {
                "name": ["my-bucket-name"]
            },
            "object": {
                "key": [{"suffix": ".pdf"}]
            }
        }
    }'
```

!!! warning "通知設定の注意点"
    - SNS/SQS/Lambdaのリソースポリシーで、S3からの呼び出しを許可する必要
    - 大量のイベントが発生する場合は、フィルタリングを適切に設定
    - 無限ループを避けるため、Lambda関数が同じバケットに書き込む場合は注意

## 10. 運用のベストプラクティス

### 10.1. セキュリティ

- **最小権限の原則**: 必要最小限の権限のみ付与
- **暗号化の有効化**: すべてのバケットで暗号化を設定
- **アクセスログ**: CloudTrailとS3アクセスログを有効化
- **MFA削除**: 重要データは多要素認証での削除保護を設定

### 10.2. コスト最適化

- **ライフサイクルルール**: 適切なストレージクラスへの移行
- **インテリジェントティアリング**: アクセス頻度に基づく自動移行
- **不完全なマルチパート削除**: 定期的な清掃ルール設定
- **コスト分析**: AWS Cost ExplorerとS3 Storage Lensの活用

### 10.3. パフォーマンス

- **リクエストパターン**: ホットスポットを避けるキー設計
- **Transfer Acceleration**: 世界各地からの高速アップロード
- **CloudFront**: 静的コンテンツの配信最適化
- **マルチパートアップロード**: 大容量ファイルの並列処理

### 10.4. 監視とアラート

```bash
# CloudWatch メトリクスの確認
aws cloudwatch get-metric-statistics \
    --namespace AWS/S3 \
    --metric-name BucketSizeBytes \
    --dimensions Name=BucketName,Value=my-bucket-name Name=StorageType,Value=StandardStorage \
    --statistics Average \
    --start-time 2024-01-01T00:00:00Z \
    --end-time 2024-01-02T00:00:00Z \
    --period 86400

# S3 Storage Lens設定
aws s3control put-storage-lens-configuration \
    --config-id default-account-dashboard \
    --account-id 123456789012 \
    --storage-lens-configuration '{
        "Id": "default-account-dashboard",
        "AccountLevel": {
            "ActivityMetrics": {
                "IsEnabled": true
            },
            "BucketLevel": {
                "ActivityMetrics": {
                    "IsEnabled": true
                }
            }
        },
        "IsEnabled": true
    }'
```

!!! success "運用チェックリスト"
    - [ ] バックアップ戦略の策定（クロスリージョンレプリケーション等）
    - [ ] 災害復旧計画の作成
    - [ ] 定期的なセキュリティ監査
    - [ ] コスト監視とアラート設定
    - [ ] パフォーマンス監視ダッシュボード構築

## まとめ

このガイドでは、AWS S3の包括的なセットアップから運用まで説明しました。セキュリティを最優先に、コスト効率とパフォーマンスを両立させた設定を心がけましょう。

継続的な監視とメンテナンスにより、安全で効率的なS3環境を維持できます。

!!! note "次のステップ"
    - **S3 Batch Operations**: 大量オブジェクトの一括処理
    - **S3 Select**: オブジェクト内データのクエリ実行
    - **Cross-Region Replication**: ディザスタリカバリ対応
    - **S3 Transfer Family**: SFTP/FTPS/AS2プロトコル対応