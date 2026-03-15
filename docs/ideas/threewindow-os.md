# 🪟 ThreeWindow OS

## 概要
「今やること」を常に3枠だけ表示する、**超軽量デイリー実行OS**。海外の“less but better”生産性トレンド（Amie/Routine系）を、日本語ユーザー向けに1画面完結で再設計。

## 海外事例分析
- Sunsama / Routine: 強力だが重い・高価格
- Amie: UIは優秀だが日本語ユーザーの導線が弱い
- Goblin Tools: 単機能が刺さる
- 日本ギャップ: 機能過多なタスク管理に疲弊、続かない

## ターゲット
- タスク管理が続かない個人
- 副業持ち会社員
- ADHD傾向で選択過多が苦手なユーザー

## 料金
- Free: 1日3枠、履歴7日
- Plus: $4/月（履歴無制限、テンプレ）
- Coach: $9/月（週次レポートPDF）

## ユーザーフロー
1. 朝に“今日の3枠”を決める
2. 1枠だけフォーカス表示
3. 完了時に1行ログ
4. 夜に自動サマリー

## デザインコンセプト
- **“Calm Terminal”**
- 黒背景＋1色アクセント
- 余計なボタンを徹底排除

## アーキテクチャ
- PWA（Next.js）
- Supabase DB
- cronで日次サマリー
- PDF出力はCloudflare Worker

## DB設計
- users(id, email, tz, plan)
- daily_windows(id, user_id, date, slot1, slot2, slot3)
- logs(id, daily_window_id, slot_no, note, done_at)
- summaries(id, user_id, date, summary_text)
- subscriptions(id, user_id, plan, status)

## コスト見積もり（月）
- Hosting: $0
- Supabase: $0
- 通知メール: $1
- LLM要約: $1
- 合計: **約$2/月**

## MVPスコープ
- 3枠作成/完了
- 日次サマリー
- 週間CSV出力
- Stripe課金

## マーケ計画
- 「Todoを捨てて3つだけ」訴求
- Xで7日チャレンジ
- 仕事術系ニュースレターに寄稿

## 技術スタック
Next.js(PWA) / Supabase / Stripe / Resend / OpenAI mini

## リスク
- シンプルすぎて無料代替される
- 習慣化しないと解約される

## 競合分析
- Todoist: 高機能 → **3枠固定**で意思決定コスト削減
- Sunsama: 高価格 → **$4**の導入しやすさ

## $20達成シナリオ
- Plus 5人（$20 MRR）
- または Coach 3人（$27 MRR）

## ユニットエコノミクス
- ARPU: $4.7
- 変動費/人: $0.2
- 粗利/人: $4.5
- 粗利率: **95.7%**
