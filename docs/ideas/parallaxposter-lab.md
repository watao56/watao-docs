# 🌊 ParallaxPoster Lab

## 概要
1枚の商品画像から**奥行き付きパララックス動画（6〜10秒）**とLP用ヒーローセクションを自動生成するマイクロSaaS。海外のLPで増えている「静止画+軽い3D演出」を日本の小規模D2C/個人開発向けに低コスト化。

## 海外事例分析
- Apple/Stripe系LPの軽量モーション演出トレンド
- FramerコミュニティのParallax Heroテンプレ
- Canva/CapCutの「短尺で映える素材」需要
- 日本は動画制作外注の心理ハードルが高く、**単一画像から即生成**にギャップあり

## ターゲット
- Shopify/BASEの小規模D2C
- ノーコードLP制作者
- Xで新機能告知する個人開発者

## 料金
- Free: 月5本、透かしあり
- Lite: $7/月（120本、透かしなし）
- Credit Pack: $9/80本（買い切り）

## ユーザーフロー
1. 画像アップロード
2. 商品カテゴリ選択（ガジェット/コスメ等）
3. AIが深度推定→カメラパス生成→BGM候補提示
4. WebM/MP4/GIFと埋め込みコード出力
5. LP/SNSへ貼り付け

## デザインコンセプト
- 「Dark + Neon Depth」
- UIは1画面完結、視差プレビューを主役に
- “before/after”比較スライダーを中央配置

## アーキテクチャ
- Front: Next.js + Tailwind + Framer Motion
- API: FastAPI (Render/Fly.io可)
- Worker: AWS Lambda (Python, depth推定+ffmpeg)
- Storage: S3互換（Cloudflare R2でも可）
- Queue: SQS or Upstash Redis

## DB設計
- users(id, email, plan, credits, created_at)
- projects(id, user_id, input_image_url, style, status, created_at)
- renders(id, project_id, output_url, duration_sec, cost_usd, created_at)
- billing_events(id, user_id, type, amount_usd, provider, created_at)

## コスト見積もり（月）
- Hosting: $7
- Storage/CDN: $3
- 推論/変換: $6（200〜300本想定）
- 合計: **$16**

## MVPスコープ
- 画像1枚→3プリセット演出
- 720p出力
- 透かしON/OFF
- Stripe課金

## マーケ計画
- Xで「1枚画像→10秒ヒーロー」比較動画を毎日投稿
- Shopifyコミュニティにテンプレ配布
- Product Hunt / Uneedで英語版検証

## 技術スタック
Next.js, FastAPI, ffmpeg, Pillow/OpenCV, AWS Lambda, S3/R2, Stripe

## リスク
- 生成品質ばらつき → プリセット制御で品質固定
- 著作権画像アップロード → 利用規約+通報導線

## 競合分析
- CapCut/Canva: 汎用すぎてLP埋め込み最適化が弱い
- Framerテンプレ: 実装知識が必要
- 本案は**“画像1枚で実運用素材が完成”**に特化

## $20達成シナリオ
- Lite $7 × 3ユーザー = $21
- もしくは Credit Pack $9 × 3 = $27

## ユニットエコノミクス
- ARPPU: $7.8
- 変動費/有料ユーザー: $1.4
- 粗利率: 約82%
