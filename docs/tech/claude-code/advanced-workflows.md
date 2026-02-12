# 上級ワークフロー

Claude Code を使いこなすための上級テクニック。サブエージェントの戦略的活用、CLAUDE.md の最適化、大規模プロジェクト運用、Git 連携、カスタムコマンドまで。

---

## サブエージェント活用パターン

### サブエージェントの戦略的な使い分け

サブエージェントは独立したコンテキストウィンドウで動作する。メインのコンテキストを消費せずにタスクを実行できるため、**コンテキスト管理の最重要ツール**。

#### 調査タスクの委任

メインのコンテキストを汚さず、結果の要約だけ受け取る：

```
サブエージェントを使って以下を調査して：
1. このプロジェクトで使っている認証ライブラリのバージョンと設定
2. 認証フローの全体像（どのファイルが関わっているか）
3. 既知の問題や TODO コメント

結果を要約して報告して。
```

#### 並列実装パターン

複数のサブエージェントに独立したタスクを割り当てる：

```
以下の3つのタスクをサブエージェントで並列に実行して：

1. src/components/Header.tsx のレスポンシブ対応
2. src/components/Footer.tsx の新デザイン実装
3. src/components/Sidebar.tsx のナビゲーション追加

各コンポーネントは独立しているので並列で進めて問題ない。
完了後にビルドして統合テストを実行して。
```

!!! tip "並列実行の判断基準"
    サブエージェントの並列実行が有効なケース：

    - 各タスクが**独立したファイル**を編集する
    - タスク間に**依存関係がない**
    - 各タスクが**中程度の複雑さ**（単純すぎるとオーバーヘッドが大きい）

    逆に、同じファイルを編集するタスクは並列化しない。コンフリクトの原因になる。

### カスタムサブエージェントの設計パターン

`.claude/agents/` にマークダウンファイルを配置してカスタムエージェントを定義できる。

#### テストエージェント

```markdown
# .claude/agents/test-writer.md
---
name: test-writer
description: テストコードの作成と改善
tools: Read, Write, Edit, Bash, Glob, Grep
model: sonnet
---

テスト作成のスペシャリストとして動作する。

## 原則
- 既存テストのスタイルとフレームワークに従う
- エッジケース・異常系を重視する
- テストは独立して実行可能にする（テスト間の依存を排除）
- モックは最小限に。可能な限り実際の依存を使う

## 手順
1. 対象コードを読んで動作を理解する
2. 既存テストのパターンを確認する
3. テストケースのリストを作成する
4. テストコードを実装する
5. テストを実行して全てパスすることを確認する
6. カバレッジを確認する
```

#### セキュリティレビューエージェント

```markdown
# .claude/agents/security-audit.md
---
name: security-audit
description: セキュリティ脆弱性のスキャン
tools: Read, Glob, Grep, Bash
model: opus
---

セキュリティ監査の専門家として動作する。

## チェック項目
- インジェクション脆弱性（SQL, NoSQL, OS コマンド, LDAP）
- 認証・認可の不備
- クロスサイトスクリプティング（XSS）
- 安全でないデシリアライゼーション
- 機密情報の露出（API キー、パスワード、トークン）
- 依存パッケージの脆弱性
- 不適切なエラーハンドリング（スタックトレースの露出）

## 出力形式
各問題について以下を報告：
- ファイルパスと行番号
- 脆弱性の種類と深刻度（Critical/High/Medium/Low）
- 問題の説明
- 修正案
```

#### ドキュメントエージェント

```markdown
# .claude/agents/doc-writer.md
---
name: doc-writer
description: ドキュメントの生成と更新
tools: Read, Write, Edit, Glob, Grep
model: sonnet
---

テクニカルライターとして動作する。

## 原則
- 正確性を最優先にする（コードを読んで実際の動作を確認）
- 簡潔だが十分な情報を含める
- コード例を必ず含める
- 対象読者のスキルレベルに合わせる
```

### Agent Teams（マルチエージェント）

Agent Teams は複数のエージェントが協調して作業する仕組み。`Shift+Tab` で Delegate Mode に切り替えて使う。

```markdown
# .claude/agents/lead.md
---
name: lead
description: タスクの分解と各エージェントへの委任
tools: Read, Bash, Glob, Grep, subagent
model: opus
---

プロジェクトリードとして動作する。
タスクを分解し、適切なエージェントに委任する。
進捗を管理し、最終的な統合を行う。
```

!!! note "Agent Teams のコスト"
    Agent Teams は各エージェントが独立したコンテキストウィンドウを使うため、トークン消費が大幅に増える。ルーチンタスクには単一セッションの方がコスト効率が良い。

---

## CLAUDE.md の効果的な書き方

### 階層構造

CLAUDE.md は複数の場所に配置でき、それぞれスコープが異なる：

| 配置場所 | スコープ | 用途 |
|---------|---------|------|
| `~/.claude/CLAUDE.md` | 全プロジェクト共通 | 個人的なスタイル、共通ツール設定 |
| `./CLAUDE.md` | プロジェクトルート | チーム共有のルール、ビルドコマンド |
| `./CLAUDE.local.md` | プロジェクトルート（個人） | .gitignore して個人設定 |
| `./src/CLAUDE.md` | サブディレクトリ | ディレクトリ固有のルール |

#### プロジェクトルートの CLAUDE.md 例

```markdown
# プロジェクト概要
ECサイトのバックエンドAPI（Express + TypeScript + Prisma）

# ビルド・テストコマンド
- ビルド: `npm run build`
- テスト全体: `npm test`
- テスト単体: `npm test -- --grep "テスト名"`
- lint: `npm run lint`
- 型チェック: `npm run typecheck`
- DB マイグレーション: `npx prisma migrate dev`

# コーディング規約
- ESM (import/export) を使う。require は使わない
- 関数は arrow function で統一
- エラーハンドリングは Result 型パターンを使う（@src/lib/result.ts 参照）
- ログは pino を使う（console.log 禁止）

# Git ルール
- Conventional Commits 形式でコミット
- ブランチ名: feature/xxx, fix/xxx, chore/xxx
- main ブランチへの直接プッシュ禁止

# テストルール
- 1つのテストファイルずつ実行して確認すること（全テスト実行は遅い）
- モックは最小限に
- DB テストは test 用データベースを使う
```

### 効果的な CLAUDE.md の原則

#### ✅ 含めるべきもの

- Claude が推測できないビルド・テストコマンド
- プロジェクト固有のコーディング規約
- デフォルトと異なるスタイルルール
- 環境の癖（特殊な環境変数など）
- よくあるミスへの注意書き

#### ❌ 含めるべきでないもの

- コードを読めばわかること
- 言語の標準的な規約（Claude は既に知っている）
- 頻繁に変わる情報
- 長い API ドキュメント（リンクで十分）
- 自明なこと（「きれいなコードを書け」など）

!!! warning "長すぎる CLAUDE.md は逆効果"
    CLAUDE.md が長すぎると、Claude が重要な指示を見落とす原因になる。各行について「これを削除したら Claude がミスするか？」と自問して、不要なものは削除しよう。

### @ インポートの活用

CLAUDE.md から他のファイルを参照できる：

```markdown
# CLAUDE.md
プロジェクト概要は @README.md を参照。
利用可能な npm スクリプトは @package.json を確認。

# 追加ルール
- Git ワークフロー: @docs/git-workflow.md
- API 設計ガイド: @docs/api-design.md
- 個人設定: @~/.claude/my-preferences.md
```

### AGENTS.md の活用

AGENTS.md は Claude Code のエージェントとしての動作を定義するファイル。CLAUDE.md がプロジェクトルールなら、AGENTS.md はエージェントの人格・行動規範：

```markdown
# AGENTS.md

## ワークスペース
作業ディレクトリ: /path/to/project

## ルール
- ファイル削除の前に必ず確認する
- テストが通らない状態でコミットしない
- 外部 API の呼び出しは Rate Limit を考慮する

## メモリ
- memory/ 以下に日次ログを残す
- 重要な決定事項は MEMORY.md に記録する

## セッション開始時
1. CLAUDE.md を読む
2. memory/ の最新ファイルを確認
3. git status で現状を把握
```

---

## 大規模プロジェクトでの運用

### コンテキスト管理戦略

大規模プロジェクトでは、コンテキストウィンドウの管理が成功の鍵。

#### .claudeignore の設定

```bash
# .claudeignore
node_modules/
dist/
build/
.next/
coverage/
*.min.js
*.min.css
*.map
*.lock
vendor/
__pycache__/
.git/objects/
```

#### タスクの分割

```
# ❌ 悪い例：一度に全てを依頼
「プロジェクト全体をリファクタリングして」

# ✅ 良い例：スコープを絞る
「src/api/routes/users.ts のエラーハンドリングをリファクタリングして。
 src/api/routes/orders.ts のパターンに合わせて。」
```

#### セッション管理

```bash
# タスクごとにセッションを分ける
claude --session "auth-refactor"
claude --session "api-tests"
claude --session "docs-update"

# セッション一覧・再開
claude --resume
```

### /clear と /compact の戦略的使用

```bash
# タスク完了ごとにクリア
/clear

# 長い会話を要約して圧縮（フォーカス指定可能）
/compact 認証システムの変更に集中

# カスタムコンパクション指示（CLAUDE.md に記載）
# コンパクション時は、変更済みファイル一覧とテスト結果を必ず保持すること
```

### モノレポでの運用

```bash
# ルートの CLAUDE.md でモノレポ全体のルールを定義
# 各パッケージの CLAUDE.md でパッケージ固有のルールを定義

monorepo/
├── CLAUDE.md              # 共通ルール
├── apps/
│   ├── web/
│   │   └── CLAUDE.md      # フロントエンド固有
│   └── api/
│       └── CLAUDE.md      # バックエンド固有
└── packages/
    └── shared/
        └── CLAUDE.md      # 共有パッケージ固有
```

---

## Git 操作との連携

### コミットの自動化

#### Conventional Commits でのコミット

```
変更内容を確認して、Conventional Commits 形式でコミットして。
関連する変更はまとめて、無関係な変更は分けて。
```

#### インタラクティブなコミット

```
git diff を見せて。変更をレビューしたい。
問題がなければコミットして。コミットメッセージは提案して確認を取って。
```

### ブランチ操作

```
# ブランチの作成と切り替え
feature/user-profile ブランチを作成して切り替えて。

# ブランチのマージ
main の最新を取り込んで。コンフリクトがあれば解決して。

# ブランチの整理
マージ済みのローカルブランチを一覧にして。
削除して問題ないか確認してから削除して。
```

### PR ワークフロー

```
# PR の作成
変更をコミットして、PR を作成して。
PR タイトルと説明を変更内容から自動生成して。
関連する Issue があればリンクして。

# PR のレビュー対応
PR #42 のレビューコメントを確認して。
指摘された問題を修正して、修正内容をコメントで返して。
```

### Git 履歴の分析

```
# 変更の多いファイルを特定
過去3ヶ月で最も変更が多かったファイルトップ10を教えて。
リファクタリングの優先度を決めたい。

# 特定の機能の歴史を追う
認証機能（src/auth/）の git log を見て、
主要な変更の流れを時系列でまとめて。

# 特定のバグの導入時期を特定
git bisect を使って、テスト X が壊れたコミットを特定して。
```

---

## 複数リポジトリの同時操作

### Git Worktrees の活用

```bash
# メインの作業ディレクトリ
cd my-project

# Worktree を作成（別ブランチを別ディレクトリで同時作業）
git worktree add ../my-project-feature feature/new-ui
git worktree add ../my-project-hotfix hotfix/login-fix

# 各 worktree で別々の Claude Code セッションを起動
# ターミナル1
cd ../my-project-feature && claude --session "new-ui"

# ターミナル2
cd ../my-project-hotfix && claude --session "login-fix"
```

### 関連リポジトリの同時変更

```
以下の2つのリポジトリに関連する変更が必要：
- frontend/ : API クライアントの型定義を更新
- backend/ : 新しいエンドポイントを追加

まず backend/ で API を実装して、
次に frontend/ で型定義とクライアントを更新して。
両方のテストが通ることを確認して。
```

---

## カスタムスラッシュコマンド

### スキル（Skills）の作成

`.claude/skills/` にマークダウンファイルを配置すると、カスタムスラッシュコマンドとして使える：

#### デプロイコマンド

```markdown
# .claude/skills/deploy.md
---
name: deploy
description: 本番環境へのデプロイ
---

以下の手順でデプロイを実行する：

1. `git status` でクリーンな状態か確認
2. `npm run typecheck` で型チェック
3. `npm run lint` でリント
4. `npm test` でテスト
5. `npm run build` でビルド
6. 全てパスしたら `git push origin main`
7. デプロイの完了を確認

いずれかのステップで失敗したら、理由を報告して停止する。
```

#### コードレビューコマンド

```markdown
# .claude/skills/review.md
---
name: review
description: 現在の変更のコードレビュー
---

`git diff --cached` (ステージ済み) または `git diff` (未ステージ) の変更をレビューする。

## チェック項目
1. **バグ**: ロジックエラー、off-by-one、null チェック漏れ
2. **パフォーマンス**: N+1 クエリ、不要な再レンダリング、メモリリーク
3. **セキュリティ**: インジェクション、XSS、認証漏れ
4. **スタイル**: プロジェクトの規約に準拠しているか
5. **テスト**: テストが追加・更新されているか

## 出力
問題がある箇所はファイル名と行番号付きで指摘する。
問題がなければ "LGTM 🎉" と報告する。
```

#### 日次レポートコマンド

```markdown
# .claude/skills/daily-report.md
---
name: daily-report
description: 今日の作業の日次レポートを生成
---

以下の情報を収集して日次レポートを生成する：

1. `git log --since="today 00:00" --oneline` で今日のコミット
2. `gh pr list --author @me` で自分の PR 状態
3. `gh issue list --assignee @me` で割り当て Issue

## 出力形式
### 📅 日次レポート - YYYY-MM-DD

#### ✅ 完了
- ...

#### 🔄 進行中
- ...

#### 📋 明日の予定
- ...
```

### スキルの使い方

```bash
# スキルの一覧を表示
/skills

# スキルを実行
/deploy
/review
/daily-report

# 引数付き
/review src/api/routes/users.ts
```

---

## SDK・プログラマティック活用

### Agent SDK（ヘッドレスモード）

Claude Code を CLI からプログラマティックに使用できる：

```bash
# 基本的な使い方
claude -p "src/utils/math.ts にテストを書いて"

# JSON 出力
claude -p "APIエンドポイント一覧をJSONで" --output-format json

# Plan Mode で分析のみ
claude --permission-mode plan -p "認証システムを分析して改善案を提示して"

# 入力をパイプ
cat error.log | claude -p "このエラーの原因を特定して"

# 権限チェックをスキップ（CI/CD 向け）
claude --dangerously-skip-permissions -p "lint エラーを修正して"
```

### TypeScript SDK

```typescript
import { Claude } from '@anthropic-ai/claude-code';

const claude = new Claude({
  workdir: '/path/to/project',
});

// タスクを実行
const result = await claude.run({
  prompt: 'src/utils/math.ts にユニットテストを追加して',
  permissionMode: 'auto',
});

console.log(result.output);
```

### Python SDK

```python
from claude_code import Claude

claude = Claude(workdir="/path/to/project")

result = claude.run(
    prompt="src/utils/math.ts にユニットテストを追加して",
    permission_mode="auto",
)

print(result.output)
```

### CI/CD での活用例

```yaml
# .github/workflows/claude-review.yml
name: Claude Code Review
on: [pull_request]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Install Claude Code
        run: curl -fsSL https://claude.ai/install.sh | bash

      - name: Review PR
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          DIFF=$(git diff origin/main...HEAD)
          claude --dangerously-skip-permissions \
            -p "以下のdiffをレビューして問題があれば報告:
            $DIFF" \
            --output-format json > review.json

      - name: Post Review
        run: |
          # review.json の内容を PR コメントとして投稿
          gh pr comment ${{ github.event.number }} \
            --body "$(cat review.json | jq -r '.output')"
```

---

## Hooks の高度な活用

### ファイル編集後の自動フォーマット

```json
// .claude/settings.json
{
  "hooks": {
    "afterEdit": [
      {
        "pattern": "*.ts",
        "command": "npx prettier --write {{file}}"
      },
      {
        "pattern": "*.tsx",
        "command": "npx prettier --write {{file}} && npx eslint --fix {{file}}"
      }
    ]
  }
}
```

### コミット前の自動チェック

```json
{
  "hooks": {
    "beforeCommit": [
      {
        "command": "npm run typecheck",
        "description": "型チェック"
      },
      {
        "command": "npm run lint",
        "description": "リント"
      }
    ]
  }
}
```

### 特定ディレクトリへの書き込み制限

```json
{
  "hooks": {
    "beforeEdit": [
      {
        "pattern": "migrations/*",
        "command": "echo '❌ マイグレーションファイルの直接編集は禁止です' && exit 1",
        "description": "マイグレーション保護"
      }
    ]
  }
}
```

!!! tip "Hooks の作成を Claude に任せる"
    ```
    ファイル編集後に自動で eslint を実行する Hook を書いて。
    マイグレーションフォルダへの書き込みをブロックする Hook も追加して。
    ```
    Claude 自身に Hooks を作成させるのが最も簡単。

---

## まとめ

| テクニック | 効果 |
|-----------|------|
| サブエージェント | コンテキスト管理、並列処理 |
| CLAUDE.md 最適化 | 一貫した出力品質 |
| セッション管理 | タスク間の分離 |
| Git Worktrees | 並列開発の物理的分離 |
| カスタムスキル | 繰り返しタスクの自動化 |
| Agent SDK | CI/CD・自動化パイプライン |
| Hooks | 品質保証の自動化 |

!!! tip "段階的に導入する"
    全てのテクニックを一度に導入しようとしないこと。まずは CLAUDE.md の整備から始めて、必要に応じてサブエージェント、スキル、Hooks を追加していくのが効果的。
