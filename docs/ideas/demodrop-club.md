# 🎬 DemoDrop Club

## 概要
毎週1本「30秒デモ」を投稿し、相互フィードバックと投票で改善を回すコミュニティSaaS。Discord連携で運営負荷を最小化。

## 海外事例分析
- 参考サービス: **WIP.co / Indie Hackers build in public / Product Hunt Ship**
- 海外では「短時間で見栄えの良い成果物を作る」需要が強い。日本市場では日本語UI・日本語テンプレ不足が参入余地。

## ターゲット
個人開発者、デザイン学習者、副業クリエイター

## 料金
Free: 投稿週1本 / Pro $6/月（無制限投稿・過去分析） / Host $15/月（コミュ運営者向け）

## ユーザーフロー
(1)動画URL貼付→(2)自動サムネ生成→(3)週次ボード公開→(4)投票/コメント→(5)改善タスク抽出

## デザインコンセプト
「文化祭の展示壁」風のカードUI。投稿が並ぶだけで楽しい見た目。

## アーキテクチャ
Next.js + Supabase Realtime + Discord Bot + Cloudinary

## DB設計（MVP）
- users, circles, demo_posts, votes, comments, weekly_reports, subscriptions

## コスト見積もり（月次）
AIは要約のみ（gpt-4o-mini）。1投稿あたり<$0.002。月間1000投稿でも$2前後。

## MVPスコープ（2週間）
投稿・投票・コメント・週次ランキング・Discord通知

## マーケ計画
Xで#buildinpublic日本語圏を毎日観測し、上位投稿者を招待。

## 技術スタック
Next.js, Supabase, Discord API, Cloudinary, OpenAI mini

## リスク
投稿が集まらない→最初はテーマ週（LP改善週など）で参加障壁を下げる。

## 競合分析
Discord単体より記録性が高く、Circleより軽い。

## $20達成シナリオ
Pro 4人で$24。Host 2人でも$30。

## ユニットエコノミクス
ARPU $6.8、粗利97%。紹介率をK>0.25目標。
