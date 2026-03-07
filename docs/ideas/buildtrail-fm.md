# 🎙️ BuildTrail FM

## 概要
作業進捗を30秒音声で投稿し、仲間がスタンプ/短い音声で返す**非同期ビルドコミュニティ**。海外のbuild-in-public文化を、音声中心で日本向けに再設計。

## 海外事例分析
- **Geneva / Circle**: クローズドコミュニティ運営
- **Loom voice notes**: 非同期報告の心理的負荷低減
- **Discord Stage文化**: 声の近さが継続率を上げる

## ターゲット
- 個人開発者
- デザイン学習者
- 小規模チームのデイリーチェックイン需要

## 料金
- Free: 1ルーム
- Crew: $6/月（5ルーム）
- Host: $12/月（50人まで）

## ユーザーフロー
1. ルーム作成（例: 30日Ship）
2. 毎日30秒ボイス投稿
3. AIが3行サマリ自動生成
4. 週次で"Build Reel"を自動作成して共有

## デザインコンセプト
- "Cassette + Neon"
- 波形アニメーションで進捗を可視化
- 1タップ録音を最優先

## アーキテクチャ
- Front: Next.js + PWA
- Backend: Cloudflare Workers
- Audio: Cloudflare R2 + ffmpeg
- Transcribe/Summary: Whisper API + GPT-5.3 mini
- Realtime: Supabase Realtime

## DB設計
- rooms(id, owner_id, name, visibility)
- members(id, room_id, user_id, role)
- voice_posts(id, room_id, user_id, audio_url, transcript, summary)
- reactions(id, post_id, user_id, emoji)
- digest_weekly(id, room_id, reel_url, created_at)

## コスト見積もり（月）
- Workers/R2: $5
- Whisper+LLM: $6
- Supabase: $0〜5
- 合計: **$11〜16**

## MVPスコープ
- 録音投稿
- 自動文字起こし
- 週次ダイジェスト生成
- ルーム招待リンク

## マーケ計画
- indiehacker系Discordに英語版デモ投稿
- 日本の個人開発コミュニティで30日チャレンジ開催
- "声だけ進捗"UGCをXで拡散

## 技術スタック
Next.js / Cloudflare Workers / Supabase / Whisper / Stripe

## リスク
- 音声投稿の習慣化失敗
- モデレーションコスト

## 競合分析
- Discord: 汎用で進捗特化UIがない
- Circle: 重めで高単価
- 差別化: 音声進捗＋AI自動ダイジェスト

## $20達成シナリオ
- Crew($6)×2 + Host($12)×1 = $24

## ユニットエコノミクス
- ARPU: $8.5
- COGS/user: $0.9
- 粗利: 89%
