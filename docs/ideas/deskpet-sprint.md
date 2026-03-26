# 🐾 DeskPet Sprint

## 概要
集中セッションを積むとデスクトップ上のペットが進化。タスク達成を「見た目報酬」に変える。

## 海外事例分析
海外のLofi/virtual pet生産性文脈、Finchの自己成長UXをデスクトップ常駐ツール化。

## ターゲット
在宅ワーカー、学生、ADHD傾向で視覚報酬が効くユーザー

## 料金
買い切り$6（基本） + テーマパック$3

## ユーザーフロー
タスク3件登録→25分集中→完了チェック→ペット進化＆部屋デコ解放

## デザインコンセプト
ピクセルアート×半透明ウィジェット。作業邪魔しない最小UI。SNS共有用の成長カードあり。

## アーキテクチャ
Tauriデスクトップアプリ + SQLite(local) + optional Supabase同期。AIはタスク分解時のみ。

## DB設計
- local_tasks, focus_sessions, pet_state, room_items, streak_logs

## コスト見積もり（AWS/無料サービス前提）
配布後のランニングほぼ$0。AI利用を月50回無料枠内に抑制。

## MVPスコープ
1匹のペット、2テーマ、ポモドーロ固定、日次レポート。

## マーケ計画
Product Hunt/Reddit r/productivity にGIF投稿、Boothでテーマ配布。

## 技術スタック
Tauri, React, Rust, SQLite, optional Supabase, OpenAI mini model

## リスク
初期体験が弱いと離脱。対策: 初回10分で必ず進化演出を出す。

## 競合分析
Forestはスマホ中心。DeskPetはPC作業導線に直結し「常時見える」強み。

## $20達成シナリオ
買い切り4本で$24達成。テーマ販売が上乗せ。

## ユニットエコノミクス
ARPU $6.8想定、粗利率97%以上。

## カテゴリ
パーソナルツール/生産性（集中で育つデスクペット）
