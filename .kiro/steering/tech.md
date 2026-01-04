# Technology Stack

## Architecture

評価プロジェクトとして、ベンチマーク実行とレポート生成を主目的とする軽量構成。

## Core Technologies

- **評価対象**: cc-mirror Multi-Agent Orchestration（Claude Code ベータ機能）
- **ベンチマーク**: MARBLE (MultiAgentBench) - ACL 2025
- **有効化方法**: `npx cc-mirror quick --provider mirror --name mclaude`

## Key Libraries

- cc-mirror: Claude Code をマルチプロバイダー対応にするツール
- MARBLE: LLM ベースマルチエージェントシステム評価フレームワーク

## Development Standards

### ドキュメント規約
- 日本語で記述（技術用語・固有名詞は英語可）
- Markdown 形式

### 評価手順
- 再現可能な手順を明示
- 環境設定・バージョン情報を記録

## Development Environment

### Required Tools
- Node.js（npx 使用のため）
- Git

### Common Commands
```bash
# cc-mirror マルチエージェント有効化
npx cc-mirror quick --provider mirror --name mclaude
```

## Key Technical Decisions

- **ベンチマーク選定**: MARBLE を採用（学術的信頼性、ACL 2025 発表）
- **評価対象**: Claude Code 公式ベータ機能に限定（サードパーティ実装は対象外）

---
_Document standards and patterns, not every dependency_
