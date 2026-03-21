# 🛡️ MeetingConsent Vault Lite

## 概要
Zoom録画や教材再利用の同意を1クリック収集し、証跡を時系列保管。月\$20達成を最短で狙うため、初期は単機能MVPで開始する。

## 海外事例分析
- 参照: DocuSign / Notta consent workflows / Teachable policy ops
- 海外で伸びる要因: 体験の即時性・共有導線・テンプレ資産
- 日本向け差分: 日本語フォント/文脈最適化、LINE/X導線、価格を低単価に調整

## ターゲット
- 主対象: オンライン講師、コーチ、小規模スクール
- 初期獲得チャネル: X、Discord、知人コミュニティ

## 料金
- Lite $5/月 / Pro $12/月
- 返金ポリシー: 初月7日返金

## ユーザーフロー
1. LPから無料登録
2. 初回テンプレを選択
3. 1分以内に最初の成果物を体験
4. 共有/保存で継続利用
5. 使用量上限で有料転換

## デザインコンセプト
「金庫」モチーフのタイムラインUI。証跡PDFを即出力。

## アーキテクチャ
Next.js + Supabase + Cloudflare R2 + e-sign API(HelloSign free tier)

## DB設計
主要テーブル: users, sessions, attendees, consents, evidence_files, audit_logs

## コスト見積もり（月次）
低頻度電子署名で月$2〜$5。保存容量小でR2$1以内。

## MVPスコープ（2週間）
(1) 同意リンク発行 (2) 参加者署名 (3) 録画URL紐付け (4) 証跡PDF出力

## マーケ計画
コーチ向けコミュニティに「同意テンプレ無料配布」で流入。

## 技術スタック
Next.js, Supabase, PDFKit, Stripe, HelloSign API

## リスク
法的有効性差異→利用規約で法的助言ではない旨明記、国別テンプレ提供

## 競合分析
DocuSignは高価で汎用。MeetingConsentは配信講座ユースケースに特化。

## \$20達成シナリオ
Lite 4人で$20達成。事故時の損失回避訴求で継続率高い。

## ユニットエコノミクス
ARPU $5, 原価$0.4/人, 粗利92%
