---
title: 信用利差走阔
created: 2026-08-31
updated: 2026-08-31
type: concept
domain: finance-investing-trading
tags:
  - finance
  - trading
  - risk-mgmt
confidence: high
---

# 信用利差走阔

## 定义

公司债收益率 = **无风险利率 + 信用利差**。利差是公司债收益率超出同期限国债的部分，单位基点（bp，1bp = 0.01%），补偿违约风险（违约概率 × 违约后损失率）与流动性溢价。

**走阔（widen）**：利差扩大——持有人抛售 → 债券价格跌 → 收益率升 → 相对国债拉开。含义是市场用真金白银要求更高风险补偿，同时新发债成本直接抬升。反向为**收窄（tighten）**。

## 数值示例

某公司 5 年期债券：国债 4.0%，公司债 5.5% → 利差 150bp。三个月后恶化：国债 4.2%（没动），公司债 7.5% → 利差 330bp，**走阔 180bp**——市场要求多 1.8 个百分点才肯借钱。

## 为什么债市先于股市（恐慌计机制）

股债对同一家公司的定价逻辑**不对称**：

| | 债券持有者 | 股票持有者 |
|---|---|---|
| 收益结构 | 上行封顶（票息+本金），下行巨亏（违约平均损失 60–80%） | 下行有限，上行无限 |
| 盯什么 | 资产负债表、现金流、偿债覆盖倍数 | 叙事、增长故事、市占率 |
| 对恶化反应 | 立即定价 | 可以长期"再等等看" |

典型剧本：基本面开始恶化时，债市交易台先跑，股市还在加仓——**利差走阔而股价坚挺的背离期，往往就是最后的窗口**。

## 历史验证

- **2007 年**：次贷 ABX 指数年初开始崩，标普 500 到 10 月才见顶——债市领先约 9 个月
- **2014–16 年**：能源高收益债利差 2014 年底开始走阔，领先油价见底
- **2020 年 3 月**：股债利差同步爆炸（同步案例）

## 追踪工具

- 全市场计：FRED 的 ICE BofA US High Yield Option-Adjusted Spread（代码 `BAMLH0A0HYM2`）
- 个券级：盯个别 AI 重度债务人（Oracle、Meta、CoreWeave 等）的单名利差与 CDS
- **中国语境**：A 股/港股基本无流动 CDS 市场 → 用**再融资条款恶化**做等效温度计（但要打折扣：可能混入制度性流动性因素，见 [[finance-concept-hk-stock-connect-southbound-liquidity|港股通与南向资金流动性]]）

## 相关概念

- [[finance-concept-cds-credit-default-swap|5Y CDS 信用违约互换]]
- [[finance-concept-circular-financing|循环融资]]
- [[finance-entity-minimax|MiniMax]]
- [[finance-source-2026-08-31-bis-aer2026-ai-capex-discussion|BIS AER 2026 讨论记录]]
