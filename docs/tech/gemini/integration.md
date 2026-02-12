# Claude Code との連携パターン

## 使い分けガイド

Gemini CLI と Claude Code はそれぞれ異なる強みを持っています。適材適所で使い分けることで、開発効率を最大化できます。

### Gemini CLI が得意な場面

| 場面 | 理由 |
|------|------|
| 大規模コードベースの分析・理解 | 大コンテキストウィンドウ + 無料枠 |
| 最新技術の調査 | Google Search グラウンディング |
| 画像/PDF/動画からのコード生成 | マルチモーダル対応が充実 |
| 気軽な質問・探索的な作業 | 無料で回数制限も十分 |
| PR レビュー自動化 | GitHub Actions 統合 |
| プロトタイピング・スケッチ実装 | 画像入力 → コード生成 |

### Claude Code が得意な場面

| 場面 | 理由 |
|------|------|
| 複雑なロジックの実装 | 推論能力が非常に高い |
| 精密なリファクタリング | コード品質が安定して高い |
| アーキテクチャ設計 | 構造的思考に優れる |
| エッジケースの洗い出し | 細部への注意力が高い |
| 既存コードの段階的改善 | 文脈理解と一貫性が強み |
| セキュリティレビュー | 脆弱性パターンの認識に強い |

!!! tip "判断基準のシンプルなルール"
    - **「広く浅く」→ Gemini CLI**（調査、概要把握、プロトタイプ）
    - **「狭く深く」→ Claude Code**（実装、リファクタリング、品質向上）
    - **コストを抑えたい → Gemini CLI**（無料枠を活用）

## 連携ワークフロー

### パターン1: Gemini で分析 → Claude Code で実装

最も基本的な連携パターンです。

```
┌─────────────────┐     ┌──────────────────┐
│   Gemini CLI    │     │   Claude Code    │
│                 │     │                  │
│ 1. コード分析    │ ──→ │ 3. 設計に基づく   │
│ 2. 改善計画作成  │     │    精密な実装     │
│                 │     │ 4. テスト作成     │
└─────────────────┘     └──────────────────┘
```

**具体例: レガシーコードのモダナイズ**

```bash
# Step 1: Gemini CLI でコードベース全体を分析
gemini
> このプロジェクト全体を読んで、技術的負債を洗い出して。
> 改善の優先順位と具体的なリファクタリング計画をまとめて。
> 結果を refactoring-plan.md に書き出して。
```

```bash
# Step 2: Claude Code で計画に基づいて実装
claude
> @refactoring-plan.md の計画に従って、優先度1の項目から
> リファクタリングを実施して。テストも書いて。
```

### パターン2: Claude Code で設計 → Gemini でレビュー

```bash
# Step 1: Claude Code で設計・実装
claude
> 新しい認証システムを設計して実装して。
> JWT + リフレッシュトークンのパターンで。
```

```bash
# Step 2: Gemini CLI でレビュー（無料で何度でも）
gemini
> src/auth/ 以下のコードをレビューして。
> セキュリティの観点で問題がないか、
> 最新のベストプラクティスに沿っているか確認して。
```

!!! note "レビューは Gemini が効率的"
    レビューは何度も繰り返す作業なので、無料枠のある Gemini CLI が向いています。Google Search でセキュリティの最新情報も参照できます。

### パターン3: 並行作業

異なるターミナルで同時に活用：

```bash
# ターミナル1: Gemini CLI でドキュメント解析
gemini
> @api-specification.pdf この仕様を読んで、
> TypeScript の型定義ファイルを生成して。types/ に保存して。

# ターミナル2: Claude Code で別の実装作業
claude
> データベースのマイグレーションスクリプトを作って。
> 既存データの整合性も考慮して。
```

## MCP 連携

Gemini CLI と Claude Code は**同じ MCP サーバー**を共有できます。

### 共通の MCP 設定

```json
// .gemini/settings.json（Gemini CLI 用）
{
  "mcpServers": {
    "github": {
      "command": "github-mcp-server",
      "args": ["stdio"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "$GITHUB_TOKEN"
      }
    },
    "database": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "DATABASE_URL": "$DATABASE_URL"
      }
    }
  }
}
```

```json
// .claude/settings.json（Claude Code 用）— 同じ MCP サーバーを設定
{
  "mcpServers": {
    "github": {
      "command": "github-mcp-server",
      "args": ["stdio"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "$GITHUB_TOKEN"
      }
    },
    "database": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "DATABASE_URL": "$DATABASE_URL"
      }
    }
  }
}
```

!!! tip "MCP サーバーの統一管理"
    同じプロジェクトで両ツールを使う場合、MCP サーバーの設定を統一することで、どちらのツールからも同じ外部リソースにアクセスできます。

## コスト最適化戦略

### 無料枠ファースト戦略

```
日常的な作業フロー:

1. まず Gemini CLI（無料）で作業開始
   - コード分析、調査、プロトタイプ
   - 簡単なコード生成、ドキュメント作成

2. 品質が重要な場面で Claude Code に切り替え
   - 本番コードの実装
   - 複雑なアルゴリズム
   - セキュリティクリティカルな部分
```

### コスト配分の目安

| 作業 | ツール | コスト |
|------|--------|--------|
| 朝の調査・分析 | Gemini CLI | 無料 |
| コードレビュー | Gemini CLI | 無料 |
| ドキュメント作成 | Gemini CLI | 無料 |
| 本番コード実装 | Claude Code | 有料 |
| 複雑なバグ修正 | Claude Code | 有料 |
| アーキテクチャ設計 | Claude Code | 有料 |

!!! warning "Gemini 無料枠の制限に注意"
    1日1,000リクエストを超えるとレート制限がかかります。大量のリクエストが必要な場合は API キーの取得を検討してください。

## 実践的なワークフロー例

### 新機能開発の一連の流れ

```bash
# 1. Gemini CLI: 要件分析と設計（無料）
gemini
> @requirements.md この要件を分析して、技術的な設計書を作って。
> design.md に保存して。

# 2. Claude Code: 実装（有料・高品質）
claude
> @design.md に基づいて実装して。TDD で進めて。

# 3. Gemini CLI: レビューと改善提案（無料）
gemini
> 今回の実装をレビューして。改善点を一覧にして。

# 4. Claude Code: 指摘事項の修正（有料・高品質）
claude
> @review-feedback.md の指摘事項を修正して。

# 5. Gemini CLI: ドキュメント・PR 作成（無料）
gemini
> この変更の PR description を書いて。
> 技術ドキュメントも更新して。
```

## コンテキストファイルの共存

プロジェクトで両ツールを使う場合のディレクトリ構成：

```
project/
├── CLAUDE.md          # Claude Code 用コンテキスト
├── GEMINI.md          # Gemini CLI 用コンテキスト
├── .claude/
│   └── settings.json  # Claude Code 設定
├── .gemini/
│   └── settings.json  # Gemini CLI 設定（MCP 等）
└── src/
```

!!! tip "コンテキストの共通化"
    `CLAUDE.md` と `GEMINI.md` の内容は大部分が共通になるはずです。共通部分は別ファイルにして、各ツールのコンテキストファイルから参照する方法もあります。
    ```markdown
    <!-- GEMINI.md -->
    このプロジェクトの情報は PROJECT.md を参照してください。
    Gemini 固有の指示: ...
    ```
