# 🎤 BuildChant Club

## 概要
「今日やること」を15秒の音声チャントで投稿し、夜に結果を返す**音声コミュニティ型実行サービス**。テキスト疲れの反動を狙い、声ベースで軽く継続する。

## 海外事例分析
- **Geneva / Discordコミュニティ運営**: 少人数濃度が継続率を作る
- **Locket/BeReal系**: “軽く毎日出す”行為に価値
- **BandLab**: 音声・短尺参加の心理的ハードルが低い

## ターゲット
- build-in-public勢
- フリーランス
- 学習コミュニティ運営者

## 料金
- Free: 1ルーム参加
- Pro: $5/月（3ルーム + 週次サマリー）
- Host: $9/月（主催機能）

## ユーザーフロー
1. 朝チャント投稿（15秒）
2. 夕方リマインド
3. 夜リザルト投稿
4. 週次で達成率カード配布

## デザインコンセプト
カセットテープUI + 波形アニメ。ノスタルジー×現代。

## アーキテクチャ
- Next.js + WebRTC録音
- Supabase Storage
- OpenAI Whisper（文字起こし）
- Discord Bot連携

## DB設計
- users(id, plan)
- rooms(id, owner_id, theme)
- chants(id, room_id, user_id, audio_url, transcript)
- checkins(id, chant_id, status, memo)

## コスト見積もり（月）
- ホスティング: $5
- Storage: $1
- Whisper: $2
- 合計: **$6〜8**

## MVPスコープ
- 録音/投稿
- 夜リマインド
- 週次達成率カード

## マーケ計画
- 「朝15秒宣言」ハッシュタグ運用
- Discord運営者にルーム無料配布
- 7日チャレンジで初回定着

## 技術スタック
Next.js / Supabase / Whisper API / Discord API

## リスク
- 参加初期の過疎化
- 音声投稿の恥ずかしさ

## 競合分析
- Focusmate: 1on1同期中心
- Discord: 汎用すぎて習慣設計が弱い
- 本案: 音声宣言×実行確認の導線特化

## $20達成シナリオ
- Pro 4人（$20）で達成

## ユニットエコノミクス
- ARPU: $5
- 変動費/ユーザー: $0.6
- 粗利/ユーザー: $4.4
- 粗利率: 88%
