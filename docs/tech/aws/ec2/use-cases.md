# 💼 EC2活用事例

実際のプロジェクトで使える具体的なEC2活用パターンとコスト最適化のコツを解説します。

## Webサーバー構築

### シンプルなWebサイト（Apache/Nginx）

#### Apache + PHP環境の構築
```bash
#!/bin/bash
# Amazon Linux 2023でのLAMP環境構築

# システム更新
sudo yum update -y

# Apache, PHP, MariaDBインストール
sudo yum install -y httpd php php-mysqli mariadb105-server

# Apache設定
sudo systemctl start httpd
sudo systemctl enable httpd

# PHP情報ページ作成
echo "<?php phpinfo(); ?>" | sudo tee /var/www/html/info.php

# ファイアウォール設定（Apache用）
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload

# MariaDB設定
sudo systemctl start mariadb
sudo systemctl enable mariadb
sudo mysql_secure_installation
```

#### Nginx + Node.js環境の構築
```bash
#!/bin/bash
# Ubuntu 22.04でのNginx + Node.js環境

# システム更新
sudo apt update && sudo apt upgrade -y

# Nginx インストール
sudo apt install -y nginx

# Node.js LTSインストール
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt install -y nodejs

# PM2（プロセス管理）
sudo npm install -g pm2

# Nginx設定例（リバースプロキシ）
sudo tee /etc/nginx/sites-available/myapp <<EOF
server {
    listen 80;
    server_name your-domain.com;
    
    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade \$http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host \$host;
        proxy_set_header X-Real-IP \$remote_addr;
        proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto \$scheme;
        proxy_cache_bypass \$http_upgrade;
    }
}
EOF

# サイト有効化
sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

### SSL証明書の設定（Let's Encrypt）

```bash
# Certbot インストール
# Amazon Linux
sudo yum install -y certbot python3-certbot-apache

# Ubuntu
sudo apt install -y certbot python3-certbot-nginx

# SSL証明書取得（Apache）
sudo certbot --apache -d your-domain.com

# SSL証明書取得（Nginx）
sudo certbot --nginx -d your-domain.com

# 自動更新設定
echo "0 12 * * * root certbot renew --quiet" | sudo tee -a /etc/crontab
```

## アプリケーションサーバー

### Python Flask アプリケーション

```bash
# Python Flask アプリ例
# app.py
from flask import Flask, jsonify
import os

app = Flask(__name__)

@app.route('/')
def hello():
    return jsonify({
        'message': 'Hello from EC2!',
        'instance_id': os.environ.get('INSTANCE_ID', 'unknown')
    })

@app.route('/health')
def health_check():
    return jsonify({'status': 'healthy'})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

```bash
# デプロイスクリプト
#!/bin/bash
# deploy-flask.sh

# Python環境セットアップ
sudo yum install -y python3 python3-pip
python3 -m venv myapp
source myapp/bin/activate

# 依存関係インストール
pip install flask gunicorn

# アプリケーション起動
gunicorn --bind 0.0.0.0:5000 app:app --daemon

# systemdサービス化
sudo tee /etc/systemd/system/myapp.service <<EOF
[Unit]
Description=My Flask App
After=network.target

[Service]
User=ec2-user
Group=ec2-user
WorkingDirectory=/home/ec2-user/myapp
Environment=PATH=/home/ec2-user/myapp/bin
ExecStart=/home/ec2-user/myapp/bin/gunicorn --bind 0.0.0.0:5000 app:app
Restart=always

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl start myapp
sudo systemctl enable myapp
```

### Node.js Express アプリケーション

```javascript
// app.js
const express = require('express');
const app = express();
const PORT = process.env.PORT || 3000;

// ミドルウェア
app.use(express.json());

// ルート
app.get('/', (req, res) => {
    res.json({
        message: 'Hello from Node.js on EC2!',
        timestamp: new Date().toISOString(),
        environment: process.env.NODE_ENV || 'development'
    });
});

app.get('/health', (req, res) => {
    res.json({ status: 'healthy' });
});

// サーバー起動
app.listen(PORT, '0.0.0.0', () => {
    console.log(`Server running on port ${PORT}`);
});
```

```json
// package.json
{
  "name": "ec2-node-app",
  "version": "1.0.0",
  "description": "Node.js app on EC2",
  "main": "app.js",
  "scripts": {
    "start": "node app.js",
    "dev": "nodemon app.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  },
  "devDependencies": {
    "nodemon": "^3.0.1"
  }
}
```

```bash
# PM2でプロセス管理
npm install -g pm2

# アプリ起動
pm2 start app.js --name "my-app"

# 自動起動設定
pm2 startup
pm2 save

# 監視・管理
pm2 status
pm2 logs my-app
pm2 restart my-app
```

## 開発環境・踏み台サーバー

### 開発環境用セットアップ

```bash
#!/bin/bash
# dev-environment.sh - 開発環境一括セットアップ

# 基本ツール
sudo yum install -y git vim htop tree wget curl unzip

# 開発言語環境
# Python
sudo yum install -y python3 python3-pip
pip3 install virtualenv

# Node.js
curl -fsSL https://rpm.nodesource.com/setup_lts.x | sudo bash -
sudo yum install -y nodejs

# Docker
sudo yum install -y docker
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -a -G docker ec2-user

# Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/download/v2.21.0/docker-compose-linux-x86_64" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# AWS CLI v2更新
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install --update

# VSCode Server（code-server）
curl -fsSL https://code-server.dev/install.sh | sh
sudo systemctl enable --now code-server@ec2-user

echo "Development environment setup completed!"
echo "VSCode Server: http://$(curl -s http://169.254.169.254/latest/meta-data/public-ipv4):8080"
```

### 踏み台サーバー（Bastion Host）

```bash
# セキュリティ強化設定
#!/bin/bash

# SSH設定強化
sudo tee -a /etc/ssh/sshd_config <<EOF
# セキュリティ設定
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
MaxAuthTries 3
ClientAliveInterval 300
ClientAliveCountMax 2

# ログ設定
LogLevel VERBOSE
EOF

sudo systemctl restart sshd

# fail2ban（侵入検知）
sudo yum install -y epel-release
sudo yum install -y fail2ban

sudo tee /etc/fail2ban/jail.local <<EOF
[DEFAULT]
bantime = 3600
findtime = 600
maxretry = 3

[sshd]
enabled = true
port = ssh
logpath = /var/log/secure
EOF

sudo systemctl start fail2ban
sudo systemctl enable fail2ban
```

## バッチ処理（スポットインスタンス活用）

### データ処理バッチの設定

```bash
# スポットインスタンス用バッチ処理
#!/bin/bash
# batch-processor.sh

# スポット価格チェック
aws ec2 describe-spot-price-history \
    --instance-types t3.medium \
    --product-descriptions "Linux/UNIX" \
    --max-items 1

# スポットインスタンス起動
aws ec2 request-spot-instances \
    --spot-price "0.05" \
    --launch-specification '{
        "ImageId": "ami-0abcdef1234567890",
        "InstanceType": "t3.medium",
        "KeyName": "my-key",
        "SecurityGroupIds": ["sg-xxxxxxxxx"],
        "SubnetId": "subnet-xxxxxxxxx",
        "UserData": "base64-encoded-script"
    }'
```

```python
# batch_job.py - Python バッチ処理例
import boto3
import json
import sys
from datetime import datetime

def process_s3_files():
    """S3からファイルを処理するバッチジョブ"""
    s3 = boto3.client('s3')
    
    bucket_name = 'my-data-bucket'
    prefix = 'input/'
    
    # ファイル一覧取得
    response = s3.list_objects_v2(
        Bucket=bucket_name,
        Prefix=prefix
    )
    
    for obj in response.get('Contents', []):
        file_key = obj['Key']
        print(f"Processing: {file_key}")
        
        # ファイル処理ロジック
        # ...
        
        # 処理済みファイルを別フォルダに移動
        copy_source = {'Bucket': bucket_name, 'Key': file_key}
        s3.copy_object(
            CopySource=copy_source,
            Bucket=bucket_name,
            Key=file_key.replace('input/', 'processed/')
        )
        s3.delete_object(Bucket=bucket_name, Key=file_key)

def main():
    start_time = datetime.now()
    print(f"Batch job started at: {start_time}")
    
    try:
        process_s3_files()
        print("Batch job completed successfully")
    except Exception as e:
        print(f"Batch job failed: {e}")
        sys.exit(1)
    
    end_time = datetime.now()
    print(f"Total processing time: {end_time - start_time}")

if __name__ == "__main__":
    main()
```

## Auto Scaling設定

### Launch Template作成

```bash
# Launch Template作成
aws ec2 create-launch-template \
    --launch-template-name web-server-template \
    --launch-template-data '{
        "ImageId": "ami-0abcdef1234567890",
        "InstanceType": "t3.micro",
        "KeyName": "my-key",
        "SecurityGroupIds": ["sg-xxxxxxxxx"],
        "UserData": "base64-encoded-startup-script",
        "IamInstanceProfile": {
            "Name": "EC2-CloudWatch-Role"
        },
        "TagSpecifications": [{
            "ResourceType": "instance",
            "Tags": [
                {"Key": "Name", "Value": "AutoScale-WebServer"},
                {"Key": "Environment", "Value": "Production"}
            ]
        }]
    }'
```

### Auto Scaling Group作成

```bash
# Auto Scaling Group作成
aws autoscaling create-auto-scaling-group \
    --auto-scaling-group-name web-server-asg \
    --launch-template '{
        "LaunchTemplateName": "web-server-template",
        "Version": "$Latest"
    }' \
    --min-size 2 \
    --max-size 6 \
    --desired-capacity 2 \
    --vpc-zone-identifier "subnet-xxxxxxxx,subnet-yyyyyyyy" \
    --target-group-arns "arn:aws:elasticloadbalancing:ap-northeast-1:123456789012:targetgroup/my-targets/1234567890123456" \
    --health-check-type ELB \
    --health-check-grace-period 300 \
    --tags '[
        {
            "Key": "Environment",
            "Value": "Production",
            "PropagateAtLaunch": true
        }
    ]'
```

### スケーリングポリシー設定

```bash
# CPU使用率ベースのスケールアウト
aws autoscaling put-scaling-policy \
    --auto-scaling-group-name web-server-asg \
    --policy-name cpu-scale-out \
    --policy-type TargetTrackingScaling \
    --target-tracking-configuration '{
        "TargetValue": 70.0,
        "PredefinedMetricSpecification": {
            "PredefinedMetricType": "ASGAverageCPUUtilization"
        },
        "ScaleOutCooldown": 300,
        "ScaleInCooldown": 300
    }'

# リクエスト数ベースのスケーリング
aws autoscaling put-scaling-policy \
    --auto-scaling-group-name web-server-asg \
    --policy-name request-count-scale \
    --policy-type TargetTrackingScaling \
    --target-tracking-configuration '{
        "TargetValue": 1000.0,
        "PredefinedMetricSpecification": {
            "PredefinedMetricType": "ALBRequestCountPerTarget",
            "ResourceLabel": "app/my-load-balancer/50dc6c495c0c9188/targetgroup/my-targets/1234567890123456"
        }
    }'
```

## Load Balancer（ALB/NLB）との連携

### Application Load Balancer設定

```bash
# ALB作成
aws elbv2 create-load-balancer \
    --name my-application-lb \
    --subnets subnet-xxxxxxxx subnet-yyyyyyyy \
    --security-groups sg-xxxxxxxxx

# ターゲットグループ作成
aws elbv2 create-target-group \
    --name my-web-servers \
    --protocol HTTP \
    --port 80 \
    --vpc-id vpc-xxxxxxxxx \
    --health-check-protocol HTTP \
    --health-check-path /health \
    --health-check-interval-seconds 30 \
    --healthy-threshold-count 2 \
    --unhealthy-threshold-count 3

# リスナー作成
aws elbv2 create-listener \
    --load-balancer-arn arn:aws:elasticloadbalancing:... \
    --protocol HTTP \
    --port 80 \
    --default-actions Type=forward,TargetGroupArn=arn:aws:elasticloadbalancing:...
```

### ヘルスチェック設定

```bash
# アプリケーションレベルのヘルスチェック例
# health.py (Flask)
from flask import Flask, jsonify
import psutil
import boto3

app = Flask(__name__)

@app.route('/health')
def health_check():
    try:
        # システムメトリクス
        cpu_percent = psutil.cpu_percent(interval=1)
        memory = psutil.virtual_memory()
        
        # データベース接続チェック（例）
        # db_status = check_database_connection()
        
        health_status = {
            'status': 'healthy',
            'timestamp': datetime.utcnow().isoformat(),
            'cpu_percent': cpu_percent,
            'memory_percent': memory.percent,
            'disk_usage': psutil.disk_usage('/').percent
        }
        
        # 閾値チェック
        if cpu_percent > 90 or memory.percent > 90:
            health_status['status'] = 'warning'
        
        return jsonify(health_status), 200
        
    except Exception as e:
        return jsonify({
            'status': 'unhealthy',
            'error': str(e)
        }), 503
```

## コスト最適化のコツ

### 1. インスタンスタイプの適正化

```bash
# CloudWatchメトリクス分析スクリプト
#!/bin/bash
# cost-optimization.sh

# CPU使用率の分析（過去1週間）
aws cloudwatch get-metric-statistics \
    --namespace AWS/EC2 \
    --metric-name CPUUtilization \
    --dimensions Name=InstanceId,Value=i-xxxxxxxxx \
    --start-time $(date -d '7 days ago' -u +%Y-%m-%dT%H:%M:%S) \
    --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
    --period 3600 \
    --statistics Average,Maximum

# メモリ使用率（カスタムメトリクス）
aws cloudwatch get-metric-statistics \
    --namespace CWAgent \
    --metric-name mem_used_percent \
    --dimensions Name=InstanceId,Value=i-xxxxxxxxx \
    --start-time $(date -d '7 days ago' -u +%Y-%m-%dT%H:%M:%S) \
    --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
    --period 3600 \
    --statistics Average,Maximum
```

### 2. Savings Plans の活用

```bash
# Savings Plans 推奨事項の確認
aws ce get-savings-plans-purchase-recommendation \
    --savings-plans-type COMPUTE_SP \
    --term-in-years ONE_YEAR \
    --payment-option PARTIAL_UPFRONT \
    --lookback-period-in-days 60

# Cost Explorer APIでコスト分析
aws ce get-cost-and-usage \
    --time-period Start=2024-01-01,End=2024-01-31 \
    --granularity MONTHLY \
    --metrics BlendedCost \
    --group-by Type=DIMENSION,Key=SERVICE
```

### 3. スケジュール型スケーリング

```bash
# 営業時間外の自動停止
aws autoscaling put-scheduled-action \
    --auto-scaling-group-name web-server-asg \
    --scheduled-action-name scale-down-evening \
    --recurrence "0 22 * * 1-5" \
    --desired-capacity 1 \
    --min-size 1 \
    --max-size 2

# 営業時間開始の自動起動
aws autoscaling put-scheduled-action \
    --auto-scaling-group-name web-server-asg \
    --scheduled-action-name scale-up-morning \
    --recurrence "0 8 * * 1-5" \
    --desired-capacity 2 \
    --min-size 2 \
    --max-size 6
```

### 4. リソースタグ付けによる管理

```bash
# コスト配分用タグ設定
aws ec2 create-tags \
    --resources i-xxxxxxxxx \
    --tags Key=Project,Value=WebApp \
           Key=Environment,Value=Production \
           Key=Team,Value=DevOps \
           Key=CostCenter,Value=IT-001

# タグベースの課金レポート
aws ce get-cost-and-usage \
    --time-period Start=2024-01-01,End=2024-01-31 \
    --granularity MONTHLY \
    --metrics BlendedCost \
    --group-by Type=TAG,Key=Project
```

## 監視・アラート設定

```bash
# CloudWatch アラーム設定
aws cloudwatch put-metric-alarm \
    --alarm-name "High-CPU-Usage" \
    --alarm-description "Alert when CPU exceeds 80%" \
    --metric-name CPUUtilization \
    --namespace AWS/EC2 \
    --statistic Average \
    --period 300 \
    --threshold 80 \
    --comparison-operator GreaterThanThreshold \
    --dimensions Name=InstanceId,Value=i-xxxxxxxxx \
    --evaluation-periods 2 \
    --alarm-actions arn:aws:sns:ap-northeast-1:123456789012:my-topic
```

!!! tip "コスト最適化のベストプラクティス"
    1. **右サイズ**: CloudWatchメトリクスを定期的に確認し、過剰スペックを避ける
    2. **混合利用**: オンデマンド + Savings Plans + スポットの組み合わせ
    3. **自動化**: スケジューリング機能を活用した自動停止・起動
    4. **タグ管理**: プロジェクトやチーム別のコスト可視化
    5. **定期見直し**: 月次でのコスト分析と最適化施策の実施

## 次のステップ

実用的なEC2活用を学んだら：

1. **[セキュリティ](security.md)** - セキュリティ強化の実装
2. **コンテナ化** - DockerやECSへの移行
3. **Infrastructure as Code** - CloudFormationやTerraformの活用
4. **マルチリージョン展開** - DR（災害復旧）対策の実装