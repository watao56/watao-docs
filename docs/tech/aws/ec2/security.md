# 🔐 EC2セキュリティ

EC2インスタンスを安全に運用するためのセキュリティベストプラクティスを体系的に解説します。

## セキュリティグループのベストプラクティス

### 最小権限の原則

```bash
# ❌ 悪い例：全開放
aws ec2 authorize-security-group-ingress \
    --group-id sg-xxxxxxxxx \
    --protocol tcp \
    --port 22 \
    --cidr 0.0.0.0/0

# ✅ 良い例：特定IPからのみ許可
aws ec2 authorize-security-group-ingress \
    --group-id sg-xxxxxxxxx \
    --protocol tcp \
    --port 22 \
    --cidr YOUR_IP/32

# ✅ 更に良い例：VPN/踏み台サーバー経由
aws ec2 authorize-security-group-ingress \
    --group-id sg-xxxxxxxxx \
    --protocol tcp \
    --port 22 \
    --source-group sg-bastion-server
```

### 階層化セキュリティ設計

```bash
# Web層（インターネット向け）
Web-SG:
  Inbound:
    - HTTP (80)   from 0.0.0.0/0
    - HTTPS (443) from 0.0.0.0/0
    - SSH (22)    from Admin-IP/32
  Outbound:
    - HTTP (80)   to App-SG
    - MySQL (3306) to DB-SG

# アプリケーション層（内部通信）
App-SG:
  Inbound:
    - Custom (8080) from Web-SG
    - SSH (22)      from Bastion-SG
  Outbound:
    - MySQL (3306)  to DB-SG
    - HTTPS (443)   to 0.0.0.0/0

# データベース層（最内部）
DB-SG:
  Inbound:
    - MySQL (3306) from App-SG
    - SSH (22)     from Bastion-SG
  Outbound:
    - HTTPS (443)  to 0.0.0.0/0 (パッチ適用用)
```

### セキュリティグループ作成スクリプト

```bash
#!/bin/bash
# create-security-groups.sh

VPC_ID="vpc-xxxxxxxxx"
ADMIN_IP="YOUR_IP/32"

# Web Tier Security Group
WEB_SG=$(aws ec2 create-security-group \
    --group-name web-tier-sg \
    --description "Security group for web servers" \
    --vpc-id $VPC_ID \
    --query 'GroupId' --output text)

aws ec2 authorize-security-group-ingress \
    --group-id $WEB_SG \
    --protocol tcp --port 80 --cidr 0.0.0.0/0

aws ec2 authorize-security-group-ingress \
    --group-id $WEB_SG \
    --protocol tcp --port 443 --cidr 0.0.0.0/0

aws ec2 authorize-security-group-ingress \
    --group-id $WEB_SG \
    --protocol tcp --port 22 --cidr $ADMIN_IP

# Application Tier Security Group
APP_SG=$(aws ec2 create-security-group \
    --group-name app-tier-sg \
    --description "Security group for application servers" \
    --vpc-id $VPC_ID \
    --query 'GroupId' --output text)

aws ec2 authorize-security-group-ingress \
    --group-id $APP_SG \
    --protocol tcp --port 8080 --source-group $WEB_SG

# Database Tier Security Group  
DB_SG=$(aws ec2 create-security-group \
    --group-name db-tier-sg \
    --description "Security group for database servers" \
    --vpc-id $VPC_ID \
    --query 'GroupId' --output text)

aws ec2 authorize-security-group-ingress \
    --group-id $DB_SG \
    --protocol tcp --port 3306 --source-group $APP_SG

echo "Web SG: $WEB_SG"
echo "App SG: $APP_SG"
echo "DB SG: $DB_SG"
```

## IAMロール設定

### EC2用IAMロールの作成

```bash
# 信頼関係ポリシー（EC2がロールを引き受け可能）
cat > trust-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF

# IAMロール作成
aws iam create-role \
    --role-name EC2-S3-CloudWatch-Role \
    --assume-role-policy-document file://trust-policy.json

# S3アクセス許可ポリシー
cat > s3-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject"
      ],
      "Resource": "arn:aws:s3:::my-app-bucket/*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket"
      ],
      "Resource": "arn:aws:s3:::my-app-bucket"
    }
  ]
}
EOF

# ポリシーを作成してロールにアタッチ
aws iam create-policy \
    --policy-name EC2-S3-Access \
    --policy-document file://s3-policy.json

aws iam attach-role-policy \
    --role-name EC2-S3-CloudWatch-Role \
    --policy-arn arn:aws:iam::ACCOUNT-ID:policy/EC2-S3-Access

# AWS管理ポリシーをアタッチ（CloudWatch）
aws iam attach-role-policy \
    --role-name EC2-S3-CloudWatch-Role \
    --policy-arn arn:aws:iam::aws:policy/CloudWatchAgentServerPolicy

# インスタンスプロファイル作成・関連付け
aws iam create-instance-profile \
    --instance-profile-name EC2-S3-CloudWatch-Profile

aws iam add-role-to-instance-profile \
    --instance-profile-name EC2-S3-CloudWatch-Profile \
    --role-name EC2-S3-CloudWatch-Role
```

### 最小権限ポリシーの例

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "CloudWatchMetrics",
      "Effect": "Allow",
      "Action": [
        "cloudwatch:PutMetricData",
        "logs:PutLogEvents",
        "logs:CreateLogGroup",
        "logs:CreateLogStream"
      ],
      "Resource": "*"
    },
    {
      "Sid": "S3SpecificBucket",
      "Effect": "Allow", 
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::my-specific-bucket/*"
    },
    {
      "Sid": "ParameterStoreRead",
      "Effect": "Allow",
      "Action": [
        "ssm:GetParameter",
        "ssm:GetParameters"
      ],
      "Resource": "arn:aws:ssm:*:*:parameter/myapp/*"
    }
  ]
}
```

!!! warning "IAMの注意点"
    - `*` リソースの使用は最小限に
    - 定期的な権限の見直しと不要権限の削除
    - 本番環境では個別アプリケーション専用のロールを作成

## SSH鍵管理

### SSH設定強化

```bash
# SSH設定ファイル（/etc/ssh/sshd_config）の強化
sudo tee -a /etc/ssh/sshd_config <<EOF

# セキュリティ強化設定
Protocol 2
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
MaxAuthTries 3
MaxStartups 3
LoginGraceTime 20

# 接続維持設定
ClientAliveInterval 300
ClientAliveCountMax 2
TCPKeepAlive no

# ユーザー制限
AllowUsers ec2-user ubuntu
DenyUsers root

# ログ設定
LogLevel VERBOSE
SyslogFacility AUTH

# 暗号化設定（強力なアルゴリズムのみ許可）
KexAlgorithms curve25519-sha256@libssh.org,diffie-hellman-group16-sha512
Ciphers aes256-gcm@openssh.com,aes128-gcm@openssh.com,aes256-ctr,aes192-ctr,aes128-ctr
MACs hmac-sha2-256-etm@openssh.com,hmac-sha2-512-etm@openssh.com
EOF

# 設定反映
sudo systemctl restart sshd
```

### 多要素認証（MFA）設定

```bash
# Google Authenticator（TOTP）のインストール
# Amazon Linux
sudo yum install -y epel-release
sudo yum install -y google-authenticator

# Ubuntu
sudo apt install -y libpam-google-authenticator

# ユーザーごとの設定
google-authenticator
# QRコードをモバイルアプリでスキャン

# PAM設定（/etc/pam.d/sshd）
sudo tee -a /etc/pam.d/sshd <<EOF
auth required pam_google_authenticator.so
EOF

# SSH設定でMFA有効化
sudo tee -a /etc/ssh/sshd_config <<EOF
ChallengeResponseAuthentication yes
AuthenticationMethods publickey,keyboard-interactive
EOF

sudo systemctl restart sshd
```

### SSH Certificate Authority (CA)

```bash
# CA用のキーペア作成
ssh-keygen -t rsa -b 4096 -f ssh_ca_key

# ホスト証明書の作成
sudo ssh-keygen -s ssh_ca_key \
    -I "web-server-01" \
    -h \
    -n "web-server-01.internal,10.0.1.100" \
    -V +365d \
    /etc/ssh/ssh_host_rsa_key.pub

# クライアント設定（~/.ssh/known_hosts）
@cert-authority *.internal,10.0.* ssh-rsa AAAAB3NzaC1yc2E...
```

## Systems Manager (SSM) Session Manager

### SSMエージェントの設定

```bash
# Amazon Linux（標準でインストール済み）
sudo systemctl status amazon-ssm-agent

# Ubuntu（手動インストール）
sudo snap install amazon-ssm-agent --classic
sudo systemctl start snap.amazon-ssm-agent.amazon-ssm-agent.service

# 設定確認
sudo systemctl status snap.amazon-ssm-agent.amazon-ssm-agent.service
```

### SSM経由でのアクセス

```bash
# Session Managerでの接続
aws ssm start-session --target i-xxxxxxxxx

# ポートフォワーディング（ローカル → EC2 → RDS）
aws ssm start-session \
    --target i-xxxxxxxxx \
    --document-name AWS-StartPortForwardingSessionToRemoteHost \
    --parameters '{"host":["mydb.cluster-xxxxxxxxx.ap-northeast-1.rds.amazonaws.com"],"portNumber":["3306"],"localPortNumber":["3306"]}'

# ファイル転送
# アップロード
aws s3 cp myfile.txt s3://my-ssm-bucket/
aws ssm send-command \
    --instance-ids "i-xxxxxxxxx" \
    --document-name "AWS-RunShellScript" \
    --parameters 'commands=["aws s3 cp s3://my-ssm-bucket/myfile.txt /tmp/"]'

# ダウンロード
aws ssm send-command \
    --instance-ids "i-xxxxxxxxx" \
    --document-name "AWS-RunShellScript" \
    --parameters 'commands=["aws s3 cp /var/log/application.log s3://my-ssm-bucket/"]'
```

### Session Manager設定（JSON）

```json
{
  "schemaVersion": "1.0",
  "description": "Session Manager preferences",
  "sessionType": "Standard_Stream",
  "inputs": {
    "s3BucketName": "my-session-logs",
    "s3KeyPrefix": "session-logs/",
    "s3EncryptionEnabled": true,
    "cloudWatchLogGroupName": "session-manager-logs",
    "cloudWatchEncryptionEnabled": true,
    "runAsEnabled": false,
    "runAsDefaultUser": "",
    "idleSessionTimeout": "20",
    "maxSessionDuration": "60",
    "shellProfile": {
      "windows": "",
      "linux": "cd /home/ec2-user && exec bash -l"
    }
  }
}
```

## パッチ管理

### AWS Systems Manager Patch Manager

```bash
# パッチベースライン作成（Amazon Linux）
aws ssm create-patch-baseline \
    --name "AL2023-Production-Baseline" \
    --operating-system AMAZON_LINUX_2 \
    --description "Security and critical patches for AL2023" \
    --approval-rules '{
        "PatchRules": [
            {
                "PatchFilterGroup": {
                    "PatchFilters": [
                        {
                            "Key": "CLASSIFICATION",
                            "Values": ["Security", "Critical"]
                        }
                    ]
                },
                "ApproveAfterDays": 0,
                "ComplianceLevel": "CRITICAL"
            }
        ]
    }'

# メンテナンスウィンドウ作成
aws ssm create-maintenance-window \
    --name "Production-Patching-Window" \
    --description "Patching window for production servers" \
    --duration 4 \
    --cutoff 1 \
    --schedule "cron(0 2 ? * SUN *)" \
    --schedule-timezone "Asia/Tokyo"

# パッチタスク登録
aws ssm register-task-with-maintenance-window \
    --window-id mw-xxxxxxxxx \
    --task-arn "AWS-RunPatchBaseline" \
    --service-role-arn "arn:aws:iam::account:role/MaintenanceWindowRole" \
    --task-type "RUN_COMMAND" \
    --targets '[
        {
            "Key": "tag:PatchGroup",
            "Values": ["Production"]
        }
    ]' \
    --task-parameters '{
        "Operation": {
            "Values": ["Install"]
        }
    }'
```

### 手動パッチ適用

```bash
# セキュリティアップデートのみ適用
# Amazon Linux
sudo yum update --security -y

# Ubuntu
sudo apt update && sudo apt upgrade -y
sudo apt install unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades

# 再起動が必要か確認
# Amazon Linux
needs-restarting -r

# Ubuntu
ls /var/run/reboot-required* 2>/dev/null && echo "reboot required"
```

## モニタリング（CloudWatch）

### CloudWatchエージェント設定

```json
{
  "agent": {
    "metrics_collection_interval": 60,
    "run_as_user": "cwagent"
  },
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_path": "/var/log/messages",
            "log_group_name": "/aws/ec2/system",
            "log_stream_name": "{instance_id}/messages",
            "timezone": "Asia/Tokyo"
          },
          {
            "file_path": "/var/log/secure",
            "log_group_name": "/aws/ec2/security",
            "log_stream_name": "{instance_id}/secure",
            "timezone": "Asia/Tokyo"
          },
          {
            "file_path": "/var/log/httpd/access_log",
            "log_group_name": "/aws/ec2/apache",
            "log_stream_name": "{instance_id}/access",
            "timezone": "Asia/Tokyo"
          }
        ]
      }
    }
  },
  "metrics": {
    "namespace": "CWAgent",
    "metrics_collected": {
      "cpu": {
        "measurement": [
          "cpu_usage_idle",
          "cpu_usage_iowait",
          "cpu_usage_user",
          "cpu_usage_system"
        ],
        "metrics_collection_interval": 60
      },
      "disk": {
        "measurement": [
          "used_percent"
        ],
        "metrics_collection_interval": 60,
        "resources": [
          "*"
        ]
      },
      "diskio": {
        "measurement": [
          "io_time",
          "read_bytes",
          "write_bytes",
          "reads",
          "writes"
        ],
        "metrics_collection_interval": 60,
        "resources": [
          "*"
        ]
      },
      "mem": {
        "measurement": [
          "mem_used_percent"
        ],
        "metrics_collection_interval": 60
      },
      "netstat": {
        "measurement": [
          "tcp_established",
          "tcp_time_wait"
        ],
        "metrics_collection_interval": 60
      },
      "swap": {
        "measurement": [
          "swap_used_percent"
        ],
        "metrics_collection_interval": 60
      }
    }
  }
}
```

### セキュリティ監視アラート

```bash
# 異常なログイン検知
aws cloudwatch put-metric-alarm \
    --alarm-name "SSH-Failed-Logins" \
    --alarm-description "Multiple SSH login failures detected" \
    --metric-name "SSHFailedLogins" \
    --namespace "Custom/Security" \
    --statistic Sum \
    --period 300 \
    --threshold 5 \
    --comparison-operator GreaterThanThreshold \
    --evaluation-periods 1 \
    --alarm-actions "arn:aws:sns:ap-northeast-1:123456789012:security-alerts"

# Root権限使用検知
aws cloudwatch put-metric-alarm \
    --alarm-name "Root-Command-Usage" \
    --alarm-description "Root command usage detected" \
    --metric-name "RootCommandUsage" \
    --namespace "Custom/Security" \
    --statistic Sum \
    --period 60 \
    --threshold 0 \
    --comparison-operator GreaterThanThreshold \
    --evaluation-periods 1
```

## バックアップ（EBSスナップショット・AMI）

### 自動スナップショット作成

```bash
# Lambda関数でEBSスナップショット自動化
import boto3
import json
from datetime import datetime, timedelta

def lambda_handler(event, context):
    ec2 = boto3.client('ec2')
    
    # タグ付きインスタンスを検索
    response = ec2.describe_instances(
        Filters=[
            {'Name': 'tag:Backup', 'Values': ['true']},
            {'Name': 'instance-state-name', 'Values': ['running', 'stopped']}
        ]
    )
    
    for reservation in response['Reservations']:
        for instance in reservation['Instances']:
            instance_id = instance['InstanceId']
            
            # EBSボリュームのスナップショット作成
            for device in instance.get('BlockDeviceMappings', []):
                volume_id = device['Ebs']['VolumeId']
                
                snapshot = ec2.create_snapshot(
                    VolumeId=volume_id,
                    Description=f'Backup of {volume_id} from {instance_id}',
                    TagSpecifications=[
                        {
                            'ResourceType': 'snapshot',
                            'Tags': [
                                {'Key': 'Name', 'Value': f'{instance_id}-backup'},
                                {'Key': 'InstanceId', 'Value': instance_id},
                                {'Key': 'CreateDate', 'Value': datetime.now().strftime('%Y-%m-%d')}
                            ]
                        }
                    ]
                )
                
                print(f"Created snapshot {snapshot['SnapshotId']} for {volume_id}")
    
    # 古いスナップショット削除（30日以上前）
    cutoff_date = datetime.now() - timedelta(days=30)
    old_snapshots = ec2.describe_snapshots(
        OwnerIds=['self'],
        Filters=[
            {'Name': 'status', 'Values': ['completed']},
            {'Name': 'start-time', 'Values': [f'{cutoff_date.strftime("%Y-%m-%d")}*']}
        ]
    )
    
    for snapshot in old_snapshots['Snapshots']:
        if snapshot['StartTime'].replace(tzinfo=None) < cutoff_date:
            ec2.delete_snapshot(SnapshotId=snapshot['SnapshotId'])
            print(f"Deleted old snapshot {snapshot['SnapshotId']}")
    
    return {
        'statusCode': 200,
        'body': json.dumps('Backup completed successfully')
    }
```

### AMI自動作成・管理

```bash
# AMI作成スクリプト
#!/bin/bash
# create-ami-backup.sh

INSTANCE_ID="i-xxxxxxxxx"
AMI_NAME="WebServer-$(date +%Y%m%d-%H%M%S)"
DESCRIPTION="Automated AMI backup for $INSTANCE_ID"

# AMI作成
AMI_ID=$(aws ec2 create-image \
    --instance-id $INSTANCE_ID \
    --name "$AMI_NAME" \
    --description "$DESCRIPTION" \
    --no-reboot \
    --query 'ImageId' \
    --output text)

echo "Created AMI: $AMI_ID"

# タグ付け
aws ec2 create-tags \
    --resources $AMI_ID \
    --tags Key=BackupDate,Value=$(date +%Y-%m-%d) \
           Key=InstanceId,Value=$INSTANCE_ID \
           Key=AutomatedBackup,Value=true

# 古いAMI削除（30日以上前）
aws ec2 describe-images \
    --owners self \
    --filters "Name=tag:AutomatedBackup,Values=true" \
              "Name=tag:InstanceId,Values=$INSTANCE_ID" \
    --query 'Images[?CreationDate<=`'$(date -d '30 days ago' -u +%Y-%m-%d)'`].[ImageId,CreationDate]' \
    --output text | \
while read ami_id creation_date; do
    if [ ! -z "$ami_id" ]; then
        echo "Deleting old AMI: $ami_id (created: $creation_date)"
        aws ec2 deregister-image --image-id $ami_id
        
        # 関連するスナップショット削除
        aws ec2 describe-images --image-ids $ami_id \
            --query 'Images[0].BlockDeviceMappings[*].Ebs.SnapshotId' \
            --output text | \
        while read snapshot_id; do
            if [ ! -z "$snapshot_id" ] && [ "$snapshot_id" != "None" ]; then
                aws ec2 delete-snapshot --snapshot-id $snapshot_id
                echo "Deleted snapshot: $snapshot_id"
            fi
        done
    fi
done
```

## セキュリティツールとサービス

### AWS Inspector（脆弱性評価）

```bash
# Inspector評価テンプレート作成
aws inspector create-assessment-template \
    --assessment-target-arn arn:aws:inspector:region:account:assessmenttarget/target-id \
    --assessment-template-name "Security-Assessment" \
    --duration-in-seconds 3600 \
    --rules-package-arns \
        arn:aws:inspector:ap-northeast-1:438731021216:rulespackage/0-gHP9oWNT \
        arn:aws:inspector:ap-northeast-1:438731021216:rulespackage/0-7WNjqgGu \
        arn:aws:inspector:ap-northeast-1:438731021216:rulespackage/0-bBUQnxMq \
        arn:aws:inspector:ap-northeast-1:438731021216:rulespackage/0-knGBhqEu

# 定期評価実行
aws inspector start-assessment-run \
    --assessment-template-arn arn:aws:inspector:region:account:assessmenttemplate/template-id
```

### GuardDuty（脅威検知）

```bash
# GuardDuty有効化
aws guardduty create-detector --enable

# 脅威インテリジェンスセット追加
aws guardduty create-threat-intel-set \
    --detector-id detector-id \
    --name "Custom-Threat-List" \
    --format TXT \
    --location s3://my-threat-intel-bucket/threats.txt \
    --activate
```

### VPC Flow Logs

```bash
# VPC Flow Logs設定
aws ec2 create-flow-logs \
    --resource-type VPC \
    --resource-ids vpc-xxxxxxxxx \
    --traffic-type ALL \
    --log-destination-type cloud-watch-logs \
    --log-group-name VPC-FlowLogs \
    --deliver-logs-permission-arn arn:aws:iam::account:role/flowlogsRole

# 異常な通信パターン検知クエリ例
aws logs start-query \
    --log-group-name "VPC-FlowLogs" \
    --start-time 1577836800 \
    --end-time 1577923200 \
    --query-string '
        fields @timestamp, srcaddr, dstaddr, srcport, dstport, protocol, action
        | filter action = "REJECT"
        | stats count() by srcaddr
        | sort count() desc
        | limit 10'
```

## 侵入検知・防止

### OSSEC HIDS設定

```bash
# OSSEC インストール（CentOS/Amazon Linux）
wget https://github.com/ossec/ossec-hids/archive/3.7.0.tar.gz
tar -xzf 3.7.0.tar.gz
cd ossec-hids-3.7.0/
sudo ./install.sh

# 基本設定（/var/ossec/etc/ossec.conf）
<ossec_config>
  <global>
    <email_notification>yes</email_notification>
    <email_to>security@company.com</email_to>
    <smtp_server>localhost</smtp_server>
    <email_from>ossec@ec2.internal</email_from>
  </global>

  <rules>
    <include>rules_config.xml</include>
    <include>pam_rules.xml</include>
    <include>sshd_rules.xml</include>
    <include>telnetd_rules.xml</include>
    <include>syslog_rules.xml</include>
    <include>arpwatch_rules.xml</include>
    <include>symantec-av_rules.xml</include>
    <include>symantec-ws_rules.xml</include>
    <include>pix_rules.xml</include>
    <include>named_rules.xml</include>
    <include>smbd_rules.xml</include>
    <include>vsftpd_rules.xml</include>
    <include>pure-ftpd_rules.xml</include>
    <include>proftpd_rules.xml</include>
    <include>ms_ftpd_rules.xml</include>
    <include>ftpd_rules.xml</include>
    <include>hordeimp_rules.xml</include>
    <include>roundcube_rules.xml</include>
    <include>wordpress_rules.xml</include>
    <include>cimserver_rules.xml</include>
    <include>vpopmail_rules.xml</include>
    <include>vmpop3d_rules.xml</include>
    <include>courier_rules.xml</include>
    <include>web_rules.xml</include>
    <include>web_appsec_rules.xml</include>
    <include>apache_rules.xml</include>
    <include>nginx_rules.xml</include>
    <include>php_rules.xml</include>
    <include>mysql_rules.xml</include>
    <include>postgresql_rules.xml</include>
    <include>ids_rules.xml</include>
    <include>squid_rules.xml</include>
    <include>firewall_rules.xml</include>
    <include>cisco-ios_rules.xml</include>
    <include>netscreenfw_rules.xml</include>
    <include>sonicwall_rules.xml</include>
    <include>postfix_rules.xml</include>
    <include>sendmail_rules.xml</include>
    <include>imapd_rules.xml</include>
    <include>mailscanner_rules.xml</include>
    <include>dovecot_rules.xml</include>
    <include>ms-exchange_rules.xml</include>
    <include>racoon_rules.xml</include>
    <include>vpn_concentrator_rules.xml</include>
    <include>spamd_rules.xml</include>
    <include>msauth_rules.xml</include>
    <include>mcafee_av_rules.xml</include>
    <include>trend-osce_rules.xml</include>
    <include>ms-se_rules.xml</include>
    <include>zeus_rules.xml</include>
    <include>solaris_bsm_rules.xml</include>
    <include>vmware_rules.xml</include>
    <include>ms_dhcp_rules.xml</include>
    <include>asterisk_rules.xml</include>
    <include>ossec_rules.xml</include>
    <include>attack_rules.xml</include>
    <include>local_rules.xml</include>
  </rules>

  <syscheck>
    <!-- File integrity monitoring -->
    <directories check_all="yes">/etc,/usr/bin,/usr/sbin</directories>
    <directories check_all="yes">/bin,/sbin,/boot</directories>
    
    <ignore>/etc/mtab</ignore>
    <ignore>/etc/hosts.deny</ignore>
    <ignore>/etc/mail/statistics</ignore>
    <ignore>/etc/random-seed</ignore>
    <ignore>/etc/adjtime</ignore>
    <ignore>/etc/httpd/logs</ignore>
    <ignore>/etc/utmpx</ignore>
    <ignore>/etc/wtmpx</ignore>
    <ignore>/etc/cups/certs</ignore>
    <ignore>/etc/dumpdates</ignore>
    <ignore>/etc/svc/volatile</ignore>
  </syscheck>

  <rootcheck>
    <rootkit_files>/var/ossec/etc/shared/rootkit_files.txt</rootkit_files>
    <rootkit_trojans>/var/ossec/etc/shared/rootkit_trojans.txt</rootkit_trojans>
  </rootcheck>

  <localfile>
    <log_format>syslog</log_format>
    <location>/var/log/messages</location>
  </localfile>

  <localfile>
    <log_format>syslog</log_format>
    <location>/var/log/secure</location>
  </localfile>
</ossec_config>
```

## セキュリティチェックリスト

### 初期設定チェックリスト

- [ ] デフォルトアカウントの無効化または削除
- [ ] SSH鍵認証の設定（パスワード認証無効化）
- [ ] セキュリティグループの最小権限設定
- [ ] IAMロールの適用（アクセスキー不使用）
- [ ] システムパッチの最新化
- [ ] 不要サービスの停止・削除
- [ ] ファイアウォール設定
- [ ] ログ設定と転送
- [ ] 監視・アラート設定
- [ ] バックアップ設定

### 運用時チェックリスト

- [ ] 定期的なパッチ適用
- [ ] セキュリティグループルールの見直し
- [ ] アクセスログの定期確認
- [ ] 不審な活動の監視
- [ ] バックアップの定期テスト
- [ ] 脆弱性スキャンの実施
- [ ] インシデント対応手順の確認

!!! tip "セキュリティの要点"
    1. **多層防御**: 単一の防御策に依存せず複数のセキュリティ層を設定
    2. **最小権限**: 必要最小限のアクセス権限のみ付与
    3. **継続監視**: 異常の早期検知と迅速な対応
    4. **定期見直し**: セキュリティ設定の定期的な確認と更新
    5. **教育と訓練**: チームメンバーのセキュリティ意識向上

## 次のステップ

セキュリティを強化したら：

1. **コンプライアンス対応** - SOC2、ISO27001などの要件確認
2. **ディザスタリカバリ** - 災害時の復旧手順策定
3. **セキュリティ自動化** - Infrastructure as Codeによる設定管理
4. **ペネトレーションテスト** - 第三者による脆弱性評価の実施