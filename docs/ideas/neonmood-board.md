# 🌈 NeonMood Board

## 概要
AIで「推し世界観」を30秒で生成し、テンプレに当て込むだけでSNS投稿用ムードボードを作れるマイクロSaaS。海外のムードボード文化を日本向けに軽量化。**デザイン映え**と**シェア動機**を最優先。

## 海外事例分析
- **Kosmik**: AI moodboard + infinite canvasが強い。学習コストは高め。
- **VSCO Canvas**: 写真文化との親和性が高く、クリエイター導線が秀逸。
- **Miro AI Moodboard**: チーム向けで重厚。個人クリエイターにはオーバースペック。
- 日本では「軽く作ってすぐ見せる」特化の単機能がまだ薄い。

## ターゲット
- Instagram/TikTok/Xで作品告知する個人クリエイター
- 同人/ハンドメイド/小規模ブランド運営者

## 料金
- Free: 月5ボード
- Lite: **$3/月**（月80ボード、透かし除去）
- One-shot: **$2/20ボード**

## ユーザーフロー
1. テーマ入力（例:「雨のネオン喫茶」）
2. AIが配色・フォント・構図を提案
3. 自動生成4案から1つ選択
4. テキスト差し替え→PNG出力
5. SNS投稿 + 作品リンク貼付

## デザインコンセプト
- 「夜のネオンサイン」配色
- 余白多め、カードUI、ミニマルアニメーション
- 「見せたくなる管理画面」重視

## アーキテクチャ
- Front: Next.js (Vercel Free)
- API: Cloudflare Workers
- 画像生成: Replicateの軽量モデル（必要時のみ）
- Storage: Cloudflare R2
- Auth/DB: Supabase

## DB設計
- users(id, email, plan, created_at)
- boards(id, user_id, prompt, theme_json, created_at)
- renders(id, board_id, output_url, cost_cents, created_at)
- events(id, user_id, type, meta_json, created_at)

## コスト見積もり（月）
- Vercel: $0
- Workers/R2: $1
- Supabase: $0
- 画像生成API: $6（有料利用者中心）
- 合計: **$7**

## MVPスコープ
- テンプレ12種
- テーマ生成（色/フォント/背景）
- PNG出力
- Stripe決済
- シェア導線（X/Instagram向け出力サイズ）

## マーケ計画
- Xで「1日1ムードボード」公開
- BOOTH/BASE制作者コミュニティで検証
- 初月は無料配布テンプレで獲得

## 技術スタック
Next.js / TypeScript / Supabase / Cloudflare Workers / Stripe

## リスク
- 画像APIコスト変動
- 著作権に近い生成物の扱い

## 競合分析
- Canva: 高機能すぎる
- Miro: チーム向け
- 本案: **個人クリエイター特化・超高速制作**

## $20達成シナリオ
- Lite $3 × 7人 = $21
- または One-shot $2 × 11件 = $22

## ユニットエコノミクス
- ARPPU: $3
- 変動費/有料ユーザー: 約$0.7
- 粗利率: 約76%
