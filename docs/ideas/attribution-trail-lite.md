# 🛡️ Attribution Trail Lite

## 概要
唯一の保険型枠。海外で増える「AI素材のライセンス/クレジット表記要件」を、投稿前チェックリストとして可視化。素材URLを貼ると、必要な表記・禁止用途を自動抽出する。

## 海外事例分析
- **Canva/Adobe Stock**: ライセンス条件の複雑化
- **Envato Elements**: 使用範囲の明文化
- **YouTube Content ID文脈**: 権利情報の未記載が実害に直結

## ターゲット
- 小規模クリエイティブチーム
- 個人YouTuber
- SNS運用代行

## 料金
- Free: 月20チェック
- Pro: $9/月（無制限）

## ユーザーフロー
1. 使用素材URLを登録
2. 表記要件を抽出
3. 投稿文テンプレにクレジット自動挿入
4. 公開前に最終チェック

## デザインコンセプト
- 「検品ボード」UI
- 赤黄緑の明確なリスク表示
- 1画面で公開可否が分かる

## アーキテクチャ
- Next.js
- ルール辞書 + LLM補助抽出
- Supabase

## DB設計
- users(id, email, plan)
- assets(id, user_id, source_url, source_name, license_text)
- checks(id, asset_id, risk_level, required_credit, prohibited_use)
- templates(id, user_id, channel, text)

## コスト見積もり（月）
- Hosting/DB: $3
- AI: $2
- 合計: $5

## MVPスコープ
- URL解析
- クレジット提案
- 公開前チェックリスト

## マーケ計画
- クリエイター向け「著作権事故回避」ショート解説
- 編集代行会社へB2Bミニ導入

## 技術スタック
- Next.js / Supabase / OpenAI mini

## リスク
- ライセンス文面の解釈差
- 法務助言との境界

## 競合分析
- 既存は契約管理が重厚
- Attribution Trail Liteは**投稿前の超軽量チェック**に特化

## $20達成シナリオ
- Pro 3人 = $27/月

## ユニットエコノミクス
- ARPU: $9
- 変動費: $0.9
- 粗利率: 90%
