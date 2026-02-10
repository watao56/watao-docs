# CLAUDE.md / AGENTS.md / Skills ガイド

Claude Code の動作をカスタマイズする設定ファイル群の書き方と管理方法。

## CLAUDE.md

### CLAUDE.md とは

Claude Code が**毎セッション開始時に自動で読み込む**設定ファイル。プロジェクトのルール、コーディング規約、ワークフローを永続的に伝える手段。

### 配置場所と優先順位

| 場所 | パス | スコープ |
|------|------|----------|
| ホームディレクトリ | `~/.claude/CLAUDE.md` | 全セッション共通 |
| プロジェクトルート | `./CLAUDE.md` | プロジェクト全体（git で共有） |
| プロジェクトルート (ローカル) | `./CLAUDE.local.md` | 個人用（.gitignore 推奨） |
| 親ディレクトリ | `../CLAUDE.md` | モノレポ用 |
| 子ディレクトリ | `./src/CLAUDE.md` | そのディレクトリのファイル作業時に読み込み |

すべての階層の CLAUDE.md が**マージされて**読み込まれる。

### 書き方のベストプラクティス

#### 基本構造

```markdown
# プロジェクト概要
TypeScript + React のフロントエンドアプリケーション

# コードスタイル
- ES Modules (import/export) を使う。CommonJS (require) は使わない
- 可能な限り import を分割代入する（例: import { foo } from 'bar'）
- コンポーネントは関数コンポーネント + hooks で書く

# テスト
- テストランナー: vitest
- 個別テストを優先。全テスト実行は避ける
- `npm run test -- --run path/to/test.ts` で単体実行

# ワークフロー
- コード変更後は必ず型チェック: `npm run typecheck`
- コミットメッセージは Conventional Commits に従う
- PR を出す前に lint + test を通す

# 参照ファイル
@README.md でプロジェクト概要を確認
@docs/architecture.md でアーキテクチャを確認
```

#### 含めるべきもの

| ✅ 含める | 理由 |
|----------|------|
| ビルド・テストコマンド | Claude が推測できない |
| コードから読み取れない規約 | デフォルトと異なるルール |
| リポジトリ固有のワークフロー | ブランチ命名、PR 手順 |
| 環境の癖 | 必須の環境変数、特殊な設定 |
| よくあるハマりポイント | 非自明な挙動 |

#### 含めないもの

| ❌ 含めない | 理由 |
|------------|------|
| 言語の標準規約 | Claude は既に知っている |
| API の詳細ドキュメント | リンクで十分 |
| 頻繁に変わる情報 | すぐ古くなる |
| ファイルごとの説明 | コードを読めばわかる |
| 「きれいなコードを書け」 | 自明すぎる |

#### 強調のテクニック

重要なルールには強調を付けると遵守率が上がる:

```markdown
# 重要なルール
IMPORTANT: テストは必ず個別実行。全テストスイートは絶対に実行しない
YOU MUST: コミット前に typecheck を通す
NEVER: node_modules を直接編集しない
```

### /init コマンドで生成する

```bash
# Claude Code 内で実行
/init
```

プロジェクトのビルドシステム、テストフレームワーク、コードパターンを自動検出して CLAUDE.md の土台を生成する。生成後に不要な記述を削除・調整する。

### @ によるファイル参照

CLAUDE.md 内で `@path/to/file` と書くと、そのファイルの内容が読み込まれる:

```markdown
プロジェクト概要: @README.md
Git ワークフロー: @docs/git-instructions.md
個人設定: @~/.claude/my-project-instructions.md
```

### メンテナンスの習慣

- **定期的に見直す**: Claude が不要な行動をしたら CLAUDE.md を確認
- **肥大化を防ぐ**: 150 行を超えたら分割を検討
- **git にコミット**: チーム全体で共有、レビュー対象にする
- **動作確認**: ルールを変更したら Claude の挙動が変わるか観察

## Skills（スキル）

### Skills とは

Claude Code の知識・能力を拡張するモジュラーな仕組み。関連する作業で**自動的に読み込まれる**か、`/skill-name` で**手動起動**できる。

[Agent Skills](https://agentskills.io) オープンスタンダードに準拠。

### ディレクトリ構成

```
.claude/skills/
└── my-skill/
    ├── SKILL.md          # メイン（必須）
    ├── template.md       # テンプレート（任意）
    ├── examples/
    │   └── sample.md     # 出力例（任意）
    └── scripts/
        └── validate.sh   # 実行スクリプト（任意）
```

### 配置場所

| 場所 | パス | スコープ |
|------|------|----------|
| 個人用 | `~/.claude/skills/<name>/SKILL.md` | 全プロジェクト |
| プロジェクト用 | `.claude/skills/<name>/SKILL.md` | そのプロジェクト |
| プラグイン | `<plugin>/skills/<name>/SKILL.md` | プラグイン有効時 |

### SKILL.md の書き方

#### リファレンス型（自動適用される知識）

```yaml
---
name: api-conventions
description: REST API 設計規約
---
# API 設計規約

- URL パスは kebab-case
- JSON プロパティは camelCase
- リストエンドポイントにはページネーション必須
- URL パスにバージョンを含める（/v1/, /v2/）
```

#### タスク型（手動起動するワークフロー）

```yaml
---
name: fix-issue
description: GitHub Issue を修正する
disable-model-invocation: true
---
GitHub Issue $ARGUMENTS を分析して修正する。

1. `gh issue view $0` で Issue の詳細を取得
2. 問題を理解する
3. 関連ファイルをコードベースから検索
4. 修正を実装
5. テストを書いて実行
6. lint・型チェックを通す
7. わかりやすいコミットメッセージで commit
8. push して PR を作成
```

使い方: `/fix-issue 1234`

### フロントマター設定

| フィールド | 説明 |
|-----------|------|
| `name` | スキル名（小文字・数字・ハイフン） |
| `description` | 説明（Claude が自動適用の判断に使用） |
| `disable-model-invocation` | `true` で自動適用を無効化（手動専用に） |
| `allowed-tools` | 許可するツール（例: `Read, Grep, Bash`） |
| `context` | `fork` でサブエージェント実行 |
| `model` | 使用モデルの指定 |

### 変数の利用

| 変数 | 説明 |
|------|------|
| `$ARGUMENTS` | `/skill-name` に渡された全引数 |
| `$ARGUMENTS[0]`, `$0` | 第1引数 |
| `$ARGUMENTS[1]`, `$1` | 第2引数 |
| `${CLAUDE_SESSION_ID}` | セッション ID |

## サブエージェント

### サブエージェントとは

独自のコンテキストとツールセットで動作する**専門化されたエージェント**。メインの会話を汚さずに特定タスクを実行できる。

### 定義方法

`.claude/agents/` に Markdown ファイルを配置:

```yaml
---
name: security-reviewer
description: コードのセキュリティレビュー
tools: Read, Grep, Glob, Bash
model: opus
---
シニアセキュリティエンジニアとしてコードをレビューする:

- インジェクション脆弱性（SQL, XSS, コマンド）
- 認証・認可の欠陥
- コード内のシークレットや認証情報
- 安全でないデータ処理

具体的な行番号と修正案を提示する。
```

使い方: 「セキュリティレビューのサブエージェントを使ってこのコードをレビューして」

## プロジェクト別の設定例

### TypeScript + React プロジェクト

```markdown
# CLAUDE.md

## ビルド・テスト
- `pnpm dev` — 開発サーバー
- `pnpm test -- --run <file>` — 個別テスト
- `pnpm typecheck` — 型チェック
- `pnpm lint` — ESLint

## コードスタイル
- 関数コンポーネント + hooks のみ
- styled-components は使わない（Tailwind CSS）
- Server Components がデフォルト、'use client' は必要な場合のみ

## アーキテクチャ
- app/ — Next.js App Router
- components/ — 再利用コンポーネント
- lib/ — ユーティリティ
- IMPORTANT: API Route は app/api/ に配置
```

### Python + FastAPI プロジェクト

```markdown
# CLAUDE.md

## 環境
- Python 3.12+, uv でパッケージ管理
- `uv run pytest tests/<file>` — テスト実行
- `uv run ruff check .` — lint
- `uv run mypy .` — 型チェック

## コードスタイル
- 型ヒント必須
- Pydantic v2 モデル使用
- async/await を優先

## NEVER
- requirements.txt を直接編集しない（uv で管理）
- グローバル変数は使わない
```

### Rust プロジェクト

```markdown
# CLAUDE.md

## ビルド・テスト
- `cargo build` — ビルド
- `cargo test <test_name>` — 個別テスト
- `cargo clippy` — lint
- `cargo fmt --check` — フォーマットチェック

## スタイル
- unwrap() は禁止。Result/Option を適切に処理
- pub は最小限に
- エラー型は thiserror で定義
```
