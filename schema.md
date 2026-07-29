# Wiki Schema

## Domains
本 Wiki 覆盖以下四个领域，旨在积累跨领域知识与洞见。页面所属领域以 frontmatter `domain` 字段为准；文件名中的领域前缀只是可读性优化，历史文件可暂不批量重命名。

| Domain ID | Name | Scope |
|-----------|------|-------|
| general | General | 全局入口、索引、跨领域综合、尚未细分的页面 |
| game-tech | 游戏技术 | 游戏引擎、渲染、性能优化、工具链、游戏设计、PuerTS、UE/蓝图等 |
| political-economy-history | 政治经济历史 | 政治体制、经济发展、历史事件、地缘政治、社会制度、国家能力、工业化、扶贫、农业治理 |
| finance-investing-trading | 金融投资交易 | 市场分析、投资策略、交易系统、风险管理、宏观经济、产业研究、政策叙事、资本市场催化 |
| knowledge-management | 阅读知识管理 | 读书笔记、知识体系、学习方法、信息处理、个人反思、记录与成长 |

- **Domain ID**: lowercase ASCII slug, used in frontmatter `domain` and filename prefixes; keep it stable (renaming an ID is a breaking change).
- **Name**: human-readable display name (may be Chinese). `wiki/index.md` shows each domain section as `## Domain: <Name> (<id>)`, pairing the display name with its ID. This Domains table is the single source of truth for the ID↔Name mapping.

## Conventions
- Keep raw sources under `raw/`; existing raw files are immutable.
- Keep generated knowledge pages physically flat under `wiki/`.
- Use lowercase domain-type-slug filenames, e.g. `general-concept-example.md` or `tech-entity-karpathy.md`. A page's authoritative domain is its frontmatter `domain`; the filename prefix is a human-scannable convenience.
- Keep `wiki/index.md` current with every durable wiki page.
- Record each operation as one standalone file under `log/` and one JSONL line in `log/manifest.jsonl`.

历史兼容规则：当前已有大量 `type-slug.md` 命名的平铺页面，升级后不强制批量重命名；维护时优先补齐 frontmatter `domain` 字段，并在新建页面时逐步采用 `domain-type-slug.md` 命名。

## Links
- **Internal wiki pages** (another file under `wiki/`): `[[concept-slug]]` or `[[concept-slug|label]]` — filename without extension, globally unique.
- **External http(s) URLs** (off-wiki sites): use `[label](https://example.com/...)`, bare URLs in bullets, or frontmatter `sources`. Never put external URLs inside wiki double brackets.
- **Forbidden**: hybrid syntax that combines Obsidian double-bracket links with Markdown URL links; choose either a wiki link or a Markdown external link.
- When an external source deserves a graph node, create a wiki page first, link to it, and keep the original URL in `sources` or bullets.

## Frontmatter
```yaml
---
title: Page Title
created: YYYY-MM-DD
updated: YYYY-MM-DD
type: entity | concept | comparison | source | synthesis | overview | til
domain: general
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

## Type Routing & Boundaries

每种类型的判断标准、创建阈值、边界声明和推荐结构。ingest 时按此路由，maintain 时按此检查误分类。

### entity — 可命名的主体
- 判断：一个人、组织、工具、项目等可命名的主体，在资料中被反复提及或为核心主体。
- 阈值：在 1 个来源中为核心主体，或在 2+ 来源中出现。
- 边界：不记录概念解释（→concept）、系统性对比（→comparison）、碎片化知识点（→til）。
- 结构：概述 → 关键属性 → 出现的来源 → 相关概念链接

### concept — 体系化概念
- 判断：一个技术概念、方法论、模式等需要解释“它是什么”的抽象知识。
- 阈值：在 1 个来源中被定义或详细解释，或在 2+ 来源中出现。
- 边界：不记录具体人物/组织/工具（→entity）、跨概念综合洞察（→synthesis）、碎片化知识点（→til）。
- 结构：定义 → 核心要点 → 常见误区（可选）→ 相关概念链接

### til — 碎片化速记
- 判断：单个原子知识点：一个技巧、一个命令、一个坑、一个小发现。一句话能说清核心。
- 阈值：只要值得记录就建页，不需要 2+ 来源。如果需要“定义→核心要点”的体系化结构，用 concept。
- 边界：不记录完整概念解释（→concept）、人/组织/工具（→entity）、系统性对比（→comparison）。
- 结构：正文（简洁叙述）→ 为什么（可选）→ 常见误区（可选）
- 独有字段：follow-up

### comparison — 系统性对比
- 判断：两个或多个实体/概念的多维度系统性对比。
- 阈值：对比覆盖 3+ 维度且有足够信息撑起独立页面时建页；单次提及的简单对比写进已有页面。
- 边界：不记录单一概念的深度解释（→concept）。
- 结构：对比维度表 → 各方分析 → 结论

### source — 原始资料摘要
- 判断：一份 raw 资料的结构化摘要页。每份 raw 资料对应一个 source 页。
- 阈值：每份新增 raw 资料创建一个 source 页。
- 边界：不记录跨来源综合（→synthesis）、单一概念深度解释（→concept）。
- 结构：来源元信息 → 摘要 → 关键引用 → 关联页面

### synthesis — 跨来源综合洞察
- 判断：综合多个来源得出的洞察，难以归入单一 entity 或 concept。
- 阈值：洞察来自 2+ 来源的交叉分析。单来源的分析写进对应 source 或 concept 页。
- 边界：不记录单来源摘要（→source）、单一概念的常规解释（→concept）。
- 结构：核心洞察 → 支撑证据（引用多来源）→ follow-up（可选）

### overview — 领域全局概览
- 判断：领域的全局概览和入口指引。只维护一个页面。
- 阈值：wiki 初始化时创建，随 ingest/maintain 持续更新。
- 边界：不记录具体实体/概念的细节（→entity）。
- 结构：领域范围 → 核心入口 → 知识地图

### 通用规则
- Passing mention → 追加到已有页面，不新建页。
- 类型难以判断时，优先选 concept（最通用的类型）。

## Update Policy
- Do not silently overwrite conflicting claims.
- Flag unresolved contradictions for user review.

历史兼容策略：升级 schema 或 index 时保留既有页面链接和文件名，不因命名规范变化进行无必要的批量重命名；批量补字段应优先选择可逆、低风险的 frontmatter 增量维护。
