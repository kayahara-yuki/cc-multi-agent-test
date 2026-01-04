# Research & Design Decisions

## Summary
- **Feature**: marble-multiagent-evaluation
- **Discovery Scope**: Complex Integration
- **Key Findings**:
  - MARBLEはモジュラー設計のPythonベースフレームワークで、Agent Graph、Cognitive Module、Coordination Engineの3コンポーネントで構成
  - Claude CodeのTaskCreate/TaskGet/TaskUpdate/TaskListツールはタスク管理APIとして機能し、MARBLEのAgent Interfaceにマッピング可能
  - MARBLEの評価メトリクス（TS、CS、KPI）は標準出力形式を持ち、独自実装不要で取得可能
  - **cc-mirror Team Mode**がタスク管理ツール（TaskCreate/TaskGet/TaskUpdate/TaskList）を有効化する
  - MARBLEカスタムエージェント統合は`marble/agent/base_agent.py`継承または`marble/llms/`へのプロバイダー追加で実現可能
  - MARBLEの協調プロトコル（Star/Tree/Chain/Graph）はClaude Codeのblocks/blockedBy依存関係構造として表現可能

## Research Log

### MARBLEフレームワーク アーキテクチャ
- **Context**: MARBLEの内部構造を理解し、統合ポイントを特定する必要があった
- **Sources Consulted**:
  - [GitHub - MultiagentBench/MARBLE](https://github.com/MultiagentBench/MARBLE)
  - [ACL 2025論文](https://arxiv.org/html/2503.01935v1)
  - [GitHub - ulab-uiuc/MARBLE](https://github.com/ulab-uiuc/MARBLE)
- **Findings**:
  - **Agent Graph Module**: 設定データをグラフ構造 G=(A,E) に変換、エージェント間関係（collaboration, supervision, negotiation）を定義
  - **Cognitive Module**: エージェントペルソナ、関係性、推論戦略（CoT, ReACT）を管理
  - **Configuration Module**: タスク仕様、ペルソナデータ、エージェントプロファイル、ロール定義を初期化
  - **Environment Module**: 関数呼び出しインターフェースでタスクシナリオをシミュレート
  - **Memory Module**: 共有・個別メモリをRAGベース検索で管理
  - **Communication Module**: コンテキスト認識型のエージェント間対話を処理
  - **Action Module**: 生成プランを実行し、結果をメモリにフィードバック
- **Implications**: Claude Codeのタスク管理ツールはAction ModuleおよびMemory Moduleと統合する必要がある

### MARBLE協調プロトコル
- **Context**: 評価時に選択可能な協調パターンを理解
- **Sources Consulted**: ACL 2025論文
- **Findings**:
  - **Star（集中型）**: 単一プランナーがすべてのアクターにタスク割り当て、フィードバック統合、強い監督
  - **Tree（階層型）**: トップレベルプランナーが下位プランナーに委譲、制御とスケーラビリティのバランス
  - **Chain（逐次型）**: エージェントが順次配置、依存関係のあるタスクに適合
  - **Graph-Mesh（分散型）**: 相互接続されたアクターによる並行計画と分散意思決定
- **Implications**: Claude CodeのTaskCreate/TaskUpdateのblocks/blockedByはChain/Tree構造と親和性が高い

### MARBLE計画戦略
- **Context**: 計画戦略の違いを理解し、評価設定に反映
- **Sources Consulted**: ACL 2025論文
- **Findings**:
  - **Vanilla Prompting**: ゼロショット命令でタスクプランを直接生成
  - **Chain-of-Thought**: タスク詳細とエージェントプロファイルを使用したステップバイステップ推論
  - **Group Discussion**: 複数エージェントが洞察を共有する協調的審議
  - **Cognitive Self-Evolving**: 反復間で期待vs実際のパフォーマンスを比較する人間的学習を模倣
- **Implications**: Claude Codeの自然言語ベースのタスク記述はすべての計画戦略と互換性がある

### MARBLE評価メトリクス
- **Context**: MARBLEが出力するメトリクスの定義と計算方法を把握
- **Sources Consulted**: ACL 2025論文
- **Findings**:
  - **Task Score (TS)**: マイルストーンベースKPI追跡 + ルールベース/LLMベースの最終出力品質評価
  - **Coordination Score (CS)**: Communication Score + Planning Scoreの平均
    - Communication Score (C_score): メッセージの明確さとアライメントの5点LLM評価
    - Planning Score (P_score): タスク整理とロール定義の5点評価
  - **KPI公式**: `KPI_overall = (1/N) × Σ(n_j/M)` (n_j=エージェントjの貢献マイルストーン、M=総マイルストーン、N=エージェント数)
- **Implications**: メトリクス計算はMARBLEに委譲、評価システムは出力取得のみで良い

### MARBLEベンチマークシナリオ
- **Context**: 利用可能なシナリオを把握し、評価対象を選定
- **Sources Consulted**: ACL 2025論文、GitHub README
- **Findings**:
  - **協調タスク**:
    - Research: 研究提案生成（5質問形式）
    - Minecraft: 協調建築
    - Database: 根本原因分析
    - Coding: マルチエージェントコーディングチャレンジ
  - **競争タスク**:
    - Werewolf: 社会推理ゲーム
    - Bargaining: 多者間交渉シミュレーション
- **Implications**: 要件ではCodingとDatabaseを必須シナリオとして指定、ResearchとMinecraftは将来拡張対象

### MARBLE技術スタック
- **Context**: 統合に必要な技術要件を把握
- **Sources Consulted**: GitHub README
- **Findings**:
  - Python 3.10必須
  - Poetry依存関係管理
  - `.env`ファイルによるAPI Key設定（OPENAI_API_KEY, Together_API_KEY）
  - Docker対応
  - `poetry install && bash run_simulation.sh`による実行
- **Implications**: cc-mirrorとの統合はPythonラッパーまたはシェルスクリプトで実現可能

### Claude Code タスク管理ツール
- **Context**: 評価対象となるClaude Codeのマルチエージェント機能を理解
- **Sources Consulted**:
  - [Claude Code Docs - Subagents](https://code.claude.com/docs/en/sub-agents)
  - [ClaudeLog - Task Agent Tools](https://claudelog.com/mechanics/task-agent-tools/)
- **Findings**:
  - **TaskCreate**: 新規タスク作成（subject, description）
  - **TaskGet**: タスク詳細取得（taskId）、依存関係（blocks/blockedBy）含む
  - **TaskUpdate**: タスク更新（status, addComment, addBlocks/addBlockedBy）
  - **TaskList**: 全タスク一覧（id, subject, status, owner, blockedBy）
  - タスク依存関係管理（blocks/blockedBy）でエージェント間の実行順序を制御
  - コメント機能でエージェント間のコミュニケーションを記録
- **Implications**: MARBLEのAgent GraphノードをClaude Codeタスクにマッピングし、エッジをblocks/blockedByに変換可能

## Architecture Pattern Evaluation

| Option | Description | Strengths | Risks / Limitations | Notes |
|--------|-------------|-----------|---------------------|-------|
| Adapter Pattern | MARBLEとClaude Code間のインターフェース変換層 | 疎結合、各システムの独立性維持、テスト容易 | 追加の抽象層によるオーバーヘッド | 推奨：要件の「統合基盤」に最適 |
| Wrapper Pattern | MARBLEをラップしてClaude Code APIを提供 | シンプル、既存MARBLEコードへの変更最小 | MARBLE内部への依存、アップデート追従必要 | 代替案 |
| Plugin Pattern | MARBLEのプラグインとしてClaude Codeを実装 | MARBLEエコシステムとの親和性 | MARBLE側の変更必要、保守性懸念 | 不採用：MARBLEのプラグインAPIが不明確 |

**選択**: Adapter Pattern - MARBLEのモジュラー設計を活かし、Claude Code固有のロジックを分離

## Design Decisions

### Decision: MARBLEメトリクス取得方法
- **Context**: MARBLEの評価メトリクス（TS, CS, KPI）をどのように取得するか
- **Alternatives Considered**:
  1. MARBLEソースコードを直接インポートしてメトリクス計算関数を呼び出す
  2. MARBLEのCLI出力をパースしてメトリクスを抽出
  3. MARBLEの実行後に出力ファイル（JSON/CSV）を読み取る
- **Selected Approach**: 出力ファイル読み取り方式（Option 3）
- **Rationale**:
  - 要件「MARBLEのメトリクス出力フォーマットをそのまま保持する」に合致
  - MARBLEの内部実装への依存を最小化
  - 再現性確保のためログファイルを保存する要件とも整合
- **Trade-offs**: リアルタイムメトリクス取得は不可、バッチ処理のみ
- **Follow-up**: MARBLEの出力ファイルフォーマットの詳細調査が必要

### Decision: Claude Codeツール呼び出しの記録方法
- **Context**: Claude Code固有分析のためにTaskCreate/TaskGet/TaskUpdate/TaskListの呼び出しを記録する方法
- **Alternatives Considered**:
  1. MARBLEのCommunication Moduleにフックを追加
  2. Claude Code側のログを直接パース
  3. Adapterレイヤーでインターセプトして記録
- **Selected Approach**: Adapterレイヤーでのインターセプト（Option 3）
- **Rationale**:
  - MARBLEコードへの変更不要
  - すべてのツール呼び出しを一元的にキャプチャ可能
  - 要件「呼び出し回数・引数・タイミングを記録」を完全に満たせる
- **Trade-offs**: Adapter実装の複雑化
- **Follow-up**: ログフォーマット設計が必要

### Decision: 協調プロトコル・計画戦略の選択インターフェース
- **Context**: 評価者がMARBLEの協調プロトコルと計画戦略を選択する方法
- **Alternatives Considered**:
  1. 設定ファイル（YAML/JSON）による静的設定
  2. CLI引数による実行時指定
  3. インタラクティブプロンプトによる選択
- **Selected Approach**: 設定ファイル + CLI引数のハイブリッド（Option 1 + 2）
- **Rationale**:
  - 設定ファイルで再現性を確保（要件6.3）
  - CLI引数で実行時の柔軟な変更が可能（要件2.3, 2.4）
- **Trade-offs**: 2つの設定方法の優先順位を明確にする必要あり
- **Follow-up**: CLI引数が設定ファイルをオーバーライドする仕様とする

## Risks & Mitigations
- **Risk 1**: MARBLEのバージョンアップによるAPI変更 — MARBLEバージョンとコミットハッシュを記録し、特定バージョンへの固定をサポート
- **Risk 2**: Claude Codeタスク管理ツールのベータ機能としての不安定性 — エラーハンドリングとリトライ機構を実装、ログを詳細に記録
- **Risk 3**: MARBLEの出力フォーマット変更 — スキーマバリデーションを実装し、不整合検出時に明確なエラーを出力
- **Risk 4**: cc-mirrorの設定不整合 — セットアップ時の検証チェックを実装（要件1.5）

## 追加調査（設計レビュー後）

### cc-mirror Team Mode によるタスク管理ツール有効化
- **Context**: Claude CodeのTaskCreate/TaskGet/TaskUpdate/TaskListツールがどのように有効化されるかを特定
- **Sources Consulted**:
  - [cc-mirror GitHub Repository](https://github.com/numman-ali/cc-mirror)
- **Findings**:
  - cc-mirrorは`~/.cc-mirror/`配下に分離されたClaude Codeインスタンスを作成
  - **Team Mode** を有効化すると、以下のツールが利用可能になる:
    - `TaskCreate`: 新規タスク作成
    - `TaskGet`: タスク詳細取得
    - `TaskUpdate`: タスク更新
    - `TaskList`: タスク一覧取得
  - Team Modeには**orchestrator skill**も含まれ、マルチエージェント協調パターンを教示
  - 有効化方法: `npx cc-mirror --enable-team-mode` または `--provider mirror`（デフォルトでTeam Mode有効）
- **Implications**:
  - cc-mirrorのTeam Modeが評価対象のタスク管理ツールを提供する
  - MARBLEとの統合はこのcc-mirror有効化環境を前提とする

### MARBLE カスタムエージェント統合ポイント
- **Context**: MARBLEに Claude Code を統合するための具体的な拡張ポイントを特定
- **Sources Consulted**:
  - [MARBLE GitHub - marble/agent/](https://github.com/ulab-uiuc/MARBLE/tree/main/marble/agent)
  - [MARBLE GitHub - marble/llms/](https://github.com/ulab-uiuc/MARBLE/tree/main/marble/llms)
- **Findings**:
  - **ディレクトリ構造**:
    ```
    marble/
    ├── agent/
    │   ├── base_agent.py      ← エージェント基底クラス
    │   ├── werewolf_agent.py  ← 実装例
    │   └── werewolf_prompts/  ← プロンプトテンプレート
    ├── llms/
    │   ├── model_prompting.py ← LLM呼び出し
    │   ├── text_embedding.py  ← 埋め込み
    │   └── error_handler.py   ← エラー処理
    ├── engine/                ← Coordination Engine
    ├── environments/          ← 環境定義
    └── configs/               ← 設定ファイル
    ```
  - **カスタムエージェント実装パターン**:
    1. `base_agent.py`を継承してカスタムエージェントクラスを作成
    2. `model_prompting.py`でLLMプロバイダーを設定
    3. `configs/`で環境変数（API Key）を設定
  - **エージェント-エンジン間フロー**:
    - Configuration Module → Agent Graph構築
    - エージェントがポリシー関数でアクション生成: `a_t = π(A_{t-1}, M_shared, M_individual)`
    - Environment Moduleがアクション処理: `o_{t+1} = Env(a_t)`
    - Memory ModuleとCommunication Moduleが結果を伝搬
- **Implications**:
  - Claude Code統合は`base_agent.py`の継承ではなく、**llms/model_prompting.py経由のプロバイダー追加**が適切
  - MARBLEの`llms/`にClaude Code（cc-mirror経由）用プロンプティングアダプターを追加
  - または、MARBLEを変更せず**外部ラッパー**としてClaude Codeを呼び出し、結果をMARBLEのAgent Interfaceに変換

### 協調プロトコル ↔ Claude Code blocks/blockedBy マッピング
- **Context**: MARBLEの協調プロトコルがClaude Codeのタスク依存関係にどうマッピングされるかを整理
- **Sources Consulted**: ACL 2025論文、設計レビュー分析
- **Findings**:

  | MARBLEプロトコル | 構造 | Claude Code blocks/blockedBy マッピング |
  |-----------------|------|----------------------------------------|
  | **Star** | 1プランナー → N アクター | プランナータスクが全アクタータスクをblocks（アクターはプランナー完了を待つ） |
  | **Tree** | 階層的委譲 | 親タスクが子タスクをblocks、子タスクの完了が親の次フェーズをunblock |
  | **Chain** | 順次実行 | 各タスクが次タスクをblocks（`T1 blocks T2 blocks T3 ...`） |
  | **Graph-Mesh** | 相互接続 | 複数のblocks/blockedBy関係、並行実行可能なタスクは依存関係なし |

  - **Claude Code Task Toolsの役割**:
    - `TaskCreate`: MARBLEがエージェントにタスク割り当て時に呼び出し
    - `TaskUpdate(addBlocks/addBlockedBy)`: 協調プロトコルに応じた依存関係設定
    - `TaskGet`: エージェントが自身の担当タスクと依存関係を確認
    - `TaskList`: 全体の進捗とブロック状況を監視

- **Implications**:
  - 協調プロトコルの違いは**blocks/blockedByの構造**として表現される
  - **Star**: 少数のblocks関係（プランナー→アクター）
  - **Chain**: 線形のblocks関係
  - **Tree/Graph**: 複雑なblocks/blockedByネットワーク
  - Claude Codeへのプロンプトでプロトコル構造を明示的に伝達する必要がある

### 統合アプローチの明確化
- **Context**: 設計レビューで指摘された「統合メカニズムの不明確さ」に対応
- **結論**: 以下の2つのアプローチが実現可能

  **アプローチA: MARBLEの`llms/`を拡張**
  ```
  MARBLE → llms/claude_code_adapter.py → cc-mirror (subprocess) → Claude Code
  ```
  - MARBLEの`model_prompting.py`パターンに従い、Claude Code用アダプターを追加
  - cc-mirrorを子プロセスとして起動し、標準入出力でやり取り
  - MARBLEの設計思想に沿った統合

  **アプローチB: 外部オーケストレーター（設計書の方針）**
  ```
  Evaluation System → MarbleAdapter → MARBLE (benchmark execution)
                   ↓
                   → cc-mirror → Claude Code (Task Tools)
  ```
  - MARBLEをブラックボックスとして使用
  - 独自のMarbleAdapterがMARBLEとClaude Codeを橋渡し
  - MARBLEコードの変更不要、保守性が高い

  **推奨**: アプローチB（外部オーケストレーター）
  - 理由: MARBLEへの変更が不要、cc-mirrorの分離環境を活用、要件「MARBLEに委譲」の方針と整合

## References
- [MARBLE GitHub Repository (MultiagentBench)](https://github.com/MultiagentBench/MARBLE) — 公式リポジトリ、最新コード
- [MARBLE GitHub Repository (ulab-uiuc)](https://github.com/ulab-uiuc/MARBLE) — 開発元リポジトリ
- [MultiAgentBench ACL 2025論文](https://arxiv.org/html/2503.01935v1) — フレームワーク詳細、メトリクス定義
- [Claude Code Subagents Documentation](https://code.claude.com/docs/en/sub-agents) — 公式ドキュメント
- [ClaudeLog Task Agent Tools](https://claudelog.com/mechanics/task-agent-tools/) — タスクツール実践ガイド
- [cc-mirror GitHub Repository](https://github.com/numman-ali/cc-mirror) — Team Mode、タスク管理ツール有効化
