# Claude Code と他ツール・MCP 連携

Claude Code は単体でも強力だが、他の CLI ツールや MCP サーバーと組み合わせることでさらに活用の幅が広がる。

## CLI ツールの活用

### gh CLI（GitHub CLI）

Claude Code と最も相性が良い CLI ツール。認証済みの gh があると、Claude が GitHub API を直接叩ける。

```bash
# インストール
brew install gh  # macOS
sudo apt install gh  # Ubuntu

# 認証
gh auth login
```

Claude Code での活用例:

```
「gh を使って Issue #42 の内容を確認して、修正を実装して PR を出して」
「最近の PR のレビューコメントを確認して対応して」
「gh api を使ってリポジトリの統計を取得して」
```

### その他の推奨 CLI ツール

| ツール | 用途 | Claude Code との連携 |
|--------|------|---------------------|
| `gh` | GitHub 操作 | Issue, PR, API 全般 |
| `jq` | JSON 処理 | API レスポンスの加工 |
| `rg` (ripgrep) | 高速検索 | コードベース探索 |
| `fd` | ファイル検索 | ファイル発見 |
| `delta` | diff 表示 | 変更確認 |
| `bat` | ファイル表示 | シンタックスハイライト付き確認 |

!!! tip "未知のツールも学習できる"
    Claude Code は `--help` を読んで未知のツールを学習できる:
    ```
    「foo-cli --help を読んで使い方を学んで、それを使って X を実行して」
    ```

## 他の AI コーディングツールとの使い分け

### 各ツールの特性

| ツール | 強み | 弱み | 最適な用途 |
|--------|------|------|-----------|
| **Claude Code** | ターミナル統合、自律実行、コンテキスト管理 | GUI なし | 実装・デバッグ・リファクタリング |
| **Cursor** | IDE 統合、インラインDiff、タブ補完 | 重い、高コスト | 日常的なコーディング |
| **GitHub Copilot** | 無料枠あり、VS Code 統合 | 大規模タスク弱い | コード補完、小さな修正 |
| **Aider** | git 統合、マルチファイル編集 | 学習コスト | git ベースのワークフロー |
| **Windsurf** | IDE 統合、Flow モード | 新しい | 探索的コーディング |

### 併用パターン

#### Claude Code + Cursor

```
1. Cursor: 日常的なコーディング、タブ補完
2. Claude Code: 大きなリファクタリング、バグ調査、テスト作成
3. 使い分け基準: 5分以内の作業 → Cursor、それ以上 → Claude Code
```

#### Claude Code + GitHub Copilot

```
1. Copilot: インライン補完（自動）
2. Claude Code: 複雑な実装タスク（手動起動）
3. 共存可能: Copilot は補完、Claude Code はタスク実行で役割が異なる
```

## パイプラインでの活用

### stdin/stdout 連携

Claude Code は Unix パイプラインに組み込める:

```bash
# ファイル内容を Claude に渡す
cat error.log | claude "このエラーログを分析して原因と対策を教えて"

# コマンド出力を渡す
git diff HEAD~5 | claude "この差分をレビューして問題点を指摘して"

# 複数ファイルのコンテキスト
cat src/auth/*.ts | claude "この認証モジュールのセキュリティレビューをして"
```

### 非対話モードでのスクリプト利用

```bash
# -p フラグで非対話実行
claude -p "package.json を読んで、未使用の依存関係を特定して"

# 出力をファイルに保存
claude -p "README.md を日本語に翻訳して" > README.ja.md

# CI/CD での利用
claude -p "このリポジトリの CHANGELOG.md を最新のコミットから生成して" \
  --allowedTools "Read,Bash,Write"
```

### シェルスクリプトとの統合

```bash
#!/bin/bash
# review.sh — 変更ファイルを自動レビュー

CHANGED_FILES=$(git diff --name-only HEAD~1)

for file in $CHANGED_FILES; do
  echo "=== Reviewing: $file ==="
  claude -p "以下のファイルをレビューして問題点があれば指摘して: @$file"
  echo ""
done
```

## MCP サーバーの活用

### MCP とは

**Model Context Protocol** — AI ツールと外部サービスを接続するオープンスタンダード。Claude Code に Issue トラッカー、DB、モニタリング等のツールを追加できる。

### MCP サーバーの追加方法

#### リモート HTTP サーバー（推奨）

```bash
# 基本構文
claude mcp add --transport http <名前> <URL>

# Notion 連携
claude mcp add --transport http notion https://mcp.notion.com/mcp

# 認証付き
claude mcp add --transport http secure-api https://api.example.com/mcp \
  --header "Authorization: Bearer YOUR_TOKEN"
```

#### リモート SSE サーバー

```bash
# Asana 連携
claude mcp add --transport sse asana https://mcp.asana.com/sse

# API キー付き
claude mcp add --transport sse private-api https://api.company.com/sse \
  --header "X-API-Key: YOUR_KEY"
```

#### ローカル stdio サーバー

```bash
# Airtable
claude mcp add --transport stdio --env AIRTABLE_API_KEY=YOUR_KEY airtable \
  -- npx -y airtable-mcp-server

# ファイルシステム
claude mcp add --transport stdio filesystem \
  -- npx -y @anthropic-ai/mcp-filesystem-server /path/to/dir

# PostgreSQL
claude mcp add --transport stdio --env DATABASE_URL=postgres://... postgres \
  -- npx -y @anthropic-ai/mcp-postgres-server
```

### MCP サーバーの管理

```bash
# 一覧表示
claude mcp list

# 詳細確認
claude mcp get github

# 削除
claude mcp remove github

# Claude Code 内でステータス確認
/mcp
```

### スコープの使い分け

| スコープ | 保存先 | 用途 |
|----------|--------|------|
| `local`（デフォルト） | `~/.claude.json`（プロジェクトパス下） | 個人的な開発用 |
| `project` | `.mcp.json`（プロジェクトルート） | チーム共有（git にコミット） |
| `user` | `~/.claude.json` | 全プロジェクト共通の個人用 |

```bash
# プロジェクトスコープで追加（チーム共有向け）
claude mcp add --transport http paypal --scope project https://mcp.paypal.com/mcp

# ユーザースコープで追加（全プロジェクトで利用）
claude mcp add --transport http hubspot --scope user https://mcp.hubspot.com/anthropic
```

### 人気の MCP サーバー

| サーバー | 用途 |
|----------|------|
| GitHub | Issue, PR, コードレビュー |
| Notion | ドキュメント管理 |
| Slack | メッセージ、通知 |
| Sentry | エラーモニタリング |
| PostgreSQL | データベースクエリ |
| Figma | デザイン統合 |
| Linear | プロジェクト管理 |
| Stripe | 決済データ |

### プロジェクトでの .mcp.json 共有

チームで MCP 設定を共有するには `.mcp.json` を git にコミット:

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-github-server"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-postgres-server"],
      "env": {
        "DATABASE_URL": "${DATABASE_URL}"
      }
    }
  }
}
```

!!! warning "セキュリティ"
    `.mcp.json` にはシークレットを直接書かない。環境変数 `${VAR_NAME}` を使う。Claude Code は初回使用時にプロジェクトスコープのサーバーの承認を求める。
