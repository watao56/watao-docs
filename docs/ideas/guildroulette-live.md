# 🎲 GuildRoulette Live

## 概要
クリエイター同士を毎日ランダムに2人マッチし、**15分で相互フィードバック**するコミュニティSaaS。Discord連携で「今夜だけ参加」が可能。

## 海外事例分析
- **ADPList/Lunchclub**: マッチング体験が強い
- **Focusmate**: 短時間セッション設計が継続を生む
- ギャップ: 日本向けに「匿名寄り・短時間・Discord中心」の導線が不足

## ターゲット
- デザイナー、動画編集者、個人開発者
- SNS発信の壁打ち相手が欲しい層

## 料金
- Free: 週2回参加
- Club: $5/月（毎日参加、過去FBログ保存）

## ユーザーフロー
1. Discord OAuth
2. 今日の参加ボタン
3. 21時に自動マッチ
4. 15分通話→テンプレFB送信

## デザインコンセプト
「**夜市の抽選会**」。ネオンカードで相手とテーマを表示。

## アーキテクチャ
- Next.js + Discord API
- Supabase Realtime
- Cloud Run（マッチングジョブ）

## DB設計
- users(id, discord_id, plan)
- sessions(id, date, theme, status)
- matches(id, session_id, user_a, user_b)
- feedbacks(id, match_id, from_user, body, score)

## コスト見積もり（月）
- Supabase: $0
- Cloud Run: $2
- Logs/monitoring: $1
- 合計: **$3**

## MVPスコープ
- 1日1セッション
- 2人マッチ固定
- フィードバックテンプレ3種

## マーケ計画
- Xで「#15分壁打ち」ハッシュタグ運用
- Discordコミュニティへの体験枠配布

## 技術スタック
Next.js, Supabase, Discord OAuth/Bot, Cloud Run

## リスク
- マッチ不成立時間帯 → 最低人数閾値で開催制御

## 競合分析
- ADPList: メンター文脈が強く気軽さが弱い
- Focusmate: 作業監視寄りでクリエイティブFB特化ではない

## $20達成シナリオ
- Club 4人 × $5 = $20/月

## ユニットエコノミクス
- ARPU: $5
- 変動費/人: $0.4
- 粗利率: 92%
