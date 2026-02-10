# Hooks・自動化ガイド

Hooks は Claude Code のライフサイクルの特定のポイントで**確定的にスクリプトを実行**する仕組み。CLAUDE.md の指示が助言的なのに対し、Hooks は**必ず実行される**ことが保証される。

> Claude は Hook を書いてくれる。「ファイル編集後に eslint を実行する hook を書いて」のようにプロンプトで指示するか、`/hooks` で対話的に設定できる。
>
> — [公式ドキュメント](https://code.claude.com/docs/en/best-practices)

## Hook の基本

### イベント一覧

| イベント | タイミング |
|----------|-----------|
| `SessionStart` | セッション開始・再開時 |
| `UserPromptSubmit` | プロンプト送信時（Claude 処理前） |
| `PreToolUse` | ツール実行前（**ブロック可能**） |
| `PermissionRequest` | 権限ダイアログ表示時 |
| `PostToolUse` | ツール実行成功後 |
| `PostToolUseFailure` | ツール実行失敗後 |
| `Notification` | Claude が通知を送信時 |
| `SubagentStart` | サブエージェント起動時 |
| `SubagentStop` | サブエージェント完了時 |
| `Stop` | Claude の応答完了時 |
| `TeammateIdle` | Agent Team のチームメイトがアイドル状態になる時 |
| `TaskCompleted` | タスク完了マーク時 |
| `PreCompact` | コンパクション実行前 |
| `SessionEnd` | セッション終了時 |

### Hook の種類

| タイプ | 説明 | 使い分け |
|--------|------|---------|
| `command` | シェルコマンドを実行 | ほとんどのケース |
| `prompt` | Claude モデルに Yes/No 判定させる | 判断が必要な場合（デフォルト: Haiku） |
| `agent` | サブエージェントにツール付きで検証させる | ファイル確認やコマンド実行が必要な場合 |

### Hook の設定場所

| 場所 | スコープ | 共有 |
|------|---------|------|
| `~/.claude/settings.json` | 全プロジェクト | 個人のみ |
| `.claude/settings.json` | プロジェクト | git コミット可能 |
| `.claude/settings.local.json` | プロジェクト | gitignore |
| 管理ポリシー設定 | 組織全体 | 管理者制御 |
| プラグインの `hooks/hooks.json` | プラグイン有効時 | プラグイン同梱 |

## 実用的な Hook 例

### デスクトップ通知（Claude が待機中に通知）

Claude がユーザーの入力待ちになったとき、ターミナルを監視していなくても気付けるようにする。

=== "macOS"
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

=== "Linux"
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

### ファイル編集後に自動フォーマット

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

`matcher` が `Edit|Write` なので、ファイル編集・書き込みツール使用後にのみ発火する。

### 保護ファイルへの編集をブロック

`.env` や `package-lock.json` など、Claude に触らせたくないファイルを保護する：

```bash
#!/bin/bash
# .claude/scripts/protect-files.sh
INPUT=$(cat)
FILE_PATH=$(echo "$INPUT" | jq -r '.tool_input.file_path')

if echo "$FILE_PATH" | grep -qE '(\.env|package-lock\.json|\.git/)'; then
  echo "Protected file: $FILE_PATH" >&2
  exit 2  # exit 2 = アクションをブロック
fi

exit 0  # exit 0 = 続行を許可
```

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "bash .claude/scripts/protect-files.sh"
          }
        ]
      }
    ]
  }
}
```

### コンパクション後にコンテキストを再注入

コンテキストウィンドウが埋まるとコンパクション（要約）が実行され、重要な詳細が失われることがある。`SessionStart` の `compact` マッチャーで、コンパクション後に重要な情報を再注入する：

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

`echo` の代わりに動的な出力を生成するコマンドも使える（例: `git log --oneline -5` で最近のコミットを表示）。

### Bash コマンドのログ記録

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.command' >> ~/.claude/command-log.txt"
          }
        ]
      }
    ]
  }
}
```

### MCP ツールの監視

MCP ツールは `mcp__<server>__<tool>` という命名規則を使う。正規表現マッチャーで特定サーバーの全ツールを対象にできる：

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "mcp__github__.*",
        "hooks": [
          {
            "type": "command",
            "command": "echo \"GitHub tool called: $(jq -r '.tool_name')\" >&2"
          }
        ]
      }
    ]
  }
}
```

## プロンプト型・エージェント型 Hook

### プロンプト型 Hook

判断が必要な場合に、Claude モデル（デフォルト: Haiku）に Yes/No 判定させる：

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "全てのタスクが完了しているか確認して。未完了なら {\"ok\": false, \"reason\": \"残りの作業内容\"} で応答して。"
          }
        ]
      }
    ]
  }
}
```

モデルが `"ok": false` を返すと、Claude はその `reason` を次の指示として作業を続ける。

### エージェント型 Hook

ファイル確認やコマンド実行を伴う検証が必要な場合に使う：

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "agent",
            "prompt": "全てのユニットテストがパスするか検証して。テストスイートを実行して結果を確認。$ARGUMENTS",
            "timeout": 120
          }
        ]
      }
    ]
  }
}
```

エージェント型 Hook はファイルを読み、コードを検索し、ツールを使って条件を検証してから判定を返す。

**使い分け**: Hook の入力データだけで判断できる → プロンプト型。コードベースの実際の状態を検証する必要がある → エージェント型。

## Hook の入出力

### 入力（stdin）

Hook はイベント固有のデータを JSON で stdin から受け取る：

```json
{
  "session_id": "abc123",
  "cwd": "/Users/sarah/myproject",
  "hook_event_name": "PreToolUse",
  "tool_name": "Bash",
  "tool_input": {
    "command": "npm test"
  }
}
```

### 出力（exit code）

| 終了コード | 動作 |
|-----------|------|
| **0** | アクション続行。`UserPromptSubmit` と `SessionStart` では stdout の内容がコンテキストに追加される |
| **2** | アクション**ブロック**。stderr に理由を書くと Claude にフィードバックされる |
| **その他** | アクション続行。stderr はログに記録されるが Claude には表示されない |

### 構造化 JSON 出力

より細かい制御が必要な場合、exit 0 で JSON オブジェクトを stdout に出力する：

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "deny",
    "permissionDecisionReason": "grep の代わりに rg を使ってください（パフォーマンス向上）"
  }
}
```

`PreToolUse` の `permissionDecision` は `"allow"`, `"deny"`, `"ask"` の3つ。

## マッチャーの仕組み

マッチャーなしの Hook は対象イベントの全発生で発火する。マッチャーで絞り込む：

| イベント | マッチ対象 | 例 |
|---------|----------|-----|
| `PreToolUse`, `PostToolUse` 等 | ツール名 | `Bash`, `Edit\|Write`, `mcp__.*` |
| `SessionStart` | 開始方法 | `startup`, `resume`, `clear`, `compact` |
| `SessionEnd` | 終了理由 | `clear`, `logout` |
| `Notification` | 通知タイプ | `permission_prompt`, `idle_prompt` |
| `SubagentStart/Stop` | エージェントタイプ | `Bash`, `Explore`, `Plan` |
| `PreCompact` | トリガー | `manual`, `auto` |

## トラブルシューティング

### Hook が発火しない

- `/hooks` で正しいイベントに表示されているか確認
- マッチャーパターンがツール名と正確に一致しているか確認（大文字小文字を区別）
- `PermissionRequest` Hook は非対話モード（`-p`）では発火しない → `PreToolUse` を使う

### Stop Hook が無限ループする

`stop_hook_active` フィールドを確認して、既に継続をトリガーしたかチェックする必要がある：

```bash
#!/bin/bash
INPUT=$(cat)
if [ "$(echo "$INPUT" | jq -r '.stop_hook_active')" = "true" ]; then
  exit 0  # Claude を停止させる
fi
# ... 残りのロジック
```

### JSON パースエラー

シェルプロファイル（`~/.zshrc` 等）に無条件の `echo` 文があると、Hook の JSON 出力の前に余計な出力が混ざる。インタラクティブシェルでのみ実行するようラップする：

```bash
# ~/.zshrc に
if [[ $- == *i* ]]; then
  echo "Shell ready"
fi
```

### デバッグ

- `Ctrl+O` で verbose モードを切り替えて Hook の出力をトランスクリプトに表示
- `claude --debug` で完全な実行詳細を確認

## よく使うカスタムスキル例

### /review — コードレビュー

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

### /commit — スマートコミット

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

### /test — テスト生成

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

### /deploy — デプロイ

```yaml
# .claude/skills/deploy/SKILL.md
---
name: deploy
description: アプリケーションを本番環境にデプロイ
disable-model-invocation: true
context: fork
---

$ARGUMENTS をデプロイする：

1. テストスイートを実行
2. アプリケーションをビルド
3. デプロイターゲットにプッシュ
4. デプロイが成功したことを確認
```

`disable-model-invocation: true` を設定しているため、Claude が勝手にデプロイすることはない。`/deploy production` のようにユーザーが明示的に呼び出す。
