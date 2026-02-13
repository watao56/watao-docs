# CloudFront活用事例とベストプラクティス

!!! info "このページについて"
    CloudFrontの実際の活用事例と、それぞれのベストプラクティス、パフォーマンス最適化、コスト削減手法について詳しく解説します。

## S3 + CloudFrontでの静的サイト配信

### 基本構成

```
Users → CloudFront → S3 Bucket
                 ↘ Origin Shield (Optional)
```

**最適化されたディストリビューション設定**

```json
{
    "Comment": "Static website with optimal caching",
    "Origins": {
        "Quantity": 1,
        "Items": [{
            "Id": "s3-static-origin",
            "DomainName": "my-static-site.s3.amazonaws.com",
            "OriginAccessControlId": "E12345EXAMPLE",
            "OriginShield": {
                "Enabled": true,
                "OriginShieldRegion": "ap-northeast-1"
            }
        }]
    },
    "CacheBehaviors": {
        "Quantity": 3,
        "Items": [
            {
                "PathPattern": "*.html",
                "CachePolicyId": "short-term-cache",
                "DefaultTTL": 300,
                "MaxTTL": 3600,
                "Compress": true
            },
            {
                "PathPattern": "/static/*",
                "CachePolicyId": "long-term-cache",
                "DefaultTTL": 31536000,
                "MaxTTL": 31536000,
                "Compress": true
            },
            {
                "PathPattern": "/sw.js",
                "CachePolicyId": "no-cache",
                "DefaultTTL": 0,
                "MaxTTL": 0
            }
        ]
    }
}
```

### Next.js Static Export + CloudFront

**next.config.js 最適化設定**

```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  output: 'export',
  trailingSlash: true,
  images: {
    unoptimized: true
  },
  // アセットにハッシュを追加
  assetPrefix: process.env.NODE_ENV === 'production' ? 'https://cdn.example.com' : '',
  
  // 静的最適化
  compiler: {
    removeConsole: process.env.NODE_ENV === 'production',
  }
}

module.exports = nextConfig
```

**自動デプロイ用GitHub Actions**

```yaml
name: Deploy to CloudFront

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Build
      run: npm run build
    
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v2
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ap-northeast-1
    
    - name: Deploy to S3
      run: |
        # 静的アセット（長期キャッシュ）
        aws s3 sync out/_next/static/ s3://${{ vars.S3_BUCKET }}/_next/static/ \
          --cache-control "public, max-age=31536000, immutable" \
          --delete
        
        # HTMLファイル（短期キャッシュ）
        aws s3 sync out/ s3://${{ vars.S3_BUCKET }}/ \
          --exclude "_next/static/*" \
          --cache-control "public, max-age=0, must-revalidate" \
          --delete
    
    - name: Invalidate CloudFront
      run: |
        aws cloudfront create-invalidation \
          --distribution-id ${{ vars.CLOUDFRONT_DISTRIBUTION_ID }} \
          --paths "/*.html" "/sw.js" "/sitemap.xml"
```

### Progressive Web App (PWA) 対応

**Service Worker対応のCloudFront設定**

```json
{
    "CacheBehaviors": {
        "Items": [
            {
                "PathPattern": "/sw.js",
                "ViewerProtocolPolicy": "https-only",
                "CachePolicyId": "4135ea2d-6df8-44a3-9df3-4b5a84be39ad",
                "TTL": {
                    "DefaultTTL": 0,
                    "MaxTTL": 0,
                    "MinTTL": 0
                },
                "Headers": {
                    "Quantity": 2,
                    "Items": ["Cache-Control", "Service-Worker-Allowed"]
                }
            },
            {
                "PathPattern": "/manifest.json",
                "ViewerProtocolPolicy": "https-only",
                "CachePolicyId": "short-term-cache",
                "TTL": {
                    "DefaultTTL": 3600,
                    "MaxTTL": 86400
                }
            }
        ]
    }
}
```

## 動的コンテンツの高速化

### API プロキシとしてのCloudFront

**マルチオリジン構成**

```json
{
    "Origins": {
        "Quantity": 3,
        "Items": [
            {
                "Id": "static-s3",
                "DomainName": "static.s3.amazonaws.com"
            },
            {
                "Id": "api-alb",
                "DomainName": "api.example.com",
                "CustomOriginConfig": {
                    "HTTPPort": 80,
                    "HTTPSPort": 443,
                    "OriginProtocolPolicy": "https-only",
                    "OriginReadTimeout": 30,
                    "OriginKeepaliveTimeout": 5
                }
            },
            {
                "Id": "websocket-nlb",
                "DomainName": "ws.example.com",
                "CustomOriginConfig": {
                    "HTTPPort": 80,
                    "HTTPSPort": 443,
                    "OriginProtocolPolicy": "https-only"
                }
            }
        ]
    },
    "CacheBehaviors": {
        "Items": [
            {
                "PathPattern": "/api/v1/static/*",
                "TargetOriginId": "api-alb",
                "CachePolicyId": "managed-caching-optimized",
                "TTL": {"DefaultTTL": 3600}
            },
            {
                "PathPattern": "/api/v1/dynamic/*",
                "TargetOriginId": "api-alb",
                "CachePolicyId": "managed-caching-disabled",
                "TTL": {"DefaultTTL": 0}
            },
            {
                "PathPattern": "/ws/*",
                "TargetOriginId": "websocket-nlb",
                "ViewerProtocolPolicy": "https-only",
                "AllowedMethods": {
                    "Items": ["GET", "HEAD", "OPTIONS", "PUT", "POST", "PATCH", "DELETE"]
                }
            }
        ]
    }
}
```

### レスポンス変換

**Lambda@Edge でのレスポンス最適化**

```python
import json
import gzip
import base64

def lambda_handler(event, context):
    request = event['Records'][0]['cf']['request']
    response = event['Records'][0]['cf']['response']
    
    # API レスポンスの圧縮
    if response['headers'].get('content-type', [{}])[0].get('value', '').startswith('application/json'):
        body = response['body']['data']
        
        # レスポンス圧縮
        if len(body) > 1024:  # 1KB以上の場合圧縮
            compressed_body = gzip.compress(body.encode('utf-8'))
            response['body']['data'] = base64.b64encode(compressed_body).decode('utf-8')
            response['body']['encoding'] = 'base64'
            response['headers']['content-encoding'] = [{'key': 'Content-Encoding', 'value': 'gzip'}]
    
    # セキュリティヘッダー追加
    response['headers']['x-frame-options'] = [{'key': 'X-Frame-Options', 'value': 'DENY'}]
    response['headers']['x-content-type-options'] = [{'key': 'X-Content-Type-Options', 'value': 'nosniff'}]
    
    return response
```

### GraphQL 最適化

**GraphQL用キャッシュ戦略**

```json
{
    "CacheBehaviors": [
        {
            "PathPattern": "/graphql",
            "TargetOriginId": "graphql-api",
            "ViewerProtocolPolicy": "https-only",
            "CachePolicyId": "custom-graphql-cache",
            "OriginRequestPolicyId": "custom-graphql-origin",
            "AllowedMethods": {
                "Items": ["GET", "HEAD", "OPTIONS", "PUT", "POST", "PATCH", "DELETE"],
                "CachedMethods": {
                    "Items": ["GET", "HEAD", "POST"]
                }
            }
        }
    ]
}
```

**GraphQL クエリベースキャッシュ用CloudFront Functions**

```javascript
function handler(event) {
    var request = event.request;
    var headers = request.headers;
    var querystring = request.querystring;
    
    // POST body からGraphQLクエリを解析
    if (request.method === 'POST' && request.body && request.body.data) {
        try {
            var body = JSON.parse(request.body.data);
            
            // クエリにキャッシュ可能なオペレーション（query）が含まれるかチェック
            if (body.query && body.query.trim().startsWith('query')) {
                // クエリハッシュをキャッシュキーに追加
                var queryHash = crypto.createHash('md5').update(body.query).digest('hex');
                querystring['qh'] = { value: queryHash };
                
                // 変数もキャッシュキーに含める場合
                if (body.variables) {
                    var varsHash = crypto.createHash('md5').update(JSON.stringify(body.variables)).digest('hex');
                    querystring['vh'] = { value: varsHash };
                }
            }
        } catch (e) {
            // JSON パースエラーは無視
        }
    }
    
    return request;
}
```

## メディア配信（動画・画像）

### HLS動画ストリーミング

**動画配信用最適化設定**

```json
{
    "CacheBehaviors": [
        {
            "PathPattern": "*.m3u8",
            "TargetOriginId": "video-s3-origin",
            "ViewerProtocolPolicy": "https-only",
            "CachePolicyId": "hls-manifest-cache",
            "TTL": {
                "DefaultTTL": 10,
                "MaxTTL": 60,
                "MinTTL": 5
            },
            "Headers": {
                "Items": ["Access-Control-Allow-Origin", "Access-Control-Allow-Methods"]
            }
        },
        {
            "PathPattern": "*.ts",
            "TargetOriginId": "video-s3-origin",
            "ViewerProtocolPolicy": "https-only",
            "CachePolicyId": "hls-segment-cache",
            "TTL": {
                "DefaultTTL": 31536000,
                "MaxTTL": 31536000
            }
        }
    ]
}
```

**S3での動画ファイル構造**

```
video-bucket/
├── live-streams/
│   ├── stream-001/
│   │   ├── master.m3u8
│   │   ├── 720p/
│   │   │   ├── playlist.m3u8
│   │   │   ├── segment-001.ts
│   │   │   └── segment-002.ts
│   │   └── 1080p/
│   │       ├── playlist.m3u8
│   │       ├── segment-001.ts
│   │       └── segment-002.ts
└── vod/
    ├── movie-001/
    │   ├── master.m3u8
    │   └── variants/
```

### 画像最適化・配信

**CloudFront Functions による画像最適化**

```javascript
function handler(event) {
    var request = event.request;
    var headers = request.headers;
    var uri = request.uri;
    
    // 画像リクエストの判定
    var imageExtensions = ['.jpg', '.jpeg', '.png', '.gif', '.webp', '.svg'];
    var isImage = imageExtensions.some(ext => uri.toLowerCase().endsWith(ext));
    
    if (!isImage) {
        return request;
    }
    
    // Accept ヘッダーから最適なフォーマットを判定
    var accept = headers.accept ? headers.accept.value : '';
    var supportsWebP = accept.includes('image/webp');
    var supportsAVIF = accept.includes('image/avif');
    
    // クエリパラメータから画像サイズを取得
    var querystring = request.querystring;
    var width = querystring.w ? querystring.w.value : null;
    var height = querystring.h ? querystring.h.value : null;
    var quality = querystring.q ? querystring.q.value : '80';
    
    // 最適化されたURIを生成
    var optimizedUri = uri;
    var transformParams = [];
    
    if (width) transformParams.push('w_' + width);
    if (height) transformParams.push('h_' + height);
    if (quality !== '80') transformParams.push('q_' + quality);
    
    // 最適なフォーマット選択
    var targetFormat = '';
    if (supportsAVIF) {
        targetFormat = 'f_avif';
    } else if (supportsWebP) {
        targetFormat = 'f_webp';
    }
    
    if (targetFormat) transformParams.push(targetFormat);
    
    if (transformParams.length > 0) {
        // パスを変更して最適化処理をトリガー
        var basePath = uri.replace(/\.[^/.]+$/, "");
        var extension = uri.match(/\.[^/.]+$/)[0];
        optimizedUri = '/transform/' + transformParams.join(',') + basePath + extension;
    }
    
    request.uri = optimizedUri;
    return request;
}
```

### Lambda@Edge による画像リサイズ

**リアルタイム画像変換**

```python
import boto3
import json
from PIL import Image
import io
import base64

def lambda_handler(event, context):
    request = event['Records'][0]['cf']['request']
    
    # パラメータ解析
    uri = request['uri']
    querystring = request.get('querystring', '')
    
    # 変換パラメータの抽出
    params = parse_transform_params(uri)
    
    if not params:
        return request  # 変換不要
    
    # S3から元画像を取得
    s3 = boto3.client('s3')
    bucket_name = 'my-image-bucket'
    original_key = params['original_key']
    
    try:
        response = s3.get_object(Bucket=bucket_name, Key=original_key)
        image_content = response['Body'].read()
        
        # 画像変換
        image = Image.open(io.BytesIO(image_content))
        optimized_image = transform_image(image, params)
        
        # 最適化後の画像をBase64エンコード
        output_buffer = io.BytesIO()
        optimized_image.save(output_buffer, format=params.get('format', 'JPEG'), quality=int(params.get('quality', 80)))
        encoded_image = base64.b64encode(output_buffer.getvalue()).decode('utf-8')
        
        # レスポンス生成
        return {
            'status': '200',
            'statusDescription': 'OK',
            'headers': {
                'content-type': [{'key': 'Content-Type', 'value': f"image/{params.get('format', 'jpeg').lower()}"}],
                'cache-control': [{'key': 'Cache-Control', 'value': 'public, max-age=31536000'}],
                'content-encoding': [{'key': 'Content-Encoding', 'value': 'base64'}]
            },
            'body': encoded_image,
            'bodyEncoding': 'base64'
        }
        
    except Exception as e:
        # エラー時はオリジナルリクエストを通す
        return request

def transform_image(image, params):
    # リサイズ
    if 'width' in params or 'height' in params:
        width = int(params.get('width', image.width))
        height = int(params.get('height', image.height))
        image = image.resize((width, height), Image.Resampling.LANCZOS)
    
    # フォーマット変換
    if params.get('format') == 'webp':
        image = image.convert('RGB')
    
    return image

def parse_transform_params(uri):
    # /transform/w_300,h_200,q_85,f_webp/images/photo.jpg
    if not uri.startswith('/transform/'):
        return None
    
    parts = uri.split('/')
    if len(parts) < 4:
        return None
    
    param_string = parts[2]
    original_path = '/'.join(parts[3:])
    
    params = {'original_key': original_path}
    for param in param_string.split(','):
        if param.startswith('w_'):
            params['width'] = param[2:]
        elif param.startswith('h_'):
            params['height'] = param[2:]
        elif param.startswith('q_'):
            params['quality'] = param[2:]
        elif param.startswith('f_'):
            params['format'] = param[2:]
    
    return params
```

## CloudFront Functions / Lambda@Edge

### 使い分けのガイドライン

| 機能 | CloudFront Functions | Lambda@Edge |
|------|---------------------|-------------|
| 実行場所 | エッジロケーション | リージョナルキャッシュ |
| 実行時間上限 | 1ms | 30秒 |
| メモリ上限 | 2MB | 128MB - 10GB |
| 言語 | JavaScript (subset) | Node.js, Python |
| 外部API呼び出し | × | ✓ |
| ファイルシステムアクセス | × | ✓ |
| 料金（100万実行あたり） | $0.10 | $0.20 + Lambda料金 |

### CloudFront Functions 実用例

**A/B テスト実装**

```javascript
function handler(event) {
    var request = event.request;
    var headers = request.headers;
    var cookies = headers.cookie ? parseCookies(headers.cookie.value) : {};
    
    // 既存のA/Bテストcookieをチェック
    var variant = cookies['ab_test_variant'];
    
    if (!variant) {
        // 新しいユーザーの場合、ランダムに振り分け
        var random = Math.random();
        variant = random < 0.5 ? 'A' : 'B';
        
        // cookieを設定（レスポンスで追加）
        request.headers['x-ab-variant'] = { value: variant };
    }
    
    // バリアントに基づいてパスを変更
    if (variant === 'B') {
        request.uri = '/variant-b' + request.uri;
    }
    
    return request;
}

function parseCookies(cookieString) {
    var cookies = {};
    cookieString.split(';').forEach(function(cookie) {
        var parts = cookie.trim().split('=');
        if (parts.length === 2) {
            cookies[parts[0]] = parts[1];
        }
    });
    return cookies;
}
```

**地域別リダイレクト**

```javascript
function handler(event) {
    var request = event.request;
    var headers = request.headers;
    
    // CloudFrontが提供する地域情報
    var country = headers['cloudfront-viewer-country'] 
        ? headers['cloudfront-viewer-country'].value 
        : 'US';
    
    var uri = request.uri;
    
    // 地域別のパス設定
    var regionMappings = {
        'JP': '/ja',
        'KR': '/ko',
        'CN': '/zh-cn',
        'TW': '/zh-tw'
    };
    
    var regionPath = regionMappings[country];
    
    // ルートパスかつ対応する地域の場合
    if (uri === '/' && regionPath) {
        return {
            statusCode: 302,
            statusDescription: 'Found',
            headers: {
                'location': { value: regionPath + '/' }
            }
        };
    }
    
    return request;
}
```

### Lambda@Edge 実用例

**JWTトークン検証**

```python
import json
import jwt
import requests
from urllib.parse import parse_qs

def lambda_handler(event, context):
    request = event['Records'][0]['cf']['request']
    headers = request['headers']
    
    # 認証が必要なパスかチェック
    protected_paths = ['/admin/', '/api/private/']
    requires_auth = any(request['uri'].startswith(path) for path in protected_paths)
    
    if not requires_auth:
        return request
    
    # Authorization ヘッダーからトークンを取得
    auth_header = headers.get('authorization', [{}])[0].get('value', '')
    
    if not auth_header.startswith('Bearer '):
        return unauthorized_response()
    
    token = auth_header[7:]  # "Bearer " を除去
    
    try:
        # JWT トークンを検証
        payload = jwt.decode(
            token, 
            get_public_key(), 
            algorithms=['RS256'],
            audience='your-app-audience',
            issuer='https://your-auth-provider.com'
        )
        
        # ユーザー情報をヘッダーに追加
        headers['x-user-id'] = [{'key': 'X-User-ID', 'value': payload['sub']}]
        headers['x-user-role'] = [{'key': 'X-User-Role', 'value': payload.get('role', 'user')}]
        
        return request
        
    except jwt.InvalidTokenError:
        return unauthorized_response()

def get_public_key():
    # JWKSエンドポイントから公開鍵を取得（キャッシュ推奨）
    response = requests.get('https://your-auth-provider.com/.well-known/jwks.json')
    jwks = response.json()
    # 実際の実装では適切なキー選択ロジックが必要
    return jwks['keys'][0]

def unauthorized_response():
    return {
        'status': '401',
        'statusDescription': 'Unauthorized',
        'headers': {
            'www-authenticate': [{'key': 'WWW-Authenticate', 'value': 'Bearer realm="protected"'}]
        },
        'body': json.dumps({'error': 'Unauthorized access'})
    }
```

## セキュリティ強化

### WAF（Web Application Firewall）連携

**WAF ルール作成**

```bash
# IPセット作成（ブロックするIP）
aws wafv2 create-ip-set \
    --name "BlockedIPs" \
    --scope CLOUDFRONT \
    --ip-address-version IPV4 \
    --addresses "192.0.2.0/24" "198.51.100.0/24"

# レート制限ルール作成
aws wafv2 create-rule-group \
    --name "RateLimitRules" \
    --scope CLOUDFRONT \
    --capacity 100 \
    --rules '[
        {
            "Name": "RateLimitRule",
            "Priority": 1,
            "Statement": {
                "RateBasedStatement": {
                    "Limit": 2000,
                    "AggregateKeyType": "IP"
                }
            },
            "Action": {"Block": {}},
            "VisibilityConfig": {
                "SampledRequestsEnabled": true,
                "CloudWatchMetricsEnabled": true,
                "MetricName": "RateLimitRule"
            }
        }
    ]'

# Web ACL作成
aws wafv2 create-web-acl \
    --name "CloudFrontWAF" \
    --scope CLOUDFRONT \
    --default-action '{"Allow": {}}' \
    --rules '[
        {
            "Name": "AWSManagedRulesCommonRuleSet",
            "Priority": 1,
            "OverrideAction": {"None": {}},
            "Statement": {
                "ManagedRuleGroupStatement": {
                    "VendorName": "AWS",
                    "Name": "AWSManagedRulesCommonRuleSet"
                }
            },
            "VisibilityConfig": {
                "SampledRequestsEnabled": true,
                "CloudWatchMetricsEnabled": true,
                "MetricName": "CommonRuleSetMetric"
            }
        }
    ]'
```

### 地理的制限（ジオブロッキング）

```json
{
    "Restrictions": {
        "GeoRestriction": {
            "RestrictionType": "whitelist",
            "Locations": ["JP", "US", "CA", "GB", "AU"],
            "Quantity": 5
        }
    }
}
```

**動的ジオブロッキング（CloudFront Functions）**

```javascript
function handler(event) {
    var request = event.request;
    var headers = request.headers;
    
    var country = headers['cloudfront-viewer-country'] 
        ? headers['cloudfront-viewer-country'].value 
        : 'UNKNOWN';
    
    // 特定のパスに対する地域制限
    var restrictedPaths = ['/admin/', '/internal/'];
    var allowedCountries = ['JP', 'US'];
    
    var isRestrictedPath = restrictedPaths.some(path => 
        request.uri.startsWith(path)
    );
    
    if (isRestrictedPath && !allowedCountries.includes(country)) {
        return {
            statusCode: 403,
            statusDescription: 'Forbidden',
            headers: {
                'content-type': { value: 'application/json' }
            },
            body: JSON.stringify({
                error: 'Access denied from your location',
                country: country
            })
        };
    }
    
    return request;
}
```

### 署名付きURL実装

**Python での署名付きURL生成**

```python
import boto3
import datetime
from botocore.signers import CloudFrontSigner
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.primitives.asymmetric import rsa
from cryptography.hazmat.primitives import serialization

class CloudFrontURLSigner:
    def __init__(self, key_id, private_key_path):
        self.key_id = key_id
        with open(private_key_path, 'rb') as key_file:
            self.private_key = serialization.load_pem_private_key(
                key_file.read(),
                password=None
            )
    
    def generate_signed_url(self, url, expiration_hours=1):
        expiration_time = datetime.datetime.utcnow() + datetime.timedelta(hours=expiration_hours)
        
        cloudfront_signer = CloudFrontSigner(self.key_id, self.rsa_signer)
        
        signed_url = cloudfront_signer.generate_presigned_url(
            url=url,
            date_less_than=expiration_time
        )
        
        return signed_url
    
    def rsa_signer(self, message):
        return self.private_key.sign(message, hashes.SHA1())

# 使用例
signer = CloudFrontURLSigner('KEYPAIRID', '/path/to/private_key.pem')
signed_url = signer.generate_signed_url('https://cdn.example.com/premium-content.mp4', expiration_hours=2)
```

**Cookie-based認証**

```python
def generate_signed_cookies(self, url_pattern, expiration_hours=1):
    expiration_time = datetime.datetime.utcnow() + datetime.timedelta(hours=expiration_hours)
    
    policy = {
        "Statement": [
            {
                "Resource": url_pattern,
                "Condition": {
                    "DateLessThan": {
                        "AWS:EpochTime": int(expiration_time.timestamp())
                    }
                }
            }
        ]
    }
    
    policy_json = json.dumps(policy, separators=(',', ':'))
    policy_b64 = base64.b64encode(policy_json.encode()).decode()
    
    signature = self.rsa_signer(policy_json.encode())
    signature_b64 = base64.b64encode(signature).decode()
    
    return {
        'CloudFront-Policy': policy_b64,
        'CloudFront-Signature': signature_b64,
        'CloudFront-Key-Pair-Id': self.key_id
    }
```

## Next.js等のSPAデプロイ

### Next.js App Router対応

**静的生成 + CloudFront Functions**

```javascript
// CloudFront Functions for Next.js routing
function handler(event) {
    var request = event.request;
    var uri = request.uri;
    
    // API routesはそのまま通す
    if (uri.startsWith('/api/')) {
        return request;
    }
    
    // 静的アセットはそのまま通す
    if (uri.startsWith('/_next/') || uri.includes('.')) {
        return request;
    }
    
    // 動的ルーティング処理
    var dynamicRoutes = {
        '/blog/[slug]': '/blog/[slug].html',
        '/user/[id]': '/user/[id].html',
        '/products/[category]/[id]': '/products/[category]/[id].html'
    };
    
    // マッチするルートを探す
    for (var pattern in dynamicRoutes) {
        var regex = new RegExp('^' + pattern.replace(/\[([^\]]+)\]/g, '([^/]+)') + '$');
        if (regex.test(uri)) {
            request.uri = dynamicRoutes[pattern];
            return request;
        }
    }
    
    // 該当しない場合はindex.htmlにフォールバック
    if (uri === '/' || !uri.includes('.')) {
        request.uri = '/index.html';
    }
    
    return request;
}
```

### Vue.js / Nuxt.js対応

**Nuxt.js Static + CloudFront設定**

```javascript
// nuxt.config.js
export default {
  target: 'static',
  ssr: false,
  
  generate: {
    fallback: true // 404.html生成
  },
  
  router: {
    base: process.env.NODE_ENV === 'production' ? '/app/' : '/'
  },
  
  build: {
    publicPath: process.env.NODE_ENV === 'production' 
      ? 'https://cdn.example.com/_nuxt/' 
      : '/_nuxt/'
  }
}
```

**CloudFrontカスタムエラーページ設定**

```json
{
    "CustomErrorResponses": {
        "Quantity": 2,
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
                "ErrorCachingMinTTL": 0
            }
        ]
    }
}
```

## コスト最適化

### 価格クラス最適化

```bash
# アジア・太平洋地域のみで配信（Price Class 200）
aws cloudfront update-distribution \
    --id E1234567890123 \
    --distribution-config '{
        "PriceClass": "PriceClass_200",
        "Comment": "Asia Pacific only distribution"
    }'

# 価格クラス比較
# PriceClass_All: 全世界（最高性能、最高価格）
# PriceClass_200: 北米、欧州、アジア、中東
# PriceClass_100: 北米、欧州のみ（最安価格）
```

### キャッシュ最適化によるコスト削減

**キャッシュヒット率向上施策**

```bash
# キャッシュ統計の確認
aws cloudwatch get-metric-statistics \
    --namespace AWS/CloudFront \
    --metric-name CacheHitRate \
    --dimensions Name=DistributionId,Value=E1234567890123 \
    --start-time $(date -d '7 days ago' --iso-8601) \
    --end-time $(date --iso-8601) \
    --period 86400 \
    --statistics Average

# オリジンリクエスト数の確認
aws cloudwatch get-metric-statistics \
    --namespace AWS/CloudFront \
    --metric-name Requests \
    --dimensions Name=DistributionId,Value=E1234567890123 \
    --start-time $(date -d '7 days ago' --iso-8601) \
    --end-time $(date --iso-8601) \
    --period 86400 \
    --statistics Sum
```

**コスト最適化のCloudFront Functions**

```javascript
function handler(event) {
    var request = event.request;
    var querystring = request.querystring;
    
    // 不要なクエリパラメータを除去してキャッシュ効率向上
    var allowedParams = ['version', 'lang', 'theme'];
    var filteredQS = {};
    
    for (var key in querystring) {
        if (allowedParams.includes(key)) {
            filteredQS[key] = querystring[key];
        }
    }
    
    request.querystring = filteredQS;
    
    // User-Agent の正規化
    var userAgent = request.headers['user-agent'] ? request.headers['user-agent'].value : '';
    var deviceType = 'desktop';
    
    if (/Mobile|Android|iPhone|iPad/.test(userAgent)) {
        deviceType = 'mobile';
    } else if (/Tablet/.test(userAgent)) {
        deviceType = 'tablet';
    }
    
    // デバイスタイプをヘッダーに追加してキャッシュキーに含める
    request.headers['x-device-type'] = { value: deviceType };
    
    return request;
}
```

### 使用量監視と自動アラート

**コスト監視用Lambda関数**

```python
import boto3
import json
import datetime
from decimal import Decimal

def lambda_handler(event, context):
    cloudwatch = boto3.client('cloudwatch')
    ce = boto3.client('ce')
    sns = boto3.client('sns')
    
    # 今月のCloudFront使用量を取得
    today = datetime.datetime.now()
    start_date = today.replace(day=1).strftime('%Y-%m-%d')
    end_date = today.strftime('%Y-%m-%d')
    
    response = ce.get_cost_and_usage(
        TimePeriod={
            'Start': start_date,
            'End': end_date
        },
        Granularity='MONTHLY',
        Metrics=['BlendedCost'],
        GroupBy=[
            {
                'Type': 'SERVICE',
                'Key': 'SERVICE'
            }
        ]
    )
    
    cloudfront_cost = 0
    for result in response['ResultsByTime']:
        for group in result['Groups']:
            if 'CloudFront' in group['Keys'][0]:
                cloudfront_cost = float(group['Metrics']['BlendedCost']['Amount'])
                break
    
    # 閾値チェック
    threshold = 100.0  # $100
    
    if cloudfront_cost > threshold:
        message = f"""
        CloudFront月額コストが閾値を超えました
        
        現在の使用料金: ${cloudfront_cost:.2f}
        設定閾値: ${threshold:.2f}
        期間: {start_date} ～ {end_date}
        
        詳細確認をお願いします。
        """
        
        sns.publish(
            TopicArn='arn:aws:sns:ap-northeast-1:123456789012:cost-alerts',
            Message=message,
            Subject='CloudFront コストアラート'
        )
    
    return {
        'statusCode': 200,
        'body': json.dumps({
            'current_cost': cloudfront_cost,
            'threshold': threshold,
            'alert_triggered': cloudfront_cost > threshold
        })
    }
```

!!! success "まとめ"
    CloudFrontは単なるCDNではなく、セキュリティ、パフォーマンス、コスト最適化の中核となるサービスです。適切な設定により、グローバルスケールでの高速・安全なコンテンツ配信が実現できます。

## 関連リンク

- [CloudFront セットアップガイド](setup.md)
- [S3との連携設定](../s3/use-cases.md#cloudfront連携)
- [AWS CloudFront 料金計算ツール](https://calculator.aws/)
- [CloudFront パフォーマンス最適化ガイド](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/ConfiguringCaching.html)