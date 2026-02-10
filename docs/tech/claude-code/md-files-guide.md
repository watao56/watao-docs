# CLAUDE.md メモリ管理ガイド

Claude Code の動作をカスタマイズするメモリシステムの全体像。CLAUDE.md、Auto Memory、Rules、Skills、サブエージェントの書き方と管理方法を、Anthropic 公式ドキュメントに基づいて解説する。

## メモリの全体像

Claude Code には**2種類のメモリ**がセッションをまたいで永続化する（[公式ドキュメント](https://code.claude.com/docs/en/memory)）：

- **CLAUDE.md ファイル**: ユーザーが書いて管理する指示・ルール・プリファレンス
- **Auto Memory**: Claude が自動的に保存する有用なコンテキスト（プロジェクトのパターン、コマンド、プリファレンスなど）

両方ともセッション開始時に Claude のコンテキストに読み込まれる。

### メモリの種類と配置場所

| メモリタイプ | 場所 | 用途 | 共有範囲 |
|------------|------|------|---------|
| **管理ポリシー** | macOS: `/Library/Application Support/ClaudeCode/CLAUDE.md` / Linux: `/etc/claude-code/CLAUDE.md` | 組織全体の指示（IT/DevOps 管理） | 組織の全ユーザー |
| **プロジェクトメモリ** | `./CLAUDE.md` または `./.claude/CLAUDE.md` | チーム共有のプロジェクト指示 | チームメンバー（ソース管理経由） |
| **プロジェクトルール** | `./.claude/rules/*.md` | モジュラーなトピック別指示 | チームメンバー（ソース管理経由） |
| **ユーザーメモリ** | `~/.claude/CLAUDE.md` | 全プロジェクト共通の個人設定 | 自分のみ |
| **プロジェクトメモリ（ローカル）** | `./CLAUDE.local.md` | 個人的なプロジェクト固有設定 | 自分のみ |
| **Auto Memory** | `~/.claude/projects/<project>/memory/` | Claude 自身のメモ・学習 | 自分のみ（プロジェクト別） |

## CLAUDE.md

### CLAUDE.md とは

Claude Code が**毎セッション開始時に自動で読み込む**設定ファイル。プロジェクトのルール、コーディング規約、ワークフローを永続的に伝える手段。

> CLAUDE.md はコードのように扱う：問題が起きたらレビューし、定期的に整理し、変更後は Claude の挙動が変わるか観察する。
>
> — [公式ベストプラクティス](https://code.claude.com/docs/en/best-practices)

### 書き方のベストプラクティス

**簡潔に保つことが最重要。** 各行について「この指示を削除したら Claude はミスするか？」と問い、答えが No なら削除する。肥大化した CLAUDE.md は、重要な指示が埋もれて無視される原因になる。

```markdown
# コードスタイル
- ES Modules (import/export) を使う。CommonJS (require) は使わない
- 可能な限り import を分割代入する（例: import { foo } from 'bar'）

# ワークフロー
- コード変更後は必ず型チェックを実行すること
- テストは個別実行を優先。全テストスイートは実行しない
```

#### 含めるべきもの vs 含めないもの

| ✅ 含める | ❌ 含めない |
|----------|-----------|
| Claude が推測できない Bash コマンド | コードを読めばわかること |
| デフォルトと異なるコードスタイルルール | 言語の標準規約（Claude は既に知っている） |
| テスト手順と推奨テストランナー | 詳細な API ドキュメント（リンクで十分） |
| リポジトリの作法（ブランチ命名、PR 手順） | 頻繁に変わる情報 |
| アーキテクチャ上の意思決定 | ファイルごとの説明 |
| 開発環境の癖（必須環境変数など） | 「きれいなコードを書け」のような自明なこと |
| よくあるハマりポイント | 長い説明やチュートリアル |

#### 強調のテクニック

重要なルールには強調を付けると遵守率が上がる：

```markdown
IMPORTANT: テストは必ず個別実行。全テストスイートは絶対に実行しない
YOU MUST: コミット前に typecheck を通す
NEVER: node_modules を直接編集しない
```

!!! tip "Claude が指示を無視する場合"
    CLAUDE.md にルールがあるのに Claude が従わない場合、ファイルが長すぎてルールが埋もれている可能性が高い。また、Claude が CLAUDE.md に答えがある質問をしてくる場合、表現が曖昧かもしれない。

### /init コマンドで生成する

```bash
# Claude Code 内で実行
/init
```

プロジェクトのビルドシステム、テストフレームワーク、コードパターンを自動検出して CLAUDE.md の土台を生成する。生成後に不要な記述を削除・調整する。

### @ によるファイルインポート

CLAUDE.md 内で `@path/to/file` と書くと、そのファイルの内容がインポートされる。相対パスはインポート元ファイルからの相対パスで解決される。

```markdown
プロジェクト概要: @README.md で確認
npm コマンド: @package.json で確認

# 追加の指示
- Git ワークフロー: @docs/git-instructions.md
- 個人設定: @~/.claude/my-project-instructions.md
```

インポートされたファイルはさらに再帰的にインポートでき、最大深度は 5 ホップ。コードスパンやコードブロック内の `@` はインポートとして扱われない。

### 配置場所と読み込み

- **ホームディレクトリ** (`~/.claude/CLAUDE.md`): 全セッション共通
- **プロジェクトルート** (`./CLAUDE.md`): git にコミットしてチーム共有
- **プロジェクトルート・ローカル** (`./CLAUDE.local.md`): 個人用（.gitignore 推奨）
- **親ディレクトリ**: モノレポで `root/CLAUDE.md` と `root/foo/CLAUDE.md` の両方が読み込まれる
- **子ディレクトリ**: Claude がそのディレクトリのファイルを操作する時にオンデマンドで読み込まれる

すべての階層の CLAUDE.md が**マージされて**読み込まれる。より具体的な指示が広い指示より優先される。

## モジュラールール（.claude/rules/）

大規模プロジェクトでは、1つの CLAUDE.md ではなく `.claude/rules/` ディレクトリに複数のルールファイルを配置できる。

### 基本構成

```
your-project/
├── .claude/
│   ├── CLAUDE.md           # メインのプロジェクト指示
│   └── rules/
│       ├── code-style.md   # コードスタイル
│       ├── testing.md      # テスト規約
│       └── security.md     # セキュリティ要件
```

`.claude/rules/` 内の全 `.md` ファイルはプロジェクトメモリとして自動的に読み込まれる。

### パス固有のルール

YAML フロントマターの `paths` フィールドで、特定のファイルにスコープを限定できる：

```markdown
---
paths:
  - "src/api/**/*.ts"
---

# API 開発ルール

- 全 API エンドポイントにバリデーションを含める
- 標準エラーレスポンス形式を使用する
- OpenAPI ドキュメントコメントを含める
```

`paths` フィールドなしのルールは無条件に全ファイルに適用される。glob パターン、ブレース展開（`*.{ts,tsx}`）、サブディレクトリ配置にも対応。

### ユーザーレベルルール

`~/.claude/rules/` に個人用ルールを配置できる。プロジェクトルールより低い優先度で読み込まれる。

## Auto Memory

### Auto Memory とは

Claude が作業中に**自動的にメモを保存する**仕組み。CLAUDE.md がユーザーの書いた指示なのに対し、Auto Memory は Claude 自身が発見したことに基づくメモ。

### 保存される内容

- **プロジェクトパターン**: ビルドコマンド、テスト規約、コードスタイル
- **デバッグの知見**: トリッキーな問題の解決策、よくあるエラーの原因
- **アーキテクチャメモ**: 重要なファイル、モジュールの関係、重要な抽象
- **ユーザーの好み**: コミュニケーションスタイル、ワークフロー、ツール選択

### ストレージ構造

```
~/.claude/projects/<project>/memory/
├── MEMORY.md              # 簡潔なインデックス（最初の200行がセッション開始時に読み込まれる）
├── debugging.md           # デバッグパターンの詳細メモ
├── api-conventions.md     # API 設計の決定事項
└── ...                    # その他のトピックファイル
```

- `MEMORY.md` の最初の 200 行がセッション開始時にシステムプロンプトに読み込まれる
- トピックファイル（`debugging.md` など）は起動時には読み込まれず、必要な時にオンデマンドで読み込まれる
- Claude に特定のことを保存させたい場合は直接指示：「pnpm を使うことを覚えて」「API テストにはローカル Redis が必要なことをメモリに保存して」

### Auto Memory の管理

```bash
# Claude Code 内で /memory コマンドでメモリファイルを開く
/memory

# 環境変数で制御
export CLAUDE_CODE_DISABLE_AUTO_MEMORY=1  # 無効化
export CLAUDE_CODE_DISABLE_AUTO_MEMORY=0  # 強制有効化
```

## Skills（スキル）

### Skills とは

Claude Code の知識・能力を拡張するモジュラーな仕組み。[Agent Skills](https://agentskills.io) オープンスタンダードに準拠。関連する作業で**自動的に読み込まれる**か、`/skill-name` で**手動起動**できる。

CLAUDE.md は毎セッション読み込まれるが、Skills は**必要な時だけ**読み込まれるため、コンテキストを膨らませない。

### スキルの配置場所

| 場所 | パス | 適用範囲 |
|------|------|---------|
| 企業管理 | マネージド設定 | 組織の全ユーザー |
| 個人用 | `~/.claude/skills/<name>/SKILL.md` | 全プロジェクト |
| プロジェクト用 | `.claude/skills/<name>/SKILL.md` | そのプロジェクトのみ |
| プラグイン | `<plugin>/skills/<name>/SKILL.md` | プラグイン有効時 |

同名のスキルは高優先度の場所が勝つ（企業 > 個人 > プロジェクト）。

### スキルの種類

#### リファレンス型（自動適用される知識）

```yaml
---
name: api-conventions
description: REST API 設計規約
---
# API 規約
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

| フィールド | 必須 | 説明 |
|-----------|------|------|
| `name` | No | スキル名（小文字・数字・ハイフン、最大64文字） |
| `description` | 推奨 | 説明（Claude が自動適用の判断に使用） |
| `argument-hint` | No | 引数のヒント（例: `[issue-number]`） |
| `disable-model-invocation` | No | `true` で自動適用を無効化（手動専用に） |
| `user-invocable` | No | `false` で /メニューから非表示（バックグラウンド知識用） |
| `allowed-tools` | No | 許可するツール（例: `Read, Grep, Glob`） |
| `model` | No | 使用モデルの指定 |
| `context` | No | `fork` でサブエージェント実行 |
| `agent` | No | `context: fork` 時のサブエージェントタイプ |
| `hooks` | No | スキルのライフサイクルにスコープされたフック |

### 変数の利用

| 変数 | 説明 |
|------|------|
| `$ARGUMENTS` | `/skill-name` に渡された全引数 |
| `$ARGUMENTS[0]`, `$0` | 第1引数 |
| `$ARGUMENTS[1]`, `$1` | 第2引数 |
| `${CLAUDE_SESSION_ID}` | セッション ID |

### 動的コンテキスト注入

`` !`command` `` 構文でシェルコマンドの出力をスキル内容に埋め込める：

```yaml
---
name: pr-summary
description: PR の変更を要約する
context: fork
agent: Explore
allowed-tools: Bash(gh *)
---

## Pull request context
- PR diff: !`gh pr diff`
- PR コメント: !`gh pr view --comments`
- 変更ファイル: !`gh pr diff --name-only`

## タスク
この Pull Request を要約して...
```

コマンドは Claude が見る前に実行され、出力で置換される。

### 呼び出し制御

| フロントマター | ユーザーが呼び出し | Claude が呼び出し | 用途 |
|--------------|----------------|----------------|------|
| （デフォルト） | ✅ | ✅ | 通常のスキル |
| `disable-model-invocation: true` | ✅ | ❌ | デプロイなど副作用のあるワークフロー |
| `user-invocable: false` | ❌ | ✅ | バックグラウンド知識 |

## プロジェクト別 CLAUDE.md の設定例

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

### メンテナンスの習慣

- **定期的に見直す**: Claude が不要な行動をしたら CLAUDE.md を確認
- **肥大化を防ぐ**: 長くなりすぎたら `.claude/rules/` に分割
- **git にコミット**: チーム全体で共有、レビュー対象にする
- **動作確認**: ルールを変更したら Claude の挙動が変わるか観察
- **Hook に変換**: Claude が既にルール通りに動いているなら削除するか、確実に実行させたい場合は Hook に変換する
