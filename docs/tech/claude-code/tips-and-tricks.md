# Claude Code Tips & Tricks

実践的な小技、コスト最適化、hooks、カスタムコマンドなど。

## コスト最適化

### コンテキスト管理が最重要

Claude Code のコストはトークン消費量に直結する。コンテキストウィンドウが埋まるほど、トークン消費も品質低下も加速する。

#### 基本戦略

```
1. /clear — タスク完了ごとにコンテキストをリセット
2. /compact — 長い会話を要約してコンテキストを縮小
3. 1セッション1タスク — 複数タスクを1セッションに詰め込まない
4. 新鮮なコンテキスト — 「AI のコンテキストは牛乳。新鮮で濃縮されたものが最高」
```

#### .claudeignore で不要ファイルを除外

```bash
# .claudeignore
node_modules/
dist/
build/
.git/
*.min.js
*.map
coverage/
__pycache__/
.next/
```

#### ステータスラインでトークン監視

Claude Code のステータスラインをカスタマイズしてトークン使用量を常時表示:

```bash
# Claude Code 内で
/config
# → status_line を設定
```

### モデルの使い分け

| シナリオ | 推奨モデル |
|----------|-----------|
| 複雑なアーキテクチャ設計 | Opus |
| 日常的な実装・デバッグ | Sonnet |
| 簡単なファイル操作・質問 | Haiku |

```bash
# モデル切り替え
/model opus
/model sonnet
/model haiku
```

## Hooks の活用

### Hooks とは

Claude Code のライフサイクルの特定のポイントで**自動的にスクリプトを実行**する仕組み。CLAUDE.md の指示は助言的だが、hooks は**確定的に実行される**。

### 主要なイベント

| イベント | タイミング |
|----------|-----------|
| `SessionStart` | セッション開始・再開時 |
| `UserPromptSubmit` | プロンプト送信時 |
| `PreToolUse` | ツール実行前（ブロック可能） |
| `PostToolUse` | ツール実行成功後 |
| `PostToolUseFailure` | ツール実行失敗後 |
| `Notification` | Claude が通知を送信時 |
| `Stop` | Claude の応答完了時 |
| `PreCompact` | コンパクション前 |
| `SessionEnd` | セッション終了時 |

### 実用的な Hook 例

#### デスクトップ通知（Claude が待機中に通知）

```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Claude Code needs your attention\" with title \"Claude Code\"'"
          }
        ]
      }
    ]
  }
}
```

Linux の場合:

```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "notify-send 'Claude Code' 'Claude Code needs your attention'"
          }
        ]
      }
    ]
  }
}
```

#### ファイル編集後に自動フォーマット

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.file_path' | xargs npx prettier --write"
          }
        ]
      }
    ]
  }
}
```

#### 保護ファイルへの編集をブロック

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.file_path' | grep -qE '(\\.env|package-lock\\.json|\\.git/)' && echo 'Protected file' >&2 && exit 2 || exit 0"
          }
        ]
      }
    ]
  }
}
```

#### コンパクション後にコンテキストを再注入

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "compact",
        "hooks": [
          {
            "type": "command",
            "command": "echo 'リマインダー: pnpm を使う（npm ではない）。コミット前に pnpm test を実行。現在のスプリント: 認証リファクタリング'"
          }
        ]
      }
    ]
  }
}
```

### Hook の設定場所

```
~/.claude/settings.json          — 全プロジェクト共通
.claude/settings.json            — プロジェクト固有
```

!!! tip "Claude に Hook を書かせる"
    「ファイル編集後に eslint を実行する hook を書いて」のようにプロンプトで指示できる。`/hooks` コマンドで対話的に設定も可能。

## カスタムスラッシュコマンド（Skills）

!!! info "コマンドとスキルの統合"
    `.claude/commands/` のカスタムスラッシュコマンドは `.claude/skills/` のスキルと**統合された**。どちらの場所に置いても `/name` で呼び出せる。同名の場合はスキルが優先。

### よく使うカスタムコマンド例

#### /review — コードレビュー

```yaml
# .claude/skills/review/SKILL.md
---
name: review
description: 変更されたファイルのコードレビュー
disable-model-invocation: true
---
git diff で変更されたファイルをレビューする。

1. `git diff --name-only HEAD~1` で変更ファイルを取得
2. 各ファイルの差分を確認
3. 以下の観点でレビュー:
   - バグの可能性
   - セキュリティリスク
   - パフォーマンス問題
   - コードスタイルの一貫性
4. 問題点と改善提案をまとめる
```

#### /commit — スマートコミット

```yaml
# .claude/skills/commit/SKILL.md
---
name: commit
description: 変更内容から適切なコミットメッセージを生成してコミット
disable-model-invocation: true
---
1. `git diff --staged` でステージされた変更を確認
2. 変更内容を分析
3. Conventional Commits 形式のメッセージを生成
4. `git commit -m "<message>"` でコミット
```

#### /test — テスト生成

```yaml
# .claude/skills/test/SKILL.md
---
name: test
description: 指定ファイルのテストを生成
disable-model-invocation: true
argument-hint: [ファイルパス]
---
$ARGUMENTS のテストを作成する。

1. 対象ファイルを読み込む
2. 既存のテストパターンを確認（テストディレクトリを検索）
3. エッジケースを含むテストを作成
4. テストを実行して全てパスすることを確認
```

## プロンプトエンジニアリング

### 効果的なプロンプトの型

#### 具体的な指示 + 検証条件

```
validateEmail 関数を書いて。
テストケース:
- user@example.com → true
- invalid → false
- user@.com → false
実装後にテストを実行して確認して。
```

#### 既存パターンの参照

```
既存のウィジェット実装を確認して（HotDogWidget.php が良い例）。
同じパターンに従ってカレンダーウィジェットを実装して。
月選択とページネーション機能付き。
```

#### スクリーンショットベースの UI 指示

```
[画像をペースト]
このデザインを実装して。完成後にスクリーンショットを撮って元のデザインと比較して。
差異があれば修正して。
```

### やってはいけないこと

| ❌ 悪い例 | ✅ 良い例 |
|----------|----------|
| 「このファイルを改善して」 | 「この関数のエラーハンドリングを追加して。null の場合のケースを処理」 |
| 「バグを直して」 | 「ログイン後にリダイレクトが失敗する。src/auth/redirect.ts を確認して」 |
| 「テストを書いて」 | 「src/utils/parser.ts のエッジケーステストを書いて。空文字列、null、巨大入力を含める」 |

## マルチリポジトリ運用

### --add-dir でマルチリポジトリ

```bash
# メインリポジトリで起動し、他リポジトリも参照可能にする
claude --add-dir ../shared-lib --add-dir ../api-server
```

### モノレポでの CLAUDE.md 配置

```
monorepo/
├── CLAUDE.md                    # 共通ルール
├── packages/
│   ├── frontend/
│   │   ├── CLAUDE.md            # フロントエンド固有ルール
│   │   └── .claude/skills/      # フロントエンド固有スキル
│   ├── backend/
│   │   ├── CLAUDE.md            # バックエンド固有ルール
│   │   └── .claude/skills/
│   └── shared/
│       └── CLAUDE.md            # 共有ライブラリのルール
```

Claude Code は作業中のディレクトリに応じて、該当する階層の CLAUDE.md を自動でマージして読み込む。

## CI/CD との連携

### GitHub Actions での利用

```yaml
# .github/workflows/claude-review.yml
name: Claude Code Review
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '22'
      - run: npm install -g @anthropic-ai/claude-code
      - name: Review PR
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          claude -p "git diff origin/main...HEAD の変更をレビューして、
            問題点があれば指摘して。結果を review.md に書き出して" \
            --allowedTools "Read,Bash,Write"
      - name: Upload review
        uses: actions/upload-artifact@v4
        with:
          name: code-review
          path: review.md
```

### pre-commit hook での利用

```bash
#!/bin/bash
# .git/hooks/pre-commit

# ステージされたファイルの簡易チェック
STAGED_FILES=$(git diff --cached --name-only --diff-filter=ACM)

if [ -n "$STAGED_FILES" ]; then
  claude -p "以下のファイルの変更に明らかなバグやセキュリティ問題がないか簡潔にチェックして:
    $STAGED_FILES" --allowedTools "Read,Bash" 2>/dev/null
fi
```

## 権限設定の最適化

### 許可リストでプロンプト疲れを防ぐ

```json
// .claude/settings.json
{
  "permissions": {
    "allow": [
      "Bash(npm run lint)",
      "Bash(npm run test*)",
      "Bash(git commit*)",
      "Bash(git push*)",
      "Read",
      "Glob",
      "Grep"
    ]
  }
}
```

### サンドボックスモード

OS レベルの分離で、ファイルシステム・ネットワークアクセスを制限しつつ Claude に自由度を与える:

```bash
# サンドボックス有効で起動
claude --sandbox
```

### 全許可モード（自動化用）

```bash
# 全権限チェックをスキップ（CI/CD 等で利用）
claude --dangerously-skip-permissions
```

!!! danger "注意"
    `--dangerously-skip-permissions` は信頼できる環境でのみ使用すること。ローカル開発では許可リストを推奨。

## プラグインの活用

### プラグインとは

Skills、hooks、サブエージェント、MCP サーバーを**ひとつのパッケージ**にまとめたもの。コミュニティや Anthropic が公開。

### コードインテリジェンスプラグイン

型付き言語を使っているなら、[コードインテリジェンスプラグイン](https://code.claude.com/docs/en/discover-plugins#code-intelligence) をインストールすると、シンボルナビゲーションや編集後の自動エラー検出が利用できる。

### awesome-claude-code

コミュニティのプラグイン・スキル集:

- [awesome-claude-code](https://github.com/hesreallyhim/awesome-claude-code)
- [awesome-claude-code-plugins](https://github.com/ccplugins/awesome-claude-code-plugins)

## よく使うコマンド早見表

| コマンド | 説明 |
|----------|------|
| `/clear` | コンテキストをクリア |
| `/compact` | 会話を要約して圧縮 |
| `/init` | CLAUDE.md を自動生成 |
| `/model` | モデル切り替え |
| `/config` | 設定変更 |
| `/hooks` | Hook の対話的設定 |
| `/mcp` | MCP サーバーのステータス |
| `/permissions` | 権限設定 |
| `/vim` | Vim モード切り替え |
| `/terminal-setup` | ターミナル設定 |
| `Shift+Tab` × 2 | Plan Mode（計画のみ、実行しない） |
| `Esc` | 実行中のタスクを中断 |
