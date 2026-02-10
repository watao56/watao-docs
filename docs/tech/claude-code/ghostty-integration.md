# Claude Code × Ghostty ターミナル

Ghostty は Mitchell Hashimoto（Vagrant / Terraform の作者）が開発したネイティブレンダリングのターミナルエミュレータ。Claude Code CLI との組み合わせは、最も軽量で高速なワークフローを実現する。

## なぜ Ghostty + CLI か

| 比較項目 | VS Code 拡張 × 2窓 | Ghostty × 2タブ |
|----------|-------------------|----------------|
| メモリ使用量 | ~8GB | ~500MB |
| レスポンス | 入力遅延あり | 即時応答 |
| ファン | 常時回転 | 静音 |
| アップデート | 遅延あり | CLI は即日反映 |

- **ネイティブレンダリング**: Electron 不使用、キー入力即応答
- **CLI ファースト**: Claude Code の新機能は CLI から先にリリースされる
- **超軽量**: 10 セッション同時起動でもマシンに負荷なし

## Ghostty のインストールと基本設定

### インストール

```bash
# macOS
# ghostty.org からダウンロードして Applications にドラッグ

# Linux (ソースビルド or パッケージマネージャ)
# ディストリビューションに応じた方法でインストール
```

### 設定ファイル

`~/.config/ghostty/config` に設定を記述する:

```ini
# テーマ（1000以上のテーマから選択可能）
theme = catppuccin-mocha

# フォント設定
font-family = JetBrains Mono
font-size = 14

# パディング（見やすさ向上）
window-padding-x = 10
window-padding-y = 10

# スクロールバック行数
scrollback-limit = 10000

# カーソル
cursor-style = bar
cursor-style-blink = false

# ウィンドウ装飾（macOS で統合タイトルバー）
macos-titlebar-style = hidden
```

### おすすめフォント

| フォント | 特徴 |
|----------|------|
| **JetBrains Mono** | リガチャ対応、コーディング最適化 |
| **Fira Code** | リガチャ豊富、視認性高い |
| **Monaspace Neon** | GitHub 製、可変フォント |
| **UDEV Gothic** | 日本語対応、Nerd Font 内蔵 |

### おすすめカラースキーム

| テーマ | 雰囲気 |
|--------|--------|
| `catppuccin-mocha` | 暖かみのあるダーク |
| `tokyonight` | クール系ダーク |
| `gruvbox-dark` | レトロ系ダーク |
| `rose-pine` | パステル系ダーク |

## ショートカットキー

### 基本操作

| 操作 | macOS | Linux |
|------|-------|-------|
| 水平分割 | `Cmd+D` | `Ctrl+Shift+E` |
| 垂直分割 | `Cmd+Shift+D` | `Ctrl+Shift+O` |
| 新規タブ | `Cmd+T` | `Ctrl+Shift+T` |
| 分割切り替え | `Cmd+[` / `Cmd+]` | フォーカス移動 |
| タブ切り替え | `Cmd+数字` | `Ctrl+数字` |

### Shift+Enter の設定

Claude Code で複数行入力するために Shift+Enter を設定する:

```ini
# ~/.config/ghostty/config に追加
keybind = shift+enter=text:\x0a
```

!!! tip "代替方法"
    `\` + Enter でも改行できる。Ghostty では Shift+Enter がデフォルトで動作するが、明示的に設定しておくと確実。

## 実践的なワークフロー

### 基本レイアウト: 2ペイン構成

```
┌─────────────────────────┬──────────────┐
│                         │              │
│     Claude Code         │  dev server  │
│     (メインペイン)       │  / テスト     │
│                         │              │
│                         │              │
└─────────────────────────┴──────────────┘
```

1. Ghostty を開いて `Cmd+D` で水平分割
2. 左ペインで `cd ~/project && claude` を起動
3. 右ペインで `npm run dev` や `cargo watch` を起動
4. Claude が変更を加えると、右ペインでリアルタイムに結果を確認

### マルチプロジェクト: タブ活用

```
[Tab 1: プロジェクトA] [Tab 2: プロジェクトB] [Tab 3: プロジェクトC]
```

各タブで別の Claude Code セッションを起動。タブ切り替え（`Cmd+数字`）で即座にプロジェクトを行き来。

### 上級パターン: 同一プロジェクト多角アプローチ

git worktree を活用して、同じリポジトリで並行作業する:

```bash
# worktree を作成
git worktree add ../myapp-tests feature/tests
git worktree add ../myapp-docs feature/docs

# タブ1: 機能開発
cd ~/projects/myapp && claude

# タブ2: テスト作成
cd ~/projects/myapp-tests && claude

# タブ3: ドキュメント作成
cd ~/projects/myapp-docs && claude
```

各タブの Claude は独立したコンテキストで動作するため、互いに干渉しない。

## パフォーマンス最適化

### Ghostty 側の最適化

```ini
# GPU レンダリング（デフォルトで有効）
# 特に設定不要だが、問題がある場合は以下で調整
# renderer = opengl

# フレームレート制限（バッテリー節約）
# repaint-hz = 60

# 大量出力時のパフォーマンス
scrollback-limit = 5000
```

### Claude Code 側の最適化

```bash
# コンテキストが重くなったらリセット
# Claude Code 内で:
/clear

# 定期的にコンパクション
/compact

# 不要なファイル読み込みを避ける .claudeignore
echo "node_modules/\ndist/\n.git/" > .claudeignore
```

### シェル統合の設定

```bash
# .zshrc / .bashrc に追加
# Claude Code のパスを通す
export PATH="$HOME/.npm-global/bin:$PATH"

# エイリアス
alias cc="claude"
alias ccc="claude --continue"
alias ccr="claude --resume"
```

## tmux との比較

| 機能 | Ghostty ネイティブ分割 | tmux |
|------|----------------------|------|
| セットアップ | 不要 | 要インストール・設定 |
| マウス操作 | 完全対応 | 設定が必要 |
| GPU レンダリング | 全ペイン対応 | 制限あり |
| コピーペースト | OS ネイティブ | tmux バッファ経由 |
| セッション永続化 | ❌ | ✅ |

!!! note "tmux が必要な場合"
    SSH 先でのセッション維持が必要なら tmux を使う。ローカル作業なら Ghostty ネイティブ分割で十分。

## まとめ

- **Ghostty + Claude Code CLI** はメモリ効率・応答速度で最強の組み合わせ
- 分割・タブを活用して複数セッションを軽量に並行運用
- Shift+Enter 設定を忘れずに
- worktree + 複数タブで同一プロジェクトの多角的開発が可能
