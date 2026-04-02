# ⏱️ FocusContracts

## 概要
「今日の作業契約（30/60/90分）」を自分と結び、完了時に**自動で成果カード化**するパーソナル生産性ツール。海外のcommitment device文化を、見せたくなるUIで再設計。

## 海外事例分析
- Beeminder/StickKのコミットメント設計
- Sunsama/Amieの日次計画体験
- Finchの“継続を可視化する感情設計”

## ターゲット
- ひとり開発者
- 副業クリエイター
- ADHD傾向で着手障壁が高い層

## 料金
- Free: 1日2契約
- Plus: $6/月（無制限・履歴分析）
- Lifetime: $19（買い切り）

## ユーザーフロー
1. タスクを1行入力
2. 契約時間を選ぶ
3. カウント中は通知を抑制
4. 完了時に成果スクショ+要約で成果カード生成
5. X/Discordへ共有（任意）

## デザインコンセプト
- 「Minimal Contract UI」
- 署名体験っぽい開始ボタン、達成時はレシート風カード

## アーキテクチャ
- Electron/Tauriデスクトップ + Web同期
- API: Hono on Cloudflare Workers
- DB: D1/Supabase
- Optional AI: 成果要約のみ

## DB設計
- users(id, email, plan)
- contracts(id, user_id, task, duration_min, status, started_at, ended_at)
- artifacts(id, contract_id, summary, image_url)
- streaks(id, user_id, date, contracts_done)

## コスト見積もり（月）
- Cloudflare: $5
- DB: $3
- AI要約: $2
- 合計: **$10以下**

## MVPスコープ
- 契約開始/終了
- 実績カード
- 連続達成カレンダー
- Stripe課金

## マーケ計画
- “作業ログ映え”を軸にXでUGC誘発
- Product Huntで「anti-procrastination minimal tool」として公開

## 技術スタック
Tauri, React, Hono, Cloudflare D1/R2, Stripe

## リスク
- 習慣化失敗 → 7日再起動シナリオ導線
- 類似タイマーとの差別化不足 → 成果カード共有を主価値に

## 競合分析
- 既存ポモドーロは“記録”止まり
- 本案は**契約体験+成果カード**で心理的完了を強化

## $20達成シナリオ
- Plus $6 × 4ユーザー = $24

## ユニットエコノミクス
- ARPPU: $6.2
- 変動費/有料ユーザー: $0.8
- 粗利率: 約87%
