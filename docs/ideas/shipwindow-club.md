# 🚢 ShipWindow Club

## 概要
「今日つくったもの」を15秒で共有する**非同期ショーケース型コミュニティ**。毎日開く“放課後の展示会”を再現し、継続制作を促進。

## 海外事例分析
- **WIP.co / Indie Hackers**: build in public需要は高い
- **Partiful**: 参加体験のデザインが成長を牽引
- テキスト中心コミュニティは継続率が下がりやすく、短尺ビジュアル化が有効

## ターゲット
- 個人開発者
- デザイナー
- 副業で制作する会社員

## 料金
- Free: 1グループ参加
- Creator: $5/月（自分の部屋を作成）
- Goal: 4人で$20

## ユーザーフロー
1. 今日の成果を1枚/15秒で投稿
2. 自動で「Showcaseカード」化
3. 他メンバーが絵文字投票
4. 週次でベスト作品を自動ハイライト

## デザインコンセプト
- ネオン掲示板UI
- 投稿をカード化して「見せたくなる」体験

## アーキテクチャ
- Next.js + Supabase Realtime
- 画像最適化: Cloudflare Images
- AI要約: OpenAI mini（週次まとめのみ）

## DB設計
- users(id, handle, plan)
- clubs(id, owner_id, theme)
- posts(id, club_id, user_id, media_url, note)
- reactions(id, post_id, user_id, emoji)
- weekly_digest(id, club_id, summary)

## コスト見積もり（月）
- Supabase: $0〜$5
- AI要約: $0.5
- 画像配信: $1
- 合計: 約$6.5

## MVPスコープ
- 画像/短動画投稿
- リアクション
- 週次まとめ
- 有料プラン

## マーケ計画
- 「7日連続シップチャレンジ」企画
- Xで作品カード自動シェア

## 技術スタック
Next.js / Supabase / Cloudflare Images / Stripe

## リスク
- 初期ネットワーク不足: テーマ別の公開部屋を先に運営

## 競合分析
- Discordは汎用すぎる
- 本サービスは「制作成果共有」導線に全振り

## $20達成シナリオ
- 初期コミュニティ30人→有料部屋オーナー4人

## ユニットエコノミクス
- ARPU: $5
- 変動費/人: $0.7
- 粗利率: 86%
