# 🧵 StreakPoster

## 概要
「毎日継続」をSNSで見せるための自動ポスター生成ツール。GitHub・学習記録・読書ログを取り込み、**1日1枚の美しい進捗カード**を自動投稿予約する。

## 海外事例分析
- **GitHub Skyline / WakaTime share cards**: 可視化需要が強い
- **Typefully / Hypefury**: 投稿支援への課金は成立済み
- 日本では継続可視化を“デザイン資産”にするツールが不足。

## ターゲット
- 個人開発者、資格学習者、発信を続けたい人

## 料金
- Basic: $2/月（1連携）
- Pro: **$5/月**（3連携、予約投稿、ブランドカラー）

## ユーザーフロー
1. GitHub/Notion/読書ログを連携
2. テーマ選択（Neon/Minimal/Paper）
3. 自動生成カードを確認
4. X/Discordへ予約投稿

## デザインコンセプト
- 「飾りたくなる進捗」
- グラデーションとタイポ重視、共有前提の高コントラスト

## アーキテクチャ
- Next.js + Edge Functions
- Scheduler: cron + queue
- Render: Satori + Resvg
- DB: Supabase

## DB設計
- users, integrations, streak_metrics, templates, renders, schedules, posts

## コスト見積もり（月）
- Hosting/DB: $0
- 投稿API維持: $1
- AIコピー補助: $1
- 合計: **$2**

## MVPスコープ
- GitHub連携
- 3テンプレ
- PNG生成
- 予約投稿（Discord優先）

## マーケ計画
- #buildinpublic ハッシュタグ向け無料テンプレ配布
- 「100日継続カード」キャンペーン

## 技術スタック
Next.js, Satori, Supabase, Cloudflare Queue, Stripe

## リスク
- SNS API制限
- 対策: 手動DL投稿導線を常備

## 競合分析
- Typefully: 投稿管理中心
- WakaTime: 開発指標中心
- 差別化: 継続のビジュアル表現特化

## $20達成シナリオ
- Pro $5 x 4人 = $20

## ユニットエコノミクス
- ARPU: $4
- 変動費: $0.2
- 粗利率: 95%
