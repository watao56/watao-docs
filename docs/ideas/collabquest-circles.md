# 🫱🏻‍🫲🏽 CollabQuest Circles

## 概要
毎週、近いスキル帯の3人を自動で組み、**48時間だけの超短期コラボ**を回すコミュニティSaaS。完成物は自動でショーケース化され、次回のマッチ精度が上がる。

## 海外事例分析
- ADPList: メンタリングは強いが共同制作は弱い
- Geneva: コミュニティ運営は強いが成果物導線が弱い
- Lunchclub: マッチング体験は強いがクリエイター特化ではない

## ターゲット
- 駆け出しデザイナー/動画編集者/AIクリエイター
- 個人で伸び悩む副業層

## 料金
- Free: 月2回参加
- Member: $5/月（無制限参加＋優先マッチ）
- Host: $15/月（主催機能）

## ユーザーフロー
1. プロフ登録（得意/欲しいスキル）
2. 毎週金曜に3人自動編成
3. Discord連携部屋で48h制作
4. 成果提出→ランキング表示

## デザインコンセプト
「Arcade Guild」：バッジ、進捗リング、ネオンUIで参加自体を楽しく。

## アーキテクチャ
- Next.js + Supabase Realtime
- Discord Bot for room auto-create
- OpenAI embeddingで相性スコア計算

## DB設計
- members(id, skills, goals, score)
- circles(id, week_key, status)
- circle_members(circle_id, member_id, role)
- submissions(id, circle_id, url, votes)

## コスト見積もり（月）
- Supabase: $0〜25（初期はFree）
- Bot hosting: $5
- AI matching: $3
- 合計: 約$8

## MVPスコープ
- 週1回の自動マッチ
- Discord部屋自動生成
- 成果物ギャラリー

## マーケ計画
- 「週末48hコラボ」ハッシュタグでUGC誘発
- Discordコミュニティとの提携
- 受賞作をショート動画で再配布

## 技術スタック
Next.js / Supabase / Discord API / OpenAI Embeddings

## リスク
- 参加率低下 → 連続参加バッジ設計
- 荒らし → 招待制＋通報即BAN

## 競合分析
一般コミュニティツールより“制作完了率”をKPIに据える点が差別化。

## $20達成シナリオ
Member($5)×5人 = $25/月

## ユニットエコノミクス
- ARPU: $5.6
- 変動費/人: $0.4
- 粗利: $5.2（92.8%）
