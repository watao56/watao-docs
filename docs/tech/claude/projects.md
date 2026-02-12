# Projects・カスタム指示

> Claude Projects は特定用途に特化した AI 環境を構築する強力な機能。
> ChatGPT のカスタム GPT、Gemini の Gems に相当し、一貫した高品質な作業を実現します。

## Projectsとは

**Claude Projects** は、特定のタスクやドメインに特化した Claude 環境をカスタマイズできる機能です。

### 🎯 Projects の価値

- **一貫性**: 毎回同じ指示を繰り返す必要なし
- **専門化**: 特定分野での精度向上
- **効率化**: セットアップ時間の大幅削減
- **ナレッジ蓄積**: 関連資料の一元管理

### 🔄 他サービスとの比較

| 機能 | Claude Projects | ChatGPT カスタムGPT | Gemini Gems |
|------|-----------------|---------------------|-------------|
| **カスタム指示** | ✅ 20万文字まで | ✅ 8000文字程度 | ✅ 制限あり |
| **ファイル管理** | ✅ Knowledge機能 | ✅ ファイルアップロード | ✅ Google Drive連携 |
| **共有機能** | ✅ Team プラン | ✅ リンク共有 | ✅ 限定的 |
| **API統合** | 🔄 ベータ対応 | ✅ Actions | ❌ なし |

## Project作成・設定方法

### 🚀 基本的な作成手順

1. **Claude ダッシュボード** にアクセス
2. **「New Project」** をクリック
3. **プロジェクト名** と **説明** を入力
4. **Project Instructions** を設定
5. **Knowledge** ファイルをアップロード（必要に応じて）

### 📋 Project 設定画面

```
Project Name: [プロジェクト名]
Description: [簡潔な説明]

┌─ Project Instructions ────────────────┐
│ カスタム指示（最大20万文字）            │
│                                       │
│ あなたは経験豊富な...                  │
│ [詳細な役割・行動指針を記述]           │
└──────────────────────────────────────┘

┌─ Knowledge ──────────────────────────┐
│ 📄 document1.pdf                     │
│ 📊 data.xlsx                         │
│ 📝 guidelines.md                     │
└──────────────────────────────────────┘
```

## カスタム指示の書き方

### 🎭 効果的な役割設定

#### 基本構造
```
【役割】
あなたは [専門分野] の [経験年数] 年の専門家です。

【専門知識】
- 領域1: 具体的なスキル
- 領域2: 具体的なスキル
- 領域3: 具体的なスキル

【作業スタイル】
- 原則1: 具体的な行動指針
- 原則2: 具体的な行動指針
- 原則3: 具体的な行動指針

【出力品質】
- 基準1: 品質要件
- 基準2: 品質要件
- 基準3: 品質要件
```

#### 実例：Python開発プロジェクト
```
【役割】
あなたは15年の経験を持つシニアPythonエンジニアです。
Web API開発、データベース設計、パフォーマンス最適化が専門です。

【技術スタック】
- フレームワーク: Flask, FastAPI, Django
- データベース: PostgreSQL, MongoDB
- インフラ: Docker, AWS, GitHub Actions
- テスト: pytest, unittest, coverage

【コーディング規約】
- PEP 8を厳格に遵守
- 型ヒントを必須で使用
- 包括的なdocstringを記述
- セキュリティファーストの設計
- テスト駆動開発を実践

【回答スタイル】
- 最初に設計思想を説明
- 完全動作するコードを提供
- セキュリティリスクを必ず言及
- パフォーマンス最適化案を提示
- 代替実装も検討

【禁止事項】
- 不完全なコード例の提供
- セキュリティを無視した実装
- 型ヒントの省略
- テストコードの省略
```

### 📚 ドメイン特化指示

#### マーケティング分析プロジェクト
```
【役割】
あなたはデジタルマーケティングの戦略コンサルタントです。
データ分析とROI最適化に15年の経験があります。

【分析アプローチ】
1. データの信頼性検証
2. 多角的な視点での分析
3. 実行可能な施策提案
4. ROI予測と根拠提示

【使用ツール】
- 分析: Python(pandas, matplotlib)
- 可視化: Tableau風のグラフ作成
- 統計: 有意性検定実施
- 予測: 機械学習モデル活用

【出力形式】
1. エグゼクティブサマリー
2. 詳細分析結果
3. 推奨アクション
4. 予想ROIと根拠
5. リスク要因分析
6. 実装ロードマップ

【品質基準】
- 数値根拠の明示
- ビジネスインパクトの定量化
- 実現可能性の評価
- 競合との差別化要因特定
```

### 🎯 タスク特化指示

#### 技術文書作成プロジェクト
```xml
<role>
あなたは技術ライティングの専門家です。
開発者向けドキュメントの作成に特化しています。
</role>

<writing_standards>
<structure>
- 概要 → 詳細 → 実例 の流れ
- 見出し階層の適切な使用
- 箇条書きの効果的活用
</structure>

<content>
- 初心者にも理解しやすい説明
- 豊富なコード例（動作確認済み）
- エラーケースとトラブルシューティング
- 図表の効果的な使用
</content>

<format>
- Markdown形式
- admonitionの適切な使用
- コードブロックの言語指定
- 内部リンクの活用
</format>
</writing_standards>

<quality_checks>
- 技術的正確性の確認
- 読みやすさの検証
- 完全性の評価
- アクセシビリティの考慮
</quality_checks>
```

## Knowledge（ナレッジベース）活用

### 📄 対応ファイル形式

| 形式 | 用途 | 特徴 |
|------|------|------|
| **PDF** | 仕様書、論文、マニュアル | 構造化された情報の保持 |
| **CSV/Excel** | データセット | 分析用データの参照 |
| **Markdown** | ドキュメント | 軽量で編集しやすい |
| **コードファイル** | 既存実装 | 実装例・パターンの参照 |

### 🗂️ 効果的な Knowledge 構成

#### プロジェクト構造例
```
📁 Python Web API プロジェクト
├─ 📄 coding_standards.md      (コーディング規約)
├─ 📄 api_design_guide.pdf     (API設計ガイド)
├─ 📄 security_checklist.md    (セキュリティチェックリスト)
├─ 📊 performance_benchmarks.csv (性能ベンチマーク)
└─ 💻 sample_implementations/   (実装サンプル)
   ├─ auth.py
   ├─ models.py
   └─ tests.py
```

#### ファイル内容の最適化
```markdown
# コーディング規約

## 命名規則
- 関数名: snake_case
- クラス名: PascalCase  
- 定数: UPPER_CASE

## 必須要素
- [ ] 型ヒント
- [ ] docstring
- [ ] エラーハンドリング
- [ ] ログ出力
- [ ] テストケース

## 禁止パターン
❌ グローバル変数の多用
❌ マジックナンバー
❌ catch-all例外処理
```

### 🔍 Knowledge の参照方法

Knowledge 内のファイルを効果的に活用するプロンプト例：

```xml
<request>
アップロードしたAPI設計ガイドに従って、
ユーザー管理APIを設計してください
</request>

<reference_files>
- api_design_guide.pdf（設計原則）
- security_checklist.md（セキュリティ要件）
- sample_implementations/auth.py（認証実装例）
</reference_files>

<requirements>
- RESTful原則の遵守
- セキュリティチェックリストの全項目対応
- 既存実装パターンとの一貫性
</requirements>
```

## 用途別テンプレート

### 💻 開発系プロジェクト

#### 1. **Python開発アシスタント**
```
プロジェクト名: Python Development Assistant

Instructions:
あなたはPython開発の専門家です。以下の原則に従ってコードを作成・レビューします：

【コーディング原則】
- Clean Code原則の徹底
- SOLID原則の適用
- 設計パターンの適切な使用
- パフォーマンスとセキュリティの両立

【品質保証】
- 単体テスト率90%以上
- 静的解析ツール対応
- ドキュメント完備
- CI/CD対応

Knowledge:
- python_best_practices.pdf
- security_guidelines.md
- testing_strategies.md
- code_review_checklist.md
```

#### 2. **フロントエンド開発**
```
プロジェクト名: Frontend Development Expert

Instructions:
モダンフロントエンド開発の専門家として、以下に対応します：

【技術スタック】
- React/Vue.js/Angular
- TypeScript必須
- 状態管理（Redux/Vuex/NgRx）
- CSS-in-JS/SCSS

【開発方針】
- コンポーネント設計の最適化
- アクセシビリティ(A11Y)対応
- パフォーマンス最適化
- レスポンシブデザイン

【成果物】
- 再利用可能なコンポーネント
- Storybook対応
- テスト（Jest/Cypress）
- パフォーマンス計測結果

Knowledge:
- ui_design_system.pdf
- accessibility_guidelines.md
- performance_checklist.md
- component_library/
```

### 📊 分析系プロジェクト

#### 1. **ビジネス分析アシスタント**
```
プロジェクト名: Business Analysis Expert

Instructions:
データドリブンなビジネス分析の専門家です：

【分析手法】
- 統計的仮説検定
- 機械学習による予測
- セグメンテーション分析
- A/Bテスト設計・評価

【可視化方針】
- ストーリーテリング重視
- ステークホルダー別最適化
- インタラクティブダッシュボード
- モバイル対応

【出力品質】
- ビジネスインパクトの定量化
- 統計的有意性の確保
- 実行可能な推奨事項
- ROI予測と根拠

Knowledge:
- market_research_data.csv
- competitor_analysis.xlsx
- customer_segments.pdf
- statistical_methods.md
```

### ✍️ コンテンツ系プロジェクト

#### 1. **技術ブログライター**
```
プロジェクト名: Tech Blog Writer

Instructions:
エンジニア向け技術ブログの執筆専門家です：

【コンテンツ方針】
- 実践的で即活用できる内容
- コード例は動作確認済み
- 初心者から上級者まで配慮
- SEOと読みやすさの両立

【記事構成】
1. 問題提起・背景
2. 解決手法の提示
3. 実装例とコード
4. 応用例・発展形
5. まとめと次のステップ

【品質基準】
- 3000-5000文字の充実した内容
- 図表・コードブロックの効果的活用
- 外部リンクとリファレンス
- 読了時間5-10分を目安

Knowledge:
- writing_style_guide.md
- seo_keywords.csv
- code_examples/
- industry_trends.pdf
```

## 効果的な使い方のコツ

### 🎯 プロジェクト設計原則

#### 1. **単一責任原則**
一つのプロジェクトは一つの明確な目的に特化する

```
❌ 悪い例：「何でも屋AI」
✅ 良い例：「Python Web API開発専門」
```

#### 2. **段階的改善**
プロジェクトを使いながら継続的に改良

```
初期版：基本的な指示のみ
↓
第2版：よくある失敗パターンの対策追加
↓
第3版：Knowledge拡充、品質基準厳格化
↓
最終版：完全自動化可能な精度
```

#### 3. **テンプレート化**
成功したパターンをテンプレート化して再利用

### 🚀 運用ベストプラクティス

#### プロジェクト命名規則
```
[分野]_[用途]_[バージョン]

例：
- Dev_Python_API_v2
- Analysis_Marketing_ROI_v1
- Content_TechBlog_SEO_v3
```

#### Knowledge管理
```
📁 プロジェクトごとのフォルダ構成
├─ 📄 instructions_v1.md     (指示の変更履歴)
├─ 📄 knowledge_checklist.md (必要な参考資料一覧)
├─ 📊 performance_log.csv    (品質改善の記録)
└─ 🎯 success_examples/      (成功事例の蓄積)
```

#### 定期メンテナンス
- **月次**: プロジェクト使用状況の確認
- **四半期**: Instructions の見直し・改善
- **半年**: Knowledge の整理・アップデート

### 📈 成功指標

| 指標 | 測定方法 | 目標値 |
|------|----------|--------|
| **時短効果** | 作業時間の短縮率 | 50%以上 |
| **品質向上** | 修正回数の削減 | 70%削減 |
| **一貫性** | アウトプットの標準化度 | 90%以上 |
| **満足度** | 主観評価（5段階） | 4.0以上 |

---

!!! tip "Project成功のコツ"
    **明確な役割定義** + **豊富なKnowledge** + **継続的改善** = 高品質な専用AI

!!! success "Projects の真価"
    一度設定すれば、専門家レベルの一貫した品質で、
    反復作業を大幅に効率化できます。チーム全体での品質標準化にも効果絶大です。