---
title: 通过 UE.Class.Load() 加载 PuerTS 生成的蓝图类
created: 2026-07-03
updated: 2026-07-03
type: concept
tags: [game-tech, unreal-engine, puerts, typescript, blueprint]
sources:
  - raw/2026-07-03-puerts-static-class-proto-chain-pitfall.md
confidence: high
---

# 通过 UE.Class.Load() 加载 PuerTS 生成的蓝图类

当需要在 TS 侧获取 PuerTS 生成的 `UTypeScriptGeneratedClass` 引用时，不应使用 `TsClass.StaticClass()`，而应使用 `UE.Class.Load()` 通过蓝图资产路径加载。

## 为什么不用 StaticClass()

PuerTS 生成的 TS 类调用 `.StaticClass()` 会返回父类的 UClass，原因见 [[concept-puerts-staticclass-proto-chain|PuerTS TS 生成类 StaticClass 返回父类 UClass 的原型链根因]]。

## 正确用法

```typescript
// 通过蓝图资产路径加载子类 UClass
const cls = UE.Class.Load(
    '/Game/Blueprints/TypeScript/Game/Dialogue/DialogueComponent.DialogueComponent_C'
);

const component = actor.GetComponentByClass(cls);
```

## 路径规则

PuerTS 生成的蓝图资产路径由 `TS_BLUEPRINT_PATH`（定义在 `JsEnv.Build.cs`）决定。DSGame 中为：

```text
/GameBlueprints/TypeScriptAutoGen/
```

路径推导规则：

| TS 源码路径 | 生成蓝图路径 |
|---|---|
| `TypeScript/Source/Game/Dialogue/DialogueComponent.ts` | `/Game/Blueprints/TypeScript/Game/Dialogue/DialogueComponent.DialogueComponent_C` |

注意末尾需要 `_C` 后缀（这是 UE 对蓝图生成的 UClass 命名约定）。

## 蓝图侧不受影响

蓝图编辑器的类选择器和 `GetComponentByClass` 节点直接使用 UE 反射系统内的 `UTypeScriptGeneratedClass` 引用，不存在原型链问题。

## 相关概念

- [[concept-puerts-staticclass-proto-chain|PuerTS TS 生成类 StaticClass 返回父类 UClass 的原型链根因]]
- [[concept-puerts-ts-blueprint-binding|PuerTS TS 到蓝图自动绑定机制]]
