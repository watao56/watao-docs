# 安全なレンダリング（XSS対策）

LLM出力を画面表示するときの最低限の安全設計をまとめます。

## 結論（先に）

- `innerHTML` / `outerHTML` への直接代入は避ける
- どうしてもHTML表示が必要なら、**サニタイズ + 許可タグ制御**を必須化
- Trusted Types（使える環境）とCSPをセットで使う

## 危険になりやすい実装

- ユーザー入力やLLM出力をそのままHTMLへ注入
- Markdown変換後のHTMLを無検証で描画
- 画像/リンクのURLスキームを検証しない（`javascript:`など）

## 推奨パターン

### 1) まずはプレーンテキスト表示

最も安全。HTMLとして解釈しない。

### 2) HTML表示が必要な場合

1. Markdown → HTML変換
2. サニタイズ（許可タグ/属性を最小化）
3. `rel="noopener noreferrer"` 付与
4. URLスキーム検証（`http/https/mailto` など）

### 3) 追加防御

- CSP（`script-src`制限）
- Trusted Types（対応環境）
- 危険な属性（`on*`）の除去

## 移行チェックリスト

- [ ] `innerHTML` 利用箇所の棚卸し
- [ ] 表示要件を「テキストで十分/HTML必要」で分類
- [ ] HTML必要箇所にサニタイザ導入
- [ ] URL/リンク属性の正規化
- [ ] CSP/Trusted Types方針を設定
- [ ] E2Eで悪性ペイロード回帰テスト
