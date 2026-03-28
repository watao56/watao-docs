# 🧘 TabSabbath

## 概要
タブ散乱を「閉じる儀式」に変える個人生産性ツール。ブラウザ拡張で開きっぱなしタブを分類し、**再開用の1画面サマリー**を生成して安心して閉じられる。

## 海外事例分析
- **Arc Browser/Edge Workspaces**: タブ管理需要は拡大
- **OneTab**: 保存はできるが再開コンテキストが弱い
- **Sunsama系**: 仕事の再開負荷削減が評価される
- 日本ギャップ: 「閉じると忘れる不安」を解消するUXが不足

## ターゲット
- リモートワーカー
- 研究・調査職
- マルチ案件のフリーランス

## 料金
- Free: 週3回までセーブ
- Pro: **$4/mo**（無制限＋日次Digest）
- Lifetime: $19

## ユーザーフロー
1. 拡張がタブ群を収集
2. AIが「次に再開すべき3タスク」要約
3. ワンクリックでタブを閉じる
4. 翌日、Digestから再開

## デザインコンセプト
"**Calm shutdown**"。夜間モード、禅庭UI、閉じる瞬間の達成演出。

## アーキテクチャ
- Extension: Plasmo
- API: Cloudflare Workers
- DB: D1 or Supabase
- AI: OpenAI Responses mini
- Sync: Web Push + Email digest

## DB設計
- users(id, email, plan)
- snapshots(id, user_id, created_at, tab_count)
- items(id, snapshot_id, url, title, cluster)
- summaries(id, snapshot_id, text, token_cost)
- resumes(id, user_id, snapshot_id, resumed_at)

## コスト見積もり（月）
- Workers + D1: $5
- AI: $6
- Email: $1
- 合計: **$12**

## MVPスコープ
- Chrome拡張
- タブクラスタリング
- 要約生成
- 翌日再開リンク

## マーケ計画
- Product Hunt mini launch
- Xで「100タブ→10秒で閉じる」デモ動画
- Notionテンプレ同梱で配布

## 技術スタック
Plasmo / TypeScript / Cloudflare Workers / D1 / Stripe

## リスク
- 拡張審査遅延
- タブURLのプライバシー懸念

## 競合分析
OneTab/Tobyより「再開しやすさ」差別化。タスク再開に最適化。

## $20達成シナリオ
- Pro 5ユーザー = $20
- 変動費$4前後で黒字化

## ユニットエコノミクス
- ARPU: $4
- 変動費/有料ユーザー: $0.8
- 粗利率: 80%
