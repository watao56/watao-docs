# 🪐 OrbitIntro

## 概要
海外の“warm intro network”文化を日本向けに実装。週1回、起業家/副業家を**相性スコアで2人マッチング**し、紹介テンプレまで自動生成するコミュニティSaaS。

## 海外事例分析
- Lunchclub: 相性マッチの継続率が高い
- Geneva/Circle: 小規模コミュニティの有料化が進む
- ADPList: メンタリング文脈の高エンゲージメント

## ターゲット
- 個人開発者、マーケター、デザイナー、フリーランス

## 料金
- Member: $5/月（週1マッチ）
- Plus: $9/月（週2マッチ + フィルタ）

## ユーザーフロー
1. プロフィール作成
2. 目的タグ選択（集客/採用/壁打ち）
3. 毎週自動マッチ通知
4. DM導線と紹介文をワンクリック送信

## デザインコンセプト
「宇宙レーダーUI」。軌道線とカードで“つながる期待感”を演出。

## アーキテクチャ
Next.js + Supabase + Discord OAuth。マッチングは軽量ルールベース（AI補正は任意）。

## DB設計
- users(id, bio, tags, timezone)
- intents(id, user_id, goal, weekly_slots)
- matches(id, user_a, user_b, score, status)
- intros(id, match_id, intro_text)

## コスト見積もり（月）
- Supabase/Vercel: $0
- OpenAI（紹介文のみ）: $2
- 合計: 約$2

## MVPスコープ
- 週次マッチ
- 紹介文生成
- Discord通知

## マーケ計画
- Xで「今週の良かった出会い」テンプレ投稿
- コミュニティ運営者と提携して導入

## 技術スタック
Next.js, Supabase, Discord Bot API, Cron, OpenAI mini

## リスク
- 初期流動性不足 → 手動キュレーション期間を設定
- ドタキャン → 信頼スコア導入

## 競合分析
Lunchclubは英語圏中心。日本語・副業文脈特化で差別化。

## $20達成シナリオ
- Member 4人で達成（$20）

## ユニットエコノミクス
- ARPU: $5.8
- 変動費/人: $0.2
- 粗利率: 96%
