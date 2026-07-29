---
title: Upgrade schema/index domain display names
created: 2026-07-29
updated: 2026-07-29
type: source
domain: general
tags:
  - source
sources: []
confidence: high
---

# Upgrade schema/index domain display names

## 操作摘要

按最新新版模板补充 Domains 表说明，并将 `wiki/index.md` 的 Domain 分区标题升级为 `## Domain: <Name> (<id>)` 显示格式。

## 更新内容

- `schema.md`
  - 保留旧版用户定制 Domains 表与标签体系。
  - 补充 Domain ID / Name 说明：Domain ID 用于 frontmatter 与文件名前缀，Name 用于 index 展示。
- `wiki/index.md`
  - 将分区标题从 `## Domain: id` 升级为 `## Domain: Name (id)`。
  - 保持页面链接和域归属不变。

## 校验

- 未新增页面。
- 未重命名历史文件。
- 未改变存量页面 frontmatter domain。

## 冲突

无。
