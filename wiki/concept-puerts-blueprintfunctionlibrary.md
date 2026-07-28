---
title: PuerTS TS BlueprintFunctionLibrary 模式
created: 2026-07-02
updated: 2026-07-02
type: concept
domain: game-tech
tags: [game-tech, unreal-engine, puerts, typescript, blueprint]
sources:
  - raw/2026-07-02-puerts-ts-blueprint-uclass-troubleshooting.md
confidence: high
---

# PuerTS TS BlueprintFunctionLibrary 模式

PuerTS 支持在 TypeScript 中定义继承 `UE.BlueprintFunctionLibrary` 的类，并将其中的 static 方法暴露成蓝图函数库节点。这是实现“TS 全局函数供蓝图调用”的推荐模式。

## 最小示例

```ts
import * as UE from "ue";

class CozyBlueprintLib extends UE.BlueprintFunctionLibrary {
    @UE.ufunction.ufunction(UE.ufunction.BlueprintCallable, UE.ufunction.Category = "CozyLife|Debug")
    static PrintHello(Name: string): void {
        console.log(`[CozyLife] Hello ${Name}`);
    }

    @UE.ufunction.ufunction(UE.ufunction.BlueprintPure, UE.ufunction.Category = "CozyLife|Math")
    static Add(A: number /*@cpp:int*/, B: number /*@cpp:int*/): number /*@cpp:int*/ {
        return A + B;
    }
}

export default CozyBlueprintLib;
```

要求：

```text
CozyBlueprintLib.ts
class CozyBlueprintLib
export default CozyBlueprintLib
```

## 源码支持点

`CodeAnalyze.js` 中会判断父类是否是 `BlueprintFunctionLibrary`：

```js
let lsFunctionLibrary = baseTypeUClass && baseTypeUClass.GetName() === "BlueprintFunctionLibrary";
```

它明确限制：

- `BlueprintFunctionLibrary` 只支持 static 方法；
- 普通 Actor/Object TS 类不支持 static 方法。

`PEBlueprintAsset.cpp` 中如果父类是 `UBlueprintFunctionLibrary`，则创建：

```cpp
BPTYPE_FunctionLibrary
```

`FJsEnvImpl` 对 `UBlueprintFunctionLibrary` 子类还有专门的 `execLazyLoadCallJS` 注入逻辑。

## 数据流

```text
TS static method
  ↓ CodeAnalyze 解析 static + decorator
PEBlueprintAsset 创建 BPTYPE_FunctionLibrary 蓝图资产
  ↓
UTypeScriptGeneratedClass 中生成 static UFunction
  ↓
蓝图右键菜单发现函数节点
  ↓
蓝图调用 UFunction
  ↓
execCallJS / execLazyLoadCallJS
  ↓
调用 TS static 方法
```

## 注意事项

- 顶层 `export function Foo()` 不会直接变成蓝图节点。
- FunctionLibrary 中非 static 方法会被忽略。
- 函数参数和返回值必须能映射到 UE 支持类型。
- 装饰器解析依赖 [[concept-puerts-typescript-version-compatibility|PuerTS TypeScript 版本兼容性]]。

## 相关概念

- [[concept-puerts-ts-blueprint-binding|PuerTS TS 到蓝图自动绑定机制]]
- [[source-2026-07-02-puerts-ts-blueprint-uclass-troubleshooting|调研与踩坑来源]]
