# Ingest: 猪肉屠宰业务知识全景

- **时间**：2026-08-04T12:55:00+08:00
- **操作类型**：ingest
- **域**：finance-investing-trading
- **来源**：`raw/2026-08-04-zhurou-tuzai-yewu-zhishi-quanjing.md`

## 背景

本轮对话从"猪肉屠宰业务具体包含哪些内容"开始，系统梳理了屠宰流程、产品体系、盈利模式（毛白差）、竞争格局，并深入拆解了屠宰业务平滑猪周期的五层机制。随后用户要求输出为文档并入库 Wiki。

已有的猪相关页面包括：PSY、育种体系、国储收储、驯化、人工授精、公猪阉割、非洲猪瘟、猪瘟与资本市场催化、本土 vs 进口对比。本次新增的是**屠宰业务**这一全新主题。

## 新增 raw 资料

| 文件 | 大小 | 内容 |
|------|------|------|
| `raw/2026-08-04-zhurou-tuzai-yewu-zhishi-quanjing.md` | 11588 bytes | 屠宰流程七步、产品体系（主产品+副产品）、盈利模式（毛白差/终端价格刚性/产能利用率）、竞争格局（三种商业模式）、平滑猪周期五层机制 |

## 新增 wiki 页面

| 页面 | 类型 | 内容 |
|------|------|------|
| `wiki/source-2026-08-04-zhurou-tuzai-yewu-zhishi-quanjing.md` | source | 原始资料摘要页 |
| `wiki/concept-tuzai-yewu.md` | concept | 屠宰业务：完整流程七步、产品体系、盈利模式（毛白差/终端价格刚性/产能利用率）、三种商业模式 |
| `wiki/concept-tuzai-pinghua-zhouqi.md` | concept | 屠宰平滑猪周期逻辑：跷跷板效应五层机制（毛白差反向/内部转移定价/出栏弹性/副产品缓冲/冻品套利），波动收窄60% |
| `wiki/entity-shuanghui-fazhan.md` | entity | 双汇发展：2025营收592.74亿/净利51.05亿，屠宰+深加工模式，屠宰毛利率5-8%/肉制品25-30% |

## 更新页面

| 页面 | 更新内容 |
|------|---------|
| `wiki/index.md` | Sources +1，Entities +1，Concepts +2 |

## 交叉引用

新增页面与已有页面的关联：

- `concept-tuzai-yewu` ↔ `concept-tuzai-pinghua-zhouqi`（互相链接）
- `concept-tuzai-yewu` → `entity-shuanghui-fazhan`（双汇是屠宰+深加工代表）
- `concept-tuzai-yewu` → `concept-feizhou-zhuwen`（屠宰验收需查非瘟检测报告）
- `concept-tuzai-yewu` → `concept-guochu-shouchu-115kg`（政策影响出栏体重进而影响屠宰）
- `concept-tuzai-yewu` → `concept-psy-pigs-per-sow-per-year`（同属生猪产业链）
- `concept-tuzai-pinghua-zhouqi` → `concept-zhuwen-ziben-shichang-cuihua`（猪周期投资分析）
- `entity-shuanghui-fazhan` → `concept-tuzai-yewu` + `concept-tuzai-pinghua-zhouqi`
- 两个 source 页面互相链接（同属生猪产业知识体系）

## 冲突

无冲突。
