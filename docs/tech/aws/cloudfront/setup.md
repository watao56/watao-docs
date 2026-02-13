# CloudFrontセットアップガイド

!!! info "前提条件"
    - AWS CLIがインストール・設定済み
    - 適切なIAM権限（CloudFrontFullAccessまたはカスタム権限）
    - オリジンサーバー（S3バケット、EC2、ALB等）が準備済み

## ディストリビューション作成

### AWS コンソールでの作成

1. **CloudFrontサービスにアクセス**
   - AWS Management Console → CloudFront

2. **ディストリビューション作成**
   - 「ディストリビューションを作成」をクリック

3. **基本設定**
   ```
   オリジンドメイン名: my-bucket.s3.ap-northeast-1.amazonaws.com
   オリジンパス: 空（または /static）
   名前: my-s3-origin
   ```

### AWS CLIでの作成

**基本的なS3オリジンディストリビューション**

```bash
# OAC（Origin Access Control）作成
aws cloudfront create-origin-access-control \
    --origin-access-control-config '{
        "Name": "my-s3-oac-2025",
        "Description": "OAC for S3 bucket access",
        "OriginAccessControlOriginType": "s3",
        "SigningBehavior": "always",
        "SigningProtocol": "sigv4"
    }'

# レスポンスからOAC IDを取得
OAC_ID="E12345EXAMPLE"
```

**ディストリビューション設定ファイル作成**

```json
{
    "CallerReference": "my-distribution-20250213-001",
    "Aliases": {
        "Quantity": 1,
        "Items": ["cdn.example.com"]
    },
    "Comment": "My S3 + CloudFront Distribution",
    "Enabled": true,
    "Origins": {
        "Quantity": 1,
        "Items": [
            {
                "Id": "my-s3-origin",
                "DomainName": "my-bucket.s3.ap-northeast-1.amazonaws.com",
                "OriginAccessControlId": "E12345EXAMPLE",
                "S3OriginConfig": {
                    "OriginAccessIdentity": ""
                }
            }
        ]
    },
    "DefaultCacheBehavior": {
        "TargetOriginId": "my-s3-origin",
        "ViewerProtocolPolicy": "redirect-to-https",
        "AllowedMethods": {
            "Quantity": 2,
            "Items": ["GET", "HEAD"],
            "CachedMethods": {
                "Quantity": 2,
                "Items": ["GET", "HEAD"]
            }
        },
        "ForwardedValues": {
            "QueryString": false,
            "Cookies": {
                "Forward": "none"
            }
        },
        "TrustedSigners": {
            "Enabled": false,
            "Quantity": 0
        },
        "MinTTL": 0,
        "DefaultTTL": 86400,
        "MaxTTL": 31536000,
        "Compress": true
    },
    "DefaultRootObject": "index.html"
}
```

```bash
# ディストリビューション作成
aws cloudfront create-distribution \
    --distribution-config file://distribution-config.json

# 作成状況確認
aws cloudfront list-distributions \
    --query 'DistributionList.Items[?Comment==`My S3 + CloudFront Distribution`].[Id,Status,DomainName]' \
    --output table
```

!!! warning "デプロイ時間"
    ディストリビューションの作成・更新には通常15-20分かかります。急いでいる場合は複数の小さな変更にまとめずに、一度に設定することを推奨します。

## S3オリジン設定

### OAC（Origin Access Control）推奨設定

**OAC作成（詳細設定版）**

```bash
aws cloudfront create-origin-access-control \
    --origin-access-control-config '{
        "Name": "my-s3-oac-production",
        "Description": "Production S3 bucket OAC with sigv4",
        "OriginAccessControlOriginType": "s3",
        "SigningBehavior": "always",
        "SigningProtocol": "sigv4"
    }'
```

**S3バケットポリシー設定**

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowCloudFrontServicePrincipal",
            "Effect": "Allow",
            "Principal": {
                "Service": "cloudfront.amazonaws.com"
            },
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::my-bucket/*",
            "Condition": {
                "StringEquals": {
                    "AWS:SourceArn": "arn:aws:cloudfront::123456789012:distribution/E1234567890123"
                }
            }
        }
    ]
}
```

```bash
# バケットポリシー適用
aws s3api put-bucket-policy \
    --bucket my-bucket \
    --policy file://s3-cloudfront-policy.json

# パブリックアクセスブロック確認（推奨：すべてtrue）
aws s3api get-public-access-block --bucket my-bucket
```

### OAI（Origin Access Identity）レガシー設定

!!! note "OAI vs OAC"
    OAC（Origin Access Control）が推奨ですが、既存システムでOAIを使用している場合の設定例も記載します。

```bash
# OAI作成（レガシー）
aws cloudfront create-cloud-front-origin-access-identity \
    --cloud-front-origin-access-identity-config '{
        "CallerReference": "my-oai-20250213",
        "Comment": "OAI for my-bucket access"
    }'

# レスポンスからOAI IDを取得
OAI_ID="E12345EXAMPLE"
```

## カスタムオリジン設定

### ALB（Application Load Balancer）オリジン

```json
{
    "Id": "my-alb-origin",
    "DomainName": "my-alb-123456789.ap-northeast-1.elb.amazonaws.com",
    "CustomOriginConfig": {
        "HTTPPort": 80,
        "HTTPSPort": 443,
        "OriginProtocolPolicy": "https-only",
        "OriginSslProtocols": {
            "Quantity": 3,
            "Items": ["TLSv1.2", "TLSv1.1", "TLSv1"]
        },
        "OriginReadTimeout": 30,
        "OriginKeepaliveTimeout": 5
    },
    "OriginPath": "/api/v1"
}
```

### EC2インスタンスオリジン

```json
{
    "Id": "my-ec2-origin",
    "DomainName": "api.example.com",
    "CustomOriginConfig": {
        "HTTPPort": 80,
        "HTTPSPort": 443,
        "OriginProtocolPolicy": "match-viewer",
        "OriginSslProtocols": {
            "Quantity": 1,
            "Items": ["TLSv1.2"]
        },
        "OriginReadTimeout": 60,
        "OriginKeepaliveTimeout": 5
    }
}
```

### オンプレミスサーバーオリジン

```json
{
    "Id": "onpremise-origin",
    "DomainName": "internal.example.com",
    "CustomOriginConfig": {
        "HTTPPort": 8080,
        "HTTPSPort": 8443,
        "OriginProtocolPolicy": "https-only",
        "OriginSslProtocols": {
            "Quantity": 1,
            "Items": ["TLSv1.2"]
        },
        "OriginReadTimeout": 30,
        "OriginKeepaliveTimeout": 5
    },
    "OriginCustomHeaders": {
        "Quantity": 2,
        "Items": [
            {
                "HeaderName": "X-CloudFront-Secret",
                "HeaderValue": "MySecretValue123"
            },
            {
                "HeaderName": "X-Forwarded-Host",
                "HeaderValue": "example.com"
            }
        ]
    }
}
```

## SSL/TLS証明書設定

### ACM証明書の準備

!!! important "証明書リージョン"
    CloudFront用のACM証明書は**必ずus-east-1（バージニア北部）**で作成する必要があります。

```bash
# バージニア北部リージョンで証明書作成
aws acm request-certificate \
    --domain-name example.com \
    --subject-alternative-names "*.example.com" \
    --validation-method DNS \
    --region us-east-1

# 証明書検証用DNS レコード取得
aws acm describe-certificate \
    --certificate-arn arn:aws:acm:us-east-1:123456789012:certificate/12345678-1234-1234-1234-123456789012 \
    --region us-east-1 \
    --query 'Certificate.DomainValidationOptions[0].ResourceRecord'
```

**Route 53での検証レコード作成**

```bash
# 検証用CNAMEレコード作成
aws route53 change-resource-record-sets \
    --hosted-zone-id Z3M3LMPEXAMPLE \
    --change-batch '{
        "Changes": [{
            "Action": "CREATE",
            "ResourceRecordSet": {
                "Name": "_abc123.example.com",
                "Type": "CNAME",
                "TTL": 300,
                "ResourceRecords": [{
                    "Value": "_xyz789.acm-validations.aws"
                }]
            }
        }]
    }'

# 証明書検証完了確認
aws acm describe-certificate \
    --certificate-arn arn:aws:acm:us-east-1:123456789012:certificate/12345678-1234-1234-1234-123456789012 \
    --region us-east-1 \
    --query 'Certificate.Status'
```

### ディストリビューション へのSSL証明書適用

```bash
# 既存ディストリビューションの設定取得
aws cloudfront get-distribution-config \
    --id E1234567890123 > current-config.json

# ViewerCertificate部分を更新
cat > ssl-config.json << 'EOF'
{
    "ViewerCertificate": {
        "ACMCertificateArn": "arn:aws:acm:us-east-1:123456789012:certificate/12345678-1234-1234-1234-123456789012",
        "SSLSupportMethod": "sni-only",
        "MinimumProtocolVersion": "TLSv1.2_2021",
        "CertificateSource": "acm"
    }
}
EOF

# 設定を更新
aws cloudfront update-distribution \
    --id E1234567890123 \
    --distribution-config file://updated-config.json \
    --if-match ETAG_VALUE
```

## 独自ドメイン設定（Route 53）

### CNAMEエイリアス作成

```bash
# CloudFrontディストリビューション情報取得
DISTRIBUTION_DOMAIN=$(aws cloudfront get-distribution \
    --id E1234567890123 \
    --query 'Distribution.DomainName' \
    --output text)

# Route 53でエイリアスレコード作成
aws route53 change-resource-record-sets \
    --hosted-zone-id Z3M3LMPEXAMPLE \
    --change-batch '{
        "Changes": [{
            "Action": "CREATE",
            "ResourceRecordSet": {
                "Name": "cdn.example.com",
                "Type": "A",
                "AliasTarget": {
                    "HostedZoneId": "Z2FDTNDATAQYW2",
                    "DNSName": "'$DISTRIBUTION_DOMAIN'",
                    "EvaluateTargetHealth": false
                }
            }
        }]
    }'
```

### サブドメイン設定

```bash
# 複数サブドメイン用のレコード作成
aws route53 change-resource-record-sets \
    --hosted-zone-id Z3M3LMPEXAMPLE \
    --change-batch '{
        "Changes": [
            {
                "Action": "CREATE",
                "ResourceRecordSet": {
                    "Name": "static.example.com",
                    "Type": "A",
                    "AliasTarget": {
                        "HostedZoneId": "Z2FDTNDATAQYW2",
                        "DNSName": "'$DISTRIBUTION_DOMAIN'",
                        "EvaluateTargetHealth": false
                    }
                }
            },
            {
                "Action": "CREATE",
                "ResourceRecordSet": {
                    "Name": "assets.example.com",
                    "Type": "A",
                    "AliasTarget": {
                        "HostedZoneId": "Z2FDTNDATAQYW2",
                        "DNSName": "'$DISTRIBUTION_DOMAIN'",
                        "EvaluateTargetHealth": false
                    }
                }
            }
        ]
    }'
```

## キャッシュポリシー設定

### カスタムキャッシュポリシー作成

**静的コンテンツ用（長期キャッシュ）**

```json
{
    "Name": "StaticContentCaching",
    "Comment": "Long-term caching for static assets",
    "DefaultTTL": 31536000,
    "MaxTTL": 31536000,
    "MinTTL": 1,
    "ParametersInCacheKeyAndForwardedToOrigin": {
        "EnableAcceptEncodingGzip": true,
        "EnableAcceptEncodingBrotli": true,
        "QueryStringsConfig": {
            "QueryStringBehavior": "none"
        },
        "HeadersConfig": {
            "HeaderBehavior": "none"
        },
        "CookiesConfig": {
            "CookieBehavior": "none"
        }
    }
}
```

```bash
aws cloudfront create-cache-policy \
    --cache-policy-config file://static-cache-policy.json
```

**動的コンテンツ用（キャッシュ無効）**

```json
{
    "Name": "DynamicContentNoCache",
    "Comment": "No caching for dynamic API responses",
    "DefaultTTL": 0,
    "MaxTTL": 1,
    "MinTTL": 0,
    "ParametersInCacheKeyAndForwardedToOrigin": {
        "EnableAcceptEncodingGzip": true,
        "EnableAcceptEncodingBrotli": true,
        "QueryStringsConfig": {
            "QueryStringBehavior": "all"
        },
        "HeadersConfig": {
            "HeaderBehavior": "whitelist",
            "Headers": {
                "Quantity": 4,
                "Items": [
                    "Authorization",
                    "Content-Type",
                    "Accept",
                    "User-Agent"
                ]
            }
        },
        "CookiesConfig": {
            "CookieBehavior": "all"
        }
    }
}
```

### キャッシュビヘイビア設定

**複数パス用のビヘイビア設定**

```json
{
    "CacheBehaviors": {
        "Quantity": 3,
        "Items": [
            {
                "PathPattern": "/api/*",
                "TargetOriginId": "my-alb-origin",
                "ViewerProtocolPolicy": "https-only",
                "AllowedMethods": {
                    "Quantity": 7,
                    "Items": ["GET", "HEAD", "OPTIONS", "PUT", "POST", "PATCH", "DELETE"],
                    "CachedMethods": {
                        "Quantity": 2,
                        "Items": ["GET", "HEAD"]
                    }
                },
                "CachePolicyId": "4135ea2d-6df8-44a3-9df3-4b5a84be39ad",
                "OriginRequestPolicyId": "88a5eaf4-2fd4-4709-b370-b4c650ea3fcf",
                "Compress": true
            },
            {
                "PathPattern": "/static/*",
                "TargetOriginId": "my-s3-origin",
                "ViewerProtocolPolicy": "redirect-to-https",
                "AllowedMethods": {
                    "Quantity": 2,
                    "Items": ["GET", "HEAD"]
                },
                "CachePolicyId": "custom-static-policy-id",
                "Compress": true
            }
        ]
    }
}
```

## 無効化（Invalidation）

### 基本的な無効化

```bash
# 特定ファイルの無効化
aws cloudfront create-invalidation \
    --distribution-id E1234567890123 \
    --paths "/index.html" "/css/style.css"

# ディレクトリ全体の無効化
aws cloudfront create-invalidation \
    --distribution-id E1234567890123 \
    --paths "/static/*"

# 全ファイル無効化（注意：コスト高）
aws cloudfront create-invalidation \
    --distribution-id E1234567890123 \
    --paths "/*"
```

### 無効化状況の確認

```bash
# 無効化リスト取得
aws cloudfront list-invalidations \
    --distribution-id E1234567890123

# 特定の無効化状況確認
aws cloudfront get-invalidation \
    --distribution-id E1234567890123 \
    --id I1234567890123
```

### デプロイ用無効化スクリプト

```bash
#!/bin/bash

DISTRIBUTION_ID="E1234567890123"
BUILD_DIR="./dist"

# ビルド実行
npm run build

# S3同期
aws s3 sync $BUILD_DIR s3://my-website-bucket/ --delete

# HTMLファイルのキャッシュ無効化
INVALIDATION_ID=$(aws cloudfront create-invalidation \
    --distribution-id $DISTRIBUTION_ID \
    --paths "/*.html" "/sw.js" "/manifest.json" \
    --query 'Invalidation.Id' \
    --output text)

echo "Invalidation created: $INVALIDATION_ID"

# 無効化完了まで待機
aws cloudfront wait invalidation-completed \
    --distribution-id $DISTRIBUTION_ID \
    --id $INVALIDATION_ID

echo "Deployment completed!"
```

!!! tip "無効化のベストプラクティス"
    - HTMLファイルのみ無効化し、CSS/JSはファイル名でバージョニング
    - 無効化は月1000回まで無料、超過分は有料（$0.005/パス）
    - `/*` による全体無効化は避ける

## Origin Shield設定

### Origin Shieldの有効化

```json
{
    "Id": "my-s3-origin-with-shield",
    "DomainName": "my-bucket.s3.ap-northeast-1.amazonaws.com",
    "OriginShield": {
        "Enabled": true,
        "OriginShieldRegion": "ap-northeast-1"
    },
    "S3OriginConfig": {
        "OriginAccessIdentity": ""
    }
}
```

**Origin Shield推奨リージョン**

| オリジンリージョン | Origin Shieldリージョン |
|-------------------|------------------------|
| ap-northeast-1 (東京) | ap-northeast-1 |
| us-east-1 (バージニア北部) | us-east-1 |
| eu-west-1 (アイルランド) | eu-west-1 |

### 複数オリジン + Origin Shield

```json
{
    "Origins": {
        "Quantity": 2,
        "Items": [
            {
                "Id": "primary-s3-origin",
                "DomainName": "primary.s3.amazonaws.com",
                "OriginShield": {
                    "Enabled": true,
                    "OriginShieldRegion": "us-east-1"
                },
                "S3OriginConfig": {
                    "OriginAccessIdentity": ""
                }
            },
            {
                "Id": "backup-s3-origin",
                "DomainName": "backup.s3.amazonaws.com",
                "OriginShield": {
                    "Enabled": true,
                    "OriginShieldRegion": "us-west-2"
                },
                "S3OriginConfig": {
                    "OriginAccessIdentity": ""
                }
            }
        ]
    }
}
```

## エラーページ設定

### カスタムエラーページ

```json
{
    "CustomErrorResponses": {
        "Quantity": 3,
        "Items": [
            {
                "ErrorCode": 404,
                "ResponsePagePath": "/404.html",
                "ResponseCode": "404",
                "ErrorCachingMinTTL": 300
            },
            {
                "ErrorCode": 403,
                "ResponsePagePath": "/index.html",
                "ResponseCode": "200",
                "ErrorCachingMinTTL": 5
            },
            {
                "ErrorCode": 500,
                "ResponsePagePath": "/error.html",
                "ResponseCode": "500",
                "ErrorCachingMinTTL": 0
            }
        ]
    }
}
```

### SPA（Single Page Application）対応

```json
{
    "CustomErrorResponses": {
        "Quantity": 1,
        "Items": [
            {
                "ErrorCode": 404,
                "ResponsePagePath": "/index.html",
                "ResponseCode": "200",
                "ErrorCachingMinTTL": 0
            }
        ]
    }
}
```

!!! note "SPA対応の注意点"
    React Router、Vue Router等のクライアントサイドルーティングを使用するSPAでは、すべての404エラーを`index.html`に200ステータスでリダイレクトする設定が必要です。

## セキュリティヘッダー設定

### CloudFront Functions によるヘッダー追加

```javascript
function handler(event) {
    var response = event.response;
    var headers = response.headers;

    // セキュリティヘッダー追加
    headers['strict-transport-security'] = { value: 'max-age=63072000; includeSubdomains; preload' };
    headers['content-security-policy'] = { value: "default-src 'self'; img-src 'self' data: https:; script-src 'self' 'unsafe-inline'" };
    headers['x-content-type-options'] = { value: 'nosniff' };
    headers['x-frame-options'] = { value: 'DENY' };
    headers['x-xss-protection'] = { value: '1; mode=block' };
    headers['referrer-policy'] = { value: 'strict-origin-when-cross-origin' };

    return response;
}
```

```bash
# CloudFront Function作成
aws cloudfront create-function \
    --name security-headers \
    --function-config '{
        "Comment": "Add security headers",
        "Runtime": "cloudfront-js-1.0"
    }' \
    --function-code fileb://security-headers.js
```

## 監視・ログ設定

### リアルタイムログの設定

```bash
# Kinesis Data Streamの作成
aws kinesis create-stream \
    --stream-name cloudfront-logs \
    --shard-count 2

# リアルタイムログ設定作成
aws cloudfront create-realtime-log-config \
    --name my-realtime-logs \
    --end-points '{
        "StreamType": "Kinesis",
        "KinesisStreamConfig": {
            "RoleArn": "arn:aws:iam::123456789012:role/CloudFrontLogsRole",
            "StreamArn": "arn:aws:kinesis:us-east-1:123456789012:stream/cloudfront-logs"
        }
    }' \
    --fields timestamp c-ip sc-status cs-method cs-uri-stem
```

### CloudWatch アラーム設定

```bash
# エラー率アラーム
aws cloudwatch put-metric-alarm \
    --alarm-name "CloudFront-HighErrorRate" \
    --alarm-description "CloudFront error rate > 5%" \
    --metric-name "4xxErrorRate" \
    --namespace "AWS/CloudFront" \
    --statistic "Average" \
    --period 300 \
    --evaluation-periods 2 \
    --threshold 5.0 \
    --comparison-operator "GreaterThanThreshold" \
    --dimensions Name=DistributionId,Value=E1234567890123 \
    --alarm-actions "arn:aws:sns:us-east-1:123456789012:cloudfront-alerts"

# オリジンレイテンシアラーム
aws cloudwatch put-metric-alarm \
    --alarm-name "CloudFront-HighOriginLatency" \
    --alarm-description "Origin latency > 5000ms" \
    --metric-name "OriginLatency" \
    --namespace "AWS/CloudFront" \
    --statistic "Average" \
    --period 300 \
    --evaluation-periods 3 \
    --threshold 5000 \
    --comparison-operator "GreaterThanThreshold" \
    --dimensions Name=DistributionId,Value=E1234567890123
```

## トラブルシューティング

### 一般的な問題と解決法

**1. オリジンアクセス権限エラー**

```bash
# S3バケットポリシー確認
aws s3api get-bucket-policy --bucket my-bucket

# OAC設定確認
aws cloudfront get-origin-access-control --id E12345EXAMPLE

# CloudTrailでアクセスログ確認
aws logs filter-log-events \
    --log-group-name CloudTrail/S3DataEvents \
    --start-time $(date -d '1 hour ago' +%s)000 \
    --filter-pattern "{ $.eventName = \"GetObject\" }"
```

**2. キャッシュ動作確認**

```bash
# レスポンスヘッダー確認
curl -I https://example.com/test.css

# X-Cache ヘッダーの意味:
# Hit from cloudfront - キャッシュヒット
# RefreshHit from cloudfront - 条件付きリクエストでキャッシュ使用
# Miss from cloudfront - キャッシュミス
```

**3. SSL証明書の問題**

```bash
# ACM証明書状況確認
aws acm describe-certificate \
    --certificate-arn arn:aws:acm:us-east-1:123456789012:certificate/12345678-1234-1234-1234-123456789012 \
    --region us-east-1

# SSL テスト
openssl s_client -connect example.com:443 -servername example.com
```

## パフォーマンステスト

### 基本的なパフォーマンステスト

```bash
#!/bin/bash

DOMAIN="example.com"
PATHS=("/" "/static/app.js" "/images/logo.png" "/api/status")

echo "CloudFront Performance Test - $(date)"
echo "=================================="

for path in "${PATHS[@]}"; do
    echo "Testing: $path"
    
    # 初回リクエスト（キャッシュミス想定）
    time1=$(curl -o /dev/null -s -w "%{time_total}" "https://$DOMAIN$path")
    
    # 2回目リクエスト（キャッシュヒット想定）
    time2=$(curl -o /dev/null -s -w "%{time_total}" "https://$DOMAIN$path")
    
    echo "  First request: ${time1}s"
    echo "  Second request: ${time2}s"
    echo "  Cache improvement: $(echo "scale=2; ($time1 - $time2) / $time1 * 100" | bc)%"
    echo
done
```

### 詳細パフォーマンス分析

```bash
# 複数地点からのテスト（curl + プロキシ）
PROXIES=("proxy1.example.com:8080" "proxy2.example.com:8080")

for proxy in "${PROXIES[@]}"; do
    echo "Testing from: $proxy"
    curl --proxy $proxy -o /dev/null -s -w "@curl-format.txt" "https://example.com/"
done

# curl-format.txt の内容
cat > curl-format.txt << 'EOF'
     time_namelookup:  %{time_namelookup}\n
        time_connect:  %{time_connect}\n
     time_appconnect:  %{time_appconnect}\n
    time_pretransfer:  %{time_pretransfer}\n
       time_redirect:  %{time_redirect}\n
  time_starttransfer:  %{time_starttransfer}\n
                     ----------\n
          time_total:  %{time_total}\n
EOF
```

!!! success "次のステップ"
    CloudFrontの設定が完了したら、[ユースケース](use-cases.md)で具体的な活用事例を確認し、パフォーマンスとコストを最適化しましょう。

## 関連リンク

- [CloudFront ユースケース](use-cases.md)
- [S3 + CloudFront連携](../s3/use-cases.md#cloudfront連携)
- [AWS CloudFront 公式ドキュメント](https://docs.aws.amazon.com/cloudfront/)