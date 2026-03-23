# 📎 ClipClaim JP

## 概要
海外で伸びたショート動画/広告をURLで入れると、**日本市場向けの訴求テンプレ（見出し・導入5秒・CTA）**を3パターン出すマイクロSaaS。

## 海外事例分析
- **Foreplay / Motion**: 広告クリエイティブ収集の需要
- **GummySearch**: 海外コミュニティの課題抽出
- **Typefully/Taplio**: 投稿テンプレ化ニーズ

## ターゲット
- 1人マーケ担当
- 小規模D2C運営

## 料金
- Starter: $10/月（60解析）
- Pro: $18/月（200解析）

## ユーザーフロー
1. URL貼り付け
2. フレーム抽出＋字幕OCR
3. 日本語訴求3案生成
4. LP/動画台本として書き出し

## デザインコンセプト
「**Swipe Lab**」: 左右スワイプで訴求比較。

## アーキテクチャ
- API: FastAPI
- Worker: ffmpeg + whisper
- LLM: OpenAI mini
- DB: PostgreSQL

## DB設計
- users(id, plan, quota)
- clips(id, user_id, source_url, transcript)
- angles(id, clip_id, headline, hook, cta, score)

## コスト見積もり（月）
- Infra: $5
- AI/OCR: $4
- 合計: **$9**

## MVPスコープ
- URL解析
- 訴求3案
- CSV/Notion export

## マーケ計画
- 海外広告の和訳スレを毎日投稿
- D2C運営向け無料5本解析

## 技術スタック
FastAPI, PostgreSQL, ffmpeg, Whisper, OpenAI mini

## リスク
- 動画取得失敗 → 手動アップロード代替
- 著作権懸念 → 入力URLの利用規約同意

## 競合分析
- Foreplay: 収集強いが日本語ローカライズ弱い
- 生成AI一般ツール: 広告文脈テンプレ不足

## $20達成シナリオ
- Starter 2人 + Pro 1人 = $38

## ユニットエコノミクス
- ARPU: $12.7
- 変動費: $1.8
- 粗利率: **85%**
