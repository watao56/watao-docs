# 🪞 FitFlick Mirror

## 概要
1枚のセルフィーと商品URLから、試着風の15秒縦動画を自動生成。SNSで「見せたくなる試着体験」を作る。

## 海外事例分析
Doji/Google Try-on機能の伸長、Canvaのテンプレ文化を参考に、日本向けに「購入前シェア」導線へ最適化。

## ターゲット
個人アパレル出品者、ハンドメイド作家、Instagram/TikTok運用の小規模EC

## 料金
Free: 月5本 / Pro: $9/月（80本） / Pack: $5（50本追加）

## ユーザーフロー
セルフィー登録→商品画像取り込み→スタイル選択→15秒動画生成→SNS投稿用コピー同時出力

## デザインコンセプト
Y2KミラーUI。ネオン枠＋ビフォーアフター2画面。出力に透過ロゴスタンプで拡散。

## アーキテクチャ
Next.js + Supabase + Cloudflare R2。生成はReplicate(try-on系)+FFmpeg合成。キューはUpstash QStash。

## DB設計
- users, projects, generated_clips, credit_ledger, style_presets, share_events

## コスト見積もり（AWS/無料サービス前提）
固定: $0〜8（Vercel/Supabase無料枠）+ 変動: 1本あたり$0.03〜0.07。月200本で約$12。

## MVPスコープ
動画長15秒固定、テンプレ3種、手動リトライ、Stripe決済のみ。

## マーケ計画
Xで#今日の試着リール企画、minis向けテンプレ配布、Etsy/BASEコミュニティでデモ投稿。

## 技術スタック
Next.js, TypeScript, Supabase, Stripe, Replicate API, FFmpeg, PostHog

## リスク
生成品質のブレ。対策: 顔保持率しきい値、失敗時クレジット返却。

## 競合分析
CapCut/Canvaは汎用編集。FitFlickは「試着特化＋商品URLから即出力」で差別化。

## $20達成シナリオ
Pro 3人($27)で達成。広告費ゼロでも既存顧客の定期制作で継続しやすい。

## ユニットエコノミクス
ARPU $9, 粗利率約62〜70%, 回収期間<1か月。

## カテゴリ
AI×クリエイティブ（セルフィー→着用リール）
