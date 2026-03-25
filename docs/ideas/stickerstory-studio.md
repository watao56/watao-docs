# 🎟️ StickerStory Studio

## 概要
1枚の写真または商品画像から、**Instagram/LINE向けのアニメーションステッカー8種**を30秒で生成するAIマイクロツール。静止画投稿より保存率が高い「動くステッカー文化」を日本向けに最適化する。

## 海外事例分析
- **Instagram AI Stickers**: ストーリー内でAIステッカー生成機能を提供
- **Canva/VEED系ステッカージェネレータ**: 生成は強いが、日本SNS向けサイズ・運用テンプレが弱い
- ギャップ: 日本の個人店・クリエイターは「投稿導線込み」の即利用素材を求める

## ターゲット
- Instagram運用中の個人店（美容室、カフェ、ハンドメイド）
- ショート動画投稿者

## 料金
- Free: 月10ステッカー
- Pro: $6/月（無制限、商用利用、ブランドプリセット）

## ユーザーフロー
1. 画像アップロード
2. スタイル選択（Pop/Kawaii/Minimal/Neon）
3. 8枚一括生成（PNG+WebM）
4. SNSプリセットでDL

## デザインコンセプト
「**貼りたくなるギャラリー**」。生成結果をカルーセル表示し、ワンタップDL。

## アーキテクチャ
- Next.js + Cloudflare R2
- Replicate API（画像生成/背景透過）
- AWS Lambda（変換バッチ）

## DB設計
- users(id, plan, created_at)
- projects(id, user_id, source_url, style, created_at)
- assets(id, project_id, type, url, size)
- usage_logs(id, user_id, credits_used, created_at)

## コスト見積もり（月）
- Vercel Hobby: $0
- R2: $1
- Replicate: $8（200生成想定）
- 合計: **$9**

## MVPスコープ
- 4スタイル
- 8枚一括生成
- IG/LINEサイズ書き出し
- Stripe課金

## マーケ計画
- Instagramで「素材配布」リール投稿
- Canvaテンプレ作者との相互紹介
- 初月無料クーポン

## 技術スタック
Next.js, TypeScript, Supabase, Stripe, Replicate, Cloudflare R2

## リスク
- 生成品質のばらつき → スタイル固定で安定化
- APIコスト増 → Proに生成上限のフェアユース

## 競合分析
- Canva: 汎用で強いが日本SNS運用に特化薄
- VEED: 動画中心でステッカー運用導線が弱い

## $20達成シナリオ
- Pro 4ユーザー × $6 = $24/月

## ユニットエコノミクス
- ARPU: $6
- 変動費/人: $1.4
- 粗利/人: $4.6（粗利率77%）
