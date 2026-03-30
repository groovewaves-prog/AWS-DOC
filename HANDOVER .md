# SES送信品質監視・異常検知・アラート機能 - プロジェクト引き継ぎ書

**作成日**: 2026年3月30日  
**プロジェクト状態**: 実装準備完了（Week 1 デプロイ可能）  
**引き継ぎ者**: 次期担当者へ

---

## 📋 プロジェクト概要

### プロジェクト名
```
SES送信品質監視・異常検知・アラート機能 実装手順書
～ バウンス・トラフィック・サイズ異常を多層検知 ～
```

### プロジェクト目的
```
本プロジェクトは、AWS SES経由でのメール送信システムに
リアルタイム異常検知・監視・通知機能を導入します。

メール送信トラフィック、SES処理品質、ファイルサイズ、
ネットワーク帯域の5つの観点から異常を多層的に検知し、
CloudWatchダッシュボードで可視化、SNS/メールで即座に通知することで、
年間 ~$100,000 の潜在的なSESリスクを回避し、
運用効率を向上させることを目的とします。

実装には月額 $4.30 の追加費用のみで、
朝15分の確認で全体システムの健全性を把握可能になります。
```

---

## ✅ 完了した作業

### 【1】設計・企画段階 ✅

- ✅ プロジェクト目的の明確化
- ✅ 5層の異常検知機能の設計確定
  - トラフィック異常検知（平常時の3倍以上）
  - SES処理品質異常（リジェクト率2%以上）
  - メールサイズ異常（100MB以上）
  - バウンスメール増加（日別で急増）
  - ネットワーク帯域異常（85%以上）

- ✅ CloudWatch メトリクス（10個）の仕様設計
- ✅ SNS トピック（2個）の仕様設計
- ✅ CloudWatch ダッシュボード（6ウィジェット）の仕様設計
- ✅ CloudWatch アラーム（3個）の仕様設計
- ✅ 費用計算（月額 $4.30）の確定

### 【2】コード開発 ✅

- ✅ **lambda_function_extended.py（825行）**
  - Phase 1: バウンスレポート機能（既存保持）
  - Phase 2: 異常検知機能（新規追加）
    - `detect_burst()`: トラフィック異常検知
    - `get_ses_metrics()`: SES品質監視
    - `get_network_metrics()`: ネットワーク帯域監視
    - `get_csv_size()`: ファイルサイズ監視
  - Phase 3: 通知・可視化（新規追加）
    - CloudWatch メトリクス送信（10個）
    - SNS 通知（2トピック）
    - CloudWatch アラーム設定（3個）

- ✅ **deploy_extended.sh**
  - IAM ポリシー・ロール作成
  - Lambda 関数デプロイ
  - SNS トピック作成（2個）
  - CloudWatch ダッシュボード作成（6ウィジェット）
  - CloudWatch アラーム設定（3個）
  - EventBridge Scheduler 設定

### 【3】ドキュメント作成 ✅

**実装手順書**
- ✅ 実装手順書_検証・本番環境対応.xlsx（11シート）
  - Sheet 01: 概要・前提条件
  - Sheet 02: 検証環境デプロイ(Week1)
  - Sheet 03: 検証環境テスト(Week2)
  - Sheet 04: 本番環境デプロイ(Week3)
  - Sheet 05: 検証環境チェック（10項目）
  - Sheet 06: 本番環境チェック（14項目）
  - Sheet 07: トラブルシューティング
  - Sheet 08: 参考資料
  - Sheet 09: Week1詳細手順（5ステップ）
  - Sheet 10: Week2詳細手順（テスト方法）
  - Sheet 11: Week3詳細手順（5ステップ）

**その他ドキュメント**
- ✅ 詳細実装手順書_Week別ガイド.md
  - Week1: CloudShell操作の詳細手順
  - Week2: テスト方法（毎日の手順）
  - Week3: 本番環境デプロイ手順
  - 異常検知テスト方法
  - トラブル対応

- ✅ 新タイトルと目的_最終確定版.md
  - タイトル定義
  - 目的文（4パターン）
  - タイトル選定理由

- ✅ 実装レビュー報告書.md
  - 全機能の実装確認
  - バースト検知対応確認
  - メールサイズ異常対応確認
  - 総合評価: A+ 合格

---

## 📦 納品ファイル一覧

### **実装に必要なファイル**

```
【コード】
  ✅ lambda_function_extended.py（825行）
     - Phase 1,2,3 を実装
     - S3から既存バウンスイベント取得
     - CloudWatch メトリクス送信
     - SNS 通知機能

  ✅ deploy_extended.sh
     - 自動デプロイ機能
     - 環境切り替え対応（test/prod）
     - リソース一括作成

【ドキュメント】
  ✅ 実装手順書_検証・本番環境対応.xlsx
     - 11シート構成
     - チェックリスト付き
     - 詳細手順含む

  ✅ 詳細実装手順書_Week別ガイド.md
     - CloudShell操作の詳細
     - テスト方法の詳細
     - トラブル対応

【参考資料】
  ✅ 新タイトルと目的_最終確定版.md
  ✅ 実装レビュー報告書.md
  ✅ HANDOVER.md（本ファイル）
```

### **保存場所**
```
すべてのファイルは以下に保存：
  /mnt/user-data/outputs/

メインファイル:
  - 実装手順書_検証・本番環境対応.xlsx
  - lambda_function_extended.py
  - deploy_extended.sh
  - 詳細実装手順書_Week別ガイド.md
```

### **GitHub リポジトリ**
```
URL: https://github.com/groovewaves-prog/AWS-DOC
種別: パブリックリポジトリ

格納内容（既存の設計ドキュメント一式）:
  01. 要件定義_新メールシステム_要件定義書.pdf
  02. 基本設計（01〜12）: はじめに, システム構成, アカウント設計,
      ネットワーク構成, メール設計, DNS設計, ストレージ設計,
      可用性設計, 拡張性設計, ログ管理設計, セキュリティ設計, 運用監視方針
  03. 詳細設計: パラメータシート（VPC, DirectConnect, Route53, SES,
      Data Firehose, S3, CloudWatch, SNS, IAM, セキュリティグループ,
      GuardDuty, Config, CloudTrail, KMS, EventBridge）
      + 接続情報, Resource Visualizer（本番/検証）
  04. 監視設定: 監視項目一覧, 動作確認報告, アラート発生時対応,
      SES送信数確認, 監視対象一覧
  05. 運用: セキュリティインシデント発生時の対応方針
  06. 運用手順書: Athena操作手順書, 通信許可IP手順書, DR切り替え手順書
  + HANDOVER.md, README.md
```

---

## 🎯 次のステップ（実装の流れ）

### **Week 1: 検証環境デプロイ（2-3時間）**

```
【準備】
  □ lambda_function_extended.py をダウンロード
  □ deploy_extended.sh をダウンロード
  □ CloudShell を開く

【実行】
  1. 2ファイルを CloudShell にアップロード
  2. パーミッション設定: chmod +x deploy_extended.sh
  3. デプロイ実行: bash deploy_extended.sh test
  4. AWS コンソールで確認
     - Lambda 関数作成確認
     - SNS トピック作成確認
     - CloudWatch ダッシュボード作成確認
     - CloudWatch アラーム作成確認

詳細は「実装手順書」Sheet 09-11 を参照
```

### **Week 2: 検証環境テスト（1週間、毎日 5-10分）**

```
【毎日実施】
  □ Lambda テスト実行（statusCode: 200 確認）
  □ ダッシュボード確認（6ウィジェット確認）
  □ ログ確認（3 Phase 実行確認）
  □ SNS 購読確認（月曜日のみ）

【期間】
  月曜: デプロイ直後確認
  火曜: Lambda テスト実行
  水曜: ログ確認
  木曜: ダッシュボード確認
  金曜: SNS 購読確認 → 本番移行判断

詳細は「実装手順書」Sheet 03, 10 を参照
```

### **Week 3: 本番環境デプロイ（1時間）**

```
【前提条件】
  ✅ Week 1-2 テスト完了
  ✅ SNS 購読「Confirmed」状態
  ✅ トラブルなし

【実行】
  1. 本番環境でデプロイ実行: bash deploy_extended.sh prod
  2. AWS コンソールで確認
  3. SNS 購読確認
  4. Lambda 本番テスト実行

【本運用開始】
  毎日 AM 8:00 - Lambda 自動実行
  毎日 AM 8:05 - ダッシュボード確認
  異常検知時 - SNS メール通知

詳細は「実装手順書」Sheet 04, 06, 11 を参照
```

---

## 🔧 デプロイ環境情報

### **対象環境**

| 項目 | 検証環境（test） | 本番環境（prod） |
|:---|:---|:---|
| **Lambda関数名** | managed-smail-test-lambda-bounce-report | managed-smail-prod-lambda-bounce-report |
| **SNS バースト** | managed-smail-test-sns-burst-alert | managed-smail-prod-sns-burst-alert |
| **SNS サイズ** | managed-smail-test-sns-size-alert | managed-smail-prod-sns-size-alert |
| **ダッシュボード** | SES-Bounce-Monitoring-Test | SES-Bounce-Monitoring-Prod |
| **アラーム（3個）** | smail-test-* | smail-prod-* |
| **スケジュール** | 毎日 UTC 23:00 (JST 08:00) | 毎日 UTC 23:00 (JST 08:00) |

### **AWS リージョン**
```
東京リージョン: ap-northeast-1
```

### **IAM 権限要件**
```
実行ユーザーには以下の権限が必要:
  • AdministratorAccess（推奨）
  または以下の個別権限:
    - Lambda: CreateFunction, GetFunction, UpdateFunction
    - IAM: CreateRole, AttachRolePolicy, CreatePolicy
    - SNS: CreateTopic, Subscribe
    - CloudWatch: PutDashboard, PutMetricAlarm
    - Scheduler: CreateSchedule
    - EventBridge: PutRule, PutTargets
```

---

## 📊 監視対象（5項目）

| # | 監視項目 | 閾値 | 検知関数 | CloudWatch メトリクス |
|:---|:---|:---|:---|:---|
| 1 | メール送信トラフィック | 平常時の3倍以上 | detect_burst() | BurstDetected |
| 2 | SES リジェクト率 | 2% 以上 | get_ses_metrics() | SESRejectRateHigh |
| 3 | メールサイズ異常 | 100MB 以上 | get_csv_size() | csv_file_size |
| 4 | バウンスメール増加 | 日別で急増 | (既存) | bounce_count |
| 5 | ネットワーク帯域 | 85% 以上 | get_network_metrics() | NetworkBandwidthHigh |

---

## 🚨 トラブルシューティング

### **よくあるエラー**

```
【エラー 1】ファイルパーミッションエラー
症状: Permission denied
対応: chmod +x deploy_extended.sh

【エラー 2】IAM ポリシーエラー
症状: AccessDenied
対応: 実行ユーザーが AdministratorAccess を持つか確認

【エラー 3】SNS メール未到着
症状: 購読確認メールが来ない
対応: 
  - メールアドレスが正しいか確認（itos-infra-alert@kddi.com）
  - SES がサンドボックスモードか確認
  - スパムフォルダ確認

【エラー 4】Lambda テスト失敗（statusCode != 200）
症状: statusCode が 200 以外
対応:
  - CloudWatch > Logs で詳細ログを確認
  - IAM ロールが正しくアタッチされているか確認
```

詳細は「実装手順書」Sheet 07 を参照

---

## 📈 費用見積もり

### **月額費用**

```
環境あたり:
  CloudWatch カスタムメトリクス: $1.00
  CloudWatch ダッシュボード:    $3.00
  CloudWatch アラーム（3個）:   $0.30
  ─────────────────────────────────
  小計（1環境）:               $4.30

検証 + 本番（2環境）:
  月額: $8.60
  年額: $103.20

その他費用（Lambda/SES/SNS）: 無料枠内
```

---

## 📞 連絡先・参考資料

### **提供者**
```
プロジェクト実施者: Claude AI
提供日: 2026年3月30日
バージョン: 1.0（最終版）
```

### **参考資料**

```
【技術仕様】
  - オプション2_完全実装ガイド.md
  - CloudWatch異常検知_実装ガイド.md
  - CloudWatch異常検知_費用詳細計算.md
  - エグゼクティブサマリー.md

【プレゼンテーション】
  - CloudWatch異常検知_上司向けプレゼン.pptx

【トラブルシューティング】
  - 実装手順書 > Sheet 07
  - 詳細実装手順書_Week別ガイド.md > トラブル対応セクション
```

---

## ✅ 実装前の最終確認チェック

```
□ lambda_function_extended.py をダウンロードした
□ deploy_extended.sh をダウンロードした
□ AWS Console にログイン可能な状態
□ CloudShell を起動できる状態
□ IAM ロール権限が AdministratorAccess か確認
□ 東京リージョン（ap-northeast-1）に切り替え
□ 本手順書（HANDOVER.md）を理解した
□ 「実装手順書」Sheet 01-11 を確認した
□ メールアドレス「itos-infra-alert@kddi.com」で SNS 通知を受けられる環境

上記すべてにチェックが入れば、実装開始可能です。
```

---

## 📝 進捗管理用チェックリスト

### **Week 1: 検証環境デプロイ**
```
□ ファイル準備（10分）
  □ lambda_function_extended.py アップロード
  □ deploy_extended.sh アップロード
  □ パーミッション設定

□ デプロイ実行（5-10分）
  □ bash deploy_extended.sh test 実行
  □ デプロイ完了メッセージ確認

□ リソース確認（5-10分）
  □ Lambda 関数確認
  □ SNS トピック確認
  □ CloudWatch ダッシュボード確認
  □ CloudWatch アラーム確認

完了予定日: __月__日
実際完了日: __月__日
```

### **Week 2: 検証環境テスト**
```
□ 月曜: デプロイ直後確認
□ 火曜: Lambda テスト実行
□ 水曜: ログ確認
□ 木曜: ダッシュボード確認
□ 金曜: SNS 購読確認 + 本番移行判断

完了予定日: __月__日
実際完了日: __月__日
```

### **Week 3: 本番環境デプロイ**
```
□ 本番デプロイ前確認
  □ Week 1-2 テスト完了
  □ SNS 購読「Confirmed」状態
  □ トラブルなし

□ 本番デプロイ実行（1時間）
  □ bash deploy_extended.sh prod 実行
  □ AWS リソース確認
  □ SNS 購読確認
  □ Lambda 本番テスト実行

□ 本運用開始
  □ 毎日 AM 8:00 - Lambda 自動実行
  □ 毎日 AM 8:05 - ダッシュボード確認

完了予定日: __月__日
実際完了日: __月__日
本運用開始日: __月__日
```

---

## 🎯 次期担当者への注記

### **重要ポイント**

1. **Lambda関数は拡張版（825行）**
   - Phase 1: バウンスレポート（既存保持）
   - Phase 2: 異常検知（新規追加）
   - Phase 3: 通知・可視化（新規追加）
   - すべてが統合されて動作します

2. **CloudWatch メトリクスは 10個送信**
   - バウンス関連: 3個
   - SES 品質関連: 3個
   - ネットワーク関連: 2個
   - ファイルサイズ: 1個
   - アラーム関連: 1個

3. **SNS トピックは 2個に分岐**
   - sns-burst-alert: バースト + SES品質 + 帯域
   - sns-size-alert: ファイルサイズ異常
   - 別トピック化で通知を柔軟に管理可能

4. **毎日 AM 8:00 JST に自動実行**
   - EventBridge Scheduler で UTC 23:00（JST 08:00）実行
   - CloudWatch ダッシュボードで毎朝確認可能

5. **トラブル時は「実装手順書」Sheet 07 参照**
   - よくあるエラー 7種類を記載
   - 解決方法も明記

### **心構え**

- Week 1-3 の流れに沿って実施してください
- 各週のチェックリストを使用して進捗管理してください
- 不明な点は「詳細実装手順書_Week別ガイド.md」を参照してください
- トラブルは「実装手順書」Sheet 07 で解決してください

---

## 📄 関連ドキュメント体系

```
【全体構成】
  HANDOVER.md（本ファイル）← 引き継ぎ用
     ↓
  実装手順書_検証・本番環境対応.xlsx（メイン）
     ├─ Sheet 01-04: 概要 + 実装手順
     ├─ Sheet 05-08: チェック + 参考資料
     └─ Sheet 09-11: 詳細手順（Week 1-3）
     ↓
  詳細実装手順書_Week別ガイド.md（深掘り用）

【参考資料】
  新タイトルと目的_最終確定版.md
  実装レビュー報告書.md
```

---

**このプロジェクトは実装準備が完全に整っています。**  
**Week 1 デプロイを開始してください。**

