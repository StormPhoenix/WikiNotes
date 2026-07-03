---
title: PuerTS TS 生成类 StaticClass 返回父类 UClass 的原型链根因
created: 2026-07-03
updated: 2026-07-03
type: concept
tags: [game-tech, unreal-engine, puerts, typescript, blueprint, debug]
sources:
  - raw/2026-07-03-puerts-static-class-proto-chain-pitfall.md
confidence: high
---

# PuerTS TS 生成类 StaticClass 返回父类 UClass 的原型链根因

PuerTS 生成的 TS 类（继承自 UE.Actor / UE.ActorComponent）在 TS 侧调用 `.StaticClass()` 时返回的是**父类的 UClass**，而非生成的子类 UClass。

## 根因

TS class 编译后产生 ES6 静态继承：

```ts
class DialogueComponent extends UE.ActorComponent { }
```

等价于：

```js
DialogueComponent.__proto__ = UE.ActorComponent
```

当调用 `DialogueComponent.StaticClass()` 时：

1. `DialogueComponent` 自身没有 `StaticClass` 属性
2. 沿 `__proto__` 向上查找
3. `DialogueComponent.__proto__` = `UE.ActorComponent`
4. 找到 `UE.ActorComponent.StaticClass()`
5. 返回 `UActorComponent::StaticClass()` ← 父类

## PuerTS 已准备了正确的函数模板，但未注入

`JsEnvImpl::GetJsClass(UTypeScriptGeneratedClass)` 为生成的子类也创建了含正确 `StaticClass()` 的 V8 函数模板，但 `MakeSureInject()` 只构建了实例 prototype 链，没有把生成类的函数模板注入到 TS 构造函数的 `__proto__` 链上。

## 源码证据

`StructWrapper.cpp`：`StaticClass()` 回调内部调用 `This->Struct.Get()`，返回创建 FStructWrapper 时绑定的原始 UStruct。

因此：
- `UE.ActorComponent.StaticClass()` → 永远返回 `UActorComponent::StaticClass()`
- 即使 `DialogueComponent` 继承了 `UE.ActorComponent`，它的 `StaticClass()` 也不会返回 `DialogueComponent_C`

## 解决方案

**禁止**在 TS 侧使用 `TsClass.StaticClass()` 获取生成类的 UClass。

应使用 [[concept-puerts-ue-class-load|UE.Class.Load() 加载生成蓝图类]]：

```ts
const cls = UE.Class.Load('/Game/Blueprints/TypeScript/Game/Dialogue/DialogueComponent.DialogueComponent_C');
```

蓝图侧无此限制（蓝图直接使用 UE 反射系统的 `UTypeScriptGeneratedClass` 引用）。

## 相关概念

- [[concept-puerts-ts-blueprint-binding|PuerTS TS 到蓝图自动绑定机制]]
- [[concept-puerts-ue-class-load|UE.Class.Load() 加载生成蓝图类]]
- [[source-2026-07-03-puerts-static-class-proto-chain-pitfall|踩坑记录来源]]
