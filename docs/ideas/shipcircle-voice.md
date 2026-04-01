# 🗣️ ShipCircle Voice

## 概要
**ShipCircle Voice**は、毎日「30秒で今日やることを音声宣言」し、夜に成果音声を返す**超軽量コミュニティSaaS**。  
テキストより心理的負担が低く、継続率を上げる。

## 海外事例分析
- **Geneva / Discord私設コミュニティ**: 小さな熱量コミュニティが増加
- **Locket**: 近しい関係に短い日常を共有する体験が定着
- **Focusmate**: 他者存在による実行率向上
- 示唆: 日本向けは「声だけ・短時間・少人数リング」が適合

## ターゲット
- 個人開発者
- 副業クリエイター
- 学習コミュニティ主催者

## 料金
- Host Plan: **$8/month**（1コミュニティ50人まで）
- Member Plus: **$2/month**（音声アーカイブ検索）

## ユーザーフロー
1. コミュニティ作成
2. 朝の宣言を音声送信
3. 自動で「夜リマインド」
4. 夜の結果報告
5. 週次でAIが継続スコア可視化

## デザインコンセプト
- 「押すと録る」巨大な1ボタン
- 円形タイムライン（声の波形がリング化）
- スマホ前提、文字最小

## アーキテクチャ
- Front: Next.js PWA
- Backend: Supabase Edge Functions
- 音声処理: Whisper API（短尺のみ）
- 通知: Discord Webhook / Email

## DB設計
- circles(id, owner_id, name, plan)
- members(id, circle_id, user_id, role)
- voice_logs(id, member_id, kind, audio_url, transcript)
- streaks(id, member_id, current_days, last_posted_at)

## コスト見積もり（月）
- Supabase Pro: $25（初期はFree運用可）
- 音声認識API: $6
- Storage: $2
- 合計: **$8〜$33**（規模依存）

## MVPスコープ
- 録音投稿
- 朝/夜2種類の投稿枠
- 週次スコア
- Discord通知

## マーケ計画
- 「30秒宣言チャレンジ」をXで開催
- 既存DiscordコミュニティにBotとして導入
- Host向けテンプレ文面を無料配布

## 技術スタック
Next.js / Supabase / Whisper / Stripe

## リスク
- 継続率低下
- 録音ハードル
- 通知過多

## 競合分析
- Discord単体運用: ログが散逸
- Focusmate: 同時接続前提
- 本案: **非同期・音声特化**で差別化

## $20達成シナリオ
- Host Plan 3件 = $24/月
- または Host 2件($16) + Member Plus 2人($4) = $20

## ユニットエコノミクス
- Host ARPU: $8
- 粗利率: 85%前後（音声短尺前提）
- CAC回収: 1か月以内
