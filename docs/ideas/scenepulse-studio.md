# 🎥 ScenePulse Studio

## 概要
ScenePulse Studioは、AI×クリエイティブ（音声メモ→シネマ틱日報リール）に特化した月$20到達を狙うマイクロプロダクト。

## 海外事例分析
- 参照潮流: Lapse / BeReal / Captions の「日常を作品化」潮流
- 日本でのギャップ: 英語圏前提・UIが重い・共有導線が弱い
- 勝ち筋: 日本語UI + 3分以内で価値体験 + 共有導線最適化

## ターゲット
日々の制作ログを見せたい個人開発者・クリエイター

## 料金
- プラン: Free / Pro $7/月（40本）
- 無料→有料転換導線: 初回の成功体験後に使用回数上限を提示

## ユーザーフロー
1. 音声30秒と写真1枚をアップロード
2. AIが3カット構成の縦リール台本+字幕生成
3. テーマ色を選んでカード/動画を書き出し
4. X/Instagramへ共有、プロフィールリンクに蓄積

## デザインコンセプト
「フィルム×ネオン」UI。余白多め、ワンタップで映える。共有時のOGPを強化。

## アーキテクチャ
Next.js(App Router) + Cloudflare Workers API + FFmpeg wasm(軽処理) + Queue

## DB設計
- 主要テーブル: users, projects, clips, share_events, subscriptions
- 監査/運用: created_at, updated_at, soft_deleteを全テーブルに付与

## コスト見積もり（月次）
- インフラ: $0〜$8（無料枠中心、超過時のみ従量）
- AIコスト: 約$0.06/ユーザー/月（Whisper small + 軽量要約）
- その他: Stripe手数料のみ

## MVPスコープ
音声→字幕要約、3テンプレ、PNG/MP4出力、Stripe課金、公開ギャラリー

## マーケ計画
Xで #buildinpublic テンプレ配布、無料3本で拡散誘発

## 技術スタック
- Frontend: Next.js / React
- Backend: Serverless Functions
- DB: Supabase or DynamoDB
- Billing: Stripe
- Analytics: PostHog

## リスク
生成品質ブレ→テンプレ固定で制御。著作権素材混入→アップロード制約。

## 競合分析
Canvaは汎用、ScenePulseは「日報映え」に特化。CapCutより作業時間が短い。

## $20達成シナリオ
Pro 3人($21) で達成

## ユニットエコノミクス
ARPU $7 / 変動費 $0.35 / 粗利95%
