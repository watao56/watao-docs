# 🎬 HookCanvas

## 概要
🎬 HookCanvasは、AI×クリエイティブ（動画広告プリプロ）に特化した月$20目標のマイクロプロダクト。**AIエージェントだけで2週間以内にMVP実装**できるスコープに限定し、初期コストを最小化する。

## 海外事例分析
- 参照トレンド: Captions / CapCut Templates / Creatify系の海外成長
- 示唆: 日本市場ではローカライズ（日本語UI・国内決済・和文最適化）で差別化余地が大きい。

## ターゲット
- 日本の個人事業主・小規模EC・SNS運用代行

## 料金
- Free / Solo $6 / Pro $12

## ユーザーフロー
1) 商品URL or メモ入力 → 2) AIが「3秒フック+絵コンテ6コマ」生成 → 3) テンプレ書き出し（CapCut/Canva） → 4) 投稿結果を保存して次回改善

## デザインコンセプト
ネオン系ダークUI＋カードスワイプ。1画面で「Hook候補」「感情トーン」「尺」を同時比較。

## アーキテクチャ
Next.js + Supabase + Cloudflare R2。生成はOpenAI Responses API(mini)を優先、画像生成は任意でReplicate。

## DB設計（最小）
- users, projects, hook_variants, storyboard_frames, exports, usage_events

## コスト見積もり（月次）
- 10有料ユーザー時: 約$4.8/月（AI $3.6 + Infra $1.2）
- AWS/無料枠超過時はCloudflare/Vercel無料枠優先で吸収

## MVPスコープ（2週間）
- フック文生成、6コマ絵コンテ、テンプレDL、履歴、Stripe決済
- 認証、決済、利用制限、簡易分析（Mixpanel代替でeventsテーブル）

## マーケ計画（最初の30日）
- Xで「1商品=3フック」デモ動画を毎日投稿。初期は制作代行アカウントへDM営業。
- LPは1ページ、CTAは1つに絞る

## 技術スタック
- Next.js, TypeScript, Supabase, Stripe, OpenAI, Cloudflare R2

## リスク
- 生成品質ぶれ→テンプレ評価機能で学習。著作権懸念→素材アップロード禁止/テキスト中心

## 競合分析
- Canva Magic/CapCutは制作機能は強いが「売れるフック設計」に特化していない

## $20達成シナリオ
- Solo $6プラン4人= $24 MRR で達成（CVR 2%なら訪問200で現実的）
- 目標: 30日以内に達成可能な「少人数有料化」設計

## ユニットエコノミクス
- ARPU $6.8, 変動費/人 $0.42, 粗利 93.8%, 回収期間 <1ヶ月
- LTV/CACは初期フェーズでLTV>3x CACを最低ライン

## 実装指示（AIエージェント向け）
1. DB migration作成
2. 認証/課金/利用制限
3. コア機能1本を先に完成
4. ログ計測とエラーハンドリング
5. LP公開→初回ユーザー5人獲得
