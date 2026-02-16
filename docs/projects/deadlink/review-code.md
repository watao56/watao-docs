# DeadLink コードレビュー

**レビュー日**: 2026-02-16  
**対象**: `deadlink/frontend/src/` 全ファイル  
**レビュアー**: シニアエンジニア視点

---

## 総合評価: C+（動くが、本番運用には要改善）

MVP として最低限動く状態だが、セキュリティ・パフォーマンス・エラーハンドリングに複数の問題がある。

---

## 1. コード品質

### 良い点
- **ファイル構造が明快**: App Router のルール通りに整理されている
- **型定義がある**: TypeScript の interface が適切に定義されている
- **コンポーネント分離**: Header, ScanForm が適切に分離
- **クローラーの設計が良い**: polite crawl（300ms間隔）、User-Agent設定、robots.txt意識

### 問題点
- **lib/dynamodb.ts**: 型が `Record<string, unknown>` で型安全でない
- **コンポーネントが大きい**: `scan/[id]/page.tsx` が200行超。結果表示部分をコンポーネントに分離すべき
- **マジックナンバー**: `20`, `100`, `300`, `10000` 等がハードコーディング。定数化すべき
- **コメントがほぼゼロ**: ビジネスロジック（特にクローラー）にはコメントが必要

---

## 2. セキュリティ

### 🔴 Critical

#### 2.1 APIレート制限がメモリベースで無効
```typescript
// api/scan/instant/route.ts
const rateLimitMap = new Map<string, { count: number; resetAt: number }>();
```
**問題**: Vercelはリクエストごとにコールドスタートする可能性があり、Mapはリクエスト間で共有されない。サーバーレス環境ではインメモリのレート制限は**事実上無効**。

**修正**: Upstash Redis（無料枠あり）でレート制限を実装する。
```typescript
import { Ratelimit } from "@upstash/ratelimit";
import { Redis } from "@upstash/redis";

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(3, "1 d"),
});
```

#### 2.2 サイトスキャンAPIに認可チェックの不備
```typescript
// api/sites/[id]/reports/route.ts
export async function GET(request, { params }) {
  const { id: siteId } = await params;
  const reports = await getReportsBySiteId(siteId);
  return NextResponse.json({ reports });
}
```
**問題**: 認証チェックが**一切ない**。UUIDを推測されれば他人のレポートが閲覧可能。

**修正**: 全ての`/api/sites/`エンドポイントでユーザー認証 + 所有権チェックを追加。

#### 2.3 SSRF（Server-Side Request Forgery）リスク
```typescript
// lib/crawler.ts
const res = await fetch(pageUrl, { ... });
```
**問題**: ユーザーが `http://169.254.169.254/latest/meta-data/` 等を入力するとAWSメタデータにアクセスされる。内部ネットワークへのアクセスも可能。

**修正**:
```typescript
function isAllowedUrl(url: string): boolean {
  const parsed = new URL(url);
  if (parsed.hostname === "localhost") return false;
  if (parsed.hostname.startsWith("127.")) return false;
  if (parsed.hostname.startsWith("169.254.")) return false;
  if (parsed.hostname.startsWith("10.")) return false;
  if (parsed.hostname.startsWith("192.168.")) return false;
  if (parsed.hostname.startsWith("172.")) {
    const second = parseInt(parsed.hostname.split(".")[1]);
    if (second >= 16 && second <= 31) return false;
  }
  if (parsed.protocol !== "http:" && parsed.protocol !== "https:") return false;
  return true;
}
```

### 🟠 High

#### 2.4 Webhook URLの検証なし
```typescript
// settings/page.tsx
await supabase.from("profiles").upsert({
  discord_webhook: discordWebhook || null,
  slack_webhook: slackWebhook || null,
});
```
**問題**: Discord/Slack以外のURLを保存できる。悪用の起点になりうる（通知発火時に任意URLにPOSTされる）。

**修正**: `discord.com/api/webhooks/` と `hooks.slack.com/services/` のプレフィックスチェック。

#### 2.5 Supabase RLS（Row Level Security）依存の不明確さ
ダッシュボードでは `supabase.from("sites").select("*")` を直接呼んでいるが、RLSが正しく設定されているか不明。RLS未設定なら全ユーザーのサイトが閲覧可能。

**確認・修正**: Supabaseダッシュボードで `sites` と `profiles` テーブルのRLSポリシーを確認し、ドキュメントに記載。

---

## 3. パフォーマンス

### 🟠 High

#### 3.1 Supabaseクライアントの毎回再生成
```typescript
// dashboard/page.tsx
const supabase = createClient(); // レンダリングごとに新規作成
```
**問題**: `createClient()` がレンダリングごとに呼ばれる。`useMemo` を使うか、Contextで共有すべき。

**修正**: settings/page.tsx では `useMemo` を使っている。統一してProviderパターンに。
```typescript
// lib/supabase-provider.tsx
const SupabaseContext = createContext(createClient());
export function SupabaseProvider({ children }) { ... }
```

#### 3.2 DynamoDBへの逐次書き込み
```typescript
for (const result of results) {
  await putResult({ ...result, report_id: reportId });
}
```
**問題**: 500リンクなら500回の逐次API呼び出し。BatchWriteを使えば1/20に削減。

**修正**: `BatchWriteCommand` で25件ずつバッチ書き込み。

#### 3.3 Headerコンポーネントが毎回API呼び出し
```typescript
useEffect(() => {
  supabase.auth.getUser().then(({ data }) => setUser(data.user));
}, [supabase]);
```
**問題**: ページ遷移のたびに `getUser()` を呼ぶ。`onAuthStateChange` リスナーを使うべき。

### 🟡 Medium

#### 3.4 クローラーでHEAD非対応サイトへのフォールバックなし
一部サイトはHEADリクエストを拒否する（405 Method Not Allowed）。GETへのフォールバックが必要。

---

## 4. エラーハンドリング

### 🟠 High

#### 4.1 空のcatchブロック
```typescript
// dashboard/sites/[id]/page.tsx
} catch {
  // ignore
}
```
**問題**: 7箇所で`catch`がエラーを握り潰している。ユーザーにフィードバックがない。

**修正**: 最低限 `console.error` + ユーザーへのエラーメッセージ表示。

#### 4.2 サイトスキャンのエラーが飲み込まれる
```typescript
// api/sites/[id]/scan/route.ts
crawlSite(site.url, site.page_limit || 100).then(async ({ results }) => {
  ...
});
// .catch がない！
```
**問題**: `crawlSite`がエラーを投げた場合、レポートが`running`のまま永久に残る。instant scanでは`after()`内でcatchしているが、site scanでは未処理。

**修正**: `.catch()` を追加してレポートを`failed`に更新。

---

## 5. テスト

### 現状: テストゼロ 🔴

テストファイルが一切存在しない。

### 推奨テスト（優先度順）

1. **クローラーのユニットテスト**: URL正規化、リンク抽出、ステータス判定
2. **APIルートの統合テスト**: レート制限、認証チェック、エラーケース
3. **E2Eテスト**: 即スキャンフローの一気通貫テスト（Playwright）
4. **SSRFフィルターのテスト**: 内部IPアドレスのブロック確認

---

## 6. スケーラビリティ

### 現在の限界
- **Vercel Hobby**: 関数実行10秒制限 → 即スキャンの `maxDuration = 60` は効かない
- **同時スキャン**: 制御がない。100人が同時スキャンしたらVercelの同時実行数制限に引っかかる
- **DynamoDB書き込み**: 逐次処理がボトルネック

### 改善案
- 長時間クロールはバックグラウンドWorker（AWS Lambda or Vercel Cron）に分離
- SQSキューイングは設計書にあるが未実装
- WebSocketかSSEでリアルタイム進捗通知

---

## 7. バグ・潜在的問題

### 🔴 Critical

#### 7.1 `after()` の挙動依存
```typescript
// api/scan/instant/route.ts
import { after } from "next/server";
after(async () => { ... });
```
`after()` はNext.js 15の実験的機能。Vercel以外のデプロイ先では動作しない可能性がある。また、Hobby planでの実行時間制限を超える可能性が高い。

#### 7.2 `loadSites` の無限再生成
```typescript
// dashboard/page.tsx
const loadSites = useCallback(async () => {
  ...
}, [supabase]); // supabaseが毎回新規作成されるのでloadSitesも毎回再生成
```
**問題**: `supabase` が `createClient()` で毎回新規オブジェクトになるため、`useCallback` の依存配列が常に変化 → `useEffect` が無限ループする可能性。

### 🟠 High

#### 7.3 認証チェックの競合状態
```typescript
useEffect(() => {
  supabase.auth.getUser().then(({ data }) => {
    if (!data.user) { router.push("/auth/login"); return; }
    loadSites();
  });
}, [supabase.auth, router, loadSites]);
```
`supabase.auth` を依存配列に入れているが、これはオブジェクト参照。不安定な動作の原因になる。

#### 7.4 レート制限のIPヘッダー偽装
```typescript
const ip = request.headers.get("x-forwarded-for") || "unknown";
```
`x-forwarded-for` はクライアントが任意に設定可能。Vercelでは `request.ip` を使うべき（Vercelが保証するIP）。

---

## 8. ベストプラクティスからの逸脱

| 項目 | 現状 | 推奨 |
|---|---|---|
| 環境変数の型安全性 | `!` で非null assertion | `zod` でバリデーション + 起動時チェック |
| APIレスポンス型 | 型なし（`any`相当） | 共通のレスポンス型定義 |
| エラーバウンダリ | なし | `error.tsx` でグローバルエラーハンドリング |
| Loading UI | 各ページで個別実装 | `loading.tsx` で統一 |
| Middleware | なし | 認証が必要なルートを middleware.ts で一括保護 |
| SEO | 最低限のmetadata | 各ページごとのmetadata + OGP設定 |
| アクセシビリティ | 考慮なし | aria-label、キーボード操作対応 |

---

## 9. 改善提案まとめ

### 🔴 Critical（本番運用前に必須）
1. **SSRFフィルター実装** — 内部IP・メタデータエンドポイントへのアクセスをブロック
2. **`/api/sites/[id]/reports` に認証追加** — 他人のデータが丸見え
3. **レート制限をRedisベースに変更** — メモリベースはサーバーレスで無効
4. **`after()` の代替実装検討** — Vercel Hobby制限を考慮

### 🟠 High（1-2週間以内）
5. **空catchブロックの修正** — エラーログ + ユーザー通知
6. **site scanの `.catch()` 追加** — runningのまま放置されるバグ修正
7. **Supabase Provider パターン導入** — クライアント再生成の無限ループ防止
8. **DynamoDB BatchWrite** — 書き込みパフォーマンス改善
9. **middleware.ts で認証ルート保護** — 各ページでの個別チェックを廃止
10. **Webhook URLのバリデーション** — Discord/Slack以外を拒否

### 🟡 Medium（1ヶ月以内）
11. **テスト追加**（クローラー単体テスト優先）
12. **環境変数のバリデーション**（zod）
13. **error.tsx / loading.tsx の追加**
14. **HEAD非対応サイトへのGETフォールバック**
15. **コンポーネント分離**（scan結果ページ）
16. **定数の一元管理**

### 🟢 Low（将来）
17. **アクセシビリティ対応**
18. **各ページのSEO最適化**
19. **WebSocket/SSEでリアルタイム進捗**
20. **SQSキューイング実装**（設計書に記載あるが未実装）

---

## ファイル別サマリー

| ファイル | 評価 | 主な問題 |
|---|---|---|
| `lib/crawler.ts` | B+ | SSRFリスク以外は良い設計 |
| `lib/dynamodb.ts` | B | 型安全でない、BatchWrite未使用 |
| `lib/supabase.ts` | A | シンプルで正しい |
| `lib/supabase-server.ts` | A | 適切な実装 |
| `api/scan/instant/route.ts` | C | レート制限無効、SSRF |
| `api/scan/[id]/route.ts` | B | 問題少ない |
| `api/sites/[id]/reports/route.ts` | D | **認証なし** |
| `api/sites/[id]/scan/route.ts` | C+ | catch不足 |
| `app/page.tsx` | A- | LP as code、シンプル |
| `app/dashboard/page.tsx` | C | 無限ループリスク、認証不安定 |
| `app/scan/[id]/page.tsx` | B | ポーリング実装は良い、コンポーネント大きい |
| `app/settings/page.tsx` | B- | Webhook検証なし |
| `app/auth/login/page.tsx` | B+ | 2パス認証、シンプル |
| `components/Header.tsx` | B | 毎回getUser() |
| `components/ScanForm.tsx` | A- | 小さく、正しい |

---

## まとめ

MVPとして素早く作った成果物としては悪くない。しかし、**セキュリティ面の問題（SSRF、認証欠如、レート制限無効）は本番公開状態では危険**。Criticalの4項目は即座に対応すべき。

コードの構造自体は良く、改善のベースが整っている。型安全性とエラーハンドリングを強化すれば、保守可能な品質に到達する。
