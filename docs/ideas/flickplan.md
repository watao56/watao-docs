# 📓 FlickPlan

## 概要
「1日を映画のショット割りで設計する」パーソナル生産性ツール。タスクを“Scene”として並べ、開始時に30秒の演出カードを出す。続けたくなるUXを最優先。

## 海外事例分析
- Sunsama: 1日計画の価値が高い
- Finch: 継続を感情UXで支える設計
- Locket/BeReal文脈: 日次共有の軽さが定着要因

## ターゲット
- クリエイター、個人開発者、ADHD傾向のある知的労働者

## 料金
- Free: 1日3シーン
- Pro: $6/月（無制限 + 共有）

## ユーザーフロー
1. 朝に3シーン作成
2. 各シーン開始でフォーカスカード表示
3. 完了で“リール”として1日を振り返り

## デザインコンセプト
フィルムUI、暗背景、巨大タイポ。進捗が“作品”に見える。

## アーキテクチャ
PWA（Next.js）+ Supabase。通知はWeb Push。

## DB設計
- users(id, timezone, theme)
- scenes(id, user_id, title, est_min, order_no, status)
- sessions(id, scene_id, start_at, end_at)
- recaps(id, user_id, summary)

## コスト見積もり（月）
- Hosting/DB: $0
- AI要約（任意）: $1.5
- 合計: 約$1.5

## MVPスコープ
- シーン作成/実行
- 日次リキャップ
- 共有画像

## マーケ計画
- 「#今日の3scene」投稿テンプレ配布
- Notionコミュニティへ配布

## 技術スタック
Next.js, Supabase, Workbox, OpenAI mini（任意）

## リスク
- 単なるToDo化 → 演出/振り返り価値を前面に

## 競合分析
一般ToDoより“使う動機”が強い感情設計が武器。

## $20達成シナリオ
- Pro 4人（$24）

## ユニットエコノミクス
- ARPU: $6
- 変動費/人: $0.15
- 粗利率: 97.5%
