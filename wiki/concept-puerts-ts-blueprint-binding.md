---
title: PuerTS TS 到蓝图自动绑定机制
created: 2026-07-02
updated: 2026-07-02
type: concept
domain: game-tech
tags: [game-tech, unreal-engine, puerts, typescript, blueprint]
sources:
  - raw/2026-07-02-puerts-ts-blueprint-uclass-troubleshooting.md
confidence: high
---

# PuerTS TS 到蓝图自动绑定机制

PuerTS 的 TS → 蓝图能力并不是让蓝图直接扫描或执行 `.ts` 文件，而是通过编辑器侧工具把 TypeScript 类转成真实的 Unreal Engine 蓝图资产和反射对象。

## 核心数据流

```text
TypeScript 类
  ↓ PuertsEditor/CodeAnalyze.js 解析 TS AST
自动生成 / 更新 UE Blueprint 资产
  ↓
生成的蓝图类使用 UTypeScriptBlueprint / UTypeScriptGeneratedClass
  ↓
UE 蓝图系统通过 AssetRegistry / 反射系统发现 UClass / UFunction / UProperty
  ↓
蓝图调用函数
  ↓
UTypeScriptGeneratedClass::execCallJS / execLazyLoadCallJS
  ↓
FJsEnvImpl / ITsDynamicInvoker 转发到 V8 中的 TypeScript 函数
```

## 为什么蓝图可以发现 TS 类

蓝图只能发现 UE 反射系统中的对象，例如 `UClass`、`UFunction`、`UBlueprint` 和 `UBlueprintGeneratedClass`。PuerTS 的策略是先生成蓝图资产：

```text
TS class
  ↓
UBlueprint asset
  ↓
UTypeScriptGeneratedClass
  ↓
UE 资产系统 / 蓝图系统可发现
```

## 编辑器侧：CodeAnalyze

`PuertsEditor/CodeAnalyze.js` 解析 TypeScript AST，识别：

- 类是否继承 UE 类型；
- 类名、父类、模块路径；
- 方法、属性、静态函数；
- `BlueprintCallable`、`BlueprintPure`、`BlueprintReadWrite` 等装饰器；
- 函数参数、返回值；
- `BlueprintFunctionLibrary` 特殊规则。

随后它会调用 `UE.PEBlueprintAsset`：

```js
let bp = new UE.PEBlueprintAsset();
bp.LoadOrCreateWithMetaData(...);
bp.AddFunctionWithMetaData(...);
bp.AddMemberVariableWithMetaData(...);
bp.Save();
```

## 蓝图资产路径

PuerTS 插件中 `JsEnv.Build.cs` 定义了 `TS_BLUEPRINT_PATH`。在 DSGame 中为：

```csharp
PublicDefinitions.Add("TS_BLUEPRINT_PATH=\"/GameBlueprints/TypeScriptAutoGen/\"");
```

`PEBlueprintAsset.cpp` 创建 package 时使用：

```cpp
FString PackageName = FString(TEXT("/Game" TS_BLUEPRINT_PATH)) / InPath / InName;
```

因此默认磁盘路径通常为：

```text
Content/GameBlueprints/TypeScriptAutoGen/<InPath>/<ClassName>.uasset
```

## 运行时：UTypeScriptGeneratedClass

`UTypeScriptGeneratedClass` 继承自 `UBlueprintGeneratedClass`，是蓝图调用回到 TS 的核心。它包含：

- `DynamicInvoker`
- `FunctionToRedirect`
- `execCallJS`
- `execLazyLoadCallJS`
- `RedirectToTypeScript`
- `StaticConstructor`
- `ObjectInitialize`

`RedirectToTypeScript` 会将目标 `UFunction` 的 native function 设置为 `UTypeScriptGeneratedClass::execCallJS`。蓝图调用节点时，最终会进入：

```cpp
DynamicInvoker->InvokeTsMethod(Context, Func, Stack, RESULT_PARAM);
```

## TS 模块绑定

运行时根据生成蓝图的 package 路径和 `TS_BLUEPRINT_PATH` 反推出 TS module name，然后：

```cpp
require(ModuleName)
```

并读取模块的 `default` export。这解释了为什么自动绑定模式要求：

```text
文件名 = 类名 = export default 的类名
```

## 相关概念

- [[concept-puerts-blueprintfunctionlibrary|PuerTS TS BlueprintFunctionLibrary 模式]]
- [[concept-puerts-typescript-version-compatibility|PuerTS TypeScript 版本兼容性]]
- [[source-2026-07-02-puerts-ts-blueprint-uclass-troubleshooting|调研与踩坑来源]]
