# 📦 AppTeardown JP Nano

## 概要
海外で伸びる新興アプリを1日3件だけ選び、**価格・導線・オンボーディングを日本語1枚カード化**して配信するマイクロSaaS。

## 海外事例分析
- **Exploding Topics / Sensor Tower分析文化**
- **Lenny's Newsletter流の分解コンテンツ**
- **App growth teardown動画文化（YouTube/TikTok）**
- 日本語で“実装に使える粒度”の定期分解は供給不足。

## ターゲット
- 小規模アプリ開発者
- デザイナー/PM
- マーケ担当の副業層

## 料金
- Weekly: $5/月
- Pro: $9/月（アーカイブ全文 + CSV）

## ユーザーフロー
1. 毎日カード3枚が届く
2. 気になったカードを保存
3. 自社向け実装TODOへ変換

## デザインコンセプト
- Notionカード×雑誌レイアウト
- 1カード1示唆（価格/UX/コピー）

## アーキテクチャ
- クローリングは手動キュレーション前提（自動化最小）
- 生成補助: LLMで要約
- 配信: Beehiiv or Resend

## DB設計
- apps(id, name, url, category)
- teardowns(id, app_id, jp_summary, pricing_note, ux_note)
- subscribers(id, email, plan, status)
- deliveries(id, subscriber_id, teardown_id, sent_at)

## コスト見積もり（月）
- メール配信: $0〜$9
- LLM要約: $2〜$6
- ホスティング: $0
- 合計: **$2〜$15**

## MVPスコープ
- 週3配信
- 検索可能アーカイブ
- Stripe課金

## マーケ計画
- Xで無料カードを毎日1枚公開
- Indie Hackers/国内開発コミュニティで検証募集

## 技術スタック
Next.js, Supabase, Stripe, Resend, OpenAI mini

## リスク
- 情報鮮度維持が労働集約
  - 対策: 対象カテゴリを「B2B SaaS」「クリエイターアプリ」に限定

## 競合分析
- 英語ニュースレターは多いが、日本語実装カードは希少
- Generic要約botとの差は「実装に落とすテンプレ」

## $20達成シナリオ
- Weekly 4人で達成
- またはPro 3人で達成

## ユニットエコノミクス
- ARPU $6.5想定
- 原価 $0.8/人
- 粗利率 **87%**
