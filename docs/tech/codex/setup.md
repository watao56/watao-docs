# Codex CLI セットアップガイド

## インストール

### npm経由（推奨）

```bash
# グローバルインストール
npm install -g @openai/codex

# バージョン確認
codex --version
```

### Homebrew（macOS）

```bash
# Homebrewでインストール
brew install --cask codex

# 確認
codex --version
```

### 事前準備

**必要な環境：**
- Node.js 18.0以上
- npm 8.0以上
- Git（リポジトリ操作用）

!!! tip "推奨環境"
    Node.js 20以上を使用すると、パフォーマンスが向上します。

## 認証設定

### OpenAI API Key設定

初回実行時に自動的にサインインプロセスが開始されます：

```bash
# 初回実行
codex

# または明示的に認証
codex auth login
```

**手動設定の場合：**

```bash
# 環境変数で設定
export OPENAI_API_KEY="your-api-key-here"

# または設定ファイルで管理
codex config set api_key "your-api-key-here"
```

### 他プロバイダ設定

**Anthropic Claude:**
```bash
codex config set provider "anthropic"
codex config set anthropic_api_key "your-anthropic-key"
```

**Google Gemini:**
```bash
codex config set provider "google"
codex config set google_api_key "your-google-key"
```

**カスタムエンドポイント:**
```bash
codex config set provider "custom"
codex config set custom_endpoint "https://your-llm-endpoint.com/v1"
codex config set custom_api_key "your-custom-key"
```

## 基本的な使い方

### プロジェクトでの初回セットアップ

```bash
# プロジェクトルートに移動
cd /path/to/your/project

# Codex設定を初期化
codex init

# 基本的なコード生成・修正
codex "ログイン機能を追加してください"
```

### よく使うコマンド

```bash
# インタラクティブモード
codex

# 特定ファイルの編集
codex edit src/components/Login.tsx

# プロジェクト全体の分析・修正
codex analyze

# 自動修正モード
codex auto-fix

# テスト実行
codex test

# コミット前チェック
codex pre-commit
```

!!! tip "効果的な指示の書き方"
    - **具体的に**：「ログイン画面を作る」ではなく「React + TypeScriptでメール/パスワード入力フォームを持つログイン画面を作成」
    - **文脈を提供**：「既存のButton コンポーネントを使って」など
    - **制約を明示**：「100行以下で」「既存のAPIスキーマに合わせて」など

## 設定ファイル

### グローバル設定（~/.codex/config.json）

```json
{
  "provider": "openai",
  "model": "gpt-4",
  "temperature": 0.2,
  "max_tokens": 4000,
  "auto_commit": false,
  "safe_mode": true,
  "editor": "code",
  "diff_tool": "git"
}
```

### プロジェクト設定（.codex/config.json）

```json
{
  "exclude_patterns": [
    "node_modules/**",
    "dist/**",
    "*.log",
    ".env*"
  ],
  "include_patterns": [
    "src/**/*.ts",
    "src/**/*.tsx",
    "src/**/*.js",
    "src/**/*.jsx"
  ],
  "rules": [
    "TypeScriptを使用",
    "ESLintルールに従う",
    "テストも合わせて更新",
    "JSDocコメントを追加"
  ],
  "auto_modes": {
    "auto_edit": true,
    "full_auto": false,
    "auto_test": true
  }
}
```

### 主要な設定項目

| 設定項目 | 説明 | 推奨値 |
|----------|------|---------|
| `provider` | LLMプロバイダ | `openai` |
| `model` | 使用モデル | `gpt-4-turbo` |
| `temperature` | 生成の多様性 | `0.2` (安定重視) |
| `safe_mode` | 安全モード | `true` |
| `auto_commit` | 自動コミット | `false` |
| `max_files` | 同時編集ファイル数 | `10` |

## トラブルシューティング

### よくある問題と解決法

**認証エラー:**
```bash
# キャッシュクリア
codex auth logout
codex auth login

# 設定確認
codex config show
```

**ファイル除外が効かない:**
```bash
# .codexignoreファイルを作成
echo "node_modules/" > .codexignore
echo "dist/" >> .codexignore
```

**メモリ不足エラー:**
```bash
# Node.jsメモリ制限を増やす
export NODE_OPTIONS="--max-old-space-size=4096"
codex
```

!!! warning "注意事項"
    - **初回実行時はプロジェクトをバックアップ**してから使用
    - **大規模プロジェクトでは段階的に導入**することを推奨
    - **APIキーは適切に管理**し、公開リポジトリにコミットしない

## 次のステップ

- [活用事例](use-cases.md) - 実際の開発ワークフローでの使用例
- [ツール比較](comparison.md) - 他のAIコーディングツールとの詳細比較

!!! success "セットアップ完了"
    設定が完了したら、小さなプロジェクトで試してみましょう：<br>
    `codex "このREADMEファイルを改善してください"`