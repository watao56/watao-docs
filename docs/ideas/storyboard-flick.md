# 🎬 Storyboard Flick

## 概要
🎬 Storyboard Flickは、AI×クリエイティブ（音声→縦型ストーリーボード）の月$20到達を狙う小粒プロダクト。

## 海外事例分析
- CapCut Templates / Canva Magic Media / Typefullyの即時編集UX
- 海外で成立している行動（短尺投稿/コミュニティ連鎖/軽量自動化）を日本語UXに再設計する。

## ターゲット
X/Instagram発信を継続したい個人開発者・小規模クリエイター

## 料金
Free（週2本） / Creator $6/月（週20本） / Pro $12/月（無制限+ブランドキット）

## ユーザーフロー
1) 音声メモ30-90秒をアップ 2) AIが要点3幕に分解 3) 9:16カード+字幕を生成 4) 1タップで書き出し

## デザインコンセプト
ネオン＋フィルムグレイン。1画面1意思決定（録る/選ぶ/出す）で迷わせない。

## アーキテクチャ
Next.js + Cloudflare R2 + AWS Lambda(Whisper/LLM) + FFmpeg + Supabase

## DB設計
- users, projects, clips, storyboard_frames, exports, billing_events

## コスト見積もり
固定費: Supabase無料枠 + Vercel無料枠 + S3互換R2 $0〜2。AI: Whisper tiny + gpt-4o-mini想定で1本$0.01〜0.03。10有料ユーザーで月$4〜8。

## MVPスコープ
音声→3幕カード生成、テンプレ2種、MP4書き出し、Stripe課金

## マーケ計画
海外の"talking head fatigue"文脈を日本向けに「声だけで映える台本化」で訴求。Xに毎日実例投稿。

## 技術スタック
TypeScript, Next.js, Supabase Postgres, OpenAI API, FFmpeg, Stripe

## リスク
生成品質のブレ→テンプレ制約で安定化。著作権→素材アップロード規約を明示。

## 競合分析
CapCutは編集自由度高いが企画化が弱い。Canvaはデザイン強いが音声起点が弱い。

## $20達成シナリオ
$6プラン4人=月$24で達成。初月目標は無料20人→有料転換20%。

## ユニットエコノミクス
ARPU $6.8, 変動費/ユーザー$0.5, 粗利約92%。
