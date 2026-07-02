---
title: PuerTS TS 继承 UE 类并暴露给蓝图调用调研与踩坑记录
created: 2026-07-02
updated: 2026-07-02
type: source
tags: [game-tech, unreal-engine, puerts, typescript, blueprint, source]
sources:
  - raw/2026-07-02-puerts-ts-blueprint-uclass-troubleshooting.md
confidence: high
---

# PuerTS TS 继承 UE 类并暴露给蓝图调用调研与踩坑记录

本页是对一次 PuerTS / Unreal Engine / TypeScript / Blueprint 集成调研的来源整理，核心问题是：**TypeScript 代码如何通过继承 UE 类，被蓝图发现并调用，以及该链路有哪些易踩坑点。**

## 资料覆盖范围

该来源综合了：

- PuerTS 官方文档中 Automatic Binding Mode / Engine calling TypeScript / Blueprint Mixin 相关机制；
- DSGame 中 PuerTS 插件源码的本地调研；
- 用户提供的 `puerts-uclass-extends-troubleshooting.md` 踩坑记录；
- CozyLifeProject 接入 PuerTS 时围绕 `build_ts.js`、类型生成和蓝图暴露的讨论。

## 关键结论

1. PuerTS 并不是让蓝图直接调用 `.ts` 文件，而是把 TS 类生成成真实 UE 蓝图资产和 `UTypeScriptGeneratedClass`。
2. 编辑器侧入口是 `PuertsEditor/CodeAnalyze.js`，它解析 TS AST 后调用 `UE.PEBlueprintAsset` 创建 / 更新蓝图资产。
3. 生成蓝图资产的默认路径由 `TS_BLUEPRINT_PATH` 控制，DSGame 中来自 `JsEnv.Build.cs`：`/GameBlueprints/TypeScriptAutoGen/`。
4. `UTypeScriptGeneratedClass` 继承自 `UBlueprintGeneratedClass`，并通过 `execCallJS` / `execLazyLoadCallJS` 将蓝图 `UFunction` 调用转发给 TS。
5. `UE.BlueprintFunctionLibrary` 是源码明确支持的特殊模式：TS 中继承它并声明 static 方法，可以生成蓝图函数库节点。
6. TypeScript 版本、`tsconfig.json` 与 PuertsEditor 运行时依赖必须严格匹配，否则 TS 类可能无法生成蓝图。

## 关联知识页

- [[concept-puerts-ts-blueprint-binding|PuerTS TS 到蓝图自动绑定机制]]
- [[concept-puerts-typescript-version-compatibility|PuerTS TypeScript 版本兼容性]]
- [[concept-puerts-blueprintfunctionlibrary|PuerTS TS BlueprintFunctionLibrary 模式]]
