# CLI・MCP 連携ガイド

Claude Code は単体でも強力だが、CLI ツールや MCP サーバーと組み合わせることでさらに活用の幅が広がる。

## CLI ツールの活用

### gh CLI（GitHub CLI）が最重要

Claude Code と最も相性が良い CLI ツール。認証済みの `gh` があれば、Claude が GitHub API を直接叩ける。

> CLI ツールは外部サービスとのやりとりにおいて**最もコンテキスト効率の良い方法**。GitHub を使うなら gh CLI をインストールすること。
>
> — [公式ベストプラクティス](https://code.claude.com/docs/en/best-practices)

```bash
# インストール
brew install gh    # macOS
sudo apt install gh  # Ubuntu

# 認証
gh auth login
```

Claude Code での活用例：

```
「gh を使って Issue #42 の内容を確認して、修正を実装して PR を出して」
「最近の PR のレビューコメントを確認して対応して」
「gh api を使ってリポジトリの統計を取得して」
```

### 未知のツールも学習できる

Claude Code は `--help` を読んで未知のツールを学習できる：

```
「foo-cli-tool --help を読んで使い方を学んで、それを使って X を実行して」
```

### 推奨 CLI ツール

| ツール | 用途 | Claude Code との連携 |
|--------|------|---------------------|
| `gh` | GitHub 操作 | Issue, PR, API 全般 |
| `jq` | JSON 処理 | API レスポンスの加工 |
| `rg` (ripgrep) | 高速検索 | コードベース探索 |
| `fd` | ファイル検索 | ファイル発見 |
| `delta` | diff 表示 | 変更確認 |
| `bat` | ファイル表示 | シンタックスハイライト付き確認 |

## リッチコンテンツの提供

Claude Code にはさまざまな方法でリッチなデータを提供できる（[公式ベストプラクティス](https://code.claude.com/docs/en/best-practices)）：

- **`@` でファイル参照**: コードの場所を説明する代わりに `@` で直接参照。Claude がファイルを読んでから応答する
- **画像を貼り付け**: コピペまたはドラッグ＆ドロップで画像をプロンプトに直接貼り付け
- **URL でドキュメント指定**: ドキュメントや API リファレンスの URL を与える。`/permissions` でよく使うドメインを許可リストに追加
- **パイプでデータ投入**: `cat error.log | claude` でファイル内容を直接送信
- **Claude に自分で取得させる**: Bash コマンド、MCP ツール、ファイル読み込みで必要なコンテキストを取得させる

## パイプラインでの活用

### Unix ユーティリティとして使う

Claude Code は Unix パイプラインに組み込める：

```bash
# ファイル内容を Claude に渡す
cat error.log | claude -p "このエラーログの根本原因を簡潔に説明して" > analysis.txt

# git diff をレビュー
git diff HEAD~5 | claude -p "この差分をレビューして問題点を指摘して"

# 構造化出力
claude -p "API エンドポイントを全てリストアップして" --output-format json

# ストリーミング
claude -p "再帰について説明して" --output-format stream-json --verbose
```

### シェルスクリプトとの統合

```bash
#!/bin/bash
# review.sh — 変更ファイルを自動レビュー

CHANGED_FILES=$(git diff --name-only HEAD~1)

for file in $CHANGED_FILES; do
  echo "=== Reviewing: $file ==="
  claude -p "以下のファイルをレビューして問題点があれば指摘して: @$file" \
    --allowedTools "Read,Bash"
  echo ""
done
```

### build スクリプトへの組み込み

```json
// package.json
{
  "scripts": {
    "lint:claude": "claude -p 'main からの変更を確認して、タイポに関する問題を報告して。ファイル名と行番号を1行目に、問題の説明を2行目に。他のテキストは返さない。'"
  }
}
```

## MCP サーバーの活用

### MCP とは

**Model Context Protocol** — AI ツールと外部サービスを接続するオープンスタンダード。Claude Code に Issue トラッカー、DB、モニタリング、Figma 等のツールを追加できる。

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
| `local`（デフォルト） | プロジェクトパス配下の設定 | 個人的な開発用 |
| `project` | `.mcp.json`（プロジェクトルート） | チーム共有（git にコミット） |
| `user` | `~/.claude.json` | 全プロジェクト共通の個人用 |

```bash
# プロジェクトスコープで追加（チーム共有向け）
claude mcp add --transport http paypal --scope project https://mcp.paypal.com/mcp

# ユーザースコープで追加（全プロジェクトで利用）
claude mcp add --transport http hubspot --scope user https://mcp.hubspot.com/anthropic
```

### .mcp.json でチーム共有

チームで MCP 設定を共有するには `.mcp.json` を git にコミット：

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

## 他の AI コーディングツールとの使い分け

### 各ツールの特性

| ツール | 強み | 最適な用途 |
|--------|------|-----------|
| **Claude Code** | ターミナル統合、自律実行、コンテキスト管理、エージェント機能 | 実装・デバッグ・リファクタリング・大規模タスク |
| **Cursor** | IDE 統合、インライン Diff、タブ補完 | 日常的なコーディング |
| **GitHub Copilot** | 無料枠あり、VS Code 統合 | コード補完、小さな修正 |
| **Windsurf** | IDE 統合、Flow モード | 探索的コーディング |

### 併用パターン

#### Claude Code + IDE ツール

```
1. IDE ツール（Cursor/Copilot）: 日常的なコーディング、タブ補完
2. Claude Code: 大きなリファクタリング、バグ調査、テスト作成、CI/CD 連携
3. 使い分け基準: 5分以内の作業 → IDE ツール、それ以上 → Claude Code
```

## よく使うコマンド早見表

| コマンド | 説明 |
|----------|------|
| `/clear` | コンテキストをクリア |
| `/compact` | 会話を要約して圧縮（フォーカス指定可） |
| `/init` | CLAUDE.md を自動生成 |
| `/model` | モデル切り替え |
| `/config` | 設定変更 |
| `/hooks` | Hook の対話的設定 |
| `/mcp` | MCP サーバーのステータス |
| `/permissions` | 権限設定 |
| `/memory` | メモリファイルを開く |
| `/agents` | サブエージェント管理 |
| `/rename` | セッション名を変更 |
| `/resume` | セッションピッカーを開く |
| `/rewind` | チェックポイントに戻る |
| `Shift+Tab` × 1 | Auto-Accept Mode |
| `Shift+Tab` × 2 | Plan Mode（計画のみ、実行しない） |
| `Esc` | 実行中のタスクを中断 |
| `Esc × 2` | Rewind メニューを開く |
| `Ctrl+O` | Verbose モード切り替え |
| `Option+T` / `Alt+T` | 拡張思考モード切り替え |
