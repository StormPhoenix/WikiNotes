# Upgrade Log — 2026-07-05T13:50:33+08:00

## 升级范围
- `schema.md` — 升级到新版模板
- `wiki/index.md` — 解决 git merge 冲突 + 升级到新版模板

## schema.md 迁移详情

### 从旧版迁移的用户定制内容
- **Domain**：四个领域描述（游戏技术、政治经济历史、金融投资交易、阅读知识管理）完整保留
- **Tag Taxonomy**：13 个领域标签 + 6 个通用标签完整保留
- **Conventions**：旧版与新版模板一致，无自定义规则需迁移
- **Update Policy**：旧版与新版模板一致，无自定义策略需迁移

### 新版系统生成章节（未从旧版迁移）
- **Links**：采用新版模板措辞（将 "standard Markdown `[label](https://url)` syntax" 简化为 `[label](https://example.com/...)`）
- **Frontmatter**：采用新版模板，`type` 字段新增 `til` 类型
- **Type Routing & Boundaries**：全新章节，定义 7 种页面类型的判断标准、创建阈值、边界声明和推荐结构

## wiki/index.md 迁移详情

### 解决 Git Merge 冲突
旧版 index.md 存在未解决的 git merge 冲突标记（`<<<<<<< HEAD` / `=======` / `>>>>>>> 073d97b`），本次升级合并了两侧内容。

### 与实际文件交叉校验
对 wiki/ 目录下全部 27 个文件进行盘点，确认：
- **Sources**：7 个 source 页面，全部收录
- **Concepts**：17 个 concept 页面，全部收录（旧版冲突两侧合计仅列 16 个，遗漏 `concept-puerts-staticclass-proto-chain` 和 `concept-puerts-ue-class-load`，已补全）
- **Entities**：0 个（无 entity- 前缀文件）
- **Comparisons**：0 个（无 comparison- 前缀文件）
- **Synthesis**：链接到 `[[synthesis]]` 页面
- **Overview**：链接到 `[[overview]]` 页面
- **TIL**：旧版索引包含 `[[til-recording-shapes-identity|记录即自我塑造]]`，但该文件在 wiki/ 目录中实际不存在（疑在 git 冲突中丢失），已从索引中移除

### 新增内容
- `## Overview` section 新增 `[[overview|领域全局概览]]` 链接
- `## Synthesis` section 新增 `[[synthesis|跨来源综合洞察]]` 链接
- Concepts 列表按字母排序，便于维护
