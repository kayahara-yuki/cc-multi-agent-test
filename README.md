# cc-mirror Multi-Agent Orchestration Evaluation

Claude Code 公式マルチエージェントオーケストレーション機能（cc-mirror 経由で有効化）を[MARBLE (MultiAgentBench)](https://github.com/ulab-uiuc/MARBLE)ベンチマークを使用して評価するプロジェクトです。

## 概要

### 背景

[cc-mirror](https://github.com/numman-ali/cc-mirror)は、Claude Code を様々な AI プロバイダーで動作させるためのツールです。作者の Numman Ali が、Claude Code の minifiedJS を解析し、無効化されているベータ機能「マルチエージェントオーケストレーション」を発見・有効化する方法を見つけました。

このプロジェクトでは、その機能を学術的なベンチマークである MARBLE を使用して定量的に評価します。

### 評価対象

- **cc-mirror Multi-Agent Orchestration**: Claude Code 公式のベータ機能
- `npx cc-mirror quick --provider mirror --name mclaude` で有効化

## MARBLE (MultiAgentBench) について

MARBLE は、ACL 2025 で発表された LLM ベースのマルチエージェントシステム評価ベンチマークです。

## ライセンス

MIT License

## 貢献者

- 評価実施: [kayahara-yuki](https://github.com/kayahara-yuki)
- cc-mirror 作者: [Numman Ali](https://github.com/numman-ali)
