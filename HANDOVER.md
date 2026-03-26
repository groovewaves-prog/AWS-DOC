# NLB導入プロジェクト 引継ぎ書

## 最終更新: 2026-03-26

## トランスクリプト
過去の全会話履歴は以下に保存されている:
- `/mnt/transcripts/2026-03-26-07-54-40-aws-nlb-ses-failover-project.txt`
- `/mnt/transcripts/2026-03-26-09-25-35-aws-nlb-ses-failover-project.txt`

---

## プロジェクト概要
オンプレミスZabbixサーバーからAWS SES経由でメール送信する際の経路問題を、NLB導入で解決するプロジェクト。ZabbixがSESのDNSを引くとVPCエンドポイントのENI IPが返るが、ZabbixはそのIPへの経路を知らずタイムアウト。NLBで固定IP(VIP)に集約し、バックエンドの切り替えはNLBが自動で行う。

### GitHubリポジトリ
https://github.com/groovewaves-prog/AWS-DOC (パブリック)

### 環境情報
- **本番VPC**: managed-smail-prod-vpc (10.133.128.0/24) AZ-a + AZ-c
- **検証VPC**: managed-smail-test-vpc (10.133.130.0/24)

### IPアドレス対応表

| リソース | 検証環境 | 本番環境 | AZ |
|:---|:---|:---|:---|
| NLB ノード | 10.133.130.50 | 10.133.128.50 | AZ-a |
| NLB ノード | 10.133.130.69 | 10.133.128.69 | AZ-c |
| VPCE ENI | 10.133.130.57 | 10.133.128.58 | AZ-a |
| VPCE ENI | 10.133.130.102 | 10.133.128.90 | AZ-c |

---

## 重要な技術的判明事項

### 1. NLBアクセスログの制限
NLBアクセスログは**TLSリスナーの場合のみ**記録される。本システムのリスナーはTCP:587（非TLS）のため、NLBアクセスログは生成されない。代替としてVPC Flow Logsを使用。

### 2. ヘルスチェック通信の除外
VPC Flow LogsにはNLBヘルスチェック通信(**268bytes**)が記録されるが、CloudWatch ProcessedBytesにはカウントされない。Athenaクエリに `AND bytes > 500` フィルタを追加してヘルスチェックを除外。

### 3. プレフィックスリスト
- **本番**: pl-01128722a023b9361（46エントリ、最大50件、**残り4件**）
- **検証**: managed-smail-test-prefix-list-mail-server（1エントリ、最大10件）

### 4. SGプレフィックスリスト欠落問題（解決済み）
本番NLBのSGにプレフィックスリスト(Zabbixセグメント)のインバウンドルールが欠落していた → ルール追加済み。

---

## Athena保存済みクエリ（最終版7件）

| No. | クエリ名 | 用途 | 技術的特徴 |
|:---|:---|:---|:---|
| 01 | NLB通信ログ確認 | 生ログ直近50件 | bytes > 500 |
| **02** | **★AZ別トラフィック分単位** | **メインクエリ。AZ横並び表示** | **sequence + LEFT JOIN + CAST + UTC→JST(+9h)** |
| 03 | 接続元別アクセス一覧 | 個社Zabbix確認・管理簿連携 | GROUP BY srcaddr |
| 04 | 特定日NLB通信ログ | 障害調査（日付指定） | date_format フィルタ |
| 05 | 拒否通信の確認 | セキュリティ調査 | action = 'REJECT'（HC除外不要） |
| 06 | 管理簿用CSV出力 | Excel管理簿連携 | CAST(... AS INTEGER/VARCHAR) |
| **07** | **ターゲット接続状況** | **●△✕ヘルス表示** | **WITH targets + LEFT JOIN（常時2行表示）** |

### VPC_FL_02の修正経緯（重要）
1. 初版: 単純GROUP BY → メール送信なし時に「結果なし」
2. sequence + LEFT JOIN で0バイト表示対応
3. `timestamp with time zone`型エラー → `CAST(... AS timestamp)` 追加
4. time_series(UTC) と mail_data(JST) のJOIN不一致 → `ts + interval '9' hour` 追加

### VPC_FL_07の修正経緯
- 初版: GROUP BY → メール送信なし時にAZ行が消える
- WITH targets + LEFT JOIN で常時2行表示対応

---

## CloudWatchダッシュボード
AZ-a/AZ-cのProcessedBytesの**2本線グラフ**で設定済み。
- メトリクス: NetworkELB > Per-AZ Metrics > ProcessedBytes
- AvailabilityZone: ap-northeast-1a, ap-northeast-1c
- 期間: 1分、統計: Sum

---

## 作成済みドキュメント一覧

| ファイル名 | 内容 |
|:---|:---|
| 【検証環境】AWS_NLB経由_SESメール送信テスト手順書_v4.xlsx | テスト手順書 |
| NLB導入_Zabbix経路設定_作業手順書_v5.xlsx | ★検証・本番統合版。SG/PL確認ステップ追加 |
| NLB_通信フロー図_v2.pptx | 通信フロー図 |
| fot_test_v8.sh | FOTスクリプト |
| 02.基本設計_NLB導入設計.xlsx / .pdf | 基本設計書 |
| 03.詳細設計_01.パラメータシート_NLB.xlsx / .pdf | 詳細設計書 |
| NLB導入_既存設計書_修正加筆一覧.xlsx | 既存設計書への修正箇所 |
| NLBセキュリティグループ_確認報告書.xlsx | SG確認報告書 |
| メールリレー_利用サーバ管理簿_v2.xlsx | 利用サーバ管理簿（Athena CSV連携対応） |
| 新メールリレーシステム_移行設定変更のお願い_v3.pptx | 移行案内資料 |
| 02.基本設計_13.バウンスレポート設計.xlsx | バウンスレポート設計書（v1.1 3段階対応） |
| bounce-report-lambda.zip | Lambda関数一式 |
| lambda_function.py | Lambda関数コード（571行、14関数、3段階対応） |
| deploy.sh | 自動デプロイスクリプト（336行、5ステップ） |
| README_bounce_report.md | README |
| バウンスレポートLambda_デプロイ運用手順書.xlsx | デプロイ・運用手順書（v1.1） |
| プレフィックスリスト運用ドキュメント.xlsx | 6シート構成（A-1～D-1 + Athenaクエリ PL_01-03 + 変更履歴） |
| Athenaクエリ設定_CloudWatchダッシュボード登録_手順書.xlsx | ★今回作成。経緯含む6シート構成 |

---

## COMPLETED
- [x] GitHubリポジトリの設計書分析
- [x] テスト手順書作成(Excel) v1～v4
- [x] 通信フロー図作成(PowerPoint)
- [x] NLB構築手順書 v1～v5（v5で検証・本番統合）
- [x] FOTスクリプト v1～v8
- [x] 基本設計書・詳細設計書 PDF→Excel変換
- [x] 既存設計書修正・加筆一覧作成
- [x] 本番環境NLB構築・SGルール追加
- [x] NLBアクセスログ有効化試行 → TLS非対応のため断念
- [x] NLBセキュリティグループ確認報告書作成
- [x] VPC Flow Logs Athenaテーブル作成（検証・本番）
- [x] Athena保存済みクエリ整理（12件→7件に統合、ヘルスチェック除外、0バイト表示対応）
- [x] 利用サーバ管理簿テンプレート作成
- [x] PowerPoint資料修正版v3作成
- [x] 検証環境SG確認・修正スクリプト作成
- [x] バウンスレポート設計書作成（v1.1 3段階対応）
- [x] Lambda関数コード・デプロイスクリプト・README作成
- [x] バウンスレポートLambda デプロイ・運用手順書作成
- [x] NLB構築手順書v5（検証・本番統合版）作成
- [x] プレフィックスリスト運用ドキュメント作成（PL_01-03含む）
- [x] **Athenaクエリ設定手順書・CloudWatchダッシュボード登録手順書の作成（経緯含む）**

## PENDING
- [ ] 検証・本番のNLBアクセスログ関連リソースのクリーンアップ
- [ ] 検証環境FOTスクリプトv8の実機実行
- [ ] 本番環境でのオンプレ側設定（hosts、スタティックルート）
- [ ] 本番環境での個社Zabbix(NAT越し)からの接続テスト
- [ ] テスト完了後のAZ-cリソース削除判断
- [ ] テスト用SMTP IAMユーザーの削除判断
- [ ] 既存設計書への修正・加筆の実施
- [ ] DR手順書へのNLB作成手順追加
- [ ] スライド4（構成図）の手動修正（NLBアイコン追加）
- [ ] 利用サーバ管理簿の初回データ投入
- [ ] 検証環境SGプレフィックスリスト欠落の確認・対応実施
- [ ] バウンスレポートLambdaの検証環境デプロイ・テスト
- [ ] バウンスレポートLambdaの本番環境デプロイ

---

## バウンスレポートLambda

### 3段階レベル対応（v1.1）
- **レベル1【正常】**: バウンス0件 → 対応不要
- **レベル2【日常】**: バウンスあり、Complaint/Suppressed 0件 → NoEmail/Transientのみ
- **レベル3【要対応】**: Complaint/Suppressed 1件以上 → 即時対応必要

### 処理フロー
EventBridge(毎日AM8:00 JST) → Lambda → S3読み込み → 分類集計 → CSV生成・S3アップロード → 署名付きURL生成(24h) → メール本文組立(3段階) → SES送信

- VPC外配置（NAT Gateway不要）
- Python 3.12、256MB、300秒タイムアウト
- コードは検証・本番共通。deploy.shの引数(test/prod)で環境変数を切り替え
