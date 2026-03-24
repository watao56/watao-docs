# 🧩 TinyRituals OS

## 概要
1日3つの「極小ルーティン」（3分以内）をAIが提案し、完了するとホーム画面用の美しい進捗カードを生成。**個人向け生産性×見せたくなるUI**。

## 海外事例分析
- Finch/stoic系: 習慣アプリは感情設計が鍵。
- Notionテンプレ運用: 習慣管理は継続導線が弱い。
- 日本市場で「短時間 + 視覚リワード」特化は余地あり。

## ターゲット
- 習慣化が苦手な会社員
- 朝活したい副業層

## 料金
- Free: 1日1提案
- Plus: **$2.5/月**（3提案、履歴、カード書き出し）

## ユーザーフロー
1. 生活タイプを選択
2. 今日の3つを受け取る
3. 完了タップ
4. 日次カード生成
5. ロック画面保存/共有

## デザインコンセプト
- 余白+グラデーション
- 「達成が積み上がる」タイムライン

## アーキテクチャ
- PWA (Next.js)
- API: Supabase Edge Functions
- 画像カード生成: Satori + Resvg

## DB設計
- users(id, chronotype, plan)
- rituals(id, user_id, text, estimated_min)
- completions(id, ritual_id, completed_at)
- cards(id, user_id, date, image_url)

## コスト見積もり（月）
- Vercel + Supabase: $0
- 画像生成: $0（自前レンダリング）
- 合計: **$0〜$3**

## MVPスコープ
- ルーティン提案
- 完了記録
- 日次カード画像生成
- 課金

## マーケ計画
- Xで「3分習慣テンプレ」連載
- Product Hunt / Reddit r/productivity

## 技術スタック
Next.js / Supabase / Satori / Stripe

## リスク
- 習慣系は解約率が高い

## 競合分析
- Habitica: ゲーム寄り
- 本案: **最小行動 + 美しい成果物配布**

## $20達成シナリオ
- Plus $2.5 × 8人 = $20

## ユニットエコノミクス
- ARPPU: $2.5
- 変動費: $0.2
- 粗利率: 90%+
