# 🎨 ChromaRecast

## 概要
海外で伸びている「短尺で映える知識コンテンツ化」を日本語向けに最適化するAIツール。URL/音声メモ/スクリプトを入れると、Instagram・X・LinkedIn向けに**統一トーンのビジュアルカード10枚**を自動生成。

## 海外事例分析
- Captions / OpusClip: 長尺→短尺の再編集需要が強い
- Gamma / Canva Magic Design: “見せられる見た目”への課金意欲が高い
- Taplio: SNS運用の下流（デザイン変換）に継続課金が成立

## ターゲット
- 個人開発者、コーチ、SNS運用代行、ニュースレター運営者

## 料金
- Free: 月10カード
- Pro: $7/月（500カード）
- Studio: $19/月（チーム3人）

## ユーザーフロー
1. URL/テキスト投入
2. トーン選択（Minimal/Pop/Editorial）
3. AIが見出し抽出・レイアウト生成
4. 一括書き出し（PNG + 投稿文）

## デザインコンセプト
「Swiss grid + 日本語可読性」。余白大きめ、縦長比率、SNSで止まる高コントラスト。

## アーキテクチャ
Next.js + Supabase + Cloudflare R2。生成は OpenAI mini + 画像はSatori/Resvg中心（外部画像生成依存を減らす）。

## DB設計
- users(id, plan, credits)
- projects(id, user_id, source_type, source_text)
- cards(id, project_id, headline, body, theme, image_url)
- exports(id, project_id, format)

## コスト見積もり（月）
- Supabase: $0
- Vercel: $0
- R2: $1
- AI: $4（Pro 5人想定）
- 合計: 約$5

## MVPスコープ
- URL要約→カード10枚生成
- テーマ3種
- PNG書き出し
- Stripe課金

## マーケ計画
- Xで「同じ文章を3テイスト化」デモ動画
- Product Hunt + 日本語版note
- 既存運用代行へのアフィリエイト20%

## 技術スタック
Next.js, TypeScript, Supabase, Stripe, OpenAI Responses API, Satori

## リスク
- 生成品質のばらつき → テンプレ優先で安定化
- 著作権懸念 → 入力コンテンツ所有確認UI

## 競合分析
Canvaは汎用。ChromaRecastは「文章→SNSカード」に特化し初速を出す。

## $20達成シナリオ
- Pro 3人（$21）で達成

## ユニットエコノミクス
- ARPU: $7
- 変動費/人: $0.7
- 粗利率: 90%
