# PuerTS TS 继承 UE 类并暴露给蓝图调用：调研与踩坑记录

> 来源：本轮对话中围绕 PuerTS 官方文档、DSGame/CozyLifeProject 插件源码、以及用户提供的 `puerts-uclass-extends-troubleshooting.md` 附件进行的调研总结。
> 时间：2026-07-02

## 主题

研究 PuerTS 中 TypeScript 代码如何通过继承 UE 类，被 Unreal 蓝图系统发现、生成蓝图资产，并在蓝图调用时转发回 TS 逻辑执行。重点包括：

- TS 类继承 `UE.Actor` / `UE.Character` / `UE.BlueprintFunctionLibrary` 的自动绑定模式；
- `PuertsEditor/CodeAnalyze.js` 如何解析 TS AST；
- `UPEBlueprintAsset` 如何创建蓝图资产；
- `UTypeScriptGeneratedClass` 如何让蓝图 UFunction 调用进入 TS；
- TS 静态函数库如何暴露为蓝图 FunctionLibrary；
- TypeScript 版本、`tsconfig.json` 和 PuertsEditor 运行时依赖的踩坑点。

## 关键事实

### 1. 蓝图不是直接调用 TS 文件

Unreal 蓝图系统只能发现 UE 反射对象，例如：

- `UClass`
- `UFunction`
- `UProperty`
- `UBlueprint`
- `UBlueprintGeneratedClass`

因此 PuerTS 的实现方式不是让蓝图直接扫描 `.ts` 文件，而是通过编辑器侧工具把 TS 类生成成真实的 UE 蓝图资产和 `UTypeScriptGeneratedClass`。

### 2. TS → Blueprint 的核心流程

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

### 3. CodeAnalyze 负责解析 TS AST

`PuertsEditor/CodeAnalyze.js` 会识别：

- 哪些类继承 UE 类型；
- 类名、父类、模块路径；
- 方法、属性、静态函数；
- `BlueprintCallable`、`BlueprintPure`、`BlueprintReadWrite` 等装饰器；
- 函数参数、返回值；
- `BlueprintFunctionLibrary` 特殊规则。

`CodeAnalyze.js` 发现符合条件的 TS 类后，会调用：

```js
let bp = new UE.PEBlueprintAsset();
bp.LoadOrCreateWithMetaData(...);
bp.AddFunctionWithMetaData(...);
bp.AddMemberVariableWithMetaData(...);
bp.Save();
```

### 4. 蓝图资产生成路径来源

PuerTS 插件中 `JsEnv.Build.cs` 定义：

```csharp
PublicDefinitions.Add("TS_BLUEPRINT_PATH=\"/GameBlueprints/TypeScriptAutoGen/\"");
```

`PEBlueprintAsset.cpp` 使用该宏创建蓝图资产：

```cpp
FString PackageName = FString(TEXT("/Game" TS_BLUEPRINT_PATH)) / InPath / InName;
```

因此默认磁盘路径通常为：

```text
Content/GameBlueprints/TypeScriptAutoGen/<InPath>/<ClassName>.uasset
```

运行时也通过 `TS_BLUEPRINT_PATH` 从蓝图 package path 反推出 TS module name。

### 5. UTypeScriptGeneratedClass 是调用转发核心

`UTypeScriptGeneratedClass` 继承自：

```cpp
UBlueprintGeneratedClass
```

其关键字段和函数包括：

- `DynamicInvoker`
- `FunctionToRedirect`
- `execCallJS`
- `execLazyLoadCallJS`
- `RedirectToTypeScript`
- `StaticConstructor`
- `ObjectInitialize`

`RedirectToTypeScript` 会把目标 `UFunction` 的 native function 设置为：

```cpp
UTypeScriptGeneratedClass::execCallJS
```

蓝图调用节点时，实际会进入：

```cpp
DynamicInvoker->InvokeTsMethod(Context, Func, Stack, RESULT_PARAM);
```

### 6. TS BlueprintFunctionLibrary 静态函数库是源码明确支持的

PuerTS 支持如下 TS 模式：

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

源码依据：

- `PEBlueprintAsset.cpp` 中如果父类是 `UBlueprintFunctionLibrary`，则使用 `BPTYPE_FunctionLibrary`；
- `CodeAnalyze.js` 中 `BlueprintFunctionLibrary` 只支持 static 方法；
- `FJsEnvImpl` 对 `UBlueprintFunctionLibrary` 子类有专门的 lazy load / native func 注入处理。

限制：

- 普通 `UE.Actor`/`UE.Object` 类不支持 static function；
- `UE.BlueprintFunctionLibrary` 只支持 static function，不支持实例方法；
- 顶层 `export function Foo()` 不会直接变成蓝图节点。

### 7. 文件名、类名、default export 必须一致

自动绑定模式要求：

```text
文件名 = 类名 = export default 的类名
```

例如：

```text
CozyBlueprintLib.ts
class CozyBlueprintLib
export default CozyBlueprintLib
```

运行时会通过蓝图路径反推出 module name，然后 `require(moduleName)` 并读取 `default` export。

### 8. 关键踩坑：TypeScript 版本一致性

PuerTS 的 `CodeAnalyze.js` 使用 TypeScript Compiler API 解析 AST。如果 PuertsEditor 构建出的 JS 产物使用的 TypeScript 版本，与项目编写/编译 TS 代码时使用的 TypeScript 版本不一致，可能导致：

- decorators 解析失败；
- `ts.SyntaxKind`、TypeChecker 行为不一致；
- `static` 判断异常；
- module path 推导异常；
- 蓝图资产不生成或函数缺失。

### 9. 踩坑：缺少 `useDefineForClassFields: false`

TypeScript 5.x 在 `target: esnext` 下默认 `useDefineForClassFields=true`，可能导致 PuerTS CodeAnalyze 或运行时对类属性识别异常。

推荐配置：

```json
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "useDefineForClassFields": false
  }
}
```

### 10. 踩坑：`ts.getDecorators is not a function`

用户提供的踩坑记录显示，某版本 PuerTS 内部存在不一致：

- `PuertsEditor/package.json` 依赖 `typescript: 4.7.4`；
- `UEMeta.ts` / `UEMeta.js` 调用了 TypeScript 5.x 才有的 `ts.getDecorators()`。

症状：

```text
gen blueprint for TsTestActor, path: Game
TypeError: ts.getDecorators is not a function
```

推荐修复：保持 TypeScript 4.7.4，并将：

```ts
ts.getDecorators(node)
```

替换为：

```ts
(node as any).decorators
```

### 11. 踩坑：TypeScript 5.x 的 `ts.sys` getter-only

如果升级 PuertsEditor 的 TypeScript 到 5.x，`CodeAnalyze.js` 中类似：

```js
t.sys = customSystem;
```

可能报：

```text
Cannot set property sys of #<Object> which has only a getter
```

因此当前更稳妥的策略是锁定 TS 4.7.4，而不是贸然升级到 TS 5.x。

### 12. 踩坑：PuertsEditor node_modules 未安装

如果 `Content/JavaScript/PuertsEditor/node_modules/typescript/` 不存在，`CodeAnalyze.js` 完全不会运行。

修复：

```bash
cd Content/JavaScript/PuertsEditor
npm install
```

验证：

```text
Content/JavaScript/PuertsEditor/node_modules/typescript/lib/typescript.js
```

存在。

## 诊断流程

```text
Step 1: 检查 UE Output Log
  ├─ 无 "start watch" 日志 → PuertsEditor node_modules 未安装 / CodeAnalyze 没启动
  ├─ 有 "start watch" 但无 "gen blueprint" → tsconfig 或 TS 编译/类结构问题
  ├─ 有 "gen blueprint" 但紧跟 Error → UEMeta / TypeScript Compiler API 版本不兼容
  └─ 有 "gen blueprint" 且无 Error → 检查 TS_BLUEPRINT_PATH 对应目录是否生成 .uasset

Step 2: 检查 tsconfig.json
  ├─ experimentalDecorators: true
  └─ useDefineForClassFields: false

Step 3: 检查 TS 文件结构
  ├─ export default <类名>
  ├─ 文件名与类名一致
  └─ extends UE.<原生类型>

Step 4: 检查 PuertsEditor TypeScript 版本
  └─ Content/JavaScript/PuertsEditor/node_modules/typescript/package.json

Step 5: 检查 UEMeta.js
  └─ 在 TS 4.7.4 环境下不应调用 ts.getDecorators
```

## 涉及的关键源码位置

- `Plugins/Puerts/Source/JsEnv/JsEnv.Build.cs`
  - 定义 `TS_BLUEPRINT_PATH`。
- `Plugins/Puerts/Content/JavaScript/PuertsEditor/CodeAnalyze.js`
  - 解析 TS AST，生成蓝图元数据。
- `Plugins/Puerts/Source/PuertsEditor/Private/PEBlueprintAsset.cpp`
  - 创建/更新 UE Blueprint 资产。
- `Plugins/Puerts/Source/JsEnv/Public/TypeScriptBlueprint.h`
  - `UTypeScriptBlueprint` 继承 `UBlueprint`。
- `Plugins/Puerts/Source/JsEnv/Public/TypeScriptGeneratedClass.h`
  - `UTypeScriptGeneratedClass` 继承 `UBlueprintGeneratedClass`。
- `Plugins/Puerts/Source/JsEnv/Private/TypeScriptGeneratedClass.cpp`
  - `execCallJS`、`execLazyLoadCallJS`、`RedirectToTypeScript`。
- `Plugins/Puerts/Source/JsEnv/Private/JsEnvImpl.cpp`
  - `MakeSureInject`、`RebindJs`、TS module 加载与函数重定向。

## 结论

PuerTS 的 TS → 蓝图能力可用，但它不是纯运行时能力，而是依赖编辑器侧 `CodeAnalyze.js` 将 TS 类转成 UE 蓝图资产和反射对象。要稳定使用该能力，必须严格控制：

- PuerTS 插件版本；
- PuertsEditor 使用的 TypeScript 版本；
- 项目 `TypeScript` 使用的 TypeScript 版本；
- `tsconfig.json` 的关键字段；
- `UEMeta.js` 与 TypeScript Compiler API 的兼容性；
- `PuertsEditor/node_modules` 是否完整安装。
