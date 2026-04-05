# catalog 担当 path 一覧（ドラフト）

このドラフトは、`ssot-distribution-system-spec-final-2026-04-04.md` の「細分化後の基本ルール」を補うための初期表である。  
最初から全 path を固定するのではなく、**衝突しやすい共通 path を先に固定する**ために使う。

## 共通入口ファイル

| path | 担当 repo | 備考 |
| --- | --- | --- |
| `AGENTS.md` | `ssot-core` | 全 AI 共通の入口 |
| `CLAUDE.md` | `ssot-core` | Claude 向け共通入口 |
| `GEMINI.md` | `ssot-core` | Gemini 向け共通入口 |
| `.github/copilot/00-index.md` | `ssot-core` | SSOT の最終入口 |

## SSOT 系の初期固定 path

| path | 担当 repo | 備考 |
| --- | --- | --- |
| `.github/copilot/00-index.md` | `ssot-core` | 共通入口 |
| `.github/copilot/10-requirements.md` | `ssot-core` | 共通ガイド |
| `.github/copilot/20-architecture.md` | `ssot-core` | 共通ガイド |
| `.github/instructions/**` | `ssot-schema` | 実装・運用ルール |
| `.github/copilot/40-testing-strategy.md` | `ssot-policies` | ポリシー差分を管理 |
| `.github/copilot/50-security.md` | `ssot-policies` | ポリシー差分を管理 |

## Skills / MCP / Infra 系の初期固定 path

| path | 担当 repo | 備考 |
| --- | --- | --- |
| `.claude/agents/common/**` | `skills-core` | 複数案件で再利用する汎用スキル |
| `.claude/agents/domain/**` | `skills-domain` | 特定アプリや案件に閉じるスキル |
| `.claude/mcp/**` | `mcp-tools` | tool 定義 |
| `docs/mcp/**` | `mcp-server-core` | MCP 実行基盤の説明資産 |
| `infra/**` | `infra-runtime` | 非機密 template / placeholder のみ |
| `adapters/**` | `adapter-layer` | OS / shell / path 差分吸収 |

## 運用ルール

* この表にない path は、repo 追加時に段階的に決める
* 同じ path を複数 repo が担当しない
* `skill-catalog` と `test-simulation` は target repo へ直接コピーしない
