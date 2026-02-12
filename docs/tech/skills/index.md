# Claude Code / OpenClaw向けSkillsまとめ

Claude CodeとOpenClawにおけるSkillsシステムの包括的なガイドです。Skillsを使ってAIエージェントの能力を拡張し、より効率的な開発ワークフローを構築しましょう。

## Skillsとは何か？

SkillsはClaude CodeやOpenClawの拡張機能システムです。`SKILL.md`ファイルに指示を記述することで、Claudeに新しい能力を追加できます。

!!! info "Skillsの特徴"
    - **簡単な作成**: テキストファイル（SKILL.md）を作成するだけで始められる
    - **自動発見**: Claudeが関連性を判断して適切なタイミングで使用
    - **手動実行**: `/skill-name`でいつでも直接呼び出し可能
    - **階層的構造**: 複数のファイルやスクリプトを含められる

## Skillsの仕組み

### 基本構造

```
my-skill/
├── SKILL.md        # メイン指示（必須）
├── template.md     # テンプレートファイル
├── examples/       # サンプル出力
│   └── sample.md
└── scripts/        # 実行可能スクリプト
    └── validate.sh
```

### SKILL.mdの構成

```markdown
---
name: my-skill
description: このスキルの用途と使うタイミング
disable-model-invocation: false
allowed-tools: Read, Write
---

Claudeへの指示をここに記述します...
```

!!! warning "重要なポイント"
    - `name`: スキル名（64文字以内、英数字とハイフンのみ）
    - `description`: Claudeが使用タイミングを判断する重要な情報
    - YAMLフロントマターは必須

## Agent Skills標準

Claude Code SkillsはAnthropic開発の[Agent Skills](https://agentskills.io)オープン標準に準拠しています。この標準により、複数のAIツール間でSkillsを共有できます。

!!! tip "Claude Code独自の拡張機能"
    - **実行制御**: 誰がSkillを呼び出せるかを制御
    - **サブエージェント実行**: 分離されたコンテキストでSkillを実行
    - **動的コンテキスト注入**: 実行時にデータを動的に取得

## Skillsの保存場所と優先度

| 場所 | パス | 適用範囲 |
|------|------|----------|
| Enterprise | 管理設定による | 組織全体 |
| Personal | `~/.claude/skills/<skill-name>/` | 全プロジェクト |
| Project | `.claude/skills/<skill-name>/` | このプロジェクトのみ |
| Plugin | `<plugin>/skills/<skill-name>/` | プラグインが有効な場所 |

**優先度**: Enterprise > Personal > Project

!!! note "OpenClaw固有の場所"
    OpenClawでは以下の場所にもSkillsを配置できます：
    - グローバル: `~/.openclaw/skills/`
    - ワークスペース: `<project>/skills/`

## 導入方法

### 1. ClawHub経由（推奨）

[ClawHub](https://clawhub.ai/)はOpenClawの公式スキルレジストリです。3,000個以上のコミュニティ作成Skillsが利用可能です。

```bash
# ClawHub CLIでインストール
npx clawhub@latest install <skill-slug>

# OpenClaw専用コマンド
clawdhub install <skill-name>
```

!!! success "ClawHubの利点"
    - **セキュリティスキャン**: VirusTotal連携による安全性チェック
    - **カテゴリ分類**: 用途別に整理された豊富なSkills
    - **コミュニティ検証**: 実績のあるSkillsが集約

### 2. 手動インストール

```bash
# スキルフォルダを適切な場所にコピー
cp -r my-skill ~/.claude/skills/
# または
cp -r my-skill .claude/skills/
```

### 3. 会話による作成

Claudeとの会話中に「このワークフローをSkillにして」と依頼することで、自動的にSkillが作成されます。

## 実際の使用例

### コード説明Skill

```markdown
---
name: explain-code
description: コードを視覚的な図解と類推で説明
---

コードを以下の方法で説明してください：

1. **概要**: 何をするコードか一行で
2. **フローチャート**: mermaid記法で処理の流れ
3. **類推**: 現実世界の例えで説明
4. **重要なポイント**: 注意すべき部分をハイライト
```

### デプロイメントSkill

```markdown
---
name: deploy
description: 本番環境へのアプリケーションデプロイ
disable-model-invocation: true
---

アプリケーションを本番環境にデプロイします：

1. テストスイートを実行
2. アプリケーションをビルド
3. デプロイターゲットにプッシュ
4. デプロイが成功したかを確認
```

## Skills管理のベストプラクティス

!!! tip "効果的なSkill作成のコツ"
    - **単一目的**: 1つのSkillは1つの機能に集中
    - **明確な説明**: Claudeが適切に判断できる具体的な説明を記述
    - **段階的構築**: シンプルなものから始めて徐々に拡張
    - **テストの実施**: 作成後は複数のプロンプトでテスト

!!! warning "セキュリティ上の注意"
    - 不明なソースからのSkillsは事前にコードレビューを実施
    - VirusTotalレポートを必ず確認
    - 本番環境で使う前にサンドボックスでテスト

## 関連リソース

- [おすすめSkills一覧](recommended.md) - カテゴリ別のおすすめSkills
- [自作Skillの作り方](create.md) - SKILL.md作成からデバッグまで
- [Claude Code公式ドキュメント](https://code.claude.com/docs/en/skills)
- [Agent Skills仕様](https://agentskills.io)
- [ClawHub](https://clawhub.ai/) - 公式スキルレジストリ