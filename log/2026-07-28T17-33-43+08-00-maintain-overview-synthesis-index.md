---
title: Maintain overview synthesis index
created: 2026-07-28
updated: 2026-07-28
type: source
tags:
  - source
sources: []
confidence: high
---

# Maintain overview synthesis index

## 操作摘要

按维护流程检查并修复 Wiki 的概览、综合洞察和索引一致性。

## 检查项

- 读取 `schema.md`、`wiki/index.md`、`log/manifest.jsonl`。
- 读取 `wiki/overview.md` 与 `wiki/synthesis.md`，发现二者仍为初始化占位内容，未覆盖当前重要页面。
- 扫描 `wiki/` 平铺文件，检查：断链、非法混写链接、外部 URL wikilink、wiki 子目录、frontmatter type 与文件名前缀不匹配、孤儿页。
- 发现 `wiki/index.md`、`wiki/overview.md`、`wiki/synthesis.md` 缺少 schema frontmatter；本次补齐。

## 修复内容

- 重建 `wiki/overview.md`：补充四大领域入口、核心 source 页面、知识地图，覆盖生猪产业、非洲猪瘟资本市场催化、中特估与长鑫科技等新增内容。
- 重建 `wiki/synthesis.md`：补充跨来源洞察，包括国家能力、政策叙事兑现链条、周期位置、指标到因果链条、可验证路径。
- 更新 `wiki/index.md`：补齐 frontmatter，保持索引链接不变。

## 未处理 / 说明

- 未发现断链、非法混写链接、外部 URL wikilink、wiki 子目录或孤儿页。
- `log/manifest.jsonl` 中 2026-07-22 两条 JSONL 之间存在历史换行/拼接格式问题，但本次维护不重写历史日志行；仅追加本次维护记录。

## 更新页面

- `wiki/overview.md`
- `wiki/synthesis.md`
- `wiki/index.md`

## 冲突

无。
