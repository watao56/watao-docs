# セットアップ手順

MCPを各種AIツールで利用するための設定手順をまとめています。環境に応じた設定方法を選択してください。

## Claude Desktop でのMCP設定

### 1. 基本設定

Claude Desktopは最も簡単にMCPを試せる環境です。

!!! info "対応プラットフォーム"
    - Windows
    - macOS
    - Linux

#### 設定ファイルの場所

=== "macOS"
    ```bash
    ~/Library/Application Support/Claude/claude_desktop_config.json
    ```

=== "Windows"
    ```bash
    %APPDATA%\Claude\claude_desktop_config.json
    ```

=== "Linux"
    ```bash
    ~/.config/Claude/claude_desktop_config.json
    ```

### 2. 設定手順

#### Step 1: 設定ファイルアクセス

!!! example "GUI経由でのアクセス"
    1. Claude Desktopを起動
    2. **Settings** > **Developer** タブ
    3. **Edit Config** ボタンをクリック

#### Step 2: 基本設定の記述

!!! example "基本設定例"
    ```json
    {
      "mcpServers": {
        "filesystem": {
          "command": "npx",
          "args": ["-y", "@anthropic/mcp-fs", "/Users/username/Documents"]
        }
      }
    }
    ```

#### Step 3: Claude Desktopの再起動

設定変更後は必ずアプリケーションを再起動してください。

!!! warning "重要"
    設定変更は再起動後に有効になります。

### 3. 複数サーバーの設定

!!! example "複数MCPサーバー設定例"
    ```json
    {
      "mcpServers": {
        "filesystem": {
          "command": "npx",
          "args": ["-y", "@anthropic/mcp-fs", "/workspace"],
          "env": {
            "NODE_ENV": "production"
          }
        },
        "github": {
          "command": "npx",
          "args": ["-y", "@modelcontextprotocol/server-github"],
          "env": {
            "GITHUB_PERSONAL_ACCESS_TOKEN": "your_token_here"
          }
        },
        "sqlite": {
          "command": "npx",
          "args": ["-y", "@modelcontextprotocol/server-sqlite", "./data.db"]
        }
      }
    }
    ```

---

## Claude Code でのMCP設定

Claude Codeは開発者向けのより高度なMCP機能を提供します。

### 1. CLI経由での設定（推奨）

#### Step 1: MCPサーバー追加

!!! example "CLIコマンドでの追加"
    ```bash
    # ファイルシステムサーバー追加
    claude mcp add filesystem

    # GitHubサーバー追加
    claude mcp add github --token YOUR_GITHUB_TOKEN

    # カスタムサーバー追加
    claude mcp add custom-server --command "node" --args "server.js"
    ```

#### Step 2: サーバー一覧確認

```bash
# 設定済みサーバー確認
claude mcp list

# サーバー詳細情報
claude mcp info filesystem
```

### 2. 設定ファイル経由

Claude Codeの設定ファイルは以下の場所にあります：

=== "macOS"
    ```bash
    ~/.claude/config.json
    ```

=== "Windows"
    ```bash
    %USERPROFILE%\.claude\config.json
    ```

=== "Linux"
    ```bash
    ~/.claude/config.json
    ```

!!! example "Claude Code設定例"
    ```json
    {
      "mcp": {
        "servers": [
          {
            "name": "filesystem",
            "command": "npx",
            "args": ["-y", "@anthropic/mcp-fs", "/workspace"]
          },
          {
            "name": "playwright",
            "command": "node",
            "args": ["./mcp-servers/playwright/index.js"],
            "env": {
              "HEADLESS": "true"
            }
          }
        ]
      }
    }
    ```

---

## OpenClaw でのMCP設定

OpenClawでMCP機能を利用する場合の設定方法です。

### 1. 設定ファイルの編集

OpenClawの設定ファイル `openclaw.json` にMCP設定を追加します。

!!! example "OpenClaw MCP設定例"
    ```json
    {
      "agents": {
        "list": [
          {
            "id": "main",
            "mcp": {
              "servers": [
                {
                  "name": "notion",
                  "command": "npx",
                  "args": ["-y", "@notionhq/mcp"]
                },
                {
                  "name": "filesystem", 
                  "command": "npx",
                  "args": ["-y", "@anthropic/mcp-fs", "/path/to/workspace"]
                }
              ]
            }
          }
        ]
      }
    }
    ```

### 2. 環境変数設定

!!! example "環境変数設定例"
    ```bash
    # .env ファイル作成
    NOTION_API_KEY=your_notion_api_key
    GITHUB_TOKEN=your_github_token

    # OpenClaw起動時に読み込み
    openclaw gateway start --env-file .env
    ```

### 3. OpenClaw専用MCPサーバー

!!! info "OpenClaw専用機能"
    ```bash
    # OpenClaw MCPサーバーの起動
    openclaw mcp serve

    # 設定例
    {
      "name": "openclaw-gateway",
      "command": "openclaw",
      "args": ["mcp", "serve"]
    }
    ```

---

## VS Code でのMCP設定

VS CodeでContinueやCopilot等の拡張機能と組み合わせてMCPを利用する方法です。

### 1. Continue拡張機能での設定

#### Step 1: Continue拡張機能のインストール

```bash
code --install-extension continue.continue
```

#### Step 2: 設定ファイル編集

`~/.continue/config.json` を編集：

!!! example "Continue MCP設定例"
    ```json
    {
      "models": [
        {
          "title": "Claude 3.5 Sonnet",
          "provider": "anthropic",
          "model": "claude-3-5-sonnet-20241022",
          "apiKey": "your_api_key"
        }
      ],
      "mcp": {
        "servers": [
          {
            "name": "codebase",
            "command": "npx",
            "args": ["-y", "@modelcontextprotocol/server-filesystem", "${workspaceFolder}"]
          }
        ]
      }
    }
    ```

### 2. GitHub Copilot Chat での利用

!!! warning "実験的機能"
    MCP対応は実験的な機能です。安定性に注意してください。

```json
{
  "github.copilot.chat.mcp.servers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-fs", "${workspaceFolder}"]
    }
  }
}
```

---

## 一般的なトラブルシューティング

### Node.js関連の問題

!!! failure "Node.jsバージョンエラー"
    ```bash
    # 現在のNode.jsバージョン確認
    node --version

    # 推奨: Node.js 18以上
    # Node.jsアップグレード（推奨方法）
    curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
    nvm install --lts
    nvm use --lts
    ```

### 権限関連の問題

!!! failure "ファイルアクセス権限エラー"
    ```bash
    # ディレクトリ権限確認
    ls -la /path/to/directory

    # 権限修正（必要に応じて）
    chmod 755 /path/to/directory
    chown $USER:$USER /path/to/directory
    ```

### ポート関連の問題

!!! failure "ポート競合エラー"
    ```bash
    # 使用中ポート確認
    lsof -i :3000

    # プロセス終了（必要に応じて）
    kill -9 <PID>
    ```

### 認証関連の問題

#### GitHub Token設定

!!! example "GitHub Personal Access Token作成"
    1. GitHub > Settings > Developer settings
    2. Personal access tokens > Tokens (classic)
    3. Generate new token (classic)
    4. 必要なスコープを選択:
        - `repo` (リポジトリアクセス)
        - `read:user` (ユーザー情報)
        - `read:org` (組織情報、必要に応じて)

#### 環境変数の設定

!!! example "セキュアな環境変数設定"
    ```bash
    # シェル設定ファイルに追加
    echo "export GITHUB_TOKEN=your_token_here" >> ~/.bashrc
    echo "export SLACK_BOT_TOKEN=xoxb-your-token" >> ~/.bashrc

    # 設定を反映
    source ~/.bashrc

    # 確認
    echo $GITHUB_TOKEN
    ```

### デバッグ機能

#### ログ出力の有効化

!!! example "デバッグログ設定"
    ```json
    {
      "mcpServers": {
        "filesystem": {
          "command": "npx",
          "args": ["-y", "@anthropic/mcp-fs", "/workspace"],
          "env": {
            "DEBUG": "mcp:*",
            "LOG_LEVEL": "debug"
          }
        }
      }
    }
    ```

#### MCP Inspector の利用

!!! tip "MCP Inspector"
    ```bash
    # MCP Inspectorで設定確認
    npx @modelcontextprotocol/inspector
    ```

---

## パフォーマンス最適化

### 1. サーバー起動時間の短縮

!!! success "最適化のコツ"
    - 不要なサーバーは無効化
    - ローカルサーバーを優先
    - キャッシュ機能を活用

### 2. メモリ使用量の削減

!!! example "軽量化設定"
    ```json
    {
      "mcpServers": {
        "lightweight-fs": {
          "command": "npx",
          "args": ["-y", "@anthropic/mcp-fs", "/workspace", "--memory-limit", "100mb"]
        }
      }
    }
    ```

### 3. 同時接続数の制限

!!! warning "リソース管理"
    ```json
    {
      "mcp": {
        "maxConcurrentServers": 3,
        "serverTimeout": 30000
      }
    }
    ```

---

## セキュリティのベストプラクティス

### 1. 最小権限の原則

!!! security "権限設定"
    - 必要最小限のディレクトリにのみアクセス許可
    - 読み取り専用モードの活用
    - 定期的な権限見直し

### 2. 機密情報の保護

!!! example "安全な設定例"
    ```json
    {
      "mcpServers": {
        "secure-fs": {
          "command": "npx",
          "args": ["-y", "@anthropic/mcp-fs", "/safe/workspace"],
          "env": {
            "READ_ONLY": "true"
          }
        }
      }
    }
    ```

### 3. 定期的なアップデート

!!! warning "セキュリティアップデート"
    ```bash
    # MCPサーバーの更新確認
    npm outdated -g

    # アップデート実行
    npm update -g @anthropic/mcp-fs
    ```

---

## まとめ

MCPの設定は、使用するツールによって方法が異なりますが、基本的な概念は共通です。まずはClaude Desktopで基本を理解し、必要に応じてより高度な設定に進むことをお勧めします。

!!! tip "設定のコツ"
    - 小さく始めて段階的に拡張
    - デバッグ機能を活用
    - セキュリティを常に意識
    - ドキュメントとコミュニティを活用