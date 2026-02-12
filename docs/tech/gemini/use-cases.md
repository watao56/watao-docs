# 活用事例

## コード生成・リファクタリング

### 新規プロジェクトのスキャフォールディング

```
> Express + TypeScript + Prisma で REST API のボイラープレートを作って。
  認証は JWT、データベースは PostgreSQL で。
```

### 既存コードのリファクタリング

```
> src/services/ 以下のファイルを読んで、重複しているロジックを
  共通ユーティリティに抽出して。DRY原則に従ってリファクタリングして。
```

### テスト生成

```
> src/utils/validation.ts のユニットテストを Vitest で書いて。
  エッジケースも網羅して。
```

## 大規模コードベース分析

Gemini CLI の最大の強みの一つが、**100万トークンのコンテキストウィンドウ**を活用した大規模分析です。

### プロジェクト全体の理解

```
> このプロジェクト全体のアーキテクチャを分析して。
  各モジュールの責務と依存関係をまとめて。
```

### 依存関係の調査

```
> この関数がどこから呼ばれているか、全ての呼び出し元を追跡して。
  影響範囲を教えて。
```

### コードベース横断のバグ調査

```
> ユーザーから「決済が二重に処理される」というバグ報告がある。
  決済関連のコードを全て読んで、原因を特定して。
```

!!! tip "大規模分析のコツ"
    Gemini CLI はプロジェクトのファイルを自動で読み込みますが、巨大なリポジトリでは `--include-directories` で対象を絞ると効率的です。

## マルチモーダル活用

### 画像からのコード生成

```
> @mockup.png このデザインモックアップを React コンポーネントとして実装して。
  Tailwind CSS を使って。
```

### PDF からの仕様実装

```
> @spec.pdf この仕様書に基づいて API エンドポイントを実装して。
  バリデーションも仕様通りに。
```

### スクリーンショットからのバグ修正

```
> @bug-screenshot.png このUIバグを修正して。
  ボタンが画面外にはみ出しているように見える。
```

!!! note "対応フォーマット"
    画像（PNG, JPEG, WebP, GIF）、PDF、動画など多様なフォーマットに対応しています。

## ドキュメント解析

### API ドキュメントの読み込みと実装

```
> @api-docs.pdf この外部 API のドキュメントを読んで、
  TypeScript のクライアントライブラリを生成して。
  エラーハンドリングとリトライロジックも含めて。
```

### 既存ドキュメントの更新

```
> コードの変更に合わせて README.md と API ドキュメントを更新して。
  新しいエンドポイントの説明を追加して。
```

## デバッグ支援

### エラーログの分析

```bash
# パイプでエラーログを渡す
cat error.log | gemini "このエラーの根本原因を分析して、修正方法を提案して"
```

### スタックトレースの解析

```
> 以下のスタックトレースを分析して、原因と修正方法を教えて：
  [スタックトレースを貼り付け]
```

### パフォーマンス問題の調査

```
> このアプリケーションの起動が遅い。プロファイルデータを見て、
  ボトルネックを特定して最適化案を出して。
```

## Google Search グラウンディング

Gemini CLI 独自の強みとして、Google Search によるリアルタイム情報の活用があります。

```
> Next.js 15 の最新の App Router のベストプラクティスを調べて、
  このプロジェクトに適用して。
```

```
> このライブラリの最新バージョンで breaking changes があるか調べて、
  マイグレーションガイドに従ってアップデートして。
```

!!! tip "Google Search の強み"
    Claude Code にはないビルトイン Web 検索機能により、最新のライブラリ情報やベストプラクティスを即座に参照できます。ドキュメントが古いOSSの調査に特に有効です。

## GitHub Actions 連携

### PR の自動レビュー

```yaml
# .github/workflows/gemini-review.yml
name: Gemini PR Review
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: google-github-actions/run-gemini-cli@v1
        with:
          prompt: |
            この PR の変更をレビューして。
            コード品質、パフォーマンス、セキュリティの観点でフィードバックして。
```

### Issue の自動トリアージ

```yaml
name: Gemini Issue Triage
on:
  issues:
    types: [opened]

jobs:
  triage:
    runs-on: ubuntu-latest
    steps:
      - uses: google-github-actions/run-gemini-cli@v1
        with:
          prompt: |
            この Issue を分析して、適切なラベルを提案して。
            優先度（high/medium/low）も判断して。
```

## 非対話モードでの自動化

```bash
# スクリプトからの実行
gemini --non-interactive "package.json の依存関係を最新にアップデートして"

# CI/CD パイプラインでの活用
gemini --non-interactive "CHANGELOG.md を最新のコミットから自動生成して"
```

## 次のステップ

- [Claude Codeとの連携](integration.md) — Gemini CLI と Claude Code を組み合わせた最強ワークフロー
