# 🎵 JamCapsule

## 概要
30秒の音声ループを投稿し、他ユーザーが重ねていく**連鎖型ミニ音楽コミュニティ**。AIでノイズ除去/簡易マスタリングを自動適用。

## 海外事例分析
- BandLab: モバイル音楽制作需要が大きい
- SoundCloud: コミュニティはあるが“短尺連鎖”特化ではない
- TikTok Duet文化: 連鎖参加体験は強い
→ 日本向けに「短尺・匿名・低ハードル」で再設計。

## ターゲット
- DTM初心者
- 配信者/BGM制作者
- 学生コミュニティ

## 料金
- Free: 投稿/参加 週3回
- Plus: $5/月（無制限参加、高音質書き出し）
- Crew: $15/月（小規模コミュニティ運営者向け）

## ユーザーフロー
1. お題ループ作成
2. 他ユーザーが重ね録り
3. AI整音
4. 完成チェーンを共有

## デザインコンセプト
「**ネオン波形UI**」。黒背景＋蛍光カラー、音の重なりが視覚で楽しい。

## アーキテクチャ
- Expo React Native + Web
- Backend: Firebase Auth/Firestore/Storage
- 音声処理: ffmpeg + 軽量DSP
- 通知: FCM

## DB設計
- users(id, handle, plan)
- loops(id, user_id, bpm, key, audio_url, waveform_url)
- chains(id, root_loop_id, status, participants_count)
- takes(id, chain_id, user_id, order_no, audio_url, mix_params)
- reactions(id, take_id, user_id, emoji)

## コスト見積もり（月）
- Firebase無料枠〜$5
- Storage超過: $3
- AI/DSP: OSS中心で$0〜$2
- 合計: **$3〜$10**

## MVPスコープ
- 30秒固定
- 3トラックまで重ね録り
- 1タップ整音
- 共有リンク生成

## マーケ計画
- 「今週のお題ループ」を毎週固定時刻で開催
- 音楽系Discordサーバーに公式Bot導入
- 完成チェーンをTikTokに自動縦動画化

## 技術スタック
Expo / Firebase / ffmpeg.wasm / Cloud Functions

## リスク
- 著作権侵害投稿 → 通報・自動ミュート対応
- 同時接続時の遅延 → 非同期アップロード中心で回避

## 競合分析
BandLabは高機能で重い。JamCapsuleは**初心者の参加体験に全振り**。

## $20達成シナリオ
Plus($5)を4人で$20達成。

## ユニットエコノミクス
- ARPU: $5
- 変動費: $0.4/人
- 粗利: 約92%
- LTV/CAC: 6以上を目標
