# 🎧 AudioBloom

- カテゴリ: AI×クリエイティブ（オーディオグラム自動生成）
- 目標: 月$20
- 想定評価: A

## 概要
ポッドキャスト/スペース録音をアップすると、字幕付きオーディオグラムを30秒で5パターン出力。日本語フォント最適化と縦動画テンプレで「見せたくなる音声切り抜き」を最短化。

## 海外事例分析
- Headliner: 音声→動画化の需要を実証。
- Riverside Magic Clips: 生成クリップ需要が拡大。
- 日本は「音声配信者の動画転用」がまだ手作業中心でギャップあり。

## ターゲット
- X/YouTube Shorts運用する個人ポッドキャスター
- 社内広報・採用広報で音声を配信する小規模企業

## 料金
- Free: 月5クリップ
- Creator $6/月: 月120クリップ、透かしなし
- Team $12/月: 共同編集2席

## ユーザーフロー
1) 音声アップロード
2) AIが見どころ候補を抽出
3) テンプレ選択
4) 字幕微調整
5) mp4書き出し・SNS共有

## デザインコンセプト
「Neon Wave」テーマ。波形アニメーション+太字日本語字幕+グラデ背景。静止画より動きで魅せる。

## アーキテクチャ
Next.js + Cloudflare R2 + ffmpeg。文字起こしはWhisper smallセルフホスト（CPU実行）でコスト抑制。

## DB設計
users(id,email,plan)
projects(id,user_id,title,audio_url,status)
clips(id,project_id,start_sec,end_sec,template,export_url)
usage_monthly(user_id,clips_count,minutes)

## コスト見積もり（月次）
- AWS Lightsail: $7
- R2ストレージ: $1
- 推論(Whisper): $2
- 合計: 約$10

## MVPスコープ（14日）
- アップロード/自動切り出し/字幕/5テンプレ/書き出し
- Stripe課金
- 使用量制限

## マーケ計画
- 「音声→縦動画3本無料」LP
- Podcast系Discordコミュニティでデモ配布
- Xでビフォーアフター動画投稿

## 技術スタック
Next.js, Tailwind, Postgres(Supabase), ffmpeg, Whisper, Stripe

## リスク
長尺処理遅延 → 非同期キュー化で緩和。著作権音源アップロード → 利用規約＋通報導線。

## 競合分析
Headlinerは英語UI中心。AudioBloomは日本語字幕品質・縦動画テンプレ・低価格で差別化。

## $20達成シナリオ
Creatorプラン4人($24 MRR)で達成。無料→有料転換率5%想定なら80無料ユーザーで到達。

## ユニットエコノミクス
ARPU $6、変動費$0.8/ユーザー、粗利率86.7%、回収期間<1ヶ月
