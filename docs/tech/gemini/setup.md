# セットアップガイド

## 前提条件

- **Node.js 20 以上**
- macOS / Linux / Windows

## インストール

### npx で即座に実行（インストール不要）

```bash
npx @google/gemini-cli
```

### npm でグローバルインストール

```bash
npm install -g @google/gemini-cli
```

### Homebrew（macOS / Linux）

```bash
brew install gemini-cli
```

### リリースチャンネル

```bash
# 安定版（毎週火曜リリース）
npm install -g @google/gemini-cli@latest

# プレビュー版（最新機能、未検証の可能性あり）
npm install -g @google/gemini-cli@preview

# ナイトリー（毎日リリース、不安定）
npm install -g @google/gemini-cli@nightly
```

!!! tip "おすすめ"
    普段は `latest`（安定版）を使い、新機能を試したいときだけ `preview` を使いましょう。

## 認証設定

### 方法1: Google アカウントでログイン（推奨）

最も簡単な方法。無料枠（60回/分、1,000回/日）がすぐに使えます。

```bash
gemini
# 初回起動時に「Login with Google」を選択
# ブラウザが開くので Google アカウントでログイン
```

!!! note "Gemini Code Assist ライセンスがある場合"
    組織の有料ライセンスを使う場合は、プロジェクト ID を設定します：
    ```bash
    export GOOGLE_CLOUD_PROJECT="YOUR_PROJECT_ID"
    gemini
    ```

### 方法2: Gemini API キー

[Google AI Studio](https://aistudio.google.com/apikey) で API キーを取得して使用します。

```bash
export GEMINI_API_KEY="YOUR_API_KEY"
gemini
```

**メリット:**

- モデルの指定が可能
- 無料枠: 1,000回/日（Gemini 3 Flash/Pro混合）
- 従量課金でアップグレード可能

### 方法3: Vertex AI（エンタープライズ向け）

Google Cloud の本番環境やエンタープライズ用途に最適です。

```bash
export GOOGLE_API_KEY="YOUR_API_KEY"
export GOOGLE_GENAI_USE_VERTEXAI=true
gemini
```

## 基本的な使い方

### 起動

```bash
# カレントディレクトリで起動
gemini

# 複数ディレクトリを含めて起動
gemini --include-directories ../other-project

# 特定のモデルを指定
gemini -m gemini-2.5-pro
```

### 対話例

```
> このプロジェクトの構造を説明して

> src/main.ts のエラーハンドリングを改善して

> テストカバレッジが低いファイルを見つけて、テストを追加して

> @image.png このデザインを HTML/CSS で実装して
```

### 非対話モード（スクリプト連携）

```bash
# 1回きりのプロンプト実行
echo "README.md を日本語に翻訳して" | gemini

# パイプ入力
cat error.log | gemini "このエラーの原因と解決策を教えて"
```

### よく使うコマンド

| コマンド | 説明 |
|----------|------|
| `/help` | ヘルプ表示 |
| `/mcp` | MCP サーバーの状態確認 |
| `/model` | 現在のモデル情報 |
| `/clear` | コンテキストのクリア |
| `/save` | セッションの保存 |
| `/restore` | セッションの復元 |
| `@ファイル名` | ファイルを参照として追加 |

## 設定ファイル

### GEMINI.md — プロジェクトコンテキスト

Claude Code の `CLAUDE.md` に相当するファイルです。プロジェクトのルートに配置すると、Gemini CLI が自動的に読み込みます。

```markdown
# GEMINI.md

## プロジェクト概要
このプロジェクトは MkDocs を使ったドキュメントサイトです。

## コーディング規約
- TypeScript を使用
- ESLint + Prettier でフォーマット
- テストは Vitest を使用

## 重要なディレクトリ
- src/ — メインのソースコード
- docs/ — ドキュメント
- tests/ — テストファイル
```

!!! tip "CLAUDE.md との互換"
    両方のツールを使う場合、`CLAUDE.md` と `GEMINI.md` を両方置くか、共通のコンテキストファイルを作って各ツールから参照するのがおすすめです。

### settings.json — 詳細設定

設定ファイルは階層的に適用されます（下が優先）：

1. **システムデフォルト**: `/etc/gemini-cli/system-defaults.json`
2. **ユーザー設定**: `~/.gemini/settings.json`
3. **プロジェクト設定**: `.gemini/settings.json`
4. **環境変数**
5. **コマンドライン引数**

```json
// ~/.gemini/settings.json の例
{
  "core": {
    "model": "gemini-2.5-pro",
    "theme": "dark"
  },
  "mcpServers": {
    "github": {
      "command": "github-mcp-server",
      "args": ["stdio"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "$GITHUB_TOKEN"
      }
    }
  }
}
```

!!! warning "環境変数の参照"
    settings.json 内では `$VAR_NAME` や `${VAR_NAME}` の構文で環境変数を参照できます。API キーなどの機密情報はハードコードせず、環境変数経由で渡しましょう。

### .gemini ディレクトリ

プロジェクトのルートに `.gemini/` ディレクトリを作成すると、プロジェクト固有の設定を管理できます：

```
.gemini/
├── settings.json    # プロジェクト設定
└── extensions/      # カスタム拡張
```

## トラブルシューティング

### 認証エラー

```bash
# 認証情報をリセット
gemini --reset-auth

# デバッグモードで詳細確認
gemini --debug
```

### Node.js バージョンの確認

```bash
node --version  # v20 以上が必要
```

### プロキシ環境

```bash
export HTTP_PROXY="http://proxy.example.com:8080"
export HTTPS_PROXY="http://proxy.example.com:8080"
gemini
```

## 次のステップ

- [活用事例](use-cases.md) — 具体的な使い方を学ぶ
- [Claude Codeとの連携](integration.md) — 両ツールを組み合わせる
