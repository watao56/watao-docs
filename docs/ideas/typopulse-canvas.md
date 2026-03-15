# 🎞️ TypoPulse Canvas

## 概要
音声メモや短文から、**日本語タイポグラフィ中心の縦型モーショングラフィック**を30秒で生成するAIクリエイティブツール。海外のCapCutテンプレ文化は強いが、日本語の可読性・改行美・縦長文脈に最適化した「文字主役」特化プロダクトはまだ薄い。

## 海外事例分析
- Captions / CapCut: テンプレ編集は強いが日本語タイポ最適化は弱い
- Submagic: 字幕演出は強いがブランド一貫性設計が弱い
- Canva Magic: 汎用性は高いが“短尺で刺さる文字演出”は手作業が多い
- 日本ギャップ: 和文フォント組版、句読点処理、縦スク向け余白設計

## ターゲット
- X/Instagram運用の個人クリエイター
- コーチ/講師業の情報発信者
- 小規模D2CのSNS担当

## 料金
- Free: 月8本、透かしあり
- Creator: $7/月（120本、ブランドプリセット3個）
- Pro: $12/月（400本、チーム2席、バッチ出力）

## ユーザーフロー
1. 音声 or テキスト入力
2. トーン（知的/熱量/ミニマル）選択
3. AIが3案生成（サムネ+30秒動画）
4. フォント・カラーを1クリック調整
5. MP4/PNG書き出し & 投稿用キャプション提案

## デザインコンセプト
- **“Neon Editorial”**
- 白黒ベースに1アクセントカラー
- 和文の行間を広く取り、視認性を最優先

## アーキテクチャ
- Front: Next.js + Tailwind
- API: FastAPI（レンダリングジョブ管理）
- Queue: SQS
- Worker: AWS Lambda + ffmpeg layer
- Storage: Cloudflare R2
- Auth/Billing: Supabase Auth + Stripe

## DB設計
- users(id, email, plan, created_at)
- projects(id, user_id, source_type, source_text, status)
- renders(id, project_id, style, duration_sec, output_url, cost_cents)
- brand_presets(id, user_id, font_family, palette_json)
- usage_daily(id, user_id, date, render_count, sec_total)
- subscriptions(id, user_id, stripe_sub_id, status)

## コスト見積もり（月）
- Vercel/Frontend: $0〜5
- Supabase: $0
- Lambda+SQS: $2
- R2: $1
- 推論/整形（miniモデル中心）: $4
- 合計: **約$7〜12/月**

## MVPスコープ
- 3スタイル固定
- 15秒/30秒のみ
- 1ブランドプリセット
- 生成履歴と再編集

## マーケ計画
- 「同じ文章→3演出比較」のショート動画配布
- Xで“今日の1本無料生成”企画
- 日本語フォント映え事例を毎日投稿

## 技術スタック
Next.js / TypeScript / FastAPI / Supabase / Stripe / AWS Lambda / SQS / ffmpeg / R2

## リスク
- レンダリング時間上振れ
- 既存動画ツールとの差別化不足

## 競合分析
- CapCut: 汎用編集 → **日本語タイポ自動最適化**で差別化
- Canva: 多機能 → **30秒以内完成**に特化

## $20達成シナリオ
- Creator 3名（$21 MRR）
- または Pro 2名（$24 MRR）

## ユニットエコノミクス
- ARPU: $7.8
- 変動費/ユーザー: $1.1
- 粗利/ユーザー: $6.7
- 粗利率: **85.9%**
