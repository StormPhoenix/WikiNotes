# Wiki Schema

## Domain
本 Wiki 覆盖以下四个领域：
- **游戏技术**：游戏引擎、渲染、性能优化、工具链、游戏设计
- **政治经济历史**：政治体制、经济发展、历史事件、地缘政治、社会制度
- **金融投资交易**：市场分析、投资策略、交易系统、风险管理、宏观经济
- **阅读知识管理**：读书笔记、知识体系、学习方法、信息处理

## Conventions
- Keep raw sources under `raw/`; existing raw files are immutable.
- Keep generated knowledge pages physically flat under `wiki/`.
- Use lowercase type-slug filenames, e.g. `concept-example.md`.
- Keep `wiki/index.md` current with every durable wiki page.
- Record each operation as one standalone file under `log/` and one JSONL line in `log/manifest.jsonl`.

## Links
- **Internal wiki pages** (another file under `wiki/`): `[[concept-slug]]` or `[[concept-slug|label]]` — filename without extension, globally unique.
- **External http(s) URLs** (off-wiki sites): use standard Markdown `[label](https://url)` syntax, bare URLs in bullets, or frontmatter `sources`. Never put external URLs inside `[[...]]`.
- **Forbidden**: mixing wiki links and Markdown links together — invalid in both Markdown and Obsidian; choose either a wiki link or a Markdown external link, never combine them.
- When an external source deserves a graph node, create a wiki page first, link to it, and keep the original URL in `sources` or bullets.

## Frontmatter
```yaml
---
title: Page Title
created: YYYY-MM-DD
updated: YYYY-MM-DD
type: entity | concept | comparison | source | synthesis | overview
tags: []
sources: []
confidence: low | medium | high
---
```

## Tag Taxonomy
### 领域标签
- `#game-tech` — 游戏技术（引擎、渲染、性能、工具链）
- `#game-design` — 游戏设计（机制、叙事、UX）
- `#politics` — 政治体制、政策分析
- `#economics` — 经济学、发展研究
- `#history` — 历史事件与比较
- `#geopolitics` — 地缘政治、国际关系
- `#finance` — 金融市场、投资
- `#trading` — 交易策略与系统
- `#risk-mgmt` — 风险管理
- `#macro` — 宏观经济
- `#reading` — 读书笔记
- `#knowledge-mgmt` — 知识管理方法论
- `#learning` — 学习方法

### 通用标签
- `#source` — 原始资料/来源
- `#synthesis` — 综合分析
- `#comparison` — 对比分析
- `#todo` — 待深入
- `#key-insight` — 关键洞见
- `#question` — 待解答问题

## Page Thresholds
- Create a page when an entity/concept is central to one source or appears in 2+ sources.
- Passing mentions → add to existing page.

## Update Policy
- Do not silently overwrite conflicting claims.
- Flag unresolved contradictions for user review.
