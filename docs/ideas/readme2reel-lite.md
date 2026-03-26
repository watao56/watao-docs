# 📼 Readme2Reel Lite

## 概要
README URLを入れると、機能要約・画面キャプチャ・字幕入りデモ動画を自動生成。

## 海外事例分析
海外のLaunch/Twitterビルドインパブリック文化、Arcade/ScreenStudioの需要を自動化で吸収。

## ターゲット
個人開発者、OSSメンテナ、受託エンジニアの実績可視化

## 料金
Starter $8/月（10本） / Solo $15/月（30本）

## ユーザーフロー
GitHub URL入力→README解析→デモ構成案確認→自動収録（Playwright）→動画DL

## デザインコンセプト
ターミナル風タイムライン＋シネマ字幕。技術者が「見せたくなる」黒基調。

## アーキテクチャ
Next.js API + Playwright recorder + Remotion render + S3。ジョブはAWS Lambda。

## DB設計
- users, repos, render_jobs, scripts, exports, usage_meter

## コスト見積もり（AWS/無料サービス前提）
固定$3〜10、1本あたり$0.04〜0.12（レンダリング時間依存）。

## MVPスコープ
公開リポジトリ限定、言語は英日、テンプレ2種。

## マーケ計画
GitHub Actionsテンプレ無料配布、Xで「README貼るだけ動画化」デモ連投。

## 技術スタック
Next.js, Playwright, Remotion, AWS Lambda, S3, DynamoDB/Supabase

## リスク
README品質依存。対策: 生成前に不足項目チェックと自動補完質問。

## 競合分析
ScreenStudioは手動録画。Readme2Reelはノーオペで量産可能。

## $20達成シナリオ
Starter 3人で$24達成。

## ユニットエコノミクス
ARPU $8.5, 粗利率80%前後。

## カテゴリ
マイクロSaaS（GitHub README→30秒デモ動画）
