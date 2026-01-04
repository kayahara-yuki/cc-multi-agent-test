# Project Structure

## Organization Philosophy

ドキュメント駆動型の評価プロジェクト。Kiro-style Spec Driven Development を採用し、仕様 → 設計 → 実装の流れで評価作業を進める。

## Directory Patterns

### Steering (`/.kiro/steering/`)
**Purpose**: プロジェクト全体のコンテキストとルールを保持
**Example**: `product.md`, `tech.md`, `structure.md`

### Specs (`/.kiro/specs/`)
**Purpose**: 個別機能・評価タスクの仕様書
**Pattern**: `{feature-name}/` 配下に `requirements.md`, `design.md`, `tasks.md`

### Settings (`/.kiro/settings/`)
**Purpose**: テンプレートとルール定義（メタデータ）
**Note**: Steering ファイルには詳細を記載しない

## Naming Conventions

- **Directories**: kebab-case (`feature-name/`)
- **Markdown files**: kebab-case (`requirements.md`, `design.md`)
- **仕様書**: 日本語で記述

## Import Organization

N/A（ドキュメント駆動プロジェクトのため）

## Code Organization Principles

1. **仕様駆動**: 実装前に Requirements → Design → Tasks の流れで仕様策定
2. **レビュー必須**: 各フェーズで承認を得てから次へ進む
3. **Steering 同期**: コードベースの変更時は Steering の更新も検討

---
_Document patterns, not file trees. New files following patterns shouldn't require updates_
