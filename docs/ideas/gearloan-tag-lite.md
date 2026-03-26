# 🏷️ GearLoan Tag Lite

## 概要
カメラ・マイクなど機材貸出時に、QRタグで受け渡し記録と返却期限を残す超小型SaaS。

## 海外事例分析
海外のpeer-to-peer gear rental運用（ShareGrid等）を、個人間貸し借りの記録用途に縮小。

## ターゲット
動画クリエイター、学生サークル、小規模制作チーム

## 料金
Free: 3台まで / Lite: $5/月（30台）

## ユーザーフロー
機材登録→QRタグ発行→借り手がスマホで受領→返却時スキャン→履歴保存

## デザインコンセプト
工業ラベル風UI。黄色/黒の注意色で期限切れが一目でわかる。

## アーキテクチャ
Supabase + PWA。通知はDiscord webhook/LINE Notify代替（メール）。

## DB設計
- items, qr_tokens, loans, return_logs, reminder_rules, audit_notes

## コスト見積もり（AWS/無料サービス前提）
固定$0〜5、AI不要。通知費用ほぼゼロ。

## MVPスコープ
手動貸出、1日1回期限通知、CSV出力。

## マーケ計画
映像系Discord/大学サークル向け無料テンプレ配布。

## 技術スタック
Next.js PWA, Supabase, Resend, Cloudflare Turnstile

## リスク
入力が面倒だと使われない。対策: QRスキャンで3タップ完了。

## 競合分析
Notion台帳より運用が早く、貸出ステータスがリアルタイム。

## $20達成シナリオ
Lite 4チームで$20達成。

## ユニットエコノミクス
ARPU $5, 粗利率95%以上。

## カテゴリ
ライト保険型（機材貸出トラブル防止）
