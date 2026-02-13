# S3活用事例とベストプラクティス

!!! info "このページについて"
    S3の実際の活用事例と、それぞれのベストプラクティス、コスト最適化手法について詳しく解説します。

## 静的サイトホスティング

### 基本設定

```bash
# 静的ウェブサイトホスティングの有効化
aws s3api put-bucket-website \
    --bucket my-website-bucket \
    --website-configuration '{
        "IndexDocument": {"Suffix": "index.html"},
        "ErrorDocument": {"Key": "error.html"}
    }'

# サイトファイルのアップロード
aws s3 sync ./website/ s3://my-website-bucket/ \
    --delete \
    --cache-control "max-age=86400"
```

### Next.js / React SPA対応

**Next.js Static Export設定**

```javascript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  output: 'export',
  trailingSlash: true,
  images: {
    unoptimized: true
  }
}

module.exports = nextConfig
```

**ビルド・デプロイスクリプト**

```bash
#!/bin/bash

# Next.jsビルド
npm run build

# S3にデプロイ
aws s3 sync ./out/ s3://my-nextjs-site/ \
    --delete \
    --cache-control "public, max-age=31536000" \
    --exclude "*.html" \
    --exclude "sw.js"

# HTMLファイルは短いキャッシュ時間
aws s3 sync ./out/ s3://my-nextjs-site/ \
    --cache-control "public, max-age=0, must-revalidate" \
    --include "*.html" \
    --include "sw.js"

echo "Deployment completed!"
```

!!! warning "SPAルーティング対応"
    React RouterやNext.js Router使用時は、すべての404エラーを`index.html`にリダイレクトする設定が必要です。S3単体では困難なため、CloudFrontとの組み合わせを推奨します。

## メディアファイル配信

### 画像・動画配信の最適化

**ライフサイクル設定例**

```json
{
    "Rules": [
        {
            "ID": "MediaOptimization",
            "Status": "Enabled",
            "Filter": {
                "Prefix": "media/"
            },
            "Transitions": [
                {
                    "Days": 90,
                    "StorageClass": "STANDARD_IA"
                },
                {
                    "Days": 365,
                    "StorageClass": "GLACIER_IR"
                }
            ]
        }
    ]
}
```

**メディアアップロード用Lambda関数**

```python
import json
import boto3
from PIL import Image
import io

def lambda_handler(event, context):
    s3 = boto3.client('s3')
    
    # 元の画像を取得
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    
    # サムネイル生成
    response = s3.get_object(Bucket=bucket, Key=key)
    image = Image.open(response['Body'])
    
    # リサイズ（複数サイズ）
    sizes = [150, 300, 600]
    for size in sizes:
        thumbnail = image.copy()
        thumbnail.thumbnail((size, size), Image.Resampling.LANCZOS)
        
        # WebP形式で保存
        output = io.BytesIO()
        thumbnail.save(output, format='WebP', quality=85)
        
        s3.put_object(
            Bucket=bucket,
            Key=f"thumbnails/{size}px/{key.replace('.jpg', '.webp')}",
            Body=output.getvalue(),
            ContentType='image/webp',
            CacheControl='public, max-age=31536000'
        )
    
    return {'statusCode': 200}
```

### 動画配信の最適化

```bash
# HLS形式の動画配信用フォルダ構造
# videos/
#   ├── video-001/
#   │   ├── master.m3u8
#   │   ├── 720p/
#   │   │   ├── playlist.m3u8
#   │   │   └── segment-*.ts
#   │   └── 1080p/
#   │       ├── playlist.m3u8
#   │       └── segment-*.ts

# 動画ファイルのアップロード
aws s3 cp video.mp4 s3://my-media-bucket/raw-videos/ \
    --metadata "processing-status=pending"

# HLSセグメントのアップロード（適切なContent-Type設定）
aws s3 sync ./hls-output/ s3://my-media-bucket/videos/video-001/ \
    --exclude "*" \
    --include "*.m3u8" \
    --content-type "application/x-mpegURL" \
    --cache-control "public, max-age=10"

aws s3 sync ./hls-output/ s3://my-media-bucket/videos/video-001/ \
    --exclude "*" \
    --include "*.ts" \
    --content-type "video/MP2T" \
    --cache-control "public, max-age=31536000"
```

## バックアップ・アーカイブ

### データベースバックアップ

**MySQL/PostgreSQL自動バックアップ**

```bash
#!/bin/bash

DB_NAME="production_db"
DB_USER="backup_user"
BUCKET="my-backup-bucket"
PREFIX="database-backups/$(date +%Y/%m/%d)"

# データベースダンプ作成
mysqldump -u $DB_USER -p$DB_PASS $DB_NAME | gzip > db_backup_$(date +%Y%m%d_%H%M%S).sql.gz

# S3にアップロード（Glacier即座取得クラス使用）
aws s3 cp db_backup_*.sql.gz s3://$BUCKET/$PREFIX/ \
    --storage-class GLACIER_IR

# ローカルファイル削除
rm db_backup_*.sql.gz

# 古いバックアップの削除（30日以上）
aws s3api list-objects-v2 \
    --bucket $BUCKET \
    --prefix database-backups/ \
    --query "Contents[?LastModified<='$(date -d '30 days ago' --iso-8601)'].Key" \
    --output text | xargs -I {} aws s3 rm s3://$BUCKET/{}

echo "Backup completed: s3://$BUCKET/$PREFIX/"
```

### システムファイルバックアップ

```bash
#!/bin/bash

# 重要なシステム設定のバックアップ
BACKUP_DATE=$(date +%Y%m%d)
BUCKET="my-system-backup"

# 設定ファイルのアーカイブ作成
tar -czf system_config_$BACKUP_DATE.tar.gz \
    /etc/nginx/ \
    /etc/apache2/ \
    /etc/ssl/ \
    /home/user/.ssh/ \
    --exclude="*.log"

# Deep Archiveに保存（長期保存・低コスト）
aws s3 cp system_config_$BACKUP_DATE.tar.gz \
    s3://$BUCKET/system-configs/ \
    --storage-class DEEP_ARCHIVE

# アプリケーションコードのバックアップ
tar -czf app_code_$BACKUP_DATE.tar.gz /var/www/html/ --exclude="node_modules"
aws s3 cp app_code_$BACKUP_DATE.tar.gz \
    s3://$BUCKET/application-code/ \
    --storage-class STANDARD_IA

rm system_config_$BACKUP_DATE.tar.gz app_code_$BACKUP_DATE.tar.gz
```

## データレイク構築

### ログデータ収集・保存

**Fluent Bit設定例**

```ini
[SERVICE]
    Flush 60
    Daemon off
    Log_Level info

[INPUT]
    Name tail
    Path /var/log/nginx/access.log
    Tag nginx.access

[OUTPUT]
    Name s3
    Match nginx.*
    bucket my-datalake-bucket
    region ap-northeast-1
    s3_key_format /logs/nginx/year=%Y/month=%m/day=%d/nginx-access-%Y%m%d-%H%M%S.log
    total_file_size 50MB
    upload_timeout 10m
    storage_class STANDARD_IA
```

### Athenaでのデータ分析

**パーティション構造の設計**

```
my-datalake-bucket/
├── logs/
│   ├── nginx/
│   │   ├── year=2025/month=02/day=13/
│   │   └── year=2025/month=02/day=14/
│   └── application/
│       ├── year=2025/month=02/day=13/
│       └── year=2025/month=02/day=14/
└── analytics/
    ├── daily-summary/
    └── monthly-reports/
```

**Athenaテーブル作成SQL**

```sql
CREATE EXTERNAL TABLE nginx_logs (
    timestamp string,
    method string,
    url string,
    status_code int,
    response_time float,
    user_agent string
)
PARTITIONED BY (
    year string,
    month string,
    day string
)
STORED AS PARQUET
LOCATION 's3://my-datalake-bucket/logs/nginx/';

-- パーティション自動発見の有効化
MSCK REPAIR TABLE nginx_logs;
```

### データ処理パイプライン

**AWS Glue Jobによる定期処理**

```python
import sys
from awsglue.transforms import *
from awsglue.utils import getResolvedOptions
from pyspark.context import SparkContext
from awsglue.context import GlueContext
from awsglue.job import Job

args = getResolvedOptions(sys.argv, ['JOB_NAME'])
sc = SparkContext()
glueContext = GlueContext(sc)
spark = glueContext.spark_session
job = Job(glueContext)
job.init(args['JOB_NAME'], args)

# 生ログを読み込み
raw_logs = glueContext.create_dynamic_frame.from_options(
    "s3",
    {"paths": ["s3://my-datalake-bucket/logs/nginx/"]},
    format="json"
)

# データ変換・クリーニング
transformed_logs = raw_logs.map(lambda x: {
    'timestamp': x['timestamp'],
    'method': x['method'].upper(),
    'status_code': int(x['status_code']),
    'response_time': float(x['response_time']),
    'date': x['timestamp'][:10]  # YYYY-MM-DD
})

# Parquet形式で出力
glueContext.write_dynamic_frame.from_options(
    transformed_logs,
    "s3",
    {
        "path": "s3://my-datalake-bucket/processed-logs/",
        "partitionKeys": ["date"]
    },
    format="parquet"
)

job.commit()
```

## ログ保存

### アプリケーションログの自動収集

**CloudWatch Logs → S3エクスポート**

```bash
# ログストリームをS3にエクスポート
aws logs create-export-task \
    --log-group-name "/aws/lambda/my-function" \
    --from-time $(date -d '1 day ago' +%s)000 \
    --to-time $(date +%s)000 \
    --destination "my-logs-bucket" \
    --destination-prefix "lambda-logs/$(date +%Y/%m/%d)/"
```

**ログローテーション用Lambda**

```python
import boto3
import json
from datetime import datetime, timedelta

def lambda_handler(event, context):
    s3 = boto3.client('s3')
    logs = boto3.client('logs')
    
    # 1日前のログをエクスポート
    yesterday = datetime.now() - timedelta(days=1)
    from_time = int(yesterday.replace(hour=0, minute=0, second=0).timestamp() * 1000)
    to_time = int(yesterday.replace(hour=23, minute=59, second=59).timestamp() * 1000)
    
    log_groups = [
        '/aws/lambda/api-handler',
        '/aws/lambda/data-processor',
        '/aws/ecs/my-application'
    ]
    
    for log_group in log_groups:
        try:
            response = logs.create_export_task(
                logGroupName=log_group,
                fromTime=from_time,
                toTime=to_time,
                destination='my-logs-archive-bucket',
                destinationPrefix=f"application-logs/{yesterday.strftime('%Y/%m/%d')}/{log_group.replace('/', '_')}"
            )
            print(f"Export task created for {log_group}: {response['taskId']}")
        except Exception as e:
            print(f"Failed to export {log_group}: {str(e)}")
    
    return {'statusCode': 200, 'body': 'Log export tasks created'}
```

## CloudFront連携

### OAC（Origin Access Control）設定

!!! tip "OAC vs OAI"
    2025年現在、OAC（Origin Access Control）がOAI（Origin Access Identity）の後継として推奨されています。OACは全リージョン対応、署名バージョン4サポートなどの利点があります。

**CloudFrontディストリビューション作成**

```bash
# OAC作成
aws cloudfront create-origin-access-control \
    --origin-access-control-config '{
        "Name": "my-s3-oac",
        "Description": "OAC for my S3 bucket",
        "OriginAccessControlOriginType": "s3",
        "SigningBehavior": "always",
        "SigningProtocol": "sigv4"
    }'

# ディストリビューション設定
cat > distribution-config.json << 'EOF'
{
    "CallerReference": "my-distribution-2025-02-13",
    "Comment": "S3 + CloudFront distribution",
    "DefaultCacheBehavior": {
        "TargetOriginId": "my-s3-origin",
        "ViewerProtocolPolicy": "redirect-to-https",
        "AllowedMethods": {
            "Quantity": 2,
            "Items": ["GET", "HEAD"]
        },
        "ForwardedValues": {
            "QueryString": false,
            "Cookies": {"Forward": "none"}
        },
        "Compress": true,
        "CachePolicyId": "4135ea2d-6df8-44a3-9df3-4b5a84be39ad"
    },
    "Origins": {
        "Quantity": 1,
        "Items": [
            {
                "Id": "my-s3-origin",
                "DomainName": "my-static-site-bucket.s3.ap-northeast-1.amazonaws.com",
                "OriginAccessControlId": "OAC_ID_HERE",
                "S3OriginConfig": {
                    "OriginAccessIdentity": ""
                }
            }
        ]
    },
    "Enabled": true
}
EOF
```

**S3バケットポリシー（OAC用）**

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
            "Resource": "arn:aws:s3:::my-static-site-bucket/*",
            "Condition": {
                "StringEquals": {
                    "AWS:SourceArn": "arn:aws:cloudfront::123456789012:distribution/E1234567890123"
                }
            }
        }
    ]
}
```

### 高度なキャッシュ設定

**複数のキャッシュビヘイビア**

```json
{
    "CacheBehaviors": {
        "Quantity": 3,
        "Items": [
            {
                "PathPattern": "/api/*",
                "TargetOriginId": "api-origin",
                "ViewerProtocolPolicy": "https-only",
                "CachePolicyId": "4135ea2d-6df8-44a3-9df3-4b5a84be39ad",
                "TTL": {
                    "DefaultTTL": 0,
                    "MaxTTL": 0,
                    "MinTTL": 0
                }
            },
            {
                "PathPattern": "/static/*",
                "TargetOriginId": "my-s3-origin",
                "ViewerProtocolPolicy": "https-only",
                "CachePolicyId": "658327ea-f89d-4fab-a63d-7e88639e58f6",
                "TTL": {
                    "DefaultTTL": 31536000,
                    "MaxTTL": 31536000,
                    "MinTTL": 0
                }
            }
        ]
    }
}
```

## コスト最適化

### Intelligent-Tiering活用

```bash
# バケット全体でIntelligent-Tiering有効化
aws s3api put-bucket-intelligent-tiering-configuration \
    --bucket my-app-storage-20250213 \
    --id "EntireBucket" \
    --intelligent-tiering-configuration '{
        "Id": "EntireBucket",
        "Status": "Enabled",
        "Filter": {},
        "Tierings": [
            {
                "Days": 90,
                "AccessTier": "ARCHIVE_ACCESS"
            },
            {
                "Days": 180,
                "AccessTier": "DEEP_ARCHIVE_ACCESS"
            }
        ]
    }'

# 特定のプレフィックスのみ対象
aws s3api put-bucket-intelligent-tiering-configuration \
    --bucket my-app-storage-20250213 \
    --id "DocumentsOnly" \
    --intelligent-tiering-configuration '{
        "Id": "DocumentsOnly",
        "Status": "Enabled",
        "Filter": {
            "Prefix": "documents/"
        },
        "Tierings": [
            {
                "Days": 90,
                "AccessTier": "ARCHIVE_ACCESS"
            }
        ]
    }'
```

### ストレージ使用量の監視

**CloudWatch メトリクス取得**

```bash
# バケット使用量取得
aws cloudwatch get-metric-statistics \
    --namespace AWS/S3 \
    --metric-name BucketSizeBytes \
    --dimensions Name=BucketName,Value=my-app-storage-20250213 Name=StorageType,Value=StandardStorage \
    --statistics Sum \
    --start-time $(date -d '7 days ago' --iso-8601) \
    --end-time $(date --iso-8601) \
    --period 86400

# オブジェクト数取得
aws cloudwatch get-metric-statistics \
    --namespace AWS/S3 \
    --metric-name NumberOfObjects \
    --dimensions Name=BucketName,Value=my-app-storage-20250213 Name=StorageType,Value=AllStorageTypes \
    --statistics Average \
    --start-time $(date -d '7 days ago' --iso-8601) \
    --end-time $(date --iso-8601) \
    --period 86400
```

**コスト分析用Lambda**

```python
import boto3
import json
from datetime import datetime, timedelta

def lambda_handler(event, context):
    s3 = boto3.client('s3')
    cloudwatch = boto3.client('cloudwatch')
    
    bucket_name = 'my-app-storage-20250213'
    
    # ストレージクラス別の使用量を取得
    storage_classes = ['StandardStorage', 'StandardIAStorage', 'GlacierIRStorage', 'GlacierStorage', 'DeepArchiveStorage']
    
    cost_summary = {}
    
    for storage_class in storage_classes:
        response = cloudwatch.get_metric_statistics(
            Namespace='AWS/S3',
            MetricName='BucketSizeBytes',
            Dimensions=[
                {'Name': 'BucketName', 'Value': bucket_name},
                {'Name': 'StorageType', 'Value': storage_class}
            ],
            StartTime=datetime.now() - timedelta(days=1),
            EndTime=datetime.now(),
            Period=86400,
            Statistics=['Average']
        )
        
        if response['Datapoints']:
            size_bytes = response['Datapoints'][0]['Average']
            size_gb = size_bytes / (1024**3)
            cost_summary[storage_class] = round(size_gb, 2)
    
    # Slackに送信
    webhook_url = 'https://hooks.slack.com/services/YOUR/WEBHOOK/URL'
    
    message = f"📊 S3 Storage Usage Report - {bucket_name}\n"
    total_size = 0
    for storage_class, size in cost_summary.items():
        message += f"• {storage_class}: {size} GB\n"
        total_size += size
    message += f"**Total: {total_size} GB**"
    
    return {
        'statusCode': 200,
        'body': json.dumps(cost_summary)
    }
```

### リクエスト料金の最適化

```bash
# S3 Transfer Accelerationの有効化（グローバルアクセス時）
aws s3api put-bucket-accelerate-configuration \
    --bucket my-app-storage-20250213 \
    --accelerate-configuration Status=Enabled

# Transfer Acceleration経由でのアップロード
aws s3 cp large-file.zip s3://my-app-storage-20250213/ \
    --endpoint-url https://s3-accelerate.amazonaws.com

# マルチパートアップロード閾値の調整
aws configure set default.s3.multipart_threshold 64MB
aws configure set default.s3.max_concurrent_requests 10
```

!!! tip "コスト最適化のベストプラクティス"
    1. **ライフサイクル自動化**: アクセスパターンに応じたストレージクラス移行
    2. **Intelligent-Tiering**: アクセスパターンが不明な場合の自動最適化
    3. **不完全アップロード削除**: 中断されたマルチパートアップロードのクリーンアップ
    4. **CloudWatch監視**: 定期的な使用量レビューと異常検知
    5. **圧縮**: 可能なファイルは事前圧縮してストレージ削減

## セキュリティベストプラクティス

### アクセスログの設定

```bash
# サーバーアクセスログの有効化
aws s3api put-bucket-logging \
    --bucket my-app-storage-20250213 \
    --bucket-logging-status '{
        "LoggingEnabled": {
            "TargetBucket": "my-access-logs-bucket",
            "TargetPrefix": "s3-access-logs/my-app-storage-20250213/"
        }
    }'
```

### VPCエンドポイント経由アクセス

```bash
# VPCエンドポイント作成
aws ec2 create-vpc-endpoint \
    --vpc-id vpc-12345678 \
    --service-name com.amazonaws.ap-northeast-1.s3 \
    --route-table-ids rtb-12345678

# VPCエンドポイント経由でのみアクセス許可するバケットポリシー
cat > vpc-only-policy.json << 'EOF'
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Deny",
            "Principal": "*",
            "Action": "s3:*",
            "Resource": [
                "arn:aws:s3:::my-secure-bucket",
                "arn:aws:s3:::my-secure-bucket/*"
            ],
            "Condition": {
                "StringNotEquals": {
                    "aws:sourceVpce": "vpce-12345678"
                }
            }
        }
    ]
}
EOF
```

### 署名付きURL生成

```python
import boto3
from botocore.exceptions import ClientError
from datetime import timedelta

def generate_presigned_url(bucket_name, object_key, expiration=3600):
    """署名付きURL生成"""
    s3_client = boto3.client('s3')
    
    try:
        response = s3_client.generate_presigned_url(
            'get_object',
            Params={'Bucket': bucket_name, 'Key': object_key},
            ExpiresIn=expiration
        )
        return response
    except ClientError as e:
        return None

# アップロード用署名付きURL
def generate_presigned_post(bucket_name, object_key, expiration=3600):
    """アップロード用署名付きURL生成"""
    s3_client = boto3.client('s3')
    
    try:
        response = s3_client.generate_presigned_post(
            Bucket=bucket_name,
            Key=object_key,
            Fields={'Content-Type': 'image/jpeg'},
            Conditions=[
                {'Content-Type': 'image/jpeg'},
                ['content-length-range', 1, 10*1024*1024]  # 1B-10MB
            ],
            ExpiresIn=expiration
        )
        return response
    except ClientError as e:
        return None
```

!!! success "次のステップ"
    S3について理解できたら、[CloudFrontとの組み合わせ](../cloudfront/index.md)でさらに高速化・コスト削減を図りましょう。

## 関連リンク

- [S3セットアップガイド](setup.md)
- [CloudFront概要](../cloudfront/index.md)
- [AWS S3 料金計算ツール](https://calculator.aws/)