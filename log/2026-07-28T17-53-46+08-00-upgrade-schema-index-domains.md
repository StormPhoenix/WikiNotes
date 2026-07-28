---
title: Upgrade schema and index to domain-aware layout
created: 2026-07-28
updated: 2026-07-28
type: source
domain: general
tags:
  - source
sources: []
confidence: high
---

# Upgrade schema and index to domain-aware layout

## 操作摘要

按新版 LLM Wiki schema/index 模板完成升级：引入 frontmatter `domain` 字段、Domains 表、按「域/类型」两级结构组织 `wiki/index.md`。

## 迁移内容

- 旧版 `schema.md` 的 Domain 自由文本迁移为新版 `## Domains` 引言与 Domains 表。
- 保留旧版 Tag Taxonomy 自定义标签体系。
- 保留 Conventions、Links、Page Thresholds、Type Routing & Boundaries、Update Policy，并补充历史兼容规则。
- 新版 Frontmatter 增加 `domain: general` 字段。
- `wiki/index.md` 从类型单级结构升级为 Domain/Type 两级结构。

## Domain 登记

- `general`：全局入口、索引、跨领域综合、尚未细分的页面
- `game-tech`：游戏技术、PuerTS、UE/蓝图等
- `political-economy-history`：政治经济历史、国家能力、工业化、扶贫、农业治理
- `finance-investing-trading`：金融投资交易、宏观、产业研究、政策叙事、资本市场催化
- `knowledge-management`：阅读知识管理、个人反思、记录与成长

## 存量页面维护

- 为所有 `wiki/` 下存量 Markdown 页面补齐 frontmatter `domain` 字段。
- 对 `source-2026-07-12-personal-reflection-passion-vs-obligation.md` 补齐 `type: source`、`domain`、`tags`、`sources`、`confidence`。
- 未批量重命名既有 `type-slug.md` 文件，遵循历史兼容策略；域归属以 frontmatter `domain` 为准。

## 更新页面

- `schema.md`
- `wiki/index.md`
- `wiki/overview.md`
- `wiki/synthesis.md`
- 所有缺少 `domain` 字段的存量 `wiki/*.md` 页面

## 校验

升级后只读扫描 `wiki/`：

- wiki 文件数：50
- 缺失 frontmatter：0
- 缺失 domain：0
- invalid type：0
- 断链：0
- 外部 URL WikiLink：0
- 混写链接：0
- wiki 子目录：0

## 冲突

无。
