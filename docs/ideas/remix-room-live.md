# 🪩 RemixRoom Live

## 概要
🪩 RemixRoom Liveは、ソーシャル/コミュニティ（毎日15分リミックス会）を狙う月$20到達向けの超小型プロダクト。

## 海外事例分析
- 参照トレンド: Focusmate / ADPList group sessions / Discord Stage culture
- 示唆: 海外では「短時間で成果が出る」「見た目がシェアしやすい」体験が伸びている。
- 日本向け再設計: 導入ハードルを下げ、テンプレ主導で初回価値を10分以内に提供。

## ターゲット
- 作業配信者・デザイナー・勉強会主催者

## 料金
- Host $8/月（開催者） 参加者無料

## ユーザーフロー
- ホストがお題作成→参加者入室→15分制作→成果1枚投稿→AIがハイライトカード生成

## デザインコンセプト
- クラブルーム風のタイマーとリアクション演出。参加したくなるライブ感。

## アーキテクチャ
- Next.js + Supabase Realtime + Discord OAuth + OpenAI summary

## DB設計
- 主テーブル: users, rooms, sessions, submissions, highlights
- 最小ER方針: usersを親に、課金対象エンティティを1:Nで接続。監査用created_atを全テーブルに付与。

## コスト見積もり（AWS or 無料サービス想定）
- 固定費$0〜10 + AI要約$0.002/セッション
- AWS移行時: CloudFront + Lambda + DynamoDBでも月$5〜15で運用可能。

## MVPスコープ
- ルーム作成、タイマー、提出ギャラリー、ハイライト画像自動化

## マーケ計画
- Discordコミュニティ3つに無料導入。週次ランキング画像で拡散。
- 初月KPI: 無料登録30 / 有料転換3〜5。

## 技術スタック
- Next.js, Supabase, Cloudinary, OpenAI API

## リスク
- 主リスク: 初期流動性不足
- 対策: テンプレ改善を週次で回し、解約理由を5件単位で反映。

## 競合分析
- Focusmateは1on1、RemixRoomは「成果可視化つき小規模群衆」

## $20達成シナリオ
- 有料ホスト3名で$24、原価<$2で達成

## ユニットエコノミクス
- ARPU $8, 変動費$0.5/ホスト, 粗利率93%
- 目安LTV/CAC: 3以上（SNSオーガニック前提）。
