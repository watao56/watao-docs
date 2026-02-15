# セットアップ手順

## 前提条件

- Node.js 20+
- Supabase プロジェクト（無料プラン）
- Vercel アカウント（デプロイ時）

## 1. Supabase テーブル作成

Supabase ダッシュボードの SQL Editor で以下を実行:

```sql
-- Profiles table
CREATE TABLE IF NOT EXISTS profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  plan TEXT DEFAULT 'free',
  stripe_customer_id TEXT,
  notification_email BOOLEAN DEFAULT true,
  discord_webhook TEXT,
  slack_webhook TEXT,
  created_at TIMESTAMPTZ DEFAULT now()
);

-- Sites table
CREATE TABLE IF NOT EXISTS sites (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  url TEXT NOT NULL,
  name TEXT,
  page_limit INT DEFAULT 100,
  check_interval TEXT DEFAULT 'weekly',
  last_checked_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT now()
);

-- Scan reports
CREATE TABLE IF NOT EXISTS scan_reports (
  id BIGSERIAL,
  site_id TEXT NOT NULL,
  scanned_at TIMESTAMPTZ NOT NULL,
  report_id TEXT UNIQUE NOT NULL,
  url TEXT,
  total_links INT DEFAULT 0,
  broken_links INT DEFAULT 0,
  redirected_links INT DEFAULT 0,
  status TEXT DEFAULT 'running',
  duration_ms INT DEFAULT 0,
  pages_scanned INT DEFAULT 0,
  PRIMARY KEY (site_id, scanned_at)
);

-- Scan results
CREATE TABLE IF NOT EXISTS scan_results (
  id BIGSERIAL,
  report_id TEXT NOT NULL,
  link_url TEXT NOT NULL,
  source_page TEXT,
  status_code INT,
  status TEXT,
  redirect_to TEXT,
  anchor_text TEXT,
  checked_at TIMESTAMPTZ,
  PRIMARY KEY (report_id, link_url)
);

-- RLS
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE sites ENABLE ROW LEVEL SECURITY;
ALTER TABLE scan_reports ENABLE ROW LEVEL SECURITY;
ALTER TABLE scan_results ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can view own profile" ON profiles FOR SELECT USING (auth.uid() = id);
CREATE POLICY "Users can update own profile" ON profiles FOR UPDATE USING (auth.uid() = id);
CREATE POLICY "Users can insert own profile" ON profiles FOR INSERT WITH CHECK (auth.uid() = id);

CREATE POLICY "Users can view own sites" ON sites FOR SELECT USING (auth.uid() = user_id);
CREATE POLICY "Users can insert own sites" ON sites FOR INSERT WITH CHECK (auth.uid() = user_id);
CREATE POLICY "Users can update own sites" ON sites FOR UPDATE USING (auth.uid() = user_id);
CREATE POLICY "Users can delete own sites" ON sites FOR DELETE USING (auth.uid() = user_id);

CREATE POLICY "Anyone can read scan reports" ON scan_reports FOR SELECT USING (true);
CREATE POLICY "Service role can manage scan reports" ON scan_reports FOR ALL USING (true);
CREATE POLICY "Anyone can read scan results" ON scan_results FOR SELECT USING (true);
CREATE POLICY "Service role can manage scan results" ON scan_results FOR ALL USING (true);

-- Auto-create profile on signup
CREATE OR REPLACE FUNCTION public.handle_new_user()
RETURNS trigger AS $$
BEGIN
  INSERT INTO public.profiles (id) VALUES (new.id) ON CONFLICT (id) DO NOTHING;
  RETURN new;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

DROP TRIGGER IF EXISTS on_auth_user_created ON auth.users;
CREATE TRIGGER on_auth_user_created
  AFTER INSERT ON auth.users
  FOR EACH ROW EXECUTE FUNCTION public.handle_new_user();
```

## 2. Supabase Auth 設定

1. Supabase ダッシュボード → Authentication → Providers
2. GitHub OAuth を有効化（GitHub Developer Settings で OAuth App 作成）
3. Email (Magic Link) を有効化

## 3. フロントエンド

```bash
cd /home/node/clawd/deadlink/frontend

# 環境変数を設定
cp .env.local.example .env.local
# .env.local を編集

# 開発サーバー起動
npm run dev
```

### 環境変数 (.env.local)

```
NEXT_PUBLIC_SUPABASE_URL=your-supabase-url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key
```

## 4. Vercel デプロイ

```bash
# Vercel CLI でデプロイ
npx vercel --prod
```

環境変数を Vercel ダッシュボードで設定すること。

## 5. AWS リソース（将来）

IAM ポリシーに以下の権限が必要:

- `dynamodb:*` (deadlink_* テーブル)
- `sqs:*` (deadlink-* キュー)
- `lambda:*` (deadlink-* 関数)
- `events:*` (deadlink-* ルール)
- `ses:*` (メール送信)

```bash
chmod +x scripts/aws-setup.sh
./scripts/aws-setup.sh
```
