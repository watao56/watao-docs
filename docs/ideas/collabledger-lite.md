# 🧾 CollabLedger Lite

## 概要
共同制作（動画/記事/デザイン）の作業履歴を自動タイムスタンプ化し、**報酬分配や貢献説明の証跡**を作るライトSaaS。保険型に近いが、監視ではなく「共同制作の信頼インフラ」に寄せる。

## 海外事例分析
- Frame.io comments: 映像制作の履歴価値が高い
- Notion/Google Docs history: 汎用だが証跡出力に弱い
- Contra/Fiverr dispute文脈: 証拠整理ニーズは強い

## ターゲット
- 2〜5人の制作チーム
- 副業コラボ（動画編集/デザイン）

## 料金
- Free: 1プロジェクト
- Lite: **$7/月**（10プロジェクト、PDF証跡エクスポート）

## ユーザーフロー
1. Slack/Discord/Drive連携
2. 作業イベントを自動記録
3. 週末に貢献サマリー生成
4. 分配交渉時にPDF共有

## デザインコンセプト
- 監査感より「制作ダッシュボード感」
- タイムラインを美しく可視化

## アーキテクチャ
Next.js + Supabase + cron集計 + PDF生成

## DB設計
- teams(id, owner_id, name)
- projects(id, team_id, title, status)
- events(id, project_id, actor, source, payload, occurred_at)
- summaries(id, project_id, week_key, contribution_json, pdf_url)

## コスト見積もり（月）
- Hosting/DB: $9
- AI要約: $3
- 合計: **$12**

## MVPスコープ
- Discord手動インポート
- 週次サマリー
- PDF出力

## マーケ計画
- クリエイター向けDiscordへ実例投稿
- 「分配トラブルを減らす」ストーリー訴求

## 技術スタック
Next.js, Supabase, BullMQ, OpenAI mini, Puppeteer(PDF)

## リスク
- 監視ツール誤認: 透明性重視のUIコピー
- 連携実装コスト: まずCSV/手動投入で検証

## 競合分析
- 汎用PMツールは証跡化が弱い
- 法務SaaSは重く高価
- 差別化: **共同制作特化の軽量証跡化**

## $20達成シナリオ
- Lite($7)×3ユーザー = $21

## ユニットエコノミクス
- ARPU: $7
- 変動費/人: $1.2
- 粗利率: 82.8%
