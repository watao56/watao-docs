# 🧠 ContextCapsule

## 概要
「今日の作業文脈」を1クリックで保存し、翌日ワンクリックで再開できるパーソナル生産性ツール。タブ、ToDo、メモ、直近Git差分を**再開パック**として束ねる。

- カテゴリ: パーソナル生産性
- 目標: 月$20（$4プラン×5人）

## 海外事例分析
- Sunsama: 日次計画の需要が高い
- Loom/Async work: 文脈共有への支払い意欲がある
- Readwise/Reflect系: 「後で再開」の価値が定着

示唆: 日本では“開始コスト”の高さが継続課題。**再開時間短縮**を主価値にすると刺さる。

## ターゲット
- 複数案件を並行する個人開発者
- 副業ワーカー
- 1人目PM/デザイナー

## 料金
- Free: 1日1カプセル
- Plus: $4/月（無制限、タグ検索、7日履歴）
- Pro: $9/月（90日履歴、Git要約、共有リンク）

## ユーザーフロー
1. 終業時に「Pack」押下
2. ブラウザタブ・メモ・タスクを自動収集
3. 翌日「Resume」押下
4. 優先順つき再開チェックリスト表示
5. 必要ならチームへ共有

## デザインコンセプト
- 朝に優しい白基調
- 「再開まで何分短縮できたか」を可視化
- 操作はPack/Resumeの2ボタン中心

## アーキテクチャ
- Chrome Extension + Next.js Web
- API: Hono on Cloudflare Workers
- DB: Turso(SQLite)
- AI要約: OpenAI mini（任意機能）

## DB設計
- users(id, email, plan)
- capsules(id, user_id, title, created_at)
- capsule_items(id, capsule_id, item_type, payload_json)
- resume_logs(id, capsule_id, resumed_at, saved_minutes)

## コスト見積もり（月）
- Workers/Turso無料枠中心: $0〜$3
- AI要約利用分: $0〜$2
- 合計想定: $2〜$5

## MVPスコープ
- Chrome拡張でタブ保存/復元
- 手動メモ添付
- 次回再開チェックリスト
- Stripe決済

## マーケ計画
- 「朝の再開5分短縮」検証データ投稿
- 個人開発者向けニュースレター提携
- Product Hunt風LPで体験動画を前面化

## 技術スタック
TypeScript / Chrome Extension API / Cloudflare Workers / Turso / Stripe / OpenAI mini

## リスク
- 拡張機能権限への心理抵抗
- タブ復元失敗時の不信感
- 他タスク管理ツールとの重複

## 競合分析
- OneTab: 保存のみで再開導線が弱い
- タスク管理ツール: 文脈（タブ/差分）まで扱わない
- 本案: **再開体験に特化**

## $20達成シナリオ
- Plus $4 × 5人 = $20
- もしくは Pro $9 × 3人 = $27

## ユニットエコノミクス
- ARPU: $5.1
- 変動費: $0.5/user
- 粗利: $4.6/user（90%）
- NPS向上要因: 「朝の開始ストレス減」
