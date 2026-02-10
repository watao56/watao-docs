# 🍎 Mac版 セットアップ完全ガイド

Claude Code を macOS にインストールして使い始めるための完全ガイドです。

---

## 📋 前提条件

### 必須要件

| 項目 | 要件 |
|------|------|
| **macOS** | macOS 12 (Monterey) 以降 |
| **アカウント** | [Claude Pro/Max/Teams/Enterprise](https://claude.com/pricing) または [Anthropic Console](https://console.anthropic.com/) アカウント |

### あると便利なもの

- **Homebrew** — パッケージ管理（インストール方法の選択肢が増える）
- **Git** — バージョン管理（Xcode CLI Tools に含まれる）
- **Ghostty** — Claude Code との相性が最高のターミナル

### Xcode Command Line Tools の確認

```bash
# インストール済みか確認
xcode-select -p

# 未インストールの場合
xcode-select --install
```

---

## 🚀 インストール

3つの方法から選べます。

### 方法1: ネイティブインストール（推奨 ⭐）

Node.js 不要で最もシンプルな方法です。

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

特定のバージョンをインストールしたい場合：

```bash
# 最新版（安定版より新しい場合がある）
curl -fsSL https://claude.ai/install.sh | bash -s latest

# バージョン指定
curl -fsSL https://claude.ai/install.sh | bash -s 1.0.58
```

インストール先: `~/.local/bin/claude`

### 方法2: Homebrew

```bash
brew install --cask claude-code
```

Homebrew で管理したい人向け。アップデートも `brew upgrade` で行えます。

### 方法3: npm（従来の方法）

```bash
# Node.js 18以上が必要
node -v  # v18.0.0 以上であることを確認

npm install -g @anthropic-ai/claude-code
```

!!! tip "どれを選ぶ？"
    特にこだわりがなければ **方法1（ネイティブインストール）** がおすすめです。依存関係が少なく、トラブルも起きにくいです。

---

## 🔑 認証設定

### 初回ログイン

```bash
cd your-project
claude
```

初回起動時にログイン画面が表示されます。ブラウザが自動で開くので、アカウントでログインしてください。

ブラウザが自動で開かない場合は `c` キーを押すとURLがクリップボードにコピーされます。

### ログイン方法の選択肢

| 方法 | 説明 |
|------|------|
| **Claude.ai アカウント** | Pro/Max/Teams/Enterprise サブスクリプション（推奨） |
| **Anthropic Console** | API利用（プリペイドクレジット制） |
| **サードパーティ** | Amazon Bedrock / Google Vertex AI / Microsoft Foundry |

### アカウント切り替え

```bash
# Claude Code 内で
/login

# 認証情報をリセットしたい場合
rm -rf ~/.config/claude-code/auth.json
claude
```

---

## 💻 ターミナル設定

### Ghostty（推奨 ⭐）

Claude Code との相性が最も良いターミナルです。

```bash
# Homebrew でインストール
brew install --cask ghostty
```

Ghostty の設定ファイル（`~/.config/ghostty/config`）：

```ini
# フォント設定
font-family = JetBrains Mono
font-size = 14

# テーマ
theme = catppuccin-mocha

# ウィンドウ設定
window-padding-x = 10
window-padding-y = 10

# クリップボード
clipboard-read = allow
clipboard-write = allow

# スクロールバック
scrollback-limit = 10000
```

!!! info "Ghostty の詳細設定"
    Ghostty と Claude Code の連携についてより詳しくは [Ghostty 連携ガイド](ghostty-integration.md) を参照してください。

### iTerm2

```bash
brew install --cask iterm2
```

推奨設定：

- **Profiles → Text → Font**: JetBrains Mono / 14pt
- **Profiles → Terminal → Scrollback Lines**: Unlimited
- **Profiles → Keys → Key Mappings**: Natural Text Editing プリセット

### Terminal.app（macOS 標準）

特別な設定なしでも動作します。以下を推奨：

- **環境設定 → プロファイル → フォント**: Menlo 14pt 以上
- **環境設定 → プロファイル → シェル**: 「シェルが終了したとき」→「ウインドウを閉じる」

---

## ⚙️ 初期設定

### CLAUDE.md の作成

プロジェクトルートに `CLAUDE.md` を作成して、Claude にプロジェクトの情報を伝えましょう。

```bash
cd your-project
cat > CLAUDE.md << 'EOF'
# プロジェクト概要

このプロジェクトは〇〇です。

## 技術スタック
- 言語: TypeScript
- フレームワーク: Next.js
- パッケージマネージャ: pnpm

## コーディング規約
- 日本語コメントを使用
- ESLint + Prettier でフォーマット

## よく使うコマンド
- `pnpm dev` — 開発サーバー起動
- `pnpm build` — ビルド
- `pnpm test` — テスト実行
EOF
```

CLAUDE.md の配置場所と用途：

| ファイル | 場所 | 用途 |
|----------|------|------|
| `CLAUDE.md` | プロジェクトルート | チーム共有の指示（Git管理） |
| `CLAUDE.local.md` | プロジェクトルート | 個人用の指示（Git管理外） |
| `~/.claude/CLAUDE.md` | ホーム | 全プロジェクト共通の個人設定 |
| `.claude/rules/*.md` | プロジェクト内 | トピック別のルールファイル |

### MCP サーバーの設定

外部ツールと連携するための MCP（Model Context Protocol）サーバーを追加できます。

```bash
# リモートサーバー（HTTP）を追加
claude mcp add --transport http notion https://mcp.notion.com/mcp

# ローカルサーバー（stdio）を追加
claude mcp add --transport stdio github -- npx -y @modelcontextprotocol/server-github

# サーバー一覧を確認
claude mcp list

# サーバーを削除
claude mcp remove notion
```

プロジェクト共有用の MCP 設定（`.mcp.json`）：

```bash
# プロジェクトスコープで追加（.mcp.json に保存、Git管理可能）
claude mcp add --transport http stripe --scope project https://mcp.stripe.com
```

### 設定ファイルの場所

| ファイル | 用途 |
|----------|------|
| `~/.claude/settings.json` | ユーザー設定（権限、フック等） |
| `.claude/settings.json` | プロジェクト設定（Git管理） |
| `.claude/settings.local.json` | ローカルプロジェクト設定 |
| `~/.claude.json` | グローバル状態（テーマ、OAuth、MCP） |
| `.mcp.json` | プロジェクトMCPサーバー |

---

## 📖 よく使うコマンド

### 起動・基本操作

```bash
# インタラクティブモードで起動
claude

# ワンショットタスク
claude "ビルドエラーを修正して"

# クエリ実行して終了
claude -p "この関数を説明して"

# 直前の会話を続ける
claude -c

# 過去の会話を再開
claude -r
```

### セッション内コマンド

```
/help          — ヘルプ表示
/clear         — 会話履歴クリア
/compact       — コンテキストを圧縮
/config        — 設定画面を開く
/login         — アカウント切り替え
/logout        — ログアウト
/memory        — メモリファイルを編集
/mcp           — MCPサーバーの状態確認
/permissions   — 権限設定
/bug           — バグレポート送信
/doctor        — インストール状態の診断
exit / Ctrl+C  — 終了
```

### Git 操作

```bash
# コミットメッセージを自動生成
claude commit

# 会話の中で
> 変更をコミットして
> PRを作成して
> マージコンフリクトを解決して
```

---

## 🔧 トラブルシューティング

### `claude: command not found`

PATH が通っていない可能性があります。

```bash
# ネイティブインストールの場合
export PATH="$HOME/.local/bin:$PATH"

# シェル設定に追加（zshの場合）
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

### npm インストールで権限エラー

```bash
# sudo は使わない！代わりに npm のグローバルディレクトリを変更
mkdir -p ~/.npm-global
npm config set prefix '~/.npm-global'
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.zshrc
source ~/.zshrc

# 再インストール
npm install -g @anthropic-ai/claude-code
```

または、ネイティブインストールに切り替えてください（推奨）。

### ブラウザが自動で開かない

ログイン時にブラウザが開かない場合：

1. `c` キーを押してURLをコピー
2. ブラウザに手動で貼り付け

### 検索やファイル参照が動かない

```bash
# ripgrep をインストール
brew install ripgrep

# 環境変数を設定
export USE_BUILTIN_RIPGREP=0
```

### CPU / メモリ使用量が高い

- `/compact` で定期的にコンテキストを圧縮
- タスク間で Claude Code を再起動
- `.gitignore` に大きなビルドディレクトリを追加

### 設定をリセットしたい

```bash
# ユーザー設定をリセット
rm ~/.claude.json
rm -rf ~/.claude/

# プロジェクト設定をリセット
rm -rf .claude/
rm .mcp.json
```

### インストール状態の診断

```bash
claude doctor
```

`/doctor` コマンドで以下をチェックできます：

- インストール方法・バージョン・検索機能
- 自動アップデートの状態
- 設定ファイルのバリデーション
- MCPサーバーの設定エラー
- コンテキスト使用量の警告

---

## 🔗 関連ページ

- [CLAUDE.md メモリ管理](md-files-guide.md)
- [エージェントモード活用](agent-mode.md)
- [Hooks・自動化](hooks-automation.md)
- [CLI・MCP 連携](cli-integration.md)
- [Ghostty 連携](ghostty-integration.md)
- [Tips & Tricks](tips-and-tricks.md)
- [公式ドキュメント](https://code.claude.com/docs/en/overview)
