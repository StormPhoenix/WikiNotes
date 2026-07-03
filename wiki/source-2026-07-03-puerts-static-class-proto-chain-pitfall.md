---
title: PuerTS TS 生成类 StaticClass 返回父类原型链踩坑记录
created: 2026-07-03
updated: 2026-07-03
type: source
tags: [game-tech, unreal-engine, puerts, typescript, source, debug]
sources:
  - raw/2026-07-03-puerts-static-class-proto-chain-pitfall.md
confidence: high
---

# PuerTS TS 生成类 StaticClass 返回父类原型链踩坑记录

来源页，对应 raw 资料：
`raw/2026-07-03-puerts-static-class-proto-chain-pitfall.md`

## 覆盖内容

| 内容 | 详情 |
|---|---|
| 踩坑现象 | TS 侧 `DialogueComponent.StaticClass()` 返回 `UActorComponent` 而非 `DialogueComponent_C` |
| 根因 | JS class `extends` 导致 `__proto__` 指向父类，PuerTS 未将生成类函数模板注入原型链 |
| 源码证据 | `StructWrapper.cpp:344-478`, `JsEnvImpl.cpp:1371-1400`, `DeclarationGenerator.cpp:1443` |
| 解决方案 | 使用 `UE.Class.Load('/Game/..._C')` 替代 `StaticClass()` |
| 蓝图侧结论 | 蓝图不受影响 |

## 关联概念

- [[concept-puerts-staticclass-proto-chain|PuerTS TS 生成类 StaticClass 返回父类 UClass 的原型链根因]]
- [[concept-puerts-ue-class-load|UE.Class.Load() 加载生成蓝图类]]
