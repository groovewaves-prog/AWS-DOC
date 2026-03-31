# HANDOVER.md — バウンスレポートLambda 拡張版 v2.1

## リポジトリ
https://github.com/groovewaves-prog/AWS-DOC

## 納品物一覧

| ファイル | 行数 | 概要 |
|---|---|---|
| `lambda_function_extended.py` | 1,170 | Lambda関数（拡張版 v2.0）日次バウンスレポート + 異常検知 + 監視 |
| `lambda_email_size_monitor.py` | 212 | Lambda関数（新規）NLBログからメールサイズをリアルタイム監視 |
| `deploy_extended.sh` | 1,196 | デプロイスクリプト（Step 0〜10、NLBログ有効化、SES設定セット自動検出） |
| `test_extended.sh` | 712 | テストスクリプト（5シナリオ） |
| `rollback.sh` | 420+ | 切り戻しスクリプト（カスタム権限保持、メールサイズ監視Lambda削除対応） |
| `バウンスレポートLambda_デプロイ運用手順書.xlsx` | 8シート | デプロイ・運用・テスト・切り戻し手順 |
| `Lambda_CloudWatch_監視比較表.xlsx` | 3シート | 既存CWアラームとLambdaの監視比較 |
| `バウンスレポートLambda_拡張版_デプロイ報告書.pptx` | 12スライド | デプロイ報告書（重複分析含む） |
| `HANDOVER.md` | 本ファイル | 引き継ぎ資料 |

## アーキテクチャ

```
【既存メール送信フロー（変更なし）】
  オンプレ → NLB → VPCエンドポイント → SES → 宛先
                ↓                        ↓
           NLBアクセスログ         SES設定セット → Data Firehose → S3（バウンスログ）
                ↓
  【リアルタイム監視（新規）】
  S3 → Lambda(email-size-monitor) → CloudWatch メトリクス + SNS アラート
                                     ↓
  【日次監視（拡張版）】           ダッシュボード表示
  EventBridge(毎日AM8:00)
    → Lambda(bounce-report)
      → Phase 1: バウンスレポート（メール送信）
      → Phase 2: 異常検知（バースト/リジェクト/帯域/メールサイズ）
      → Phase 3: CloudWatch メトリクス + SNS アラート
```

## 環境情報

| 項目 | 検証環境 | 本番環境 |
|---|---|---|
| アカウントID | 114091642122 | 192706392021 |
| Lambda（日次） | managed-smail-test-lambda-bounce-report | managed-smail-prod-lambda-bounce-report |
| Lambda（サイズ） | managed-smail-test-lambda-email-size-monitor | managed-smail-prod-lambda-email-size-monitor |
| S3バウンスログ | managed-smail-test-s3-bucket-bounce-event | managed-smail-prod-s3-bucket-bounce-event |
| S3 NLBログ | managed-smail-test-s3-bucket-nlb-accesslog | managed-smail-prod-s3-bucket-nlb-accesslog |
| NLB | managed-smail-test-nlb | managed-smail-prod-nlb |
| SES設定セット | — | managed-smail-prod-ses-configset |
| ダッシュボード | SES-Bounce-Monitoring-Test | SES-Bounce-Monitoring-Prod |

## メール送信ポリシー

- バウンス0件 → メール送信しない（CloudWatchメトリクスで稼働確認）
- バウンス1件以上（Complaint/Suppressedなし）→ レベル2【日常】メール
- Complaint/Suppressed検出 → レベル3【要対応】メール
- 異常検知 → SNS経由でアラートメール（日次メールとは別経路）

## 異常検知（4項目）

| 項目 | 閾値 | データソース | 検知間隔 |
|---|---|---|---|
| バースト | 過去7日平均の3倍 | SES Send（24時間） | 1日1回 |
| SESリジェクト率 | 2%以上 | SES Reject（1時間） | 1日1回 |
| ネットワーク帯域 | 85%以上 | NLB ProcessedBytes（5分） | 1日1回 |
| メールサイズ（日次平均） | 450KB超 | NLB bytes ÷ SES送信数（24時間） | 1日1回 |
| メールサイズ（リアルタイム） | 450KB超 | NLBアクセスログ received_bytes | 約5分 |

## 保持期間（すべて90日）

- CloudWatch Logs（Lambda実行ログ）
- バウンスCSVレポート（S3 `reports/daily/`）
- NLBアクセスログ（S3 `nlb-logs/`）

## デプロイ手順

```bash
# CloudShellで5ファイルをアップロード後
bash deploy_extended.sh test   # 検証環境
bash deploy_extended.sh prod   # 本番環境
```

## 切り戻し手順

```bash
bash rollback.sh test   # 検証環境
bash rollback.sh prod   # 本番環境
# オプション: --keep-dashboard --keep-sns --keep-alarms --keep-nlb-logs
```

## PENDING（未完了）

- 本番IAMポリシーにSES設定セットARN追加（deploy再実行で自動適用）
- 本番SNS購読確認メールのConfirm subscription
- test_extended.sh prod 再実行でテスト2,3,4のPASS確認
- 既存監視との重複整理の方針決定（案B' 推奨）
- 基本設計・詳細設計の改訂

## 変更履歴

| 版 | 日付 | 内容 |
|---|---|---|
| v1.0 | 2026/03 | 初版（Phase 1のみ） |
| v1.1 | 2026/03 | メールレベル3段階化 |
| v2.0 | 2026/03 | Phase 2/3追加（異常検知・CloudWatch・SNS） |
| v2.1 | 2026/03 | メールサイズリアルタイム監視追加、閾値450KB、OK通知停止、保持期間90日統一 |
