# おすすめMCPサーバー一覧

!!! tip "サーバー選びのポイント"
    2025年現在、**7,260以上**のMCPサーバーが利用可能です。公式サーバーを中心に、コミュニティサーバーも活用してワークフローを効率化しましょう。

## 公式 vs コミュニティサーバー

### 公式サーバー（推奨）
- **Anthropic公式**メンテナンス
- **安定性・セキュリティ**が保証
- **長期サポート**が期待できる

### コミュニティサーバー
- **豊富な選択肢**（7,000以上）
- **ニッチな用途**にも対応
- **品質にばらつき**があるため注意

## カテゴリ別おすすめサーバー

### 📁 ファイルシステム

#### **@anthropic/mcp-fs** (公式)
ローカルファイルシステムへの基本的なアクセス機能を提供。

!!! example "セットアップ例"
    ```bash
    # インストール
    npx @anthropic/mcp-fs

    # 設定例（Claude Desktop）
    {
      "name": "filesystem",
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-fs", "/path/to/allowed/directory"]
    }
    ```

**主な機能:**
- ファイル読み書き
- ディレクトリ操作
- ファイル検索

---

### 🗄️ データベース

#### **PostgreSQL MCP** (コミュニティ)
PostgreSQLデータベースとの連携。

!!! example "セットアップ例"
    ```bash
    # 環境変数設定
    export POSTGRES_URL="postgresql://user:pass@localhost:5432/dbname"

    # 設定例
    {
      "name": "postgresql",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"]
    }
    ```

**主な機能:**
- 自然言語でのSQL生成
- テーブル構造の自動取得
- クエリ結果の解析

#### **SQLite MCP** (コミュニティ)
軽量データベースSQLiteとの連携。

!!! example "設定例"
    ```json
    {
      "name": "sqlite",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-sqlite", "/path/to/database.db"]
    }
    ```

---

### 🌐 Web・ブラウザ

#### **Playwright MCP** (コミュニティ)
ブラウザ自動操作のための強力なツール。

!!! success "おすすめポイント"
    - **高精度**なWeb操作
    - **複雑なフォーム**入力対応
    - **スクリーンショット**機能

!!! example "セットアップ例"
    ```bash
    # Playwrightブラウザのインストール
    npx playwright install

    # 設定例
    {
      "name": "playwright",
      "command": "node",
      "args": ["/path/to/playwright-mcp-server/index.js"]
    }
    ```

**活用例:**
- Webフォームの自動入力
- データの自動収集
- 定期的なサイト監視

#### **Browserbase MCP** (公式)
クラウドベースのブラウザ自動化サービス。

!!! info "特徴"
    - **クラウド実行**でリソース節約
    - **大規模スクレイピング**に対応
    - **プロキシ・キャプチャ**対応

---

### 🔧 開発ツール

#### **GitHub MCP** (コミュニティ)
GitHubとの包括的な連携。

!!! example "セットアップ例"
    ```bash
    # GitHub Personal Access Token設定
    export GITHUB_PERSONAL_ACCESS_TOKEN="your_token_here"

    # 設定例
    {
      "name": "github",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"]
    }
    ```

**主な機能:**
- リポジトリ操作
- プルリクエスト管理
- イシュー作成・更新
- コードレビュー自動化

#### **Git MCP** (公式)
ローカルGitリポジトリ操作。

!!! example "設定例"
    ```json
    {
      "name": "git",
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-git", "/path/to/repo"]
    }
    ```

---

### 💬 コミュニケーション

#### **Slack MCP** (コミュニティ)
Slackワークスペースとの連携。

!!! example "セットアップ例"
    ```bash
    # Slack Bot Tokenを設定
    export SLACK_BOT_TOKEN="xoxb-your-token"

    # 設定例
    {
      "name": "slack",
      "command": "npx",
      "args": ["-y", "@slack-mcp/server"]
    }
    ```

**活用例:**
- 自動通知送信
- チャンネル監視
- ワークフロー自動化

#### **Discord MCP** (コミュニティ)
Discordサーバーとの連携。

!!! warning "注意"
    Discord利用規約を遵守し、適切なBot設定を行ってください。

---

### ☁️ クラウドサービス

#### **AWS MCP** (コミュニティ)
Amazon Web Servicesとの連携。

!!! example "主なサービス対応"
    - **S3** - ファイルストレージ
    - **EC2** - インスタンス管理
    - **Lambda** - サーバーレス関数
    - **DynamoDB** - NoSQLデータベース

#### **Google Cloud MCP** (コミュニティ)
Google Cloud Platformとの連携。

#### **Azure MCP** (コミュニティ)
Microsoft Azureとの連携。

---

### 📊 データ分析

#### **Pandas MCP** (コミュニティ)
Pythonデータ分析ライブラリとの連携。

!!! success "データ分析の自動化"
    ```python
    # 自然言語でデータ操作
    "売上データの月別トレンドを計算して"
    "顧客セグメントごとの統計を出して"
    ```

#### **Neo4j MCP** (コミュニティ)
グラフデータベースとの連携。

**活用例:**
- 関係性の可視化
- ネットワーク分析
- 推薦システム構築

---

### 🏥 特殊分野

#### **FHIR MCP** (コミュニティ)
医療データ標準FHIRとの連携。

!!! info "医療分野での活用"
    - 患者履歴の自然言語検索
    - 医療データの統合分析
    - 診療記録の自動整理

---

## サーバー選択のガイドライン

### 1. **信頼性重視** → 公式サーバー優先

!!! success "公式サーバーの見分け方"
    - `@anthropic/*` パッケージ
    - GitHub: `modelcontextprotocol` organization
    - 公式ドキュメントでの紹介

### 2. **機能重視** → コミュニティサーバー活用

!!! warning "コミュニティサーバー選択時の注意点"
    - **GitHubスター数**をチェック
    - **最終更新日**を確認
    - **メンテナー**の信頼性を確認
    - **セキュリティリスク**を評価

### 3. **パフォーマンス重視** → 軽量サーバー選択

- SQLiteよりPostgreSQL
- ローカルよりクラウド（用途次第）

## 設定のベストプラクティス

### 環境変数管理

!!! example "セキュアな設定例"
    ```bash
    # .env ファイル
    GITHUB_TOKEN=your_github_token
    SLACK_BOT_TOKEN=your_slack_token
    DATABASE_URL=postgresql://user:pass@localhost:5432/db

    # Claude Desktop設定で参照
    {
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
    ```

### 複数サーバーの組み合わせ

!!! tip "効果的な組み合わせ例"
    ```json
    {
      "mcpServers": {
        "filesystem": { "command": "npx", "args": ["-y", "@anthropic/mcp-fs", "/workspace"] },
        "github": { "command": "npx", "args": ["-y", "@modelcontextprotocol/server-github"] },
        "sqlite": { "command": "npx", "args": ["-y", "@modelcontextprotocol/server-sqlite", "./data.db"] },
        "playwright": { "command": "node", "args": ["./playwright-server.js"] }
      }
    }
    ```

## トラブルシューティング

### よくある問題

#### サーバーが起動しない
!!! failure "原因と対処法"
    - **Node.js バージョン**: 18以上を推奨
    - **権限問題**: ファイルアクセス権限を確認
    - **ポート競合**: 他のサービスとの競合チェック

#### 認証エラー
!!! failure "対処法"
    - 環境変数の設定確認
    - トークンの有効期限確認
    - API制限の確認

## まとめ

MCPサーバーの選択は、**用途と信頼性のバランス**が重要です。まずは公式サーバーで基本を理解し、必要に応じてコミュニティサーバーを追加することをお勧めします。

---

!!! info "参考リンク"
    - [Awesome MCP Servers](https://github.com/punkpeye/awesome-mcp-servers)
    - [公式サーバー一覧](https://github.com/modelcontextprotocol/servers)
    - [MCP Server Inspector](https://glama.ai/mcp/inspector)