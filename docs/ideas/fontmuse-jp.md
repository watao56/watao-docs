# 🎨 FontMuse JP

## 概要
🎨 FontMuse JPは、AI×クリエイティブ（日本語フォント演出）に特化した月$20達成向けマイクロプロダクト。AIエージェントのみで2週間MVP実装を前提に、低コスト運用で黒字化を狙う。

## 海外事例分析
- 参照トレンド: Canvaのフォントペアリング需要、Adobe Expressのテンプレ市場
- 示唆: 海外では「汎用ツール」か「高価格SaaS」が主流。日本向けには**狭い用途+日本語UX最適化**が刺さる。
- 日本でのギャップ: 使い方が難しい/英語前提/日本文化の導線不足。

## ターゲット
X/Instagram運用中の個人クリエイター・小規模ブランド（日本語デザインが苦手）

## 料金
Free / Pro $6 / Studio $12

## ユーザーフロー
①テーマ入力→②AIが和文フォントペア+余白+色を提案→③3パターン比較→④PNG/Canva書き出し

## デザインコンセプト
「和紙×ネオン」ミックス。大きい見出し、余白多め、1クリックで“映える”構図。

## アーキテクチャ
Next.js + Supabase + Cloudflare R2 + Replicate(低価格画像補助) + OpenAI mini

## DB設計
主要テーブル: users, projects, style_presets, generations, exports, subscriptions

## コスト見積もり
固定: Vercel/Supabase無料枠中心。10有料ユーザーで月$4.8（AI $3.1 + storage $1.2 + misc $0.5）

## MVPスコープ
テキスト→レイアウト3案生成、フォント推薦、PNG書き出し、Stripe課金

## マーケ計画
Xで「フォント改善Before/After」短尺投稿、デザイン系Discordで無料クレジット配布

## 技術スタック
TypeScript, Next.js, Tailwind, Supabase, Stripe, OpenAI Responses API

## リスク
フォントライセンス。MVPはGoogle Fonts/Noto中心で法務リスク回避。

## 競合分析
Canvaは汎用、FontMuse JPは「日本語タイポ最適化」に特化

## $20達成シナリオ
Pro $6×4人= $24/月（達成）。初月は無料配布で20トライアル→CVR20%を目標

## ユニットエコノミクス
ARPU $6、変動費/人 $0.35、粗利率94%
