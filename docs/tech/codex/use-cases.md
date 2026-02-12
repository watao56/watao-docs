# Codex CLI 活用事例

## コード生成・リファクタリング

### 🆕 新機能の実装

**シナリオ：** React プロジェクトにユーザー認証機能を追加

```bash
# 認証機能の一括生成
codex "React + TypeScriptでユーザー認証機能を実装してください。
- ログイン/ログアウト画面
- JWT トークン管理
- 認証状態管理（Context API使用）
- 保護されたルート機能"

# 生成後の調整
codex edit src/auth/ "セキュリティを強化し、エラーハンドリングを改善"
```

**生成例：**
```
✓ src/context/AuthContext.tsx
✓ src/components/LoginForm.tsx  
✓ src/components/ProtectedRoute.tsx
✓ src/hooks/useAuth.ts
✓ src/utils/tokenManager.ts
✓ src/types/auth.ts
```

### 🔧 レガシーコードのモダン化

```bash
# 古いJavaScriptをTypeScriptに変換
codex "このJSファイルをTypeScriptに変換し、モダンな書き方に更新"

# クラスコンポーネントを関数コンポーネントに
codex "React クラスコンポーネントをHooksを使った関数コンポーネントに変換"

# 非同期処理の改善
codex "Promise チェーンを async/await に書き換え、エラーハンドリングを強化"
```

!!! example "実際の変換例"
    **変換前（レガシー）:**
    ```javascript
    class UserService {
      getUser(id) {
        return fetch(`/api/users/${id}`)
          .then(res => res.json())
          .catch(err => console.error(err));
      }
    }
    ```
    
    **変換後（モダン）:**
    ```typescript
    export class UserService {
      async getUser(id: string): Promise<User | null> {
        try {
          const response = await fetch(`/api/users/${id}`);
          if (!response.ok) {
            throw new Error(`User fetch failed: ${response.status}`);
          }
          return await response.json() as User;
        } catch (error) {
          console.error('Failed to fetch user:', error);
          return null;
        }
      }
    }
    ```

## マルチファイル編集

### 📁 プロジェクト横断的な変更

**API エンドポイント変更の一括対応：**

```bash
# プロジェクト全体のAPI URL更新
codex full-auto "APIのベースURLを '/api/v1' から '/api/v2' に変更し、
新しいレスポンス形式に対応してください。影響するすべてのファイルを更新"
```

**Codex が自動的に検出・更新：**
- API クライアント設定
- コンポーネント内のフェッチ処理  
- 型定義ファイル
- テストファイル
- 設定ファイル

### 🎨 デザインシステム導入

```bash
# 新しいデザインシステムの適用
codex "Material-UIからChakra UIに移行してください。
既存のすべてのコンポーネントを新しいライブラリで書き直し、
統一されたテーマシステムを適用"
```

!!! tip "マルチファイル編集のコツ"
    - **具体的なスコープを指定**：「すべてのReactコンポーネント」など
    - **段階的な実行**：大規模変更は小分けして実行
    - **事前バックアップ**：`git commit` してから実行

## 自律モードの使い分け

### 🤖 auto-edit モード

**最適な用途：**
- 単一ファイルの修正
- 明確で限定的なタスク  
- 既存コードの改善

```bash
# 特定ファイルの自動改善
codex auto-edit src/components/Header.tsx

# 複数ファイルの類似修正
codex auto-edit "src/components/*.tsx" "TypeScriptの型注釈を改善"
```

**設定例：**
```json
{
  "auto_edit": {
    "max_changes_per_file": 10,
    "require_confirmation": false,
    "backup_before_edit": true
  }
}
```

### 🚀 full-auto モード

**最適な用途：**
- プロジェクト全体のリファクタリング
- 新機能の包括的な実装
- アーキテクチャ変更

```bash
# プロジェクト全体の自動化
codex full-auto "このプロジェクトにESLint + Prettier + Husky を導入し、
コード品質を向上させてください"

# 包括的なテスト追加
codex full-auto "すべてのコンポーネントにJest + React Testing Library
でテストを追加してください"
```

!!! warning "full-auto 使用時の注意"
    - **小さなプロジェクト**から試す
    - **重要な作業前は必ずバックアップ**
    - **段階的に適用**して動作確認

## 実用的なワークフロー

### 🔄 日常的な開発フロー

**朝の準備：**
```bash
# 昨日からの変更を確認
git log --oneline -10

# Codexでコードレビュー
codex "昨日のコミットを確認し、改善点があれば指摘してください"

# 今日のタスクを計画
codex "今日実装予定の機能について、必要なファイルと手順を教えて"
```

**開発中：**
```bash
# 新機能の実装
codex "ユーザープロフィール編集機能を実装"

# バグ修正
codex "このエラーログを見て原因を特定し、修正してください"

# テスト追加
codex "この関数のテストケースを追加"
```

**コミット前：**
```bash
# コード品質チェック
codex pre-commit

# コミットメッセージの提案
codex "今回の変更に適したコミットメッセージを提案してください"
```

### 🎯 特定シナリオ別ワークフロー

**新人研修・オンボーディング：**
```bash
# プロジェクト構造の説明
codex "このプロジェクトの構造と主要ファイルを説明してください"

# コーディング規約の学習
codex "このプロジェクトのコーディング規約を調べ、
新人向けのガイドラインを作成してください"
```

**レガシーシステム改善：**
```bash
# 技術的負債の特定
codex analyze "技術的負債と改善すべきコードを特定してください"

# 段階的なリファクタリング計画
codex "このモジュールをモダンな構造に移行する計画を作成してください"
```

**パフォーマンス最適化：**
```bash
# ボトルネック分析
codex "このコードのパフォーマンス問題を特定し、最適化してください"

# バンドルサイズ削減
codex "不要なライブラリを特定し、軽量な代替案を提案してください"
```

### 💡 チーム開発でのベストプラクティス

**コードレビューサポート：**
```bash
# レビュー前のセルフチェック
codex "このプルリクエストの変更を確認し、
潜在的な問題や改善点を指摘してください"

# 説明文の作成
codex "この変更の目的と影響を説明するPR説明文を作成してください"
```

**ドキュメント自動生成：**
```bash
# API ドキュメント生成
codex "この API の OpenAPI 仕様書を生成してください"

# README更新  
codex "プロジェクトの変更に合わせて README.md を更新してください"
```

**ペアプログラミング支援：**
```bash
# アルゴリズムの相談
codex "この問題を解決する最適なアプローチは何ですか？"

# 設計レビュー
codex "このクラス設計について、SOLID原則の観点から評価してください"
```

!!! success "効果的な活用のポイント"
    1. **小さく始める** - 重要でないファイルで練習
    2. **段階的に拡大** - 成功したパターンを他に応用  
    3. **チーム共有** - 効果的な使い方を共有
    4. **継続的改善** - 設定とワークフローを定期的に見直し

## 次のステップ

- [ツール比較](comparison.md) - 他のAIコーディングツールとの詳細比較
- [セットアップ](setup.md) - より詳細な設定オプション

!!! tip "さらなる学習リソース"
    - **公式ドキュメント**: [developers.openai.com/codex](https://developers.openai.com/codex/)
    - **GitHub リポジトリ**: [github.com/openai/codex](https://github.com/openai/codex)
    - **コミュニティ**: Codex CLI ユーザー向けDiscussions