# Python Quest - 設計書

> 子供向けPython学習プラットフォーム  
> Scratch → 変換 → Python の段階的アプローチでプログラミング的思考を育てる  
> ※ 本設計書はシニアエンジニアレビュー第1回・第2回（2026-02-16）の指摘を反映済み

---

## 1. コンセプト

### 1.1 ビジョン
小中学生がプログラミングに興味を持ち、論理的思考を身につけるためのゲーム型学習プラットフォーム。いきなりコードを書かせるのではなく、ビジュアルプログラミング（Scratch風ブロック）から段階的にPythonへ移行する。

### 1.2 ターゲット
- **年齢層:** 小学校高学年〜中学生
- **前提知識:** プログラミング経験なし
- **ゴール:** プログラミングへの興味喚起 + 論理的思考の習得

### 1.3 コア体験
- 問題に対してコード（またはブロック）を書き、**結果が視覚的・アニメーション付きで表示**される
- RPG風のクエスト形式で**ゲームとして楽しめる**
- 同じ概念をブロック→変換→コードの3段階で繰り返す**螺旋型カリキュラム**

---

## 2. ワールド構成（カリキュラム設計）

### 2.1 全体構成

```
🌱 ワールド1：スクラッチの森（ブロックプログラミング）
🔀 ワールド2：変換の洞窟（ブロック⇔コード対応）
🐍 ワールド3：Pythonの大地（テキストコーディング）
🎨 ワールド4：turtleの草原（応用・お絵かき）
```

### 2.2 ワールド1：スクラッチの森 🌱

**モード:** ブロックエディタ（Blockly）  
**目的:** プログラミングの基本概念をビジュアルで理解する  
**問題数:** 18問（基本・練習・チャレンジの3段階構成）

| クエスト | 学習内容 | 段階 | 概要 |
|---------|---------|------|------|
| 1-1 | 出力 | 基本 | ブロックを使って文字を表示する |
| 1-2 | 出力 | 練習 | 複数行の出力を組み立てる |
| 1-3 | 文字列結合 | 基本 | 文字をつなげるブロックを組む |
| 1-4 | 文字列結合 | 練習 | 名前と挨拶を組み合わせる |
| 1-5 | 計算 | 基本 | 足し算・引き算のブロック |
| 1-6 | 計算 | チャレンジ | 複雑な計算式を組む |
| 1-7 | 変数 | 基本 | 箱に値を入れるイメージ |
| 1-8 | 変数 | 練習 | 変数を使って計算する |
| 1-9 | 条件分岐 | 基本 | if / else のブロック |
| 1-10 | 条件分岐 | 練習 | 複数条件の分岐 |
| 1-11 | 条件分岐 | チャレンジ | ネストした条件分岐 |
| 1-12 | くり返し | 基本 | forループのブロック |
| 1-13 | くり返し | 練習 | ループと変数の組み合わせ |
| 1-14 | リスト | 基本 | 複数の値をまとめる |
| 1-15 | リスト | 練習 | リストをループで表示する |
| 1-16 | 関数 | 基本 | 処理をまとめるブロック（W3との螺旋対応） |
| 1-17 | 関数 | 練習 | 引数を持つ関数ブロック |
| 👹 ボス | FizzBuzz | — | 条件分岐 + くり返しの総合問題 |

> **螺旋型設計:** ワールド1で「関数ブロック」を体験させることで、ワールド3の `def` との対応関係を自然に理解できるようにする。

### 2.3 ワールド2：変換の洞窟 🔀

**モード:** ブロックとPythonコードの並列表示  
**目的:** ブロックとコードの対応関係を理解する  
**問題数:** 16問（基本・練習・チャレンジの3段階構成）

| クエスト | 学習内容 | 段階 | 概要 |
|---------|---------|------|------|
| 2-1 | print() | 基本 | ブロック→コードの対応を見る |
| 2-2 | print() | 練習 | 出力コードの穴埋め |
| 2-3 | 文字列 | 基本 | 文字列操作の対応 |
| 2-4 | 文字列 | 練習 | 文字列結合のコード変換 |
| 2-5 | 演算子 | 基本 | 計算式の書き方 |
| 2-6 | 変数 | 基本 | 変数宣言・代入の対応 |
| 2-7 | 変数 | 練習 | 変数を使った式の変換 |
| 2-8 | if文 | 基本 | 条件分岐のコード表現 |
| 2-9 | if文 | 練習 | elif/elseの対応 |
| 2-10 | if文 | チャレンジ | 複雑な条件式の変換 |
| 2-11 | forループ | 基本 | くり返しのコード表現 |
| 2-12 | forループ | 練習 | range()の対応 |
| 2-13 | リスト | 基本 | リスト操作の対応 |
| 2-14 | リスト | チャレンジ | リストとループの組み合わせ |
| 2-15 | 関数 | 基本 | def文の対応 |
| 👹 ボス | コード当てクイズ | — | ブロックを見てPythonコードを選択 |

**クエスト形式例:**
- ブロックが表示され、対応するPythonコードの穴埋め
- Pythonコードが表示され、同じ動きをするブロックを組む
- ブロックとコードを線で結ぶマッチング問題

### 2.4 ワールド3：Pythonの大地 🐍

**モード:** コードエディタ（Monaco Editor）  
**目的:** 自力でPythonコードを書けるようになる  
**問題数:** 18問（基本・練習・チャレンジの3段階構成）

| クエスト | 学習内容 | 段階 | 概要 |
|---------|---------|------|------|
| 3-1 | print() | 基本 | はじめてのコーディング |
| 3-2 | print() | 練習 | 複数行の出力 |
| 3-3 | 文字列操作 | 基本 | 結合、f-string |
| 3-4 | 文字列操作 | チャレンジ | 文字列メソッド |
| 3-5 | 計算 | 基本 | 四則演算、型変換 |
| 3-6 | 計算 | 練習 | 複合演算 |
| 3-7 | 変数 | 基本 | 変数の宣言と使い方 |
| 3-8 | 変数 | 練習 | 変数の更新と再代入 |
| 3-9 | if/elif/else | 基本 | 条件分岐を書く |
| 3-10 | if/elif/else | 練習 | 複数条件の分岐 |
| 3-11 | if/elif/else | チャレンジ | 論理演算子を使った条件 |
| 3-12 | forループ | 基本 | range(), リスト反復 |
| 3-13 | forループ | 練習 | ネストしたループ |
| 3-14 | リスト | 基本 | リスト操作 |
| 3-15 | リスト | チャレンジ | リスト内包表記（入門） |
| 3-16 | 関数 | 基本 | def, 引数, 戻り値 |
| 3-17 | 関数 | 練習 | 関数の組み合わせ |
| 👹 ボス | FizzBuzz | — | コードで解く総合問題 |

### 2.5 ワールド4：turtleの草原 🎨

**モード:** コードエディタ + Canvas描画  
**目的:** コードで視覚的な作品を作る楽しさを知る  
**問題数:** 15問（基本・練習・チャレンジの3段階構成）

| クエスト | 学習内容 | 段階 | 概要 |
|---------|---------|------|------|
| 4-1 | 直線 | 基本 | forward(), backward() |
| 4-2 | 直線 | 練習 | 長さを変えて複数の線を引く |
| 4-3 | 回転 | 基本 | right(), left() |
| 4-4 | 回転 | 練習 | 角度を変えてジグザグ |
| 4-5 | 四角形 | 基本 | ループで図形を描く |
| 4-6 | 四角形 | チャレンジ | 入れ子の四角形 |
| 4-7 | 色 | 基本 | pencolor(), fillcolor() |
| 4-8 | 色 | 練習 | カラフルな模様 |
| 4-9 | 多角形 | 基本 | 関数で多角形を描く |
| 4-10 | 多角形 | チャレンジ | 星型を描く |
| 4-11 | 螺旋 | 基本 | ループ + 変数で螺旋模様 |
| 4-12 | 螺旋 | チャレンジ | 色を変えながら螺旋 |
| 4-13 | 自由制作 | — | お題に沿って自由に描く（★評価なし、提出でXP獲得） |
| 4-14 | 自由制作 | — | テーマを選んで自由に描く（★評価なし、提出でXP獲得） |
| 👹 ボス | アート課題 | — | 指定された図形を再現 |

> **自由制作について:** 自由制作クエスト（4-13, 4-14）は★評価の対象外。提出するだけでXP（50XP）を獲得できる。創造性を制限しない設計。

---

## 3. ゲームシステム

### 3.1 RPG要素

| 要素 | 説明 |
|------|------|
| **レベル** | XPを貯めてレベルアップ。称号が変わる |
| **XP（経験値）** | クエストクリアで獲得。★評価でボーナス |
| **コイン** | クリア報酬。将来的にキャラカスタマイズ等に使用 |
| **スター（★1〜3）** | クエスト毎の評価。コード行数・実行回数等で判定 |
| **実績** | 特定条件で解放される称号・バッジ |
| **連続ログイン** | 🔥ストリーク。モチベーション維持 |

> **連続ログインのペナルティなし設計:** ストリークが途切れてもペナルティはなし。再開時は「おかえり！」と褒めるメッセージを表示。子供にプレッシャーを与えない設計を徹底する。3日以上のブランク後は「またいっしょに冒険しよう！」とNPCが声かけ。

### 3.2 レベル・称号テーブル

| レベル | 必要XP | 称号 | 解放要素 |
|--------|--------|------|---------|
| 1 | 0 | コードのたまご | — |
| 2 | 100 | コードのひよこ | — |
| 3 | 300 | コードの見習い | — |
| 4 | 500 | コードの冒険者 | ワールド2解放 |
| 5 | 800 | コードの騎士 | — |
| 6 | 1200 | コードの魔法使い | ワールド3解放 |
| 7 | 1800 | コードの賢者 | — |
| 8 | 2500 | コードの達人 | ワールド4解放 |
| 9 | 3500 | コードの伝説 | — |
| 10 | 5000 | Pythonマスター | 全解放 |

### 3.3 NPCキャラクター

| キャラ | 名前 | 役割 | 登場ワールド |
|--------|------|------|-------------|
| 🧙 | パイソン先生 | メインガイド。問題の説明・ヒント | 全ワールド |
| 🐍 | パイちゃん | プレイヤーの相棒。リアクション担当 | 全ワールド |
| 🧩 | ブロッくん | ブロックの案内役 | ワールド1 |
| 🔀 | ヘンカンジャー | 変換の案内役 | ワールド2 |
| 🐢 | カメさん | turtle描画の案内役 | ワールド4 |

> **つまづき時の声かけ:** 一定時間（3分以上）操作がない場合、NPCが「大丈夫？ヒントを見てみよう！」と声かけする。離脱防止の重要な仕組み。

### 3.4 正解判定ロジック

| モード | 判定方法 |
|--------|---------|
| ブロック（W1） | 生成されたコードの出力を期待値と比較 |
| 変換（W2） | 穴埋め/選択の正誤判定 + コード出力比較 |
| コード（W3） | 標準出力を期待値と完全一致 or パターンマッチ |
| turtle（W4） | 座標・角度のステップ比較（下記参照） |

#### turtle（W4）の正解判定アルゴリズム

画像比較は色や線の太さのわずかな差で不正解になりがちなため、**座標・角度のステップ比較方式**を採用する：

```typescript
interface TurtleStep {
  action: 'forward' | 'backward' | 'right' | 'left' | 'penup' | 'pendown' | 'pencolor' | 'fillcolor';
  value: number | string;
}

// 正解判定: ステップ列の比較
function judgeTurtle(expected: TurtleStep[], actual: TurtleStep[]): boolean {
  // 1. ステップ数の一致確認
  // 2. 各ステップのアクション一致確認
  // 3. 数値は±5%の許容範囲で比較（子供の入力ミス許容）
  // 4. 色はRGB値の近似比較
}
```

- turtleのコマンドをフックしてステップ列を記録
- 期待するステップ列と比較（順序・値）
- 数値には±5%の許容範囲を設ける
- ボスクエストのみ最終座標の一致も確認

### 3.5 ★評価基準

| ★ | 条件 |
|---|------|
| ★0 | 「答えを見る」でクリア（XP半減） |
| ★1 | クリア |
| ★2 | ヒント未使用でクリア |
| ★3 | ヒント未使用 + 実行1回でクリア |

> **自由制作クエストは★評価対象外。** 提出するだけでXP獲得。

### 3.6 つまづき救済システム

クエストで行き詰まった生徒が離脱しないための段階的救済フロー：

```
ヒント1（方向性のヒント）
  ↓ 解けない場合
ヒント2（具体的なヒント）
  ↓ 解けない場合
ヒント3（ほぼ答えのヒント）
  ↓ 解けない場合
┌────────────────────────────────────┐
│ 選択肢:                            │
│ 📖 「答えを見る」（★0扱い、XP半減）  │
│ 🔄 「似た練習問題をやる」（同概念の別問題）│
│ 🧙 パイソン先生のアドバイス           │
│    （将来的にLLMで個別指導）          │
└────────────────────────────────────┘
```

- **「答えを見る」:** コードの答えを表示。★0としてクリア扱い、XPは通常の50%。後からやり直して★を上げられる
- **「似た練習問題をやる」:** 同じ概念の別問題へ誘導。クリアすると元の問題に戻る
- **NPCの声かけ:** 3分以上操作がない場合「大丈夫？」と表示。離脱防止

---

## 4. 画面設計

### 4.1 画面一覧

| 画面 | パス | 説明 |
|------|------|------|
| トップ | `/` | ログイン/招待コード入力 |
| ワールドマップ | `/map` | ワールド選択・クエスト選択 |
| クエスト（ブロック） | `/quest/block/:id` | ワールド1用。Blocklyエディタ |
| クエスト（変換） | `/quest/convert/:id` | ワールド2用。並列表示 |
| クエスト（コード） | `/quest/code/:id` | ワールド3,4用。Monacoエディタ |
| クリア演出 | モーダル | 正解時のアニメーション |
| 失敗画面 | モーダル | 不正解時のフィードバック + エラーメッセージ日本語化 |
| レベルアップ | モーダル | レベルアップ演出 |
| ダッシュボード | `/dashboard` | 進捗・ステータス・実績 |
| 先生用管理画面 | `/admin` | 生徒の進捗一覧・アカウント管理 |

### 4.2 ワールドマップ画面

```
┌────────────────────────────────────────────────────┐
│  🐍 Python Quest           Lv.3 ████░░ 350/500    │
├────────────────────────────────────────────────────┤
│                                                    │
│  🌱 スクラッチの森    [クリア済み ✓]                 │
│    ⭐-⭐-⭐-⭐-⭐-⭐-...-👹✓  (18問)               │
│                                                    │
│  🔀 変換の洞窟        [進行中 ▶]                    │
│    ⭐-⭐-⭐-●-○-○-...-🔒   (16問)                 │
│                                                    │
│  🐍 Pythonの大地      [ロック 🔒]                    │
│    🔒-🔒-🔒-...-🔒           (18問)                │
│                                                    │
│  🎨 turtleの草原      [ロック 🔒]                    │
│    🔒-🔒-🔒-...-🔒           (15問)                │
│                                                    │
└────────────────────────────────────────────────────┘
```

### 4.3 クエスト画面（ブロックモード / ワールド1）

```
┌──────────┬──────────────────┬──────────────────┐
│  問題文   │  Blockly         │  結果表示         │
│          │  ブロックエディタ  │                   │
│  🧩 NPC  │                  │  アニメーション     │
│  ミッション│  [ブロック置き場]  │  or              │
│          │                  │  コンソール出力     │
│  報酬     │                  │                   │
│          │    [▶ 実行する！]  │                   │
│  [💡ヒント] [📖答えを見る]    │                   │
└──────────┴──────────────────┴──────────────────┘
```

### 4.4 クエスト画面（変換モード / ワールド2）

```
┌──────────┬──────────┬──────────┬───────────────┐
│  問題文   │ ブロック  │ Python   │  結果表示      │
│          │ (左側)   │ (右側)   │               │
│  🔀 NPC  │          │          │  「同じ動き    │
│  ミッション│ [自動生成]│ [穴埋め] │   してるね！」 │
│          │    ↔     │          │               │
│  報酬     │  対応表示 │          │               │
└──────────┴──────────┴──────────┴───────────────┘
```

### 4.5 クエスト画面（コードモード / ワールド3,4）

```
┌──────────┬──────────────────┬──────────────────┐
│  問題文   │  Monaco Editor   │  結果表示         │
│          │  (コードエディタ)  │                   │
│  🧙 NPC  │                  │  出力             │
│  ミッション│  シンタックス     │  or              │
│          │  ハイライト付き    │  Canvas描画       │
│  報酬     │                  │  (turtle)        │
│          │    [▶ 実行する！]  │                   │
│  [💡ヒント] [📖答えを見る]    │  [エラー: 日本語] │
└──────────┴──────────────────┴──────────────────┘
```

---

## 5. 技術スタック

### 5.1 フロントエンド

| 役割 | 技術 | 備考 |
|------|------|------|
| フレームワーク | Next.js (React) | App Router, SSR/SSG |
| 言語 | TypeScript | 型安全 |
| ブロックエディタ | Blockly | Google製。ブロック→Python変換を標準サポート |
| コードエディタ | Monaco Editor | VSCode同等。**遅延ロード + 部分インポートで最適化**（5.7参照） |
| Python実行 | Skulpt | ブラウザ内実行。**実行層を抽象化して将来Pyodide移行可能**（5.6参照） |
| 可視化 | Canvas API | turtle描画 + 結果表示 |
| アニメーション | Framer Motion | UI演出（正解エフェクト等） |
| スタイリング | Tailwind CSS | ユーティリティファースト |
| 状態管理 | Zustand | 軽量。**ストア分割設計**（5.8参照） |
| データフェッチ | React Query (TanStack Query) | キャッシュ戦略（5.9参照）。リアルタイム更新対応が強み |

### 5.2 バックエンド

| 役割 | 技術 | 備考 |
|------|------|------|
| API | Next.js API Routes | フロントと同居 |
| DB | Supabase (PostgreSQL) | 認証+DB+リアルタイム |
| 認証 | Supabase Auth | **招待コード + ニックネーム + パスワード方式**（6.1参照） |
| ストレージ | Supabase Storage | アバター画像等 |
| RLS | Supabase RLS | **Row Level Security必須**（8.3参照） |

### 5.3 インフラ

| 役割 | 技術 | 備考 |
|------|------|------|
| ホスティング | Vercel | Next.jsとの相性◎。**Hobbyは商用不可、MVP後はPro前提**（11.1参照） |
| DB | Supabase Cloud | 無料枠あり |
| CDN | Vercel Edge | 自動 |
| 監視 | Sentry | エラー監視・パフォーマンス監視 |

### 5.4 開発ツール

| 役割 | 技術 |
|------|------|
| パッケージ管理 | pnpm |
| リンター | ESLint + Prettier |
| テスト | Vitest + Playwright |
| Git | GitHub |
| CI/CD | GitHub Actions → Vercel |

### 5.5 ブラウザ最低動作環境

学校PC環境を想定し、以下を最低動作環境として定義する。Blockly・Monaco Editor・Skulptの各ライブラリの要件を考慮し、ES2020+をベースラインとする。

| ブラウザ | 最低バージョン | 備考 |
|---------|--------------|------|
| **Google Chrome** | 90+ | GIGAスクール端末の標準ブラウザ |
| **Microsoft Edge** | 90+ | Chromiumベース。Windows学校PC |
| **Chromebook (ChromeOS)** | 90+ | GIGAスクール端末で採用多数 |
| **Safari** | 15+ | iPad導入校向け |
| **Firefox** | 90+ | 一部自治体で採用 |

**非サポート:** Internet Explorer（全バージョン）、Chrome 89以下

**動作確認の優先順位:**
1. **最優先:** Chrome（Chromebook含む） — GIGAスクール端末の大半を占める
2. **高:** Edge — Windows PC環境
3. **中:** Safari — iPad環境
4. **低:** Firefox — 利用率が低いため後回し

**必要なブラウザAPI:**
- Web Worker（無限ループ対策のSkulpt実行に必須）
- ES2020（Optional chaining, Nullish coalescing等）
- IndexedDB（オフライン時のコード自動保存）
- CSS Grid / Flexbox（レイアウト）

> **📌 GIGAスクール対応:** 文部科学省のGIGAスクール構想で配布された端末（Chromebook、iPad、Windows）を主要ターゲットとする。Phase 1のユーザーテスト時に実機確認を行い、パフォーマンス問題があれば対処する。

---

### 5.6 Python実行層の抽象化

MVPはSkulptで進め、制約にぶつかったらPyodideに移行できるよう、Python実行層を抽象化しておく。

```typescript
interface PythonRunner {
  execute(code: string): Promise<ExecutionResult>;
  setStdout(callback: (text: string) => void): void;
  setStderr(callback: (text: string) => void): void;
  setCanvas(canvas: HTMLCanvasElement): void;
  terminate(): void; // 無限ループ対策
}

interface ExecutionResult {
  success: boolean;
  output: string;
  error?: PythonError;
  turtleSteps?: TurtleStep[]; // turtle描画のステップ記録
  executionTimeMs: number;
}

interface PythonError {
  type: string;      // 例: "SyntaxError"
  message: string;   // 英語の元メッセージ
  messageJa: string; // 日本語化されたメッセージ
  line?: number;
}

// 実装を差し替え可能に
class SkulptRunner implements PythonRunner { ... }
class PyodideRunner implements PythonRunner { ... }

// ファクトリー
function createPythonRunner(type: 'skulpt' | 'pyodide' = 'skulpt'): PythonRunner {
  return type === 'skulpt' ? new SkulptRunner() : new PyodideRunner();
}
```

**Skulpt vs Pyodide 比較:**

| | Skulpt | Pyodide |
|---|---|---|
| サイズ | 約300KB | 約10MB（初回） |
| Python 3互換性 | 部分的 | 完全 |
| 起動速度 | 即座 | 数秒（初回） |
| turtle対応 | 組み込み | カスタム実装必要 |
| 標準ライブラリ | 限定的 | ほぼ完全 |

### 5.7 Monaco Editorの最適化

Monaco Editorはバンドルサイズが大きい（数MB）。学校のPCは低スペックなことが多いため、以下の最適化を行う：

```typescript
// 遅延ロード: ワールド3以降に到達するまでMonacoを読み込まない
const MonacoEditor = dynamic(() => import('@monaco-editor/react'), {
  ssr: false,
  loading: () => <EditorSkeleton />,
});

// 部分インポート: Python言語サポートのみ
// monaco-editor/esm/vs/basic-languages/python/python.js
// 不要な言語サポートを除外してバンドルサイズを削減

// Web Worker設定: エディタのシンタックスハイライト等をWeb Workerで実行
// next.config.js でMonaco用Worker設定
```

- **初回ロード目標:** 3秒以内（3G回線想定）
- **キャッシュ:** Service Workerでモジュールをキャッシュし、2回目以降は即時ロード
- **フォールバック:** ロード中はシンプルな `<textarea>` を表示

### 5.8 Zustand ストア分割設計

状態管理のカオス化を防ぐため、ストアを役割別に分割する：

```typescript
// ゲーム状態（レベル、XP、コイン、実績）
const useGameStore = create<GameState>((set) => ({
  level: 1,
  xp: 0,
  coins: 0,
  achievements: [],
  streak: 0,
  // ...
}));

// クエスト進捗状態
const useProgressStore = create<ProgressState>((set) => ({
  worldProgress: {},  // ワールドごとの進捗
  currentQuest: null,
  // ...
}));

// エディタ状態（コード、ブロック、実行結果）
const useEditorStore = create<EditorState>((set) => ({
  code: '',
  blocks: null,
  output: '',
  isRunning: false,
  error: null,
  // ...
}));

// UI状態（モーダル、アニメーション、NPC会話）
const useUIStore = create<UIState>((set) => ({
  showClearModal: false,
  showLevelUpModal: false,
  npcDialogue: null,
  // ...
}));

// 認証状態
const useAuthStore = create<AuthState>((set) => ({
  user: null,
  isTeacher: false,
  classId: null,
  // ...
}));
```

### 5.9 キャッシュ戦略

```
問題データ（JSON）    → SSGで静的生成（ビルド時に全問題をプリレンダ）
ユーザー進捗         → SWR/React Query でキャッシュ（stale-while-revalidate）
ワールド/クエスト一覧  → SSG + ISR（問題追加時に再生成）
ランキング           → SWR（60秒キャッシュ）
先生ダッシュボード    → React Query（30秒キャッシュ + リアルタイム更新オプション）
```

### 5.10 API Routesのドメイン層分離

67問+管理画面のビジネスロジックがAPI Routesに肥大化するのを防ぐため、ドメイン層を分離する：

```
app/api/quest/submit/route.ts  ← ルーティング+バリデーションのみ
  ↓
lib/services/quest-service.ts  ← ビジネスロジック（正解判定、XP計算、★評価）
  ↓
lib/repositories/quest-repo.ts ← データアクセス（Supabase操作）
lib/repositories/progress-repo.ts
```

```typescript
// Service層の例
class QuestService {
  constructor(
    private questRepo: QuestRepository,
    private progressRepo: ProgressRepository,
  ) {}

  async submitAnswer(userId: string, questId: string, code: string, output: string) {
    const quest = await this.questRepo.findById(questId);
    const isCorrect = this.compareOutput(output, quest.expected_output);
    if (!isCorrect) return { success: false };

    const progress = await this.progressRepo.findByUserAndQuest(userId, questId);
    const stars = this.calculateStars(progress);
    const rewards = this.calculateRewards(quest.rewards, stars, progress.saw_answer);

    await this.progressRepo.updateCleared(userId, questId, stars);
    await this.progressRepo.addXP(userId, rewards.xp);

    return { success: true, stars, ...rewards };
  }
}
```

- **API Route:** リクエスト解析・認証チェック・レスポンス整形のみ
- **Service層:** ビジネスロジック（正解判定、XP計算、進捗管理）
- **Repository層:** Supabaseへのデータアクセスを抽象化

> **📌 メリット:** テスタビリティの向上（Service層の単体テストが容易）、ロジックの再利用（管理画面と生徒画面で同じServiceを使用）、将来のDB移行への備え。

---

### 5.11 無限ループ対策

生徒が `while True` を書いた場合にブラウザがフリーズしないよう、以下の対策を実装する：

```typescript
// Web Worker内でPythonコードを実行
const worker = new Worker('/python-worker.js');

// 実行ステップ上限: 100,000ステップ
// タイムアウト: 10秒
const EXECUTION_LIMITS = {
  maxSteps: 100_000,
  timeoutMs: 10_000,
};

// タイムアウトで自動停止
const timeoutId = setTimeout(() => {
  worker.terminate();
  showError('プログラムの実行に時間がかかりすぎたよ！\nくり返しが止まらなくなっていないかな？');
}, EXECUTION_LIMITS.timeoutMs);
```

- Python実行はWeb Worker内で行い、メインスレッドをブロックしない
- 実行ステップ数の上限（100,000ステップ）を設定
- タイムアウト（10秒）で自動停止
- 停止時は日本語でわかりやすいメッセージを表示

### 5.12 エラーハンドリング・オフライン対策

```typescript
// オフライン時のコード保存
const useOfflineStore = create<OfflineState>((set) => ({
  pendingSubmissions: [],  // オフライン時に保存された解答
  // ...
}));

// IndexedDBにコードを自動保存（30秒ごと）
// オンライン復帰時に自動同期

// ネットワークリトライ
const fetchWithRetry = async (url: string, options: RequestInit, retries = 3) => {
  for (let i = 0; i < retries; i++) {
    try {
      return await fetch(url, options);
    } catch (e) {
      if (i === retries - 1) throw e;
      await sleep(1000 * (i + 1)); // exponential backoff
    }
  }
};
```

- **自動保存:** コードを30秒ごとにIndexedDBに保存。ブラウザを閉じても復元可能
- **オフライン検知:** `navigator.onLine` + 定期的なヘルスチェック
- **オフラインモード:** API通信なしでもコード編集・実行（Skulptはブラウザ内完結）が可能。進捗同期はオンライン復帰後に実行
- **リトライ:** ネットワークエラー時はexponential backoffで最大3回リトライ

#### オフライン同期のコンフリクト解決

学校PCと自宅PCなど、複数デバイスでオフライン状態のまま学習した場合の進捗コンフリクトを解決する戦略：

```typescript
// 同期戦略: タイムスタンプベースのラストライトウィン + XP最大値採用
interface SyncableProgress {
  quest_id: string;
  stars: number;
  status: 'locked' | 'available' | 'cleared';
  code: string;
  attempts: number;
  hints_used: number;
  saw_answer: boolean;
  updated_at: number; // Unix timestamp (ms)
}

async function syncProgress(localData: SyncableProgress[], userId: string) {
  const serverData = await fetchServerProgress(userId);

  for (const local of localData) {
    const server = serverData.find(s => s.quest_id === local.quest_id);

    if (!server) {
      // サーバーにデータなし → ローカルをアップロード
      await uploadProgress(userId, local);
      continue;
    }

    // コンフリクト解決
    const merged = resolveConflict(local, server);
    if (merged.needsUpdate) {
      await uploadProgress(userId, merged.data);
    }
  }

  // XP/レベルはサーバー側で全進捗から再計算（整合性保証）
  await recalculateXP(userId);
}

function resolveConflict(
  local: SyncableProgress,
  server: SyncableProgress
): { needsUpdate: boolean; data: SyncableProgress } {
  // ルール1: ★評価は常に最大値を採用（巻き戻りを防止）
  const stars = Math.max(local.stars, server.stars);

  // ルール2: クリア済みは巻き戻さない
  const status = (local.status === 'cleared' || server.status === 'cleared')
    ? 'cleared' : (local.updated_at > server.updated_at ? local.status : server.status);

  // ルール3: コード（最後に書いた内容）はタイムスタンプが新しい方を採用
  const code = local.updated_at > server.updated_at ? local.code : server.code;

  // ルール4: 挑戦回数は合算（両デバイスでの試行を反映）
  const attempts = Math.max(local.attempts, server.attempts);

  return {
    needsUpdate: true,
    data: { ...local, stars, status, code, attempts, updated_at: Date.now() },
  };
}
```

**同期ルールまとめ:**
| データ | 戦略 | 理由 |
|--------|------|------|
| ★評価 | **最大値を採用** | ★の巻き戻りは子供のモチベーションを大きく損なう |
| クリア状態 | **クリア済みは巻き戻さない** | 一度クリアした問題がロックに戻るのは不正 |
| コード | **タイムスタンプが新しい方** | ラストライトウィン。最新の作業を優先 |
| XP/レベル | **サーバー側で全進捗から再計算** | 部分同期でもXPの整合性を保証 |
| 挑戦回数 | **大きい方を採用** | 正確な合算は困難なため、保守的に最大値 |

> **📌 設計判断:** 「学校で途中まで→自宅で続き」は十分あり得るシナリオ。完全な双方向同期（CRDT等）は複雑すぎるため、上記のシンプルなルールで対応する。ユーザーテストで問題が出れば改善する。

---

## 6. データ設計

### 6.1 ER図（概要）

```
User (ユーザー)
├── id, nickname, role (student/teacher), avatar
├── password_hash
├── class_id (所属クラス)
├── level, xp, coins
├── streak_count, last_login_at
└── created_at

Class (クラス)
├── id, name, invite_code (8桁英数字)
├── teacher_id (FK → User)
└── created_at

Quest (クエスト / 問題)
├── id, world_id, order, title
├── description (Markdown)
├── mode (block/convert/code)
├── difficulty (basic/practice/challenge)
├── initial_code (テンプレート)
├── expected_output
├── expected_turtle_steps (W4用)
├── hints[] (段階的ヒント)
├── solution_code (答えを見る用)
├── related_quest_id (似た練習問題のID)
├── npc_dialogue (NPC台詞)
├── rewards { xp, coins }
└── star_criteria

World (ワールド)
├── id, name, order, icon
├── unlock_level (解放レベル)
└── description

UserProgress (進捗)
├── user_id, quest_id
├── status (locked/available/cleared)
├── stars (0-3)
├── attempts (挑戦回数)
├── hints_used (使用したヒント数)
├── saw_answer (答えを見たか)
├── code (最後に書いたコード)
└── cleared_at

Achievement (実績)
├── id, name, icon, description
└── condition (解放条件)

UserAchievement (ユーザー実績)
├── user_id, achievement_id
└── earned_at

AnalyticsEvent (分析イベント)
├── id, user_id, event_type
├── quest_id (nullable)
├── payload (JSON)
└── created_at
```

> **個人情報最小化:** ユーザーテーブルにメールアドレスは持たない。ニックネーム + パスワードのみ。個人を特定できる情報は収集しない。

### 6.2 問題データ例（JSON）

```json
{
  "id": "1-1",
  "world_id": "world-1",
  "order": 1,
  "title": "はじめてのprint",
  "mode": "block",
  "difficulty": "basic",
  "npc": {
    "character": "block-kun",
    "dialogue": "ようこそ、冒険者！まずはブロックを使って文字を表示してみよう！"
  },
  "description": "「こんにちは、Python！」と表示するブロックを組み立てよう",
  "expected_output": "こんにちは、Python！",
  "initial_blocks": [],
  "available_blocks": ["print", "text"],
  "hints": [
    "「出力」カテゴリにあるブロックを使ってみよう",
    "printブロックの中に文字ブロックを入れてみよう",
    "文字ブロックに「こんにちは、Python！」と入力しよう"
  ],
  "solution_code": "print('こんにちは、Python！')",
  "related_quest_id": "1-2",
  "rewards": {
    "xp": 30,
    "coins": 20
  },
  "star_criteria": {
    "0": "答えを見てクリア（XP半減）",
    "1": "クリア",
    "2": "ヒント未使用",
    "3": "ヒント未使用 + 実行1回"
  }
}
```

---

## 7. 先生用機能

### 7.1 管理画面

| 機能 | 説明 |
|------|------|
| 生徒一覧 | 各生徒のレベル・進捗を一覧表示 |
| 進捗詳細 | 特定の生徒のクエスト別クリア状況 |
| クラス管理 | クラス作成・招待コード発行 |
| アカウント管理 | 生徒のパスワードリセット・アカウント無効化 |
| 成績エクスポート | CSV出力 |
| カスタム問題 | オリジナルクエストの追加（将来） |

### 7.2 先生ダッシュボード

```
┌─────────────────────────────────────────────────────┐
│  📊 クラスの進捗       招待コード: ABCD1234          │
│                                                      │
│  名前        レベル  W1    W2    W3    W4    最終     │
│  たろう       Lv.3  18/18  3/16  0/18  0/15  今日    │
│  はなこ       Lv.4  18/18  10/16 1/18  0/15  今日    │
│  けんた       Lv.2   9/18  0/16  0/18  0/15  昨日    │
│  ゆい         Lv.1   3/18  0/16  0/18  0/15  3日前   │
│                                                      │
│  [CSV出力] [パスワードリセット] [招待コード再発行]      │
└─────────────────────────────────────────────────────┘
```

---

## 8. セキュリティ設計

### 8.1 認証フロー（COPPA/個人情報保護対応）

子供向けサービスとして、**個人情報の最小化**を最優先に設計する。

```
認証フロー:
1. 先生がクラスを作成 → 8桁の招待コード（例: ABCD1234）が発行される
2. 生徒は招待コード + ニックネーム + パスワードで登録
3. メールアドレスは不要（子供の個人情報を収集しない）
4. 先生がパスワードリセット可能（生徒が忘れた場合）
5. オプションでGoogleログイン連携（中学生向け・保護者同意の上）

個人情報最小化原則:
- 収集するのは「ニックネーム」と「パスワード」のみ
- 本名・メールアドレス・生年月日は収集しない
- IPアドレスはログに記録するがユーザーには紐づけない
```

**法的考慮事項:**
- 日本の「青少年が安全に安心してインターネットを利用できる環境の整備等に関する法律」を遵守
- 13歳未満のユーザーは先生（大人）経由でのみ登録可能
- プライバシーポリシーを策定し、サービス内に掲示する（必須）
- 保護者への通知と同意取得の仕組みを用意する

### 8.2 プライバシーポリシー（策定必須項目）

- 収集する情報の範囲（ニックネーム、パスワードハッシュ、学習進捗）
- 情報の利用目的（学習進捗の管理、サービス改善）
- 情報の保存期間と削除方針
- 第三者提供の有無（Supabase等のサービス利用を明記）
- 保護者の権利（データの閲覧・削除請求）
- 問い合わせ先

### 8.3 Supabase RLS（Row Level Security）

他人の進捗データを閲覧・改ざんできないようにするRLSポリシー：

```sql
-- UserProgress: 自分の進捗のみ閲覧・更新可能
CREATE POLICY "Users can view own progress"
  ON user_progress FOR SELECT
  USING (auth.uid() = user_id);

CREATE POLICY "Users can update own progress"
  ON user_progress FOR UPDATE
  USING (auth.uid() = user_id);

-- 先生は自分のクラスの生徒の進捗を閲覧可能
CREATE POLICY "Teachers can view class progress"
  ON user_progress FOR SELECT
  USING (
    EXISTS (
      SELECT 1 FROM users u
      JOIN classes c ON c.teacher_id = auth.uid()
      WHERE u.id = user_progress.user_id
        AND u.class_id = c.id
    )
  );

-- User: 自分のプロフィールのみ更新可能
CREATE POLICY "Users can update own profile"
  ON users FOR UPDATE
  USING (auth.uid() = id);

-- 先生は自分のクラスの生徒情報を閲覧可能
CREATE POLICY "Teachers can view class students"
  ON users FOR SELECT
  USING (
    auth.uid() = id
    OR EXISTS (
      SELECT 1 FROM classes c
      WHERE c.teacher_id = auth.uid()
        AND users.class_id = c.id
    )
  );
```

### 8.4 XP改ざん防止 — サーバー側正解判定の具体設計

XP・コインの加算は必ずサーバー側で行う。クライアントからの直接加算リクエストは受け付けない。

```typescript
// ❌ NG: クライアントサイドでXPを加算してAPIに送信
// POST /api/progress { questId: "1-1", xp: 9999 }

// ✅ OK: サーバー側で正解判定→XP付与
// POST /api/quest/submit { questId: "1-1", code: "print('hello')" }
// → サーバーが正解判定 → XPを計算 → DBに保存
```

#### サーバー側正解判定フロー

サーバー側でPythonコードを再実行するのはインフラコスト・セキュリティリスクが大きいため、**「クライアント実行結果 + サーバー側検証」のハイブリッド方式**を採用する。

```
┌─────────────┐     POST /api/quest/submit      ┌─────────────────┐
│  クライアント  │ ──────────────────────────────→ │  API Route       │
│              │   {                              │                  │
│  1. Skulptで  │     questId: "1-1",             │  1. コード整合性   │
│     コード実行 │     code: "print('こんにちは')", │     チェック       │
│  2. 出力を取得 │     output: "こんにちは\n",     │  2. 期待出力と比較 │
│  3. サーバーに │     stars: 3                    │  3. ★再計算       │
│     送信      │   }                             │  4. XP/コイン付与 │
│              │ ←────────────────────────────── │  5. DB保存        │
│  4. 結果表示  │   { success, xp, level, ... }   │                  │
└─────────────┘                                  └─────────────────┘
```

```typescript
// API Route: /api/quest/submit
export async function POST(req: Request) {
  const { questId, code, output } = await req.json();
  const user = await getAuthUser(req); // Supabase Auth検証

  // 1. クエスト情報をDBから取得
  const quest = await getQuest(questId);
  if (!quest) return error(404, 'Quest not found');

  // 2. コード整合性チェック（カジュアルな不正を防ぐ）
  const codeCheck = validateCode(code, quest);
  if (!codeCheck.valid) return error(400, codeCheck.reason);

  // 3. 期待出力との比較（サーバー側で判定）
  const isCorrect = compareOutput(output, quest.expected_output, quest.mode);
  if (!isCorrect) return json({ success: false, message: '不正解です' });

  // 4. ★評価をサーバー側で再計算（クライアントの自己申告を信用しない）
  const progress = await getUserProgress(user.id, questId);
  const stars = calculateStars(progress);

  // 5. XP・コインをサーバー側で計算・付与
  const rewards = calculateRewards(quest.rewards, stars, progress.saw_answer);
  await updateProgress(user.id, questId, { stars, cleared: true });
  await addXPAndCoins(user.id, rewards.xp, rewards.coins);

  return json({ success: true, stars, xp: rewards.xp, coins: rewards.coins });
}
```

```typescript
// コード整合性チェック
function validateCode(code: string, quest: Quest): { valid: boolean; reason?: string } {
  // 空コードでないか
  if (!code || code.trim().length === 0) {
    return { valid: false, reason: 'コードが空です' };
  }

  // 最低限のキーワードを含むか（例: print問題なら print が含まれるべき）
  if (quest.required_keywords?.length) {
    const missing = quest.required_keywords.filter(kw => !code.includes(kw));
    if (missing.length > 0) {
      return { valid: false, reason: 'コードに必要な要素が含まれていません' };
    }
  }

  // コードの長さが異常でないか（1文字や10000文字など）
  if (code.length > 5000) {
    return { valid: false, reason: 'コードが長すぎます' };
  }

  return { valid: true };
}

// 出力比較
function compareOutput(actual: string, expected: string, mode: string): boolean {
  // 末尾の改行・空白を正規化して比較
  const normalize = (s: string) => s.replace(/\s+$/gm, '').trim();
  return normalize(actual) === normalize(expected);
}
```

**設計方針:**
- **サーバー側ではPythonコードを再実行しない。** Skulptはブラウザ向け設計でありNode.js上での実行は非推奨。Vercel Serverless Functionsでのサンドボックス実行もコスト・複雑性が高すぎる
- **代わりに「出力文字列の比較 + コード整合性チェック」で検証する。** 100%の改ざん防止は諦め、カジュアルな不正（DevToolsでXPを書き換える等）を防ぐレベルで十分
- **★評価はサーバー側で再計算する。** ヒント使用回数・実行回数はサーバーのUserProgressで管理しているため、クライアントの自己申告に依存しない
- **教育プラットフォームでは、完全な改ざん防止よりUXが優先。** 子供が不正してXPを水増ししても、学習効果がないだけで実害は小さい

### 8.5 招待コード総当たり対策

8桁英数字の招待コードは`36^8 ≈ 2.8兆通り`で十分なエントロピーだが、APIレベルでの保護を実装する：

- **レート制限:** 同一IPから招待コード入力は10回/時間まで。超過時は一時ブロック
- **遅延レスポンス:** 招待コード検証APIは意図的に500ms〜1000msの遅延を入れ、高速な総当たりを抑制
- **失敗ログ:** 連続失敗をログに記録し、異常なパターンをSentryでアラート

### 8.6 セッション管理

学校の共有PC環境を考慮したセッション設計：

- **セッション有効期限:** ブラウザセッション（ブラウザを閉じたらセッション切れ）をデフォルトとする。子供がログアウトを忘れても、次の生徒がアクセスできないようにする
- **「ログイン状態を保持する」チェックボックス:** 自宅PCなど個人端末向け。チェック時のみリフレッシュトークンをlocalStorageに保存（有効期限7日）
- **同時セッション:** 同一アカウントの複数デバイス同時ログインを許可（学校と自宅での利用を想定）
- **ログアウトボタン:** 画面上部に常に表示。子供でも見つけやすい位置に配置

### 8.7 パスワード要件

子供が使える範囲で最低限のセキュリティを確保する：

- **最低文字数:** 4文字以上（子供が覚えられる長さ）
- **最大文字数:** 100文字
- **使用可能文字:** 英数字 + 基本的な記号
- **先生によるパスワード配布:** クラス作成時に先生が初期パスワードを設定・配布することも可能

### 8.8 その他のセキュリティ対策

- **XSS対策:** ユーザー入力（コード、ニックネーム）の表示時にサニタイズ。特にコード表示部分は `dangerouslySetInnerHTML` を使わず、専用のコード表示コンポーネントを使用
- **CSRF対策:** Supabase Authのトークンベース認証で対応
- **レート制限:** API Routesにレート制限を実装（連続実行の抑制）

---

## 9. エラーハンドリング設計

### 9.1 エラーメッセージ日本語化マッピング

Skulpt/Pythonのエラーメッセージは英語。子供には理解不能なため、日本語化マッピングを実装する。

```typescript
const errorMessageMap: Record<string, (detail: string) => string> = {
  // 構文エラー
  'SyntaxError: unexpected EOF while parsing':
    () => 'コードが途中で終わっているよ！括弧 () や : を忘れていないかな？',

  'SyntaxError: invalid syntax':
    () => 'コードの書き方がちょっとちがうみたい。スペルミスや記号の間違いがないか確認してね！',

  'SyntaxError: EOL while scanning string literal':
    () => '文字列（クォーテーション）が閉じられていないよ！\' か \" を確認してね',

  // 名前エラー
  'NameError: name \'(.+)\' is not defined':
    (name) => `"${name}" って何だろう？もしかしてスペルミスかな？\n似ている名前がないか確認してみよう！`,

  // インデントエラー
  'IndentationError: expected an indented block':
    () => 'if文やfor文の中身は、スペース4つ分右にずらしてね！\nこれを「インデント」って言うよ',

  'IndentationError: unexpected indent':
    () => 'スペースが多すぎるところがあるよ！行の最初のスペースを確認してね',

  // 型エラー
  'TypeError: unsupported operand type':
    () => '数字と文字を足し算しようとしているよ！\n数字にするには int() を、文字にするには str() を使ってね',

  'TypeError: (.+) takes (.+) arguments':
    () => '関数に渡す値の数が合っていないよ！関数の定義を確認してね',

  // インデックスエラー
  'IndexError: list index out of range':
    () => 'リストの範囲外にアクセスしようとしているよ！\nリストの長さを確認してね（0から始まるのを忘れずに！）',

  // ゼロ除算
  'ZeroDivisionError: division by zero':
    () => '0で割り算はできないよ！割る数が0になっていないか確認してね',
};
```

### 9.2 よくあるミスのパターンマッチ

エラーメッセージだけでなく、コード自体を分析してよくあるミスを検出する：

```typescript
const commonMistakes = [
  {
    pattern: /pritn\s*\(/,
    suggestion: 'もしかして print() の間違いかな？'
  },
  {
    pattern: /Print\s*\(/,
    suggestion: 'Pythonでは print は小文字だよ！ Print → print'
  },
  {
    pattern: /if .+[^:]\s*$/m,
    suggestion: 'if文の最後に : （コロン）を忘れていないかな？'
  },
  {
    pattern: /for .+[^:]\s*$/m,
    suggestion: 'for文の最後に : （コロン）を忘れていないかな？'
  },
  {
    pattern: /def .+[^:]\s*$/m,
    suggestion: 'def文の最後に : （コロン）を忘れていないかな？'
  },
];
```

---

## 10. 分析基盤

### 10.1 追跡イベント一覧

問題改善・カリキュラム調整のために、以下のイベントを収集する：

| イベント | 説明 | ペイロード |
|---------|------|-----------|
| `quest_start` | クエスト開始 | quest_id, world_id |
| `quest_clear` | クエストクリア | quest_id, stars, attempts, time_spent_sec |
| `quest_abandon` | クエスト離脱 | quest_id, time_spent_sec, last_code |
| `hint_used` | ヒント使用 | quest_id, hint_level (1-3) |
| `answer_viewed` | 答えを見た | quest_id |
| `code_execute` | コード実行 | quest_id, has_error, error_type |
| `error_occurred` | エラー発生 | quest_id, error_type, error_message |
| `world_enter` | ワールド遷移 | world_id |
| `level_up` | レベルアップ | new_level |
| `session_start` | セッション開始 | — |
| `session_end` | セッション終了 | duration_sec |

### 10.2 分析ダッシュボード（先生用）

- **つまづきポイント:** どのクエストで離脱・ヒント使用が多いか
- **エラー頻度:** よく発生するエラーの種類と回数
- **進捗分布:** クラス全体の進捗ヒートマップ
- **学習時間:** 1クエストあたりの平均所要時間

### 10.3 データ保存

分析イベントはSupabaseの `analytics_events` テーブルに保存。個人を特定できる情報は含めない（user_idはUUID）。

---

## 11. アクセシビリティ

教育プラットフォームとして、以下のアクセシビリティ対応を実施する：

### 11.1 キーボードナビゲーション

- 全ての操作がキーボードで完結可能
- Tab順序の適切な設定
- フォーカス状態の視覚的表示（アウトライン）
- コードエディタ内のキーボードショートカット（Ctrl+Enter で実行）

### 11.2 色覚多様性への配慮

- 色だけで情報を伝えない（正解=緑+チェックマーク、不正解=赤+×マーク）
- ★評価はアイコン+テキストで表示
- コードのシンタックスハイライトは色覚多様性対応のテーマを用意
- WCAG 2.1 AA基準のコントラスト比を確保

### 11.3 フォントサイズ調整

- エディタのフォントサイズを設定画面で変更可能（12px〜24px）
- ブラウザのズーム機能で崩れないレスポンシブ設計
- NPC会話テキストは最低16px

### 11.4 スクリーンリーダー対応

- 適切なARIAラベルの設定
- コンソール出力部分のlive region設定（実行結果を読み上げ）
- Blocklyのアクセシビリティ機能を有効化

---

## 12. 画面設計（ワイヤーフレーム）

**[👉 ワイヤーフレームを見る（インタラクティブ HTML）](python-quest-wireframe.html)**

全10ページ構成:
1. 🗺 ワールドマップ（4ワールド+クエストノード）
2. 🧩 W1: ブロックモード（Blocklyエディタ）
3. 🔀 W2: 変換モード（ブロック⇔コード並列表示）
4. 🐍 W3: コードモード（Monacoエディタ）
5. 🎨 W4: turtleモード（コード+Canvas描画）
6. 🎉 正解演出
7. 💀 不正解演出
8. ⬆ レベルアップ演出
9. 👤 生徒ダッシュボード
10. 👨‍🏫 先生用管理画面

---

## 13. コスト見積もり

### 13.1 インフラ・サービス費用

#### 無料枠で運用できる範囲（生徒 〜50人規模・MVP検証期間）

| サービス | プラン | 月額 | 備考 |
|---------|--------|------|------|
| **Vercel** | Hobby（無料） | **¥0** | ⚠️ **商用利用不可。** 個人の非商用利用限定。MVP検証期間のみ |
| **Supabase** | Free | **¥0** | DB 500MB、Auth 5万MAU、Storage 1GB、帯域2GB |
| **GitHub** | Free | **¥0** | パブリック/プライベートリポジトリ、Actions 2000分/月 |
| **ドメイン** | .com | **約 ¥1,500/年** | 任意。Vercelのサブドメイン(*.vercel.app)なら無料 |
| **Sentry** | Developer（無料） | **¥0** | エラー監視。5Kイベント/月 |
| | | | |
| **合計（無料ドメイン）** | | **¥0/月** | MVP検証期間のみ |
| **合計（独自ドメインあり）** | | **約 ¥125/月** | MVP検証期間のみ |

> **⚠️ 重要:** Vercel Hobbyプランは利用規約上、商用利用不可。教育機関に提供する段階でProプラン（$20/月）への移行が必須。MVP検証後は速やかにProプランへ切り替えること。

> **💡 ポイント:** PythonはSkulptでブラウザ内実行するため、サーバー側の計算コストがゼロ。これが最大のコスト削減要因。

#### MVP後の本格運用（生徒 〜500人規模）

| サービス | プラン | 月額 | 備考 |
|---------|--------|------|------|
| **Vercel** | Pro | **$20（約 ¥3,000）** | 帯域1TB、商用利用可 |
| **Supabase** | Pro | **$25（約 ¥3,750）** | DB 8GB、Auth 10万MAU、帯域250GB |
| **Sentry** | Team | **$26（約 ¥3,900）** | 50Kイベント/月、パフォーマンス監視 |
| | | | |
| **合計** | | **約 ¥10,650/月** | |

#### 大規模運用時（生徒 500人〜）

| サービス | プラン | 月額 | 備考 |
|---------|--------|------|------|
| **Vercel** | Pro | **$20〜（約 ¥3,000〜）** | 帯域超過分は$40/100GB |
| **Supabase** | Pro + 従量 | **$25〜（約 ¥3,750〜）** | DB・帯域の超過分は従量課金 |
| **Sentry** | Team | **$26〜（約 ¥3,900〜）** | イベント超過分は従量課金 |
| | | | |
| **合計** | | **約 ¥11,000〜/月** | 利用量に応じて増加 |

### 13.2 開発人件費（個人開発前提）

| フェーズ | 工数目安 | 備考 |
|---------|---------|------|
| Phase 1（MVP） | 6〜8週間 | Blockly + Skulpt統合が大半 |
| Phase 2a | 4〜5週間 | 認証 + ワールド2 |
| Phase 2b | 4〜5週間 | ワールド3 + ゲームシステム |
| Phase 3 | 6〜8週間 | ワールド4 + 全問題作成 + 管理画面 |
| Phase 4 | 継続 | 磨き込み |
| **合計** | **約20〜26週間（5〜7ヶ月）** | 個人開発（週20〜30時間想定） |

> **📌 注意:** 最大のコストは人件費。個人開発の場合、自分の時間のみだが、5〜7ヶ月の開発期間を見込むこと。

### 13.3 開発ツール費用

| ツール | 費用 | 備考 |
|--------|------|------|
| **Next.js** | 無料 | OSS |
| **TypeScript** | 無料 | OSS |
| **Blockly** | 無料 | Google製 OSS（Apache 2.0） |
| **Monaco Editor** | 無料 | Microsoft製 OSS（MIT） |
| **Skulpt** | 無料 | OSS（MIT） |
| **Tailwind CSS** | 無料 | OSS（MIT） |
| **Framer Motion** | 無料 | OSS（MIT） |
| **Zustand** | 無料 | OSS（MIT） |
| | | |
| **合計** | **¥0** | 全てOSS |

### 13.4 オプション・その他費用

| 項目 | 費用 | 必要性 |
|------|------|--------|
| **BGM・SE素材** | ¥0〜¥5,000 | フリー素材で対応可。有料素材を使う場合 |
| **イラスト・キャラ素材** | **¥30,000〜¥100,000** | **投資推奨。** キャラクターイラスト5体+ワールド背景。子供向けは見た目の魅力が継続率に大きく影響。絵文字ベースのMVPから段階的にアップグレード |
| **フォント** | ¥0 | Google Fonts で無料 |
| **SSL証明書** | ¥0 | Vercel が自動対応 |
| **テストデバイス** | ¥0〜¥50,000 | Chromebook（中古¥15,000〜）、タブレット。学校環境での動作確認に必要 |

### 13.5 コストまとめ

| フェーズ | 想定規模 | 月額コスト | 備考 |
|---------|---------|-----------|------|
| **MVP検証期間** | 〜10人 | **¥0** | Vercel Hobby（商用不可に注意） |
| **Phase 2〜（本格運用）** | 〜500人 | **約 ¥10,650/月** | Vercel Pro + Supabase Pro + Sentry |
| **大規模展開** | 500人〜 | **約 ¥11,000〜/月** | 従量課金で増加 |
| **初期投資（一時費用）** | — | **¥30,000〜¥150,000** | イラスト + テストデバイス |

> **📌 結論:** MVP検証は無料で開始可能だが、教育機関への提供開始時にはVercel Pro（$20/月）への移行が必須。Python実行をブラウザ側で行う設計のおかげで、サーバーコストを最小限に抑えられるのが本プラットフォームの強み。キャラクターイラストへの投資は子供の継続率に直結するため推奨。

---

## 14. 開発ロードマップ

### Phase 1: MVP（6〜8週間）

**スコープ:** Blockly + Skulpt統合 + ワールド1の3問で「動くもの」を作る

- [ ] プロジェクト初期セットアップ（Next.js + TypeScript + Tailwind）
- [ ] Blockly統合（日本語化 + カスタムブロック定義 + 利用ブロック制限）
- [ ] Skulpt統合（Python実行層の抽象化インターフェース実装）
- [ ] 無限ループ対策（Web Worker + タイムアウト）
- [ ] ワールド1のクエスト3問（基本のみ）
- [ ] 基本的な正解判定（出力一致）
- [ ] エラーメッセージ日本語化（主要10パターン）
- [ ] 正解/不正解の演出
- [ ] ワールドマップ画面
- [ ] ローカル進捗保存（localStorage）
- [ ] **🧪 ユーザーテスト（実際の子供2〜3人にプロトタイプを触らせる）**

> **⚠️ Phase 1完了直後にユーザーテストを実施すること。** 子供のUXは大人の想像と大きく異なる。テスト結果に基づいてPhase 2以降の優先順位を調整する。

### Phase 2a: 認証 + ワールド2（4〜5週間）

- [ ] Supabase統合（認証 + DB）
- [ ] 招待コード + ニックネーム + パスワード認証
- [ ] Supabase RLS設計・実装
- [ ] ワールド2（変換モード）の実装
- [ ] ワールド1の残りクエスト（計18問）
- [ ] ワールド2のクエスト（計16問）
- [ ] つまづき救済システム（答えを見る、似た練習問題）
- [ ] **プライバシーポリシー策定・掲示**（認証機能の公開前に必須。§8.2参照）

### Phase 2b: ワールド3 + ゲームシステム（4〜5週間）

- [ ] Monaco Editor統合（遅延ロード + 部分インポート最適化）
- [ ] ワールド3（コードモード）の実装
- [ ] ワールド3のクエスト（計18問）
- [ ] XP・レベル・称号システム（サーバー側でXP付与）
- [ ] レベルアップ演出
- [ ] 実績システム
- [ ] ★評価システム
- [ ] 連続ログインストリーク（ペナルティなし設計）

### Phase 3: 拡充（6〜8週間）

- [ ] ワールド4（turtle）の実装
- [ ] turtle正解判定（座標・角度ステップ比較）
- [ ] ワールド4のクエスト（計15問）
- [ ] NPC会話システム（つまづき時の声かけ含む）
- [ ] ヒント段階表示
- [ ] 先生用管理画面
- [ ] クラス管理・招待コード発行
- [ ] アカウント管理（パスワードリセット等）
- [ ] 分析基盤（イベント収集 + ダッシュボード）
- [ ] レスポンシブ対応（タブレット）
- [ ] **🧪 2回目のユーザーテスト（10人規模）**

### Phase 4: 磨き込み（継続）

優先順位はユーザーテスト結果に基づいて決定する。

- [ ] BGM・SE
- [ ] キャラカスタマイズ（コインで購入）
- [ ] キャラクターイラスト導入（絵文字→オリジナルイラスト）
- [ ] ランキング
- [ ] カスタム問題作成（先生用）
- [ ] アクセシビリティ改善
- [ ] 多言語対応
- [ ] PWA対応（オフライン利用）

---

## 15. 備考

### 設計方針
- **サーバーでPythonを実行しない** — Skulptでブラウザ完結。セキュリティリスクとサーバーコストを排除
- **Blockly→Python変換はBlockly標準機能** — 自前実装不要
- **問題データはJSONで管理** — MVPではファイル、後からDB移行可能
- **モバイル対応は後回し** — ブロックエディタ/コードエディタはPC/タブレット前提
- **Python実行層を抽象化** — Skulpt→Pyodide移行パスを確保（第1回レビュー反映）
- **個人情報の最小化** — メールアドレス不要、ニックネーム+パスワードのみ（第1回レビュー反映）
- **XP付与はサーバー側** — 出力比較+コード整合性チェックで検証（第2回レビュー反映で具体化）
- **エラーメッセージの日本語化** — 子供が理解できるフィードバック（第1回レビュー反映）
- **ペナルティなし設計** — 連続ログインが途切れても罰しない（第1回レビュー反映）
- **早期ユーザーテスト** — Phase 1直後に実際の子供にテスト（第1回レビュー反映）
- **GIGAスクール端末対応** — Chrome 90+/Edge 90+/Chromebook を最低動作環境に設定（第2回レビュー反映）
- **プライバシーポリシーは認証実装時に策定** — Phase 4ではなくPhase 2aで必須（第2回レビュー反映）
- **ドメイン層の分離** — API Routes肥大化防止のためService/Repository層を分離（第2回レビュー反映）
- **オフライン同期はラストライトウィン+XP最大値採用** — シンプルなルールでコンフリクト解決（第2回レビュー反映）

### 参考サービス
- CodeCombat（ゲーム×プログラミング）
- CheckiO（Python問題）
- Blockly Games（Google）
- EduBlocks（ブロック→Python）
