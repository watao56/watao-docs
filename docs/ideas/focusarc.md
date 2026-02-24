# 🌀 FocusArc

- カテゴリ: パーソナル生産性（集中セッション可視化）
- 目標: 月$20
- 想定評価: A

## 概要
1日を「90分アーク（弧）」で設計し、実績を自動記録。作業ログが美しいタイムライン画像になるパーソナル集中ツール。

## 海外事例分析
- Time-blocking系(Sunsama/Motion)は高価格帯。
- Flowtime/Deep Work記録アプリは海外で伸びるが日本語UI特化が少ない。

## ターゲット
- 在宅フリーランス
- 受験/資格勉強ユーザー
- 毎日の集中可視化をしたい個人

## 料金
- Free: 1日2アーク
- Plus $4/月: 無制限+週次レポート
- Plus Yearly $36

## ユーザーフロー
1) 今日の3アーク設定
2) タイマー開始
3) 終了時に達成メモ
4) 日次カード自動生成
5) SNS/日報へ共有

## デザインコンセプト
「Glass Arc」。半透明カード、円弧グラフ、夜間モード重視で習慣化を促進。

## アーキテクチャ
PWA中心。ローカル優先保存+クラウド同期は任意。画像生成はSatori+Resvg。

## DB設計
users(id,plan,timezone)
arcs(id,user_id,start_at,end_at,goal,result)
streaks(user_id,current_days,max_days)
exports(id,user_id,date,image_url)

## コスト見積もり（月次）
- Cloudflare Pages無料
- Supabase Free
- 画像生成関数: $1
- 合計: $1〜$3

## MVPスコープ（14日）
- 90分タイマー
- 目標/結果ログ
- ストリーク
- 日次画像エクスポート
- 課金

## マーケ計画
- #今日の集中ログ ハッシュタグ導線
- Study系YouTuberに紹介依頼
- 無料テンプレ配布

## 技術スタック
Next.js(PWA), Supabase, Cloudflare, Stripe

## リスク
競合多い → 「見せたくなる日次カード」で差別化。離脱 → 7日オンボーディング導入。

## 競合分析
Togglは計測寄り、FocusArcは習慣化と共有体験寄り。低価格で導入しやすい。

## $20達成シナリオ
Plusユーザー5人で$20到達。B2Cなので薄利多売だが運用コスト極小。

## ユニットエコノミクス
ARPU $4、変動費$0.15、粗利96.3%、1人獲得CPA<$3を維持。
