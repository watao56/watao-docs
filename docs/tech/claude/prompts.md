# 効果的なプロンプト設計

> Claude で最高の結果を得るためのプロンプト設計技法。
> XMLタグ活用から長文コンテキストまで、Claude 特有の強みを活かしたテクニックを解説。

## Claude プロンプトの特徴

### 🎯 Claude が得意なこと

- **XMLタグ構造**: 構造化されたプロンプトの処理
- **長文理解**: 200K〜1M トークンの大容量処理
- **自然な日本語**: 文脈を理解した自然な応答
- **段階的思考**: Extended Thinking による深い推論

### 💡 基本原則

1. **明確性**: 求める結果を具体的に指定
2. **構造化**: XMLタグで情報を整理
3. **文脈提供**: 十分な背景情報を含める
4. **段階分割**: 複雑なタスクは段階的に指示

## XMLタグ活用（Claude独自の強み）

Claude は XMLタグを使った構造化プロンプトが非常に得意です。

### 📋 基本的なXMLタグパターン

#### 情報整理タグ
```xml
<context>
背景情報や前提条件
</context>

<task>
具体的に実行してほしいタスク
</task>

<requirements>
- 要件1
- 要件2
- 要件3
</requirements>

<output_format>
出力形式の指定
</output_format>
```

#### 実例：技術文書作成
```xml
<context>
新しいAPIサービスのドキュメントを作成します。
対象読者は開発者で、RESTful APIの基本知識はあります。
</context>

<task>
以下のAPIエンドポイントの詳細ドキュメントを作成してください。
</task>

<api_spec>
POST /api/users
- 新規ユーザー登録
- パラメータ: name, email, password
- レスポンス: user_id, created_at
</api_spec>

<requirements>
- OpenAPI形式準拠
- サンプルリクエスト/レスポンス付き
- エラーケースの説明
- cURL例も含める
</requirements>

<output_format>
Markdown形式で、以下の構成：
1. エンドポイント概要
2. パラメータ詳細
3. レスポンス例
4. エラー一覧
5. 使用例
</output_format>
```

### 🔄 複数データ処理タグ

```xml
<documents>
<document name="仕様書A">
[内容A]
</document>
<document name="仕様書B">
[内容B]
</document>
</documents>

<analysis_criteria>
- 機能比較
- 性能差
- コスト分析
</analysis_criteria>
```

### 💭 思考プロセスタグ

```xml
<thinking>
この問題について段階的に考えてください：
1. 現状分析
2. 課題特定
3. 解決策検討
4. 実装手順
</thinking>

<deliverable>
最終的に以下の成果物を提供：
- 分析レポート
- 実装計画
- リスク評価
</deliverable>
```

## 長文コンテキスト活用法

Claude の 200K〜1M トークンコンテキストを最大限活用する方法。

### 📚 大量文書処理

#### 効果的な構造化
```xml
<analysis_request>
以下の複数文書を分析してください
</analysis_request>

<document_set>
<document type="specification" name="現行システム仕様">
[数十ページの仕様書]
</document>
<document type="specification" name="新システム仕様">
[数十ページの仕様書]
</document>
<document type="requirements" name="要件定義書">
[要件定義の詳細]
</document>
</document_set>

<analysis_goals>
1. 仕様変更点の詳細比較
2. 移行時の影響範囲分析
3. 開発工数見積もり
4. リスク要因の特定
</analysis_goals>

<output_requirements>
- 項目ごとの詳細比較表
- 影響度マトリクス
- 移行計画案
- 各段階でのチェックポイント
</output_requirements>
```

### 🗂️ コードベース分析

```xml
<codebase_analysis>
大規模なコードベースのリファクタリング計画を立ててください
</codebase_analysis>

<code_files>
<file name="main.py" path="src/">
[Pythonメインファイル]
</file>
<file name="models.py" path="src/">
[モデル定義]
</file>
<file name="views.py" path="src/">
[ビュー実装]
</file>
<!-- 数十ファイルを含める -->
</code_files>

<refactoring_criteria>
- 可読性向上
- 性能最適化
- テスタビリティ改善
- 保守性向上
</refactoring_criteria>
```

## System Prompt設計

Projects や API で使用する System Prompt の効果的な設計法。

### 🎯 専門家キャラクター設定

```
あなたは15年の経験を持つシニアPythonエンジニアです。

専門分野：
- Web API開発（Flask/FastAPI）
- データベース設計
- パフォーマンス最適化
- セキュリティ対策

コーディングスタイル：
- PEP 8完全準拠
- 型ヒント必須
- 包括的なdocstring
- 防御的プログラミング
- テスト駆動開発

回答形式：
- 最初に概要を説明
- コード例は完全動作するものを提供
- セキュリティリスクがあれば必ず言及
- 代替案も提示
```

### 📋 タスク特化設定

```
あなたは技術ドキュメント作成の専門家です。

作成するドキュメント：
- API仕様書
- 設計書
- 手順書
- トラブルシューティングガイド

品質基準：
- 読みやすい構成（見出し、箇条書き活用）
- 具体例を豊富に含める
- 初心者にも理解しやすい説明
- 図表を効果的に使用
- エラーケースも詳述

出力形式：Markdown、適切なadmonition使用
```

## タスク別プロンプト例

### 🖥️ コーディング

#### Python開発支援
```xml
<coding_request>
以下の要件でPythonコードを作成してください
</coding_request>

<requirements>
- FastAPIでREST API作成
- PostgreSQLとの連携
- JWT認証実装
- 非同期処理対応
- エラーハンドリング充実
</requirements>

<deliverables>
1. 完全動作するコード
2. 必要な依存関係（requirements.txt）
3. 環境構築手順
4. API使用例
5. テストコード
</deliverables>

<coding_standards>
- 型ヒント必須
- docstring完備
- PEP 8準拠
- セキュリティ考慮
</coding_standards>
```

#### コードレビュー
```xml
<code_review>
以下のコードをレビューしてください
</code_review>

<code>
[レビュー対象コード]
</code>

<review_criteria>
- 可読性
- 性能
- セキュリティ
- 保守性
- ベストプラクティス準拠
</review_criteria>

<output_format>
1. 総合評価（S/A/B/C）
2. 良い点
3. 改善点（重要度順）
4. 具体的な修正提案
5. 追加の推奨事項
</output_format>
```

### 📝 文書作成

#### 技術仕様書作成
```xml
<document_creation>
新機能の技術仕様書を作成してください
</document_creation>

<feature_description>
[機能概要と背景]
</feature_description>

<stakeholders>
- 開発チーム
- QAチーム
- プロダクトマネージャー
</stakeholders>

<document_sections>
1. 機能概要
2. 技術要件
3. アーキテクチャ設計
4. API仕様
5. データ構造
6. セキュリティ考慮事項
7. 性能要件
8. テスト戦略
9. 実装計画
10. リスク分析
</document_sections>

<quality_standards>
- 各セクション500-1000文字
- 図表を効果的に使用
- 具体例を豊富に含める
- 技術的詳細と全体像のバランス
</quality_standards>
```

### 📊 データ分析

```xml
<data_analysis>
売上データの詳細分析を行ってください
</data_analysis>

<dataset>
[CSVデータまたはデータ説明]
</dataset>

<analysis_objectives>
1. トレンド分析（時系列）
2. 商品カテゴリ別パフォーマンス
3. 地域別売上分析
4. 季節性の特定
5. 異常値の検出と原因分析
</analysis_objectives>

<deliverables>
1. データクリーニング済みセット
2. 各分析の可視化グラフ
3. 統計的検証結果
4. インサイトと推奨アクション
5. 来期予測モデル
</deliverables>

<tools>
Python (pandas, matplotlib, seaborn, scikit-learn)
</tools>
```

### 🌍 翻訳・ローカライゼーション

```xml
<translation_request>
技術文書の日英翻訳を行ってください
</translation_request>

<source_text>
[翻訳対象テキスト]
</source_text>

<translation_requirements>
- 技術用語の正確性
- 文脈の保持
- 読みやすさの確保
- 文化的配慮
</translation_requirements>

<target_audience>
英語圏の開発者（中級〜上級レベル）
</target_audience>

<output_format>
1. 翻訳文
2. 用語集（技術用語の対訳）
3. 翻訳ノート（判断に迷った箇所の説明）
4. カルチャライゼーション提案
</output_format>
```

## ChatGPT・Geminiプロンプトとの違い

### 🔄 ChatGPT vs Claude

| 側面 | ChatGPT | Claude |
|------|---------|--------|
| **構造化** | Markdown中心 | XML タグが効果的 |
| **長文処理** | 分割して処理 | 一括処理が得意 |
| **思考プロセス** | ステップバイステップ | Extended Thinking |
| **コード生成** | 実行可能性重視 | 設計・品質重視 |

#### ChatGPT スタイル（参考）
```
ステップバイステップで考えてください：

1. まず問題を分析
2. 次に解決策を検討
3. 最後に実装案を提示

各ステップで詳しく説明してください。
```

#### Claude 最適化スタイル
```xml
<analysis_process>
以下の手順で段階的に分析してください：
1. 問題の構造化
2. 根本原因分析
3. 複数解決策の比較検討
4. 最適解の選定と根拠
</analysis_process>

<thinking_depth>
Extended Thinking を活用して、
表面的でない洞察を提供してください
</thinking_depth>
```

### 💎 Gemini vs Claude

| 側面 | Gemini | Claude |
|------|--------|--------|
| **検索連携** | Google検索統合 | Web検索機能 |
| **長文処理** | 1M+トークン | 1Mトークン（構造化得意） |
| **コーディング** | 幅広い対応 | 深い設計思考 |
| **日本語品質** | 自然 | より洗練された表現 |

### 🚀 Claude 特化テクニック

#### 1. **階層的情報整理**
```xml
<project_structure>
  <overview>全体概要</overview>
  <phases>
    <phase number="1">
      <goals>目標</goals>
      <tasks>具体的タスク</tasks>
      <deliverables>成果物</deliverables>
    </phase>
  </phases>
</project_structure>
```

#### 2. **多角的分析指示**
```xml
<analysis_perspectives>
<perspective name="技術的">技術実装の観点</perspective>
<perspective name="ビジネス的">ROIとコスト</perspective>
<perspective name="ユーザー的">UX とユーザビリティ</perspective>
<perspective name="運用的">保守性とスケーラビリティ</perspective>
</analysis_perspectives>
```

#### 3. **品質基準の明示化**
```xml
<quality_criteria>
<technical>
- コードの可読性
- 性能要件達成
- セキュリティ考慮
</technical>
<documentation>
- 完全性
- 正確性
- わかりやすさ
</documentation>
</quality_criteria>
```

---

!!! tip "プロンプト作成のコツ"
    **XMLタグ**: 情報を構造化  
    **具体例**: 期待値を明確に  
    **段階分割**: 複雑タスクは分解  
    **品質基準**: 明確な評価軸を設定

!!! success "Claude の真価を引き出す"
    XMLタグ + 長文コンテキスト + Extended Thinking の組み合わせで、
    他のAIでは不可能な高品質・高精度な結果を得られます。