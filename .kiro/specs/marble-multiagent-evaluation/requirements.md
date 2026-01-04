# Requirements Document

## Introduction

本ドキュメントは、cc-mirror Multi-Agent Orchestration 評価システムの要件を定義する。MARBLE（MultiAgentBench / ACL 2025）ベンチマークフレームワークを活用し、Claude Code 公式マルチエージェント機能（TaskCreate/TaskGet/TaskUpdate/TaskList ツール）の性能を定量的に評価することを目的とする。

**設計方針**: MARBLEが提供する機能（メトリクス計算、シナリオ実行、評価基盤）は独自実装せず、フレームワークをそのまま活用する。独自実装はMARBLEとcc-mirrorの統合部分、およびClaude Code固有の分析に限定する。

## Requirements

### Requirement 1: MARBLE統合基盤

**Objective:** As a 評価者, I want MARBLEフレームワークとcc-mirrorを統合する基盤, so that Claude Codeマルチエージェント機能をMARBLEで評価できる

#### Acceptance Criteria

1. When 評価者が統合セットアップコマンドを実行した時, the 統合基盤 shall MARBLEフレームワークをインストール・設定する
2. When 評価者が統合セットアップコマンドを実行した時, the 統合基盤 shall cc-mirror経由でマルチエージェント機能を有効化する
3. The 統合基盤 shall Claude CodeのTaskCreate/TaskGet/TaskUpdate/TaskListをMARBLEエージェントインターフェースにマッピングする
4. When MARBLEがエージェントにタスクを割り当てた時, the 統合基盤 shall Claude Codeのタスク管理ツールを呼び出す
5. If 統合設定に不整合がある場合, the 統合基盤 shall エラー内容を明示して処理を中断する

### Requirement 2: ベンチマーク実行

**Objective:** As a 評価者, I want MARBLEベンチマークを実行する機能, so that 標準シナリオでマルチエージェント性能を測定できる

#### Acceptance Criteria

1. When 評価者がベンチマーク実行コマンドを実行した時, the 評価システム shall MARBLEのCodingシナリオを起動する
2. When 評価者がベンチマーク実行コマンドを実行した時, the 評価システム shall MARBLEのDatabaseシナリオを起動する
3. The 評価システム shall MARBLEの協調プロトコル（Star/Tree/Graph/Chain）を選択可能にする
4. The 評価システム shall MARBLEの計画戦略（Vanilla/CoT/Group Discussion/Cognitive）を選択可能にする
5. While ベンチマーク実行中, the 評価システム shall MARBLEの標準ログ出力を保存する

### Requirement 3: メトリクス取得（MARBLE委譲）

**Objective:** As a 評価者, I want MARBLEが算出したメトリクスを取得する機能, so that 標準化された評価結果を利用できる

#### Acceptance Criteria

1. When MARBLEがベンチマークを完了した時, the 評価システム shall Task Score (TS)をMARBLE出力から取得する
2. When MARBLEがベンチマークを完了した時, the 評価システム shall Coordination Score (CS)をMARBLE出力から取得する
3. When MARBLEがベンチマークを完了した時, the 評価システム shall Milestone KPIをMARBLE出力から取得する
4. When MARBLEがベンチマークを完了した時, the 評価システム shall Communication ScoreとPlanning Scoreを個別に取得する
5. The 評価システム shall MARBLEのメトリクス出力フォーマットをそのまま保持する

### Requirement 4: Claude Code固有分析（独自実装）

**Objective:** As a 評価者, I want Claude Code固有のツール使用パターンを分析する機能, so that タスク管理ツールの協調動作を評価できる

#### Acceptance Criteria

1. While ベンチマーク実行中, the 評価システム shall TaskCreateの呼び出し回数・引数・タイミングを記録する
2. While ベンチマーク実行中, the 評価システム shall TaskGet/TaskUpdateの呼び出しパターンを記録する
3. While ベンチマーク実行中, the 評価システム shall TaskListの呼び出しタイミングと結果を記録する
4. When ベンチマークが完了した時, the 評価システム shall タスク依存関係（blocks/blockedBy）の使用統計を集計する
5. When ベンチマークが完了した時, the 評価システム shall エージェント間のタスク割り当て分布を可視化する

### Requirement 5: 評価レポート生成

**Objective:** As a 評価者, I want MARBLEメトリクスとClaude Code固有分析を統合したレポートを生成する機能, so that 評価結果を共有・報告できる

#### Acceptance Criteria

1. When レポート生成コマンドが実行された時, the 評価システム shall Markdown形式の評価レポートを生成する
2. The 評価レポート shall MARBLEメトリクス（TS, CS, KPI, Communication, Planning）のサマリーを含む
3. The 評価レポート shall Claude Code固有分析（タスク管理ツール使用統計）セクションを含む
4. The 評価レポート shall 評価環境情報（MARBLEバージョン、cc-mirror設定、実行日時）を含む
5. The 評価レポート shall 使用した協調プロトコルと計画戦略を明記する

### Requirement 6: 再現性確保

**Objective:** As a 評価者, I want 評価の再現性を確保する機能, so that 評価結果を検証・再実行できる

#### Acceptance Criteria

1. The 評価システム shall 使用したMARBLEバージョンとコミットハッシュを記録する
2. The 評価システム shall cc-mirrorの設定パラメータを記録する
3. The 評価システム shall 選択した協調プロトコルと計画戦略を設定ファイルとして保存する
4. When 評価設定ファイルが提供された時, the 評価システム shall 同一条件でMARBLEベンチマークを再実行する
5. The 評価システム shall MARBLE標準の乱数シード設定をサポートする
