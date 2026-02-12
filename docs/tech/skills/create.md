# 自作Skillの作り方

オリジナルのSkillを作成して、Claude Code・OpenClawを自分の用途に最適化する完全ガイドです。基本的なSKILL.mdの作成から高度なスクリプト連携、テスト、公開まで段階的に解説します。

## 🚀 基本的なSkill作成

### Step 1: ディレクトリ構造の準備

```bash
# Skillディレクトリを作成
mkdir -p ~/.claude/skills/my-first-skill

# 基本ファイルを作成
touch ~/.claude/skills/my-first-skill/SKILL.md
```

!!! info "保存場所の選択"
    | 場所 | パス | 用途 |
    |------|------|------|
    | Personal | `~/.claude/skills/` | 全プロジェクトで使用 |
    | Project | `.claude/skills/` | 特定プロジェクトのみ |
    | OpenClaw | `~/.openclaw/skills/` | OpenClaw専用 |

### Step 2: 基本的なSKILL.md作成

```markdown
---
name: my-first-skill
description: 簡単な挨拶を行うサンプルスキル
---

# 挨拶スキル

ユーザーに挨拶を返します。

## 使用方法

「こんにちは」または「hello」と言われたら、親しみやすい挨拶を返してください。

## 返答例

- 日本語: 「こんにちは！今日はどんなことをお手伝いしましょうか？」
- 英語: "Hello! How can I assist you today?"
```

### Step 3: テスト実行

Claude Codeで以下を実行してテスト：

```
/my-first-skill
```

または自然な会話で：
```
こんにちは
```

!!! success "成功の確認"
    Claudeが適切に挨拶を返し、`/skills`コマンドでスキルが表示されることを確認

## 📝 SKILL.mdの詳細な書き方

### フロントマターの全オプション

```yaml
---
# 必須項目
name: skill-name                    # スキル名（64文字以内、英数字とハイフン）
description: スキルの説明と使用タイミング  # Claudeが判断に使う重要な情報

# オプション項目
argument-hint: "[引数名]"            # 引数のヒント表示
disable-model-invocation: false    # Claudeによる自動実行を無効化
user-invocable: true               # ユーザーによる手動実行を許可
allowed-tools: "Read, Write"       # 許可するツール一覧
model: "claude-3-sonnet"           # 使用モデル指定
context: fork                      # サブエージェントで実行
agent: Explore                     # サブエージェント種類
---
```

### 効果的な説明文の書き方

!!! tip "良い説明文の例"
    ```yaml
    # ❌ 悪い例
    description: "ファイルを処理する"
    
    # ✅ 良い例  
    description: "CSVファイルを読み込んでデータを分析し、可視化グラフを作成する。データ分析、CSV処理、グラフ作成が必要な時に使用"
    ```

**ポイント**:
- 具体的な機能を明記
- 使用するタイミングを明確化
- キーワードを含める（Claudeの判断材料）

### 引数の活用

```markdown
---
name: file-processor
description: 指定されたファイルを処理する
argument-hint: "[ファイルパス] [処理タイプ]"
---

# ファイル処理スキル

$ARGUMENTSで指定されたファイルを処理します。

## 処理内容

1. ファイル $ARGUMENTS[0] を読み込み
2. 処理タイプ $ARGUMENTS[1] に応じて変換
3. 結果を出力

## サポートされる処理タイプ

- `analyze`: 内容分析
- `convert`: フォーマット変換
- `validate`: 検証
```

**使用例**:
```
/file-processor data.csv analyze
```

## 🔧 高度なSkill作成

### マルチファイル構成

```
advanced-skill/
├── SKILL.md          # メイン指示
├── REFERENCE.md      # 詳細リファレンス
├── templates/        # テンプレート集
│   ├── email.md
│   └── report.md
├── examples/         # サンプル出力
│   └── sample.json
└── scripts/          # 実行スクリプト
    ├── analyze.py
    └── generate.sh
```

#### メインSKILL.md

```markdown
---
name: advanced-skill
description: 高度なデータ分析とレポート生成
allowed-tools: Bash, Read, Write
---

# 高度分析スキル

データを分析してレポートを生成します。

## 利用可能なリソース

- 詳細な分析手法: [REFERENCE.md](REFERENCE.md)
- テンプレート集: [templates/](templates/)  
- サンプル出力: [examples/sample.json](examples/sample.json)

## 実行手順

1. データを[scripts/analyze.py](scripts/analyze.py)で分析
2. 結果を[templates/report.md](templates/report.md)でフォーマット
3. 最終レポートを生成
```

### 実行可能スクリプトの作成

#### Python スクリプト例

```python
#!/usr/bin/env python3
"""データ分析スクリプト"""

import sys
import json
from pathlib import Path

def analyze_data(file_path):
    """データ分析を実行"""
    data = Path(file_path).read_text()
    
    # 分析ロジック
    result = {
        "file_size": len(data),
        "line_count": len(data.split('\n')),
        "word_count": len(data.split()),
        "analysis_summary": "データ分析完了"
    }
    
    return result

if __name__ == "__main__":
    if len(sys.argv) != 2:
        print("使用法: analyze.py <ファイルパス>")
        sys.exit(1)
        
    file_path = sys.argv[1]
    result = analyze_data(file_path)
    print(json.dumps(result, ensure_ascii=False, indent=2))
```

#### SKILL.mdでの連携

```markdown
---
name: data-analyzer
description: Pythonスクリプトでデータ分析を実行
allowed-tools: Bash
---

# データ分析スキル

以下の手順でデータ分析を実行：

```bash
python ~/.claude/skills/data-analyzer/scripts/analyze.py $ARGUMENTS
```

分析結果をJSON形式で出力し、わかりやすく解釈します。
```

### 動的コンテキスト注入

```markdown
---
name: github-pr-summary  
description: GitHub PRの内容を要約
context: fork
agent: Explore
allowed-tools: Bash(gh *)
---

## プルリクエストコンテキスト

- PR差分: !`gh pr diff`
- PRコメント: !`gh pr view --comments`
- 変更ファイル一覧: !`gh pr diff --name-only`

## タスク

このプルリクエストの内容を以下の観点で要約してください：

1. **変更の概要**: 何が変更されたか
2. **影響範囲**: どの部分に影響するか  
3. **注意点**: レビューで確認すべきポイント
```

!!! note "動的コンテキスト注入の仕組み"
    - `!`command`` 記法で実行時にコマンド実行
    - 出力がSkill内容に注入される
    - Claudeには最終的な内容のみが渡される

## 🧪 テスト・デバッグ

### 基本テスト手法

```markdown
# テスト用SKILL.md例
---
name: test-skill
description: テスト専用スキル
---

# テストスキル

以下をテスト：

1. **引数テスト**: $ARGUMENTS
2. **ファイル読み込み**: [test-data.txt](test-data.txt) があるか確認
3. **スクリプト実行**: scripts/test.sh を実行

## デバッグ情報

- スキル名: test-skill
- 実行時刻: 現在時刻を表示
- 引数: すべての引数を列挙
```

### デバッグのベストプラクティス

!!! tip "効果的なデバッグ方法"
    1. **単純化**: 複雑な機能は段階的に追加
    2. **ログ出力**: デバッグ情報を豊富に含める
    3. **エラーハンドリング**: 期待される失敗ケースを考慮
    4. **バリデーション**: 入力値の検証を含める

#### デバッグ用テンプレート

```markdown
---
name: debug-template
description: デバッグ機能付きスキルテンプレート
---

# デバッグモード

## 引数確認
- 引数数: $ARGUMENTSの数を確認
- 各引数: $0, $1, $2... を個別表示

## 環境確認  
- 実行パス: 現在のディレクトリ
- ファイル存在: 必要ファイルの存在確認
- 権限確認: スクリプト実行権限

## 実行結果
- 成功/失敗の明確な表示
- エラーメッセージの詳細化
```

## 📚 パフォーマンス最適化

### 軽量化のテクニック

```markdown
---
name: optimized-skill
description: 軽量化されたスキル（参考情報使用時のみ詳細読み込み）
---

# 最適化されたスキル

基本的な処理を実行します。

## 詳細情報が必要な場合

- 高度な設定: [CONFIG.md](CONFIG.md) を参照
- トラブルシューティング: [TROUBLESHOOT.md](TROUBLESHOOT.md) を確認
- API仕様: [API_REFERENCE.md](API_REFERENCE.md) をチェック

必要に応じて上記ファイルを読み込んでください。
```

!!! info "パフォーマンス考慮点"
    - **プログレッシブ開示**: 基本情報 → 詳細情報の段階的読み込み
    - **文字数制限**: SKILL.mdは簡潔に、詳細は別ファイルへ
    - **条件分岐**: 必要な時だけ追加リソースを読み込み

## 🌐 Skills公開・共有

### ClawHub への公開準備

#### 1. ディレクトリ構造の確認

```bash
my-skill/
├── SKILL.md              # 必須
├── README.md             # 推奨
├── LICENSE               # 公開時必須
├── examples/             # オプション
└── scripts/              # オプション
```

#### 2. README.mdの作成

```markdown
# My Skill

## 概要
このスキルの簡潔な説明

## インストール
```bash
npx clawhub@latest install my-skill
```

## 使用方法
基本的な使用例とコマンド

## 機能
- 機能1: 説明
- 機能2: 説明

## ライセンス
MIT
```

#### 3. セキュリティ対応

```markdown
# セキュリティチェックリスト

- [ ] 機密情報の削除（API キー等）
- [ ] 適切な権限設定
- [ ] エラーハンドリング実装
- [ ] VirusTotal スキャン対応
- [ ] 依存関係の明記
```

### GitHub での管理

```bash
# Skillリポジトリ作成
git init
git add .
git commit -m "Initial skill version"
git branch -M main
git remote add origin https://github.com/username/my-skill.git
git push -u origin main

# ClawHub登録用のPRを作成
# openclaw/skillsリポジトリにPull Request
```

## 💡 実践的なSkill作成例

### プロジェクト管理Skill

```markdown
---
name: project-manager
description: プロジェクトのタスク管理とステータス追跡
allowed-tools: Read, Write, Bash
---

# プロジェクト管理スキル

## 機能一覧

1. **タスク追加**: `add-task [タスク名] [優先度]`
2. **進捗更新**: `update-status [タスクID] [ステータス]`  
3. **レポート生成**: `generate-report [期間]`

## 使用方法

### タスクファイルの確認

```bash
ls -la tasks/
```

### 新しいタスクの追加

タスクを以下の形式でtasks/ディレクトリに保存：

```json
{
  "id": "task-001",
  "name": "$ARGUMENTS[0]",
  "priority": "$ARGUMENTS[1]", 
  "status": "todo",
  "created": "現在日時"
}
```

### 進捗レポートの生成

現在のタスク一覧から進捗状況を集計し、Markdown形式でレポート出力。
```

!!! success "成功要因"
    - **明確な機能分割**: 1つのSkillが1つの責務を持つ
    - **豊富な例**: 使用方法が具体的
    - **エラー対応**: 失敗時の動作も明記

## 🔍 トラブルシューティング

### よくある問題と解決策

| 問題 | 原因 | 解決策 |
|------|------|--------|
| Skillが認識されない | ディレクトリ構造が不正 | `SKILL.md`がディレクトリ直下にあることを確認 |
| 自動実行されない | 説明文が不適切 | より具体的な`description`に修正 |
| スクリプトが実行されない | 権限・パスの問題 | 実行権限と絶対パスを確認 |
| 引数が正しく渡されない | `$ARGUMENTS`の記述ミス | `$0`, `$1`等の正しい記法を使用 |

### デバッグコマンド

```bash
# Skillsの一覧確認
/skills

# 特定Skillの詳細表示  
/skill-name --help

# Claudeの思考過程確認（Claude Code）
/thinking on
```

## 📈 次のステップ

!!! tip "スキル向上のロードマップ"
    1. **基本Skills**: シンプルな指示ベース → **完了** ✅
    2. **実行系Skills**: スクリプト連携 → **習得中** 🔄  
    3. **統合Skills**: 外部API・サービス連携 → **次の目標** 🎯
    4. **共有Skills**: コミュニティ貢献 → **将来目標** 🌟

### さらなる学習リソース

- [Agent Skills仕様書](https://agentskills.io) - 標準仕様の詳細
- [Claude Code公式ドキュメント](https://code.claude.com/docs/en/skills) - 最新機能
- [Awesome OpenClaw Skills](https://github.com/VoltAgent/awesome-openclaw-skills) - 優秀な実例集
- [ClawHub](https://clawhub.ai/) - 公開Skillsの研究

## 関連ページ

- [Skillsの基礎知識](index.md)
- [おすすめSkills一覧](recommended.md)