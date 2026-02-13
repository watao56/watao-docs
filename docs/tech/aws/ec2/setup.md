# 🚀 EC2セットアップガイド

EC2インスタンスの作成から初期設定まで、コンソールとCLI両方での手順を詳しく解説します。

## 事前準備

### 1. AWS CLIの設定確認
```bash
# AWS CLIが設定済みか確認
aws sts get-caller-identity

# 実行結果例
{
    "UserId": "AIDAXXXXXXXXXXXX",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/myuser"
}

# リージョン設定確認
aws configure get region
```

### 2. VPCとサブネットの確認
```bash
# デフォルトVPCの確認
aws ec2 describe-vpcs --filters "Name=isDefault,Values=true"

# パブリックサブネットの確認
aws ec2 describe-subnets \
    --filters "Name=vpc-id,Values=vpc-xxxxxxxx" \
              "Name=map-public-ip-on-launch,Values=true"
```

## AMI選択ガイド

### 主要なAMI

#### Amazon Linux 2023
```bash
# Amazon Linux 2023の最新AMI検索
aws ec2 describe-images \
    --owners amazon \
    --filters "Name=name,Values=al2023-ami-*" \
              "Name=architecture,Values=x86_64" \
              "Name=state,Values=available" \
    --query 'Images | sort_by(@, &CreationDate) | [-1].ImageId' \
    --output text

# 実行結果例: ami-0abcdef1234567890
```

- **特徴**: AWSが推奨する最新OS、最適化済み
- **用途**: 本番環境、新規プロジェクト

#### Ubuntu Server
```bash
# Ubuntu 22.04 LTS AMI検索
aws ec2 describe-images \
    --owners 099720109477 \
    --filters "Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-jammy-22.04*" \
              "Name=architecture,Values=x86_64" \
    --query 'Images | sort_by(@, &CreationDate) | [-1].ImageId' \
    --output text
```

- **特徴**: 豊富なパッケージ、大きなコミュニティ
- **用途**: 開発環境、オープンソースソフトウェア

#### Windows Server
```bash
# Windows Server 2022 AMI検索
aws ec2 describe-images \
    --owners amazon \
    --filters "Name=name,Values=Windows_Server-2022*English*Base*" \
    --query 'Images | sort_by(@, &CreationDate) | [-1].ImageId' \
    --output text
```

### AMI選択のポイント

| 項目 | Amazon Linux | Ubuntu | CentOS/RHEL | Windows |
|------|-------------|--------|-------------|---------|
| **コスト** | 安い | 安い | 普通 | 高い（ライセンス料） |
| **パフォーマンス** | 最適化済み | 良い | 良い | 普通 |
| **パッケージ** | yum/dnf | apt | yum/dnf | GUI/PowerShell |
| **サポート** | AWS公式 | コミュニティ | Red Hat | Microsoft |

## キーペア作成・管理

### コンソールでの作成
```bash
# AWSコンソール手順
1. EC2コンソール → ネットワーク&セキュリティ → キーペア
2. 「キーペアの作成」
3. 名前: my-ec2-key
4. ファイル形式: .pem（Linux/Mac）、.ppk（Windows PuTTY）
5. 「キーペアを作成」→ 秘密鍵ファイルダウンロード
```

### CLI での作成
```bash
# キーペア作成
aws ec2 create-key-pair \
    --key-name my-ec2-key \
    --key-format pem \
    --query 'KeyMaterial' \
    --output text > my-ec2-key.pem

# 権限設定（重要）
chmod 400 my-ec2-key.pem

# 既存キーペア一覧
aws ec2 describe-key-pairs
```

!!! danger "秘密鍵の管理"
    - 秘密鍵ファイルは絶対に第三者と共有しない
    - バックアップを安全な場所に保管
    - 権限は400（所有者のみ読み取り可能）に設定

### SSH接続用のconfig設定

```bash
# ~/.ssh/configに追加
Host my-ec2
    HostName <EC2のパブリックIP>
    User ec2-user
    IdentityFile ~/my-ec2-key.pem
    ServerAliveInterval 60

# 使用例
ssh my-ec2
```

## セキュリティグループ設定

### 基本的なセキュリティグループ作成

```bash
# セキュリティグループ作成
aws ec2 create-security-group \
    --group-name web-server-sg \
    --description "Security group for web server" \
    --vpc-id vpc-xxxxxxxx

# HTTP（80番）許可
aws ec2 authorize-security-group-ingress \
    --group-id sg-xxxxxxxxx \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/0

# HTTPS（443番）許可
aws ec2 authorize-security-group-ingress \
    --group-id sg-xxxxxxxxx \
    --protocol tcp \
    --port 443 \
    --cidr 0.0.0.0/0

# SSH（22番）許可（特定IPからのみ）
aws ec2 authorize-security-group-ingress \
    --group-id sg-xxxxxxxxx \
    --protocol tcp \
    --port 22 \
    --cidr YOUR-IP/32
```

### 推奨セキュリティグループ設定

```bash
# Web Server（公開）
Inbound Rules:
- HTTP (80)    0.0.0.0/0
- HTTPS (443)  0.0.0.0/0  
- SSH (22)     YOUR-IP/32

# Application Server（内部）
Inbound Rules:
- Custom (8080) sg-web-server-sg
- SSH (22)      YOUR-IP/32

# Database Server（最内部）
Inbound Rules:
- MySQL (3306)  sg-app-server-sg
- SSH (22)      sg-bastion-sg
```

!!! warning "セキュリティグループのポリシー"
    - 最小権限の原則：必要最小限のポートのみ開放
    - 0.0.0.0/0の使用は最小限に
    - 定期的な見直しと不要ルールの削除

## インスタンス作成手順

### コンソールでの作成

```bash
# AWSコンソール手順
1. EC2コンソール → インスタンス → 「インスタンスを起動」

2. AMI選択
   - Amazon Linux 2023 AMI

3. インスタンスタイプ
   - t3.micro（無料枠）

4. キーペア
   - 既存: my-ec2-key
   - 新規作成も可

5. ネットワーク設定
   - VPC: デフォルト
   - サブネット: パブリックサブネット
   - パブリックIP: 有効

6. セキュリティグループ
   - 新規作成または既存選択

7. ストレージ設定
   - EBS: 8GB gp3（無料枠）

8. 詳細設定（オプション）
   - ユーザーデータスクリプト
   - IAMロール
   - 削除保護

9. 確認・起動
```

### CLI での作成

```bash
# インスタンス作成（基本）
aws ec2 run-instances \
    --image-id ami-0abcdef1234567890 \
    --instance-type t3.micro \
    --key-name my-ec2-key \
    --security-group-ids sg-xxxxxxxxx \
    --subnet-id subnet-xxxxxxxxx \
    --associate-public-ip-address

# 詳細設定付きで作成
aws ec2 run-instances \
    --image-id ami-0abcdef1234567890 \
    --instance-type t3.micro \
    --key-name my-ec2-key \
    --security-group-ids sg-xxxxxxxxx \
    --subnet-id subnet-xxxxxxxxx \
    --associate-public-ip-address \
    --block-device-mappings '[
        {
            "DeviceName": "/dev/xvda",
            "Ebs": {
                "VolumeSize": 20,
                "VolumeType": "gp3",
                "DeleteOnTermination": true
            }
        }
    ]' \
    --tag-specifications 'ResourceType=instance,Tags=[
        {Key=Name,Value=MyWebServer},
        {Key=Environment,Value=Development}
    ]' \
    --user-data file://user-data.sh \
    --iam-instance-profile Name=EC2-S3-Role
```

### ユーザーデータスクリプト例

```bash
#!/bin/bash
# user-data.sh - インスタンス起動時に実行

# システム更新
yum update -y

# 基本パッケージインストール
yum install -y git htop vim

# Apache インストール・起動
yum install -y httpd
systemctl start httpd
systemctl enable httpd

# 簡単なWebページ作成
echo "<h1>Hello from EC2!</h1>" > /var/www/html/index.html

# CloudWatchエージェントインストール
wget https://s3.amazonaws.com/amazoncloudwatch-agent/amazon_linux/amd64/latest/amazon-cloudwatch-agent.rpm
rpm -U ./amazon-cloudwatch-agent.rpm
```

## Elastic IP設定

### Elastic IP（静的パブリックIP）の作成

```bash
# Elastic IP割り当て
aws ec2 allocate-address --domain vpc

# 実行結果例
{
    "AllocationId": "eipalloc-xxxxxxxxx",
    "PublicIp": "203.0.113.25",
    "Domain": "vpc"
}

# インスタンスに関連付け
aws ec2 associate-address \
    --instance-id i-xxxxxxxxx \
    --allocation-id eipalloc-xxxxxxxxx

# 関連付け解除
aws ec2 disassociate-address --allocation-id eipalloc-xxxxxxxxx

# 削除（課金停止）
aws ec2 release-address --allocation-id eipalloc-xxxxxxxxx
```

!!! warning "Elastic IPの課金"
    Elastic IPは使用していない（インスタンスに未関連付け）状態で課金されます。使わない場合は必ず削除しましょう。

## SSH接続方法

### Linux/Mac からの接続
```bash
# 基本接続
ssh -i my-ec2-key.pem ec2-user@<パブリックIP>

# 初回接続時（フィンガープリント確認）
The authenticity of host '203.0.113.25' can't be established.
ECDSA key fingerprint is SHA256:xxxxxxxxxxxxxxx.
Are you sure you want to continue connecting (yes/no)? yes

# 接続オプション
ssh -i my-ec2-key.pem ec2-user@<パブリックIP> \
    -o ServerAliveInterval=60 \
    -o ServerAliveCountMax=3
```

### Windows からの接続

#### PuTTY使用
```bash
# 事前準備
1. PuTTYgen で .pem を .ppk に変換
2. PuTTY設定
   - Host Name: ec2-user@<パブリックIP>
   - Port: 22
   - Connection Type: SSH
   - Auth: Private key file (.ppk)
```

#### Windows PowerShell/WSL
```bash
# Windows 10/11 標準のOpenSSH使用
ssh -i my-ec2-key.pem ec2-user@<パブリックIP>
```

### デフォルトユーザー名

| AMI | ユーザー名 |
|-----|-----------|
| Amazon Linux | ec2-user |
| Ubuntu | ubuntu |
| CentOS | centos |
| RHEL | ec2-user |
| Windows | Administrator |

## 初期セットアップスクリプト例

### Amazon Linux 2023 用
```bash
#!/bin/bash
# setup-amazon-linux.sh

# システム更新
sudo yum update -y

# 開発ツール
sudo yum groupinstall -y "Development Tools"
sudo yum install -y git vim htop tree wget curl

# Docker インストール
sudo yum install -y docker
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -a -G docker ec2-user

# Node.js インストール（最新LTS）
curl -fsSL https://rpm.nodesource.com/setup_lts.x | sudo bash -
sudo yum install -y nodejs

# Python3 & pip
sudo yum install -y python3 python3-pip

# AWS CLI v2 更新
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install --update

# CloudWatch エージェント
wget https://s3.amazonaws.com/amazoncloudwatch-agent/amazon_linux/amd64/latest/amazon-cloudwatch-agent.rpm
sudo rpm -U ./amazon-cloudwatch-agent.rpm

echo "Setup completed! Please logout and login again to apply group changes."
```

### Ubuntu 22.04 用
```bash
#!/bin/bash
# setup-ubuntu.sh

# システム更新
sudo apt update && sudo apt upgrade -y

# 基本パッケージ
sudo apt install -y curl wget git vim htop tree build-essential

# Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker ubuntu

# Node.js LTS
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt install -y nodejs

# Python3 (通常は既にインストール済み)
sudo apt install -y python3-pip python3-venv

# AWS CLI v2
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# snap パッケージ
sudo apt install -y snapd

echo "Setup completed! Please logout and login again."
```

## インスタンス管理コマンド

```bash
# インスタンス一覧
aws ec2 describe-instances \
    --query 'Reservations[].Instances[].[InstanceId,State.Name,PublicIpAddress,PrivateIpAddress,Tags[?Key==`Name`].Value | [0]]' \
    --output table

# インスタンス停止
aws ec2 stop-instances --instance-ids i-xxxxxxxxx

# インスタンス開始
aws ec2 start-instances --instance-ids i-xxxxxxxxx

# インスタンス再起動
aws ec2 reboot-instances --instance-ids i-xxxxxxxxx

# インスタンス削除
aws ec2 terminate-instances --instance-ids i-xxxxxxxxx

# インスタンス詳細
aws ec2 describe-instances --instance-ids i-xxxxxxxxx
```

## 次のステップ

セットアップが完了したら：

1. **[活用事例](use-cases.md)** - Webサーバーやアプリケーション構築の実例
2. **[セキュリティ](security.md)** - セキュリティ強化のベストプラクティス
3. **モニタリング設定** - CloudWatchでパフォーマンス監視
4. **バックアップ設定** - EBSスナップショットとAMI作成

!!! tip "トラブルシューティング"
    接続できない場合は：
    1. セキュリティグループでSSH（22番）が許可されているか確認
    2. パブリックIPが割り当てられているか確認  
    3. キーペアのパスと権限（400）を確認
    4. インスタンスが「running」状態か確認