# 🫂 CreatorRelay Rooms

## 概要
1人が投稿→次の1人が24h以内に改善案を返す「バトン」形式で継続しやすい。月\$20達成を最短で狙うため、初期は単機能MVPで開始する。

## 海外事例分析
- 参照: ADPList / Lunchclub / Geneva / Build in Public
- 海外で伸びる要因: 体験の即時性・共有導線・テンプレ資産
- 日本向け差分: 日本語フォント/文脈最適化、LINE/X導線、価格を低単価に調整

## ターゲット
- 主対象: 副業クリエイター、インディーハッカー、学生クリエイター
- 初期獲得チャネル: X、Discord、知人コミュニティ

## 料金
- Free 1ルーム / Host $5/月 / Crew $15/月
- 返金ポリシー: 初月7日返金

## ユーザーフロー
1. LPから無料登録
2. 初回テンプレを選択
3. 1分以内に最初の成果物を体験
4. 共有/保存で継続利用
5. 使用量上限で有料転換

## デザインコンセプト
駅伝UI。バトン進捗がリングで可視化され、達成時に記念カード生成。

## アーキテクチャ
Next.js + Supabase Realtime + Discord OAuth + Cloudflare Images

## DB設計
主要テーブル: users, rooms, batons, feedbacks, streaks, showcases

## コスト見積もり（月次）
Supabase Pro不要(無料枠運用) + 画像配信$1未満。AI要約は月$2上限。

## MVPスコープ（2週間）
(1) ルーム作成 (2) バトン投稿 (3) 期限通知 (4) 週次ショーケース自動生成

## マーケ計画
Discordコミュニティへ無料導入、週次ベスト改善をX連携投稿。

## 技術スタック
Next.js, Supabase, Discord API, Cloudflare Worker Cron

## リスク
過疎化リスク→3人最小ルーム制と週1自動ハイライトで再活性化

## 競合分析
Circle/Genevaは汎用コミュニティ。CreatorRelayは「改善バトン」体験に特化。

## \$20達成シナリオ
Host 5人で$25 MRR。運営者課金モデルで少人数でも成立。

## ユニットエコノミクス
ARPU $5, 原価$0.25/人, 粗利95%
