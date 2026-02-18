# 🛡️ DnsShield — DNS/WHOIS変更監視＆ドメインハイジャック検知

## 概要

自分が管理するドメインのDNSレコード（A, CNAME, MX, NS, TXT）とWHOIS情報を定期監視し、意図しない変更を検知したら即座に通知するサービス。ドメインハイジャック・DNS設定ミス・ネームサーバー変更・ドメイン期限切れを早期発見。「知らないうちにDNSが書き換えられてフィッシングサイトに誘導されてた」を防ぐ。

## ターゲット

- **メイン**: Web制作会社・フリーランス（クライアントのドメインを管理）
- **サブ**: SaaS運営者（自社ドメインのセキュリティ）
- **ペルソナ**: 中村さん（40歳）Web制作会社経営。15社のドメインを管理。先月、クライアントがレジストラの更新メールを無視してドメイン失効。別の誰かに取得され、クライアントのブランドサイトが3週間ダウン。損害賠償の話に発展しかけた。

## 料金

| プラン | 月額 | 内容 |
|--------|------|------|
| Free | $0 | 3ドメイン、日次チェック、メール通知 |
| Pro | $5/月 | 30ドメイン、1時間ごとチェック、Slack/LINE通知、WHOIS期限アラート |
| Agency | $12/月 | 100ドメイン、15分ごとチェック、チーム通知、月次レポート |

## ユーザーフロー

1. サインアップ（メール or GitHub OAuth）
2. 監視したいドメインを登録（example.com等）
3. 初回スキャン: 現在のDNSレコード＆WHOIS情報をベースラインとして保存
4. 通知先設定（Slack / メール / LINE）
5. 定期スキャン開始
6. 変更検知時:
   - DNSレコード変更 → 「example.comのAレコードが 1.2.3.4 → 5.6.7.8 に変更されました」
   - NS変更 → 🚨 緊急アラート「ネームサーバーが変更されました（ハイジャックの可能性）」
   - WHOIS期限30日前 → ⚠️ 「ドメイン期限切れまで30日です」
7. ダッシュボードで全ドメインの状態一覧

## アーキテクチャ

```
[EventBridge (1時間ごと)] → [Lambda (DNSチェッカー)]
                                    ↓
                        [DNS Resolve (A/CNAME/MX/NS/TXT)]
                        [WHOIS Lookup]
                                    ↓
                        [ベースラインと比較]
                                    ↓ (変更あり)
                        [Supabase (変更履歴保存)]
                                    ↓
                        [SNS → Slack/Email/LINE通知]

[ユーザー] → [Next.js on Vercel] → [Supabase]
```

## DB設計

```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email TEXT NOT NULL UNIQUE,
  plan TEXT DEFAULT 'free',
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE domains (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  domain TEXT NOT NULL,
  whois_expiry DATE,
  whois_registrar TEXT,
  whois_nameservers TEXT[],
  last_checked_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- DNSレコードベースライン
CREATE TABLE dns_baselines (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  domain_id UUID REFERENCES domains(id),
  record_type TEXT NOT NULL, -- A, AAAA, CNAME, MX, NS, TXT
  record_value TEXT NOT NULL,
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- 変更履歴
CREATE TABLE dns_changes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  domain_id UUID REFERENCES domains(id),
  record_type TEXT,
  old_value TEXT,
  new_value TEXT,
  change_type TEXT, -- added, removed, modified, whois_expiry, ns_change
  severity TEXT, -- info, warning, critical
  detected_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE notification_channels (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  type TEXT NOT NULL,
  config JSONB NOT NULL,
  enabled BOOLEAN DEFAULT true
);

CREATE TABLE alerts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  domain_id UUID REFERENCES domains(id),
  change_id UUID REFERENCES dns_changes(id),
  channel_id UUID REFERENCES notification_channels(id),
  sent_at TIMESTAMPTZ DEFAULT NOW()
);
```

## コスト見積もり

| 項目 | 月額 |
|------|------|
| Vercel (Hobby) | $0 |
| Supabase (Free) | $0 |
| AWS Lambda (DNS/WHOISチェック) | $0.20 (100ドメイン×24回/日) |
| WHOIS API（whoisxml等） | $0（月500クエリ無料） or $2（有料） |
| **合計** | **$0.20〜$2.20/月** (100ドメイン時) |

## MVPスコープ（2週間）

### Week 1
- 認証（Supabase Auth）
- ドメイン登録UI
- Lambda: DNS解決（A/CNAME/MX/NS/TXT）＆ベースライン保存
- 差分検知ロジック

### Week 2
- WHOIS期限チェック
- Slack/メール通知（重要度別）
- ダッシュボード（ドメイン一覧、変更履歴）
- LP + Stripe決済

## マーケ計画

1. **SEO**: 「ドメイン ハイジャック 対策」「DNS 変更 通知」「ドメイン 期限切れ 防止」
2. **Twitter/X**: ドメインハイジャック事件のニュースに引用リプ
3. **Web制作コミュニティ**: 「クライアントのドメイン管理、ちゃんと監視してますか？」
4. **セキュリティ系記事**: 「DNSハイジャックの実態と対策」→ DnsShield紹介
5. **Product Hunt**: セキュリティツールとしてローンチ

## 技術スタック

- **フロントエンド**: Next.js 14 + Tailwind CSS + shadcn/ui
- **バックエンド**: AWS Lambda (Node.js) — dns.resolve + whois
- **DB**: Supabase (PostgreSQL)
- **認証**: Supabase Auth
- **決済**: Stripe
- **DNS解決**: Node.js標準 `dns` モジュール
- **WHOIS**: whois-json npm or WHOIS XML API

## リスク

| リスク | 影響 | 対策 |
|--------|------|------|
| WHOISのRate Limit | 中 | WHOIS XML API（有料）でフォールバック。日次チェックで十分 |
| 偽陽性（CDNのIPローテーション等） | 中 | CDN検知してAレコード変更を「info」に格下げ。NSとWHOIS変更は常にcritical |
| 既存のドメイン監視ツール | 中 | ほとんどがエンタープライズ価格。$5で30ドメインは差別化 |
| DNSキャッシュによる検知遅延 | 低 | 複数リゾルバ（8.8.8.8, 1.1.1.1）で並列チェック |

## 競合分析

| サービス | 月額 | 特徴 | DnsShieldの優位性 |
|----------|------|------|-------------------|
| DNSimple | $5〜 | DNS管理（監視は付録） | 監視特化、WHOIS期限アラート |
| Uptime Robot | $7〜 | 死活監視（DNS変更は非対応） | DNS/WHOIS特化 |
| SecurityTrails | $50〜 | エンタープライズDNS情報 | 個人/小規模向け価格 |
| 手動whoisチェック | $0 | 忘れる | 自動、見落としゼロ |

## $20達成シナリオ

```
Month 1: 無料ユーザー40人（Web制作コミュニティ）
Month 2: Pro 2人 = $10/月
Month 3: Pro 3人 + Agency 1人 = $27/月 ✅
```

**必要有料ユーザー数: Pro 4人で$20達成**

## ユニットエコノミクス

| 指標 | 値 |
|------|-----|
| CAC | $0 |
| ARPU | $5.00 |
| 粗利率 | 97.8%（$5.00 - $0.022/ユーザー） |
| LTV（12ヶ月） | $60 |
| LTV/CAC | ∞ |
| チャーン率（予測） | 2%/月（ドメイン管理は継続必須→極低チャーン） |
| Payback Period | 即時 |
