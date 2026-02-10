# 🪟 Windows版 セットアップ完全ガイド

Claude Code を Windows にインストールして使い始めるための完全ガイドです。

---

## 📋 前提条件

### 必須要件

| 項目 | 要件 |
|------|------|
| **Windows** | Windows 10 以降 |
| **Git for Windows** | [Git for Windows](https://git-scm.com/downloads/win)（Git Bash 含む） |
| **アカウント** | [Claude Pro/Max/Teams/Enterprise](https://claude.com/pricing) または [Anthropic Console](https://console.anthropic.com/) アカウント |

!!! warning "Git for Windows は必須"
    Claude Code のネイティブ Windows 版は **Git for Windows（Git Bash）** が必要です。事前にインストールしておいてください。

### 推奨環境

- **Windows Terminal** — モダンなターミナル体験
- **WSL2** — Linux環境での利用（オプション）

---

## 🚀 インストール

Windows では主に2つの方法があります。

### 方法A: ネイティブ Windows（推奨 ⭐）

Git Bash がインストールされた Windows 上で直接動作します。

=== "PowerShell"

    ```powershell
    irm https://claude.ai/install.ps1 | iex
    ```

=== "コマンドプロンプト (CMD)"

    ```cmd
    curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
    ```

=== "WinGet"

    ```powershell
    winget install Anthropic.ClaudeCode
    ```

特定のバージョンをインストールしたい場合（PowerShell）：

```powershell
# 最新版
& ([scriptblock]::Create((irm https://claude.ai/install.ps1))) latest

# バージョン指定
& ([scriptblock]::Create((irm https://claude.ai/install.ps1))) 1.0.58
```

インストール先: `%USERPROFILE%\.local\bin\claude.exe`

### 方法B: WSL2 経由

Linux 環境で動かしたい場合。ファイルシステム性能が良く、Linux ツールチェーンがそのまま使えます。

#### WSL2 のセットアップ

```powershell
# PowerShell（管理者）で実行
wsl --install

# 再起動後、Ubuntu が起動する
# ユーザー名とパスワードを設定
```

#### WSL2 内でインストール

```bash
# WSL2 の Ubuntu ターミナルで実行
curl -fsSL https://claude.ai/install.sh | bash
```

!!! tip "どちらを選ぶ？"
    - **ネイティブ Windows**: Windows のファイルをそのまま編集したい場合
    - **WSL2**: Linux 開発環境を使いたい場合、Docker を多用する場合

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

# 認証情報をリセットしたい場合（PowerShell）
Remove-Item -Recurse -Force "$env:USERPROFILE\.config\claude-code\auth.json"
claude
```

---

## 💻 ターミナル設定

### Windows Terminal（推奨 ⭐）

Windows 11 には標準搭載。Windows 10 の場合は Microsoft Store からインストール。

推奨設定（`settings.json`）：

```json
{
    "profiles": {
        "defaults": {
            "font": {
                "face": "JetBrains Mono",
                "size": 14
            },
            "padding": "10",
            "scrollbarState": "visible"
        }
    },
    "actions": [
        { "command": "paste", "keys": "ctrl+v" },
        { "command": "copy", "keys": "ctrl+c" }
    ]
}
```

!!! tip "フォントのインストール"
    [JetBrains Mono](https://www.jetbrains.com/lp/mono/) または [Cascadia Code](https://github.com/microsoft/cascadia-code) がおすすめです。リガチャ対応でコードが読みやすくなります。

### PowerShell での利用

ネイティブ Windows 版はPowerShellから直接起動できます。

```powershell
cd C:\Users\YourName\projects\your-project
claude
```

### Git Bash での利用

```bash
cd /c/Users/YourName/projects/your-project
claude
```

### WSL2 ターミナルでの利用

Windows Terminal で WSL2 プロファイルを選択して使います。

```bash
cd ~/projects/your-project
claude
```

---

## ⚙️ 初期設定

### CLAUDE.md の作成

プロジェクトルートに `CLAUDE.md` を作成して、Claude にプロジェクトの情報を伝えましょう。

```bash
cd your-project
```

`CLAUDE.md` の内容例：

```markdown
# プロジェクト概要

このプロジェクトは〇〇です。

## 技術スタック
- 言語: TypeScript
- フレームワーク: Next.js
- パッケージマネージャ: npm

## コーディング規約
- 日本語コメントを使用
- ESLint + Prettier でフォーマット

## よく使うコマンド
- `npm run dev` — 開発サーバー起動
- `npm run build` — ビルド
- `npm test` — テスト実行
```

CLAUDE.md の配置場所と用途：

| ファイル | 場所 | 用途 |
|----------|------|------|
| `CLAUDE.md` | プロジェクトルート | チーム共有の指示（Git管理） |
| `CLAUDE.local.md` | プロジェクトルート | 個人用の指示（Git管理外） |
| `~/.claude/CLAUDE.md` | ホーム | 全プロジェクト共通の個人設定 |
| `.claude/rules/*.md` | プロジェクト内 | トピック別のルールファイル |

!!! info "Windows でのホームディレクトリ"
    `~` は `C:\Users\YourName` を指します。

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

### 設定ファイルの場所（Windows）

| ファイル | 場所 |
|----------|------|
| ユーザー設定 | `C:\Users\YourName\.claude\settings.json` |
| グローバル状態 | `C:\Users\YourName\.claude.json` |
| マネージド設定 | `C:\Program Files\ClaudeCode\managed-settings.json` |
| プロジェクト設定 | `.claude\settings.json`（プロジェクトルート） |
| プロジェクトMCP | `.mcp.json`（プロジェクトルート） |

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

## ⚠️ Windows 固有の注意点

### ファイルパスの違い

Windows はバックスラッシュ（`\`）がパス区切りですが、Claude Code は内部的にスラッシュ（`/`）も受け付けます。

```bash
# どちらでもOK
claude -p "C:\Users\YourName\project\src\index.ts を説明して"
claude -p "C:/Users/YourName/project/src/index.ts を説明して"
```

### WSL2 のファイルシステムに注意

WSL2 を使う場合、**プロジェクトは Linux ファイルシステム上に置く**のがベストです。

```bash
# ✅ 推奨: Linux ファイルシステム
cd ~/projects/my-app
claude

# ⚠️ 非推奨: Windows ファイルシステム（遅い）
cd /mnt/c/Users/YourName/projects/my-app
claude
```

Windows ファイルシステム（`/mnt/c/`）上で作業すると、ファイル読み書きが遅くなり、検索結果が不完全になることがあります。

### 改行コードの問題

Windows（CRLF）と Linux（LF）の改行コードの違いに注意。Git の設定で制御できます。

```bash
# Git で改行コードを自動変換
git config --global core.autocrlf true
```

---

## 🔧 トラブルシューティング

### 「Claude Code on Windows requires git-bash」エラー

Git for Windows がインストールされていないか、パスが通っていません。

```powershell
# Git Bash のパスを明示的に設定
$env:CLAUDE_CODE_GIT_BASH_PATH="C:\Program Files\Git\bin\bash.exe"

# システム環境変数に永続的に追加する場合
# システムのプロパティ → 環境変数 から設定
```

### `claude` コマンドが見つからない

```powershell
# PATH を確認
$env:PATH -split ";"

# 手動で PATH に追加
$env:PATH += ";$env:USERPROFILE\.local\bin"

# 永続化（PowerShell プロファイルに追加）
Add-Content $PROFILE "`n`$env:PATH += `";$env:USERPROFILE\.local\bin`""
```

### WSL2 での npm / Node.js の問題

WSL が Windows 側の npm/Node.js を参照してしまう問題：

```bash
# どの npm/node が使われているか確認
which npm    # /mnt/c/... なら Windows 版が使われている
which node

# nvm で Linux 版 Node.js をインストール（推奨）
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.0/install.sh | bash
source ~/.bashrc
nvm install --lts
nvm use --lts

# シェル設定に nvm のロードを追加
cat >> ~/.bashrc << 'EOF'
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
EOF
```

### WSL2 で npm インストール時の OS エラー

```bash
# OS検出の問題を回避
npm config set os linux
npm install -g @anthropic-ai/claude-code --force --no-os-check
```

⚠️ `sudo` は使わないでください。

### WSL2 でサンドボックスが動かない

```bash
# 必要なパッケージをインストール
# Ubuntu/Debian
sudo apt-get install bubblewrap socat

# Fedora
sudo dnf install bubblewrap socat
```

WSL1 はサンドボックス非対応です。WSL2 にアップグレードしてください。

### WSL2 で検索が遅い・結果が不完全

Windows ファイルシステム（`/mnt/c/`）上で作業している場合に発生します。

対処法：

1. プロジェクトを Linux ファイルシステム（`/home/`）に移動
2. より具体的な検索クエリを使う（ディレクトリやファイルタイプを指定）
3. ネイティブ Windows 版の使用を検討

### 検索やファイル参照が動かない

```powershell
# ripgrep をインストール（WinGet）
winget install BurntSushi.ripgrep.MSVC
```

環境変数 `USE_BUILTIN_RIPGREP=0` を設定してください。

### JetBrains IDE が WSL2 から検出されない

WSL2 のネットワーク設定が原因の場合があります。

```powershell
# PowerShell（管理者）でファイアウォールルールを追加
New-NetFirewallRule -DisplayName "Allow WSL2 Internal Traffic" `
    -Direction Inbound -Protocol TCP -Action Allow `
    -RemoteAddress 172.21.0.0/16 -LocalAddress 172.21.0.0/16
```

または `.wslconfig` でミラードネットワークを有効化：

```ini
# %USERPROFILE%\.wslconfig
[wsl2]
networkingMode=mirrored
```

設定後、PowerShell で `wsl --shutdown` を実行して WSL を再起動してください。

### 設定をリセットしたい

```powershell
# PowerShell
Remove-Item "$env:USERPROFILE\.claude.json"
Remove-Item -Recurse -Force "$env:USERPROFILE\.claude"

# プロジェクト設定もリセット
Remove-Item -Recurse -Force ".claude"
Remove-Item ".mcp.json"
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

---

## 🔗 関連ページ

- [CLAUDE.md メモリ管理](md-files-guide.md)
- [エージェントモード活用](agent-mode.md)
- [Hooks・自動化](hooks-automation.md)
- [CLI・MCP 連携](cli-integration.md)
- [Tips & Tricks](tips-and-tricks.md)
- [公式ドキュメント](https://code.claude.com/docs/en/overview)
