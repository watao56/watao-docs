# 🚀 WaitKit

## 概要
🚀 WaitKitは、Indie Hacker向けマイクロSaaS（待機リスト拡散）に特化した月$20目標のマイクロプロダクト。**AIエージェントだけで2週間以内にMVP実装**できるスコープに限定し、初期コストを最小化する。

## 海外事例分析
- 参照トレンド: Prefinery / Viral Loops / GetWaitlist の海外実績
- 示唆: 日本市場ではローカライズ（日本語UI・国内決済・和文最適化）で差別化余地が大きい。

## ターゲット
- 個人開発者・小規模SaaS運営者

## 料金
- Starter $9 / Growth $19

## ユーザーフロー
1) LPに1行スクリプト埋め込み → 2) 登録時に紹介URL発行 → 3) 順位/特典を自動表示 → 4) CSV/メール配信へ連携

## デザインコンセプト
レトロ端末風ランキングUI。紹介したくなる可視化。

## アーキテクチャ
Cloudflare Workers + D1 + R2 + 軽量管理画面(Next.js)

## DB設計（最小）
- projects, waitlist_users, referrals, rewards, email_events

## コスト見積もり（月次）
- 15有料ユーザー時: 約$1.9/月（Workers無料枠中心）
- AWS/無料枠超過時はCloudflare/Vercel無料枠優先で吸収

## MVPスコープ（2週間）
- 埋め込みwidget、紹介URL、順位表示、CSV export、簡易メール
- 認証、決済、利用制限、簡易分析（Mixpanel代替でeventsテーブル）

## マーケ計画（最初の30日）
- Xで「1時間で導入できるWaitlist」を連投。個人開発コミュニティに無料枠提供。
- LPは1ページ、CTAは1つに絞る

## 技術スタック
- Cloudflare Workers/D1, Next.js, Resend, Stripe

## リスク
- 不正紹介→IP+端末指紋の軽量検知。到達率→SPF/DKIMガイド

## 競合分析
- Viral Loopsは高機能高価格。WaitKitは「小規模ローンチ専用」で低価格

## $20達成シナリオ
- Starter $9 ×3人 = $27 MRR
- 目標: 30日以内に達成可能な「少人数有料化」設計

## ユニットエコノミクス
- ARPU $10.8, 変動費/人 $0.11, 粗利 99.0%
- LTV/CACは初期フェーズでLTV>3x CACを最低ライン

## 実装指示（AIエージェント向け）
1. DB migration作成
2. 認証/課金/利用制限
3. コア機能1本を先に完成
4. ログ計測とエラーハンドリング
5. LP公開→初回ユーザー5人獲得
