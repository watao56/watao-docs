# CI/CD・GitHub Actions 連携ガイド

Claude Code をヘッドレスモードで CI/CD パイプラインに統合する方法。GitHub Actions での自動化、PR レビュー、Issue 対応のワークフローを解説する。

## ヘッドレスモード（プログラマティック実行）

`claude -p` フラグで Claude Code を非対話的に実行できる。スクリプト、CI/CD パイプライン、自動化ワークフローに組み込める。

### 基本的な使い方

```bash
# 基本的なクエリ
claude -p "このプロジェクトが何をするか説明して"

# ツールの自動許可
claude -p "テストスイートを実行してエラーを修正して" \
  --allowedTools "Read,Edit,Bash"

# Plan Mode（読み取り専用）で実行
claude --permission-mode plan -p "認証システムを分析して改善提案を出して"
```

### 出力フォーマット

| フォーマット | フラグ | 用途 |
|------------|--------|------|
| テキスト（デフォルト） | `--output-format text` | 人間が読む出力 |
| JSON | `--output-format json` | 構造化データ、メタデータ付き |
| ストリーム JSON | `--output-format stream-json` | リアルタイムストリーミング |

```bash
# JSON で出力（セッション ID、使用量等のメタデータ付き）
claude -p "プロジェクトを要約して" --output-format json

# JSON スキーマで構造化出力
claude -p "auth.py のメイン関数名を抽出して" \
  --output-format json \
  --json-schema '{"type":"object","properties":{"functions":{"type":"array","items":{"type":"string"}}},"required":["functions"]}'

# ストリーミング（リアルタイム処理）
claude -p "再帰について説明して" --output-format stream-json --verbose
```

### パイプライン統合

```bash
# ファイル内容を渡す
cat error.log | claude -p "このエラーログの根本原因を簡潔に説明して"

# git diff を渡してレビュー
git diff HEAD~5 | claude -p "この差分をレビューして問題点を指摘して"

# 出力をファイルに保存
claude -p "README.md を日本語に翻訳して" > README.ja.md

# 他のコマンドにパイプ
claude -p "<your prompt>" --output-format json | your_command
```

### 会話の継続

```bash
# 最初のリクエスト
claude -p "このコードベースのパフォーマンス問題をレビューして"

# 直前の会話を継続
claude -p "データベースクエリに焦点を当てて" --continue

# セッション ID を指定して継続
session_id=$(claude -p "レビューを開始" --output-format json | jq -r '.session_id')
claude -p "レビューを続けて" --resume "$session_id"
```

### システムプロンプトのカスタマイズ

```bash
# デフォルトプロンプトに追加
gh pr diff "$1" | claude -p \
  --append-system-prompt "あなたはセキュリティエンジニアです。脆弱性をレビューしてください。" \
  --output-format json

# システムプロンプトを完全に置き換え
claude -p "コードを分析して" --system-prompt "あなたはシニアコードレビューアーです。"
```

## GitHub Actions 連携

### Claude Code GitHub Actions とは

GitHub のワークフローに AI 自動化を追加する公式アクション。PR や Issue のコメントで `@claude` とメンションするだけで、Claude がコードを分析し、PR を作成し、機能を実装し、バグを修正する。

- [公式リポジトリ: anthropics/claude-code-action](https://github.com/anthropics/claude-code-action)

### セットアップ

#### 推奨: Claude Code CLI から

```bash
# Claude Code 内で実行
/install-github-app
```

ガイドに従って GitHub App と必要なシークレットをセットアップする。

#### 手動セットアップ

1. [Claude GitHub App](https://github.com/apps/claude) をリポジトリにインストール
2. `ANTHROPIC_API_KEY` をリポジトリシークレットに追加
3. [ワークフローファイル](https://github.com/anthropics/claude-code-action/blob/main/examples/claude.yml)を `.github/workflows/` にコピー

### 基本的なワークフロー

```yaml
# .github/workflows/claude.yml
name: Claude Code
on:
  issue_comment:
    types: [created]
  pull_request_review_comment:
    types: [created]

jobs:
  claude:
    runs-on: ubuntu-latest
    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          # @claude メンションに自動応答
```

### スキルを使ったコードレビュー

```yaml
name: Code Review
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: "/review"
          claude_args: "--max-turns 5"
```

### 定時レポート生成

```yaml
name: Daily Report
on:
  schedule:
    - cron: "0 9 * * *"

jobs:
  report:
    runs-on: ubuntu-latest
    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: "昨日のコミットとオープンイシューのサマリーを生成して"
          claude_args: "--model opus"
```

### @claude メンションの使い方

Issue や PR のコメントで：

```
@claude この Issue の内容に基づいて機能を実装して
@claude このエンドポイントのユーザー認証をどう実装すべき？
@claude ユーザーダッシュボードコンポーネントの TypeError を修正して
```

Claude が自動的にコンテキストを分析して対応する。

### アクションパラメータ

| パラメータ | 説明 | 必須 |
|-----------|------|------|
| `prompt` | Claude への指示（テキストまたは `/review` 等のスキル） | いいえ |
| `claude_args` | Claude Code CLI 引数 | いいえ |
| `anthropic_api_key` | Claude API キー | はい* |
| `github_token` | GitHub API アクセストークン | いいえ |
| `trigger_phrase` | カスタムトリガーフレーズ（デフォルト: `@claude`） | いいえ |
| `use_bedrock` | AWS Bedrock を使用 | いいえ |
| `use_vertex` | Google Vertex AI を使用 | いいえ |

\* Bedrock/Vertex 使用時は不要

### CLI 引数の渡し方

```yaml
claude_args: "--max-turns 5 --model claude-sonnet-4-5-20250929 --mcp-config /path/to/config.json"
```

よく使う引数：

- `--max-turns`: 最大会話ターン数（デフォルト: 10）
- `--model`: 使用モデル
- `--mcp-config`: MCP 設定ファイルのパス
- `--allowed-tools`: 許可するツール（カンマ区切り）

## 実践的な CI/CD パターン

### PR ごとの自動レビュー + コメント

```yaml
name: Auto Review
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: |
            この PR の変更をレビューして。以下の観点で確認：
            - バグの可能性
            - セキュリティリスク
            - パフォーマンス問題
            - テストカバレッジ
            問題があれば具体的な修正案を提示して。
          claude_args: "--max-turns 5"
```

### Issue からの自動 PR 作成

```yaml
name: Auto Fix Issue
on:
  issues:
    types: [labeled]

jobs:
  fix:
    if: contains(github.event.label.name, 'claude-fix')
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: |
            この Issue を修正して PR を作成して。
            テストも書いて、lint と型チェックを通して。
```

### ステージされた変更の自動コミット

```bash
#!/bin/bash
# CI/CD やスクリプトでの自動コミット
claude -p "ステージされた変更を確認して適切なコミットメッセージでコミットして" \
  --allowedTools "Bash(git diff *),Bash(git log *),Bash(git status *),Bash(git commit *)"
```

### 変更ファイルの自動チェック（pre-commit hook）

```bash
#!/bin/bash
# .git/hooks/pre-commit

STAGED_FILES=$(git diff --cached --name-only --diff-filter=ACM)

if [ -n "$STAGED_FILES" ]; then
  claude -p "以下のファイルの変更に明らかなバグやセキュリティ問題がないか
    簡潔にチェックして: $STAGED_FILES" \
    --allowedTools "Read,Bash" 2>/dev/null
fi
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

## AWS Bedrock / Google Vertex AI での利用

エンタープライズ環境では、自社のクラウドインフラ経由で Claude Code GitHub Actions を使用できる。

### AWS Bedrock

```yaml
- name: Configure AWS Credentials
  uses: aws-actions/configure-aws-credentials@v4
  with:
    role-to-assume: ${{ secrets.AWS_ROLE_TO_ASSUME }}
    aws-region: us-west-2

- uses: anthropics/claude-code-action@v1
  with:
    github_token: ${{ steps.app-token.outputs.token }}
    use_bedrock: "true"
    claude_args: '--model us.anthropic.claude-sonnet-4-5-20250929-v1:0 --max-turns 10'
```

### Google Vertex AI

```yaml
- name: Authenticate to Google Cloud
  uses: google-github-actions/auth@v2
  with:
    workload_identity_provider: ${{ secrets.GCP_WORKLOAD_IDENTITY_PROVIDER }}
    service_account: ${{ secrets.GCP_SERVICE_ACCOUNT }}

- uses: anthropics/claude-code-action@v1
  with:
    github_token: ${{ steps.app-token.outputs.token }}
    use_vertex: "true"
    claude_args: '--model claude-sonnet-4@20250514 --max-turns 10'
  env:
    ANTHROPIC_VERTEX_PROJECT_ID: ${{ steps.auth.outputs.project_id }}
    CLOUD_ML_REGION: us-east5
```

## コスト最適化

### GitHub Actions のコスト

- Claude Code は GitHub ホストランナーで動作し、GitHub Actions の分数を消費する
- API トークンの使用量はタスクの複雑さとコードベースのサイズに依存

### 最適化のヒント

- `@claude` コマンドを具体的にして不要な API 呼び出しを減らす
- `--max-turns` を適切に設定して過度な反復を防ぐ
- ワークフローレベルのタイムアウトで暴走ジョブを防止
- GitHub の concurrency 制御で並列実行を制限

```yaml
# タイムアウトと並行制御の例
jobs:
  claude:
    runs-on: ubuntu-latest
    timeout-minutes: 15
    concurrency:
      group: claude-${{ github.ref }}
      cancel-in-progress: true
```

## セキュリティの注意事項

- API キーは必ず GitHub Secrets を使用する（ワークフローに直接書かない）
- Claude の提案はマージ前に必ずレビューする
- アクション権限を必要最小限にする
- `--dangerously-skip-permissions` は信頼できる環境（サンドボックス、CI）でのみ使用する

```yaml
# 最小権限の例
permissions:
  contents: write
  pull-requests: write
  issues: write
  id-token: write  # OIDC 認証時のみ必要
```
