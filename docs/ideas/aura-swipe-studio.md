# 🌌 AuraSwipe Studio

## 概要
🌌 AuraSwipe Studioは、AI×クリエイティブ（写真→トレンド演出スワイプ動画）を狙う月$20到達向けの超小型プロダクト。

## 海外事例分析
- 参照トレンド: Canva Magic Studio / Captions / PhotoRoom
- 示唆: 海外では「短時間で成果が出る」「見た目がシェアしやすい」体験が伸びている。
- 日本向け再設計: 導入ハードルを下げ、テンプレ主導で初回価値を10分以内に提供。

## ターゲット
- Instagram/TikTok運用する個人クリエイター・小規模D2C

## 料金
- Free + Pro $6/月（動画書き出し無制限）

## ユーザーフロー
- 素材3枚アップロード→AIが5パターン演出提案→テンプレ選択→30秒動画出力→SNS投稿リンク生成

## デザインコンセプト
- ネオン粒子＋カードスワイプUI。生成結果を「見せたくなる」質感に寄せる。

## アーキテクチャ
- Next.js + Cloudflare R2 + Replicate API + Supabase

## DB設計
- 主テーブル: users, projects, assets, renders, subscriptions
- 最小ER方針: usersを親に、課金対象エンティティを1:Nで接続。監査用created_atを全テーブルに付与。

## コスト見積もり（AWS or 無料サービス想定）
- 固定費 $0〜5（Vercel/Supabase無料枠）+ 従量$0.03/動画
- AWS移行時: CloudFront + Lambda + DynamoDBでも月$5〜15で運用可能。

## MVPスコープ
- 画像3枚から縦動画を自動生成、字幕テンプレ2種、透かしON/OFF

## マーケ計画
- Xで「1素材→5演出」デモ動画を毎日投稿。初月は無料テンプレ配布。
- 初月KPI: 無料登録30 / 有料転換3〜5。

## 技術スタック
- Next.js, Tailwind, Supabase, Stripe, Replicate

## リスク
- 主リスク: 生成品質のムラ
- 対策: テンプレ改善を週次で回し、解約理由を5件単位で反映。

## 競合分析
- Canvaは汎用、AuraSwipeは「短尺訴求動画」特化で時短価値

## $20達成シナリオ
- Pro 4人（$24）で達成。従量原価約$3.6、粗利約$20.4

## ユニットエコノミクス
- ARPU $6, 変動費$0.9/人, 粗利率85%
- 目安LTV/CAC: 3以上（SNSオーガニック前提）。
