# Claude Code ベストプラクティス

Claude Code は Anthropic が提供するターミナルネイティブな AI コーディングエージェント。単なるチャットボットではなく、ファイルの読み書き、コマンド実行、自律的な問題解決が可能。

このセクションでは、Claude Code を最大限活用するためのベストプラクティスをまとめる。

## ガイド一覧

| ガイド | 内容 |
|--------|------|
| [Ghostty × Claude Code](ghostty-integration.md) | Ghostty ターミナルとの組み合わせによる最速ワークフロー |
| [CLAUDE.md / AGENTS.md / Skills ガイド](md-files-guide.md) | 設定ファイルの書き方・管理方法 |
| [CLI 連携ガイド](cli-integration.md) | 他の AI ツールや MCP サーバーとの連携 |
| [Tips & Tricks](tips-and-tricks.md) | コスト最適化、hooks、カスタムコマンド等の小技集 |

## 最も重要な原則

### 1. コンテキストウィンドウを管理する

Claude Code のパフォーマンスに最も影響するのは**コンテキストウィンドウの使用量**。ウィンドウが埋まるほど性能が劣化する。

- `/clear` を積極的に使い、コンテキストをリセットする
- 1 セッション = 1 タスクを心がける
- `/compact` で会話を要約してコンテキストを解放する

### 2. 検証手段を与える

Claude に「自分で確認する方法」を与えると品質が劇的に向上する。

```
❌ 「メール検証関数を書いて」
✅ 「validateEmail 関数を書いて。test@example.com は true、invalid は false になるテストも書いて実行して」
```

### 3. 探索 → 計画 → 実装

いきなりコーディングさせず、Plan Mode（`Shift+Tab` × 2）で段階的に進める。

1. **探索**: コードベースを理解する
2. **計画**: 実装方針をレビューする
3. **実装**: 計画に基づいてコードを書く
4. **検証**: テスト・ビルドで確認する

### 4. 具体的なコンテキストを提供する

```
❌ 「ログインのバグを直して」
✅ 「セッションタイムアウト後にログインが失敗する。src/auth/ のトークンリフレッシュ処理を確認して、
    再現テストを書いてから修正して」
```

## 公式ドキュメント

- [Claude Code Docs](https://code.claude.com/docs/en/best-practices)
- [Claude Code GitHub](https://github.com/anthropics/claude-code)
- [Model Context Protocol](https://modelcontextprotocol.io/)
