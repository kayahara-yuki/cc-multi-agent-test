# Implementation Plan

## Tasks

- [ ] 1. プロジェクト基盤セットアップ
- [ ] 1.1 (P) Python/Poetry環境とプロジェクト構造の構築
  - Python 3.10+環境をPoetryで初期化
  - ディレクトリ構成（src/evaluation/, tests/, logs/, reports/）を作成
  - 基本的な依存関係（PyYAML, Click）をpyproject.tomlに追加
  - pytestとmypy設定を追加
  - _Requirements: 1.1_

- [ ] 1.2 (P) MARBLE依存関係の設定
  - MARBLEフレームワークをPoetry依存関係として追加
  - MARBLEバージョン制約とコミットハッシュ取得ロジックを実装
  - MARBLE環境検証ユーティリティを作成
  - _Requirements: 1.1, 6.1_

- [ ] 2. 設定管理（ConfigLoader）
- [ ] 2.1 設定スキーマとデータクラスの実装
  - BenchmarkConfig, CoordinationProtocol, PlanningStrategyデータクラスを定義
  - ValidationResult, ValidationErrorモデルを実装
  - 協調プロトコル（star/tree/graph/chain）と計画戦略（vanilla/cot/group_discussion/cognitive）の列挙型を定義
  - _Requirements: 2.3, 2.4_

- [ ] 2.2 設定ファイル読み込みとCLI引数処理
  - YAML設定ファイルのパースロジックを実装
  - CLI引数によるオーバーライド機能を追加
  - 乱数シード設定の読み込みをサポート
  - _Requirements: 2.3, 2.4, 6.3, 6.5_

- [ ] 2.3 環境情報収集と検証
  - MARBLEバージョン・コミットハッシュの収集機能を実装
  - cc-mirror設定パラメータの検出・記録機能を実装
  - 統合設定の整合性検証ロジックを実装
  - 不整合検出時のエラーメッセージ生成
  - _Requirements: 1.2, 1.5, 6.1, 6.2_

- [ ] 3. ツール呼び出し記録（ToolLogger）
- [ ] 3.1 (P) ToolCallRecordデータ構造の実装
  - ToolCallRecordデータクラス（call_id, agent_id, tool_name, parameters, response, timestamps）を定義
  - ツール名（TaskCreate/TaskGet/TaskUpdate/TaskList）の列挙型を定義
  - 呼び出しID生成ロジック（UUID）を実装
  - _Requirements: 4.1, 4.2, 4.3_

- [ ] 3.2 (P) ログ記録サービスの実装
  - log_call_start/log_call_end APIを実装
  - 呼び出し開始・終了タイミングの記録機能を実装
  - 引数とレスポンスの完全記録機能を実装
  - エラー情報の記録機能を追加
  - _Requirements: 4.1, 4.2, 4.3_

- [ ] 3.3 ログ永続化
  - JSONL形式でのログ書き込み機能を実装
  - ログファイルパス管理（logs/{run_id}/tool_calls.jsonl）を実装
  - get_records APIを実装
  - _Requirements: 2.5_

- [ ] 4. ツール使用分析（ToolAnalyzer）
- [ ] 4.1 タスク依存関係統計の分析
  - blocks/blockedByの使用回数集計を実装
  - タスクあたりの平均依存関係数の計算を実装
  - 最大依存深度の計算を実装
  - DependencyStatsデータクラスへの集計結果格納
  - _Requirements: 4.4_

- [ ] 4.2 エージェント間タスク分布の分析
  - エージェントごとのタスク作成・更新・完了数を集計
  - TaskDistributionデータクラスへの結果格納
  - 分布データの可視化用フォーマット生成
  - _Requirements: 4.5_

- [ ] 5. MARBLEメトリクス収集（MetricsCollector）
- [ ] 5.1 (P) MARBLEメトリクス読み取り
  - MARBLE出力ファイル（JSON）のパースを実装
  - Task Score (TS)、Coordination Score (CS)の抽出を実装
  - Milestone KPIの抽出を実装
  - Communication Score、Planning Scoreの個別抽出を実装
  - _Requirements: 3.1, 3.2, 3.3, 3.4_

- [ ] 5.2 (P) メトリクスフォーマット保持と検証
  - MarbleMetricsデータクラスでraw_outputを保持
  - MARBLEの元フォーマットを変換せずそのまま保存
  - フォーマット検証ロジックと欠損値検出を実装
  - _Requirements: 3.5_

- [ ] 6. MARBLE統合アダプター（MarbleAdapter）
- [ ] 6.1 ツールマッピング基盤の実装
  - Claude CodeタスクツールとMARBLEエージェントアクション間のマッピングを定義
  - AgentAction, AgentActionResultデータクラスを実装
  - translate_to_claude_tool変換ロジックを実装
  - translate_from_claude_tool変換ロジックを実装
  - _Requirements: 1.3_

- [ ] 6.2 協調プロトコル別blocks/blockedBy構造生成
  - Star構造（1:N プランナー→アクター）のblocks生成を実装
  - Chain構造（線形依存）のblocks生成を実装
  - Tree構造（階層的親子関係）のblocks/blockedBy生成を実装
  - Graph構造（多対多接続）のblocks/blockedBy生成を実装
  - build_dependency_structure APIを実装
  - _Requirements: 1.4, 2.3_

- [ ] 6.3 MARBLEベンチマーク初期化と実行
  - MARBLEフレームワークとの接続初期化を実装
  - シナリオ（Coding/Database）の選択・起動を実装
  - 協調プロトコルと計画戦略の設定伝達を実装
  - 乱数シードの設定を実装
  - initialize/run_benchmark APIを実装
  - _Requirements: 2.1, 2.2, 2.4, 6.5_

- [ ] 6.4 cc-mirrorプロセス管理とツール呼び出し中継
  - cc-mirror子プロセスの起動・管理を実装
  - MARBLEからのエージェントアクション受信時にClaude Codeツールを呼び出す
  - ツール呼び出し前後でToolLoggerに記録
  - レスポンスをMARBLEフォーマットに変換して返却
  - _Requirements: 1.4_

- [ ] 7. 実行オーケストレーター（Orchestrator）
- [ ] 7.1 ベンチマーク実行フロー制御
  - 実行ID（run_id）生成とディレクトリ作成を実装
  - 設定検証 → アダプター初期化 → ベンチマーク実行のフロー制御を実装
  - 実行開始・完了時刻の記録を実装
  - RunResultデータクラスへの結果格納
  - _Requirements: 2.1, 2.2_

- [ ] 7.2 再実行機能とエラーハンドリング
  - 過去の設定ファイルを使用した再実行機能を実装
  - 実行状態取得APIを実装
  - エラー発生時のクリーンアップ処理を実装
  - 部分結果の保存機能を実装
  - _Requirements: 6.4_

- [ ] 8. レポート生成（ReportGenerator）
- [ ] 8.1 Markdownレポートテンプレートと生成
  - Jinja2テンプレートを使用したMarkdownレンダリングを実装
  - MARBLEメトリクスサマリーセクション（TS, CS, KPI, Communication, Planning）を生成
  - Claude Code固有分析セクション（ツール使用統計、依存関係統計）を生成
  - _Requirements: 5.1, 5.2, 5.3_

- [ ] 8.2 評価環境情報と設定記録
  - 評価環境情報（MARBLEバージョン、cc-mirror設定、実行日時）セクションを生成
  - 使用した協調プロトコルと計画戦略を明記
  - generate APIを実装し、レポートファイル出力
  - _Requirements: 5.4, 5.5_

- [ ] 8.3 タスク分布可視化
  - エージェント間タスク割り当て分布をMermaid pie chartで可視化
  - 可視化セクションをレポートに統合
  - _Requirements: 4.5_

- [ ] 9. CLIインターフェース
- [ ] 9.1 Clickベースのコマンド体系構築
  - メインCLIエントリーポイントを実装
  - setupサブコマンド（統合セットアップ）を実装
  - runサブコマンド（ベンチマーク実行、--scenario, --protocol, --strategy オプション）を実装
  - reportサブコマンド（レポート生成、--run-id オプション）を実装
  - _Requirements: 1.1, 2.1, 2.2, 5.1_

- [ ] 9.2 MARBLE標準ログ出力保存
  - ベンチマーク実行中のMARBLE標準出力キャプチャを実装
  - ログファイルへの保存機能を実装
  - _Requirements: 2.5_

- [ ] 10. 統合テストとE2Eテスト
- [ ] 10.1 ユニットテストスイート
  - ConfigLoaderのテスト（パース、検証、オーバーライド）を実装
  - ToolLoggerのテスト（記録、シリアライズ）を実装
  - ToolAnalyzerのテスト（統計計算）を実装
  - MetricsCollectorのテスト（抽出、検証）を実装
  - _Requirements: 1.5_

- [ ] 10.2 統合テスト
  - MarbleAdapter + MARBLEの統合テスト（初期化、ツールマッピング）を実装
  - Orchestrator + MarbleAdapter + ToolLoggerの実行フローテストを実装
  - ReportGenerator + MetricsCollector + ToolAnalyzerのレポート生成テストを実装
  - _Requirements: 1.3, 1.4_

- [ ] 10.3 E2Eテスト
  - Codingシナリオでのフルベンチマーク実行テストを実装
  - Databaseシナリオでのフルベンチマーク実行テストを実装
  - 同一設定での再実行による再現性検証テストを実装
  - _Requirements: 2.1, 2.2, 6.4_

## Requirements Coverage

| Requirement | Tasks |
|-------------|-------|
| 1.1 | 1.1, 1.2, 9.1 |
| 1.2 | 2.3 |
| 1.3 | 6.1, 10.2 |
| 1.4 | 6.2, 6.4, 10.2 |
| 1.5 | 2.3, 10.1 |
| 2.1 | 6.3, 7.1, 9.1, 10.3 |
| 2.2 | 6.3, 7.1, 9.1, 10.3 |
| 2.3 | 2.1, 6.2 |
| 2.4 | 2.1, 6.3 |
| 2.5 | 3.3, 9.2 |
| 3.1 | 5.1 |
| 3.2 | 5.1 |
| 3.3 | 5.1 |
| 3.4 | 5.1 |
| 3.5 | 5.2 |
| 4.1 | 3.1, 3.2 |
| 4.2 | 3.1, 3.2 |
| 4.3 | 3.1, 3.2 |
| 4.4 | 4.1 |
| 4.5 | 4.2, 8.3 |
| 5.1 | 8.1, 9.1 |
| 5.2 | 8.1 |
| 5.3 | 8.1 |
| 5.4 | 8.2 |
| 5.5 | 8.2 |
| 6.1 | 1.2, 2.3 |
| 6.2 | 2.3 |
| 6.3 | 2.2 |
| 6.4 | 7.2, 10.3 |
| 6.5 | 1.2, 2.2, 6.3 |
