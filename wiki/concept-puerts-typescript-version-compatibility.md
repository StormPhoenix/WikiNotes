---
title: PuerTS TypeScript 版本兼容性
created: 2026-07-02
updated: 2026-07-02
type: concept
domain: game-tech
tags: [game-tech, puerts, typescript, unreal-engine, blueprint]
sources:
  - raw/2026-07-02-puerts-ts-blueprint-uclass-troubleshooting.md
confidence: high
---

# PuerTS TypeScript 版本兼容性

PuerTS 的 TS → 蓝图自动绑定链路依赖 `PuertsEditor/CodeAnalyze.js` 使用 TypeScript Compiler API 解析 AST。因此 PuertsEditor 运行时使用的 TypeScript 版本，必须与项目编写、编译 TS 代码时的 TypeScript 版本保持兼容。

## 为什么重要

CodeAnalyze 依赖 TypeScript Compiler API，例如：

- AST 节点结构；
- decorator 表示方式；
- `ts.SyntaxKind`；
- `TypeChecker` 行为；
- `tsconfig.json` 解析；
- `ts.sys` 行为；
- module path 推导。

版本不一致可能导致：

- TS 类无法生成蓝图；
- 蓝图资产生成但函数缺失；
- 装饰器解析失败；
- module path 与 JS 产物路径不一致；
- 运行时 require module 失败。

## 关键配置

项目 `tsconfig.json` 应包含：

```json
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "useDefineForClassFields": false
  }
}
```

其中：

- `experimentalDecorators: true` 用于让 `@UE.ufunction` / `@UE.uproperty` 可用；
- `useDefineForClassFields: false` 避免 TS 5.x 在 `target: esnext` 下采用 class field / defineProperty 语义，提升 PuerTS 兼容性。

## 已知踩坑

### `ts.getDecorators is not a function`

某些 PuerTS 版本中，`PuertsEditor/package.json` 依赖 `typescript: 4.7.4`，但 `UEMeta.ts` / `UEMeta.js` 调用了 TS 5.x 才有的：

```ts
ts.getDecorators(node)
```

在 TS 4.7.4 下会报：

```text
ts.getDecorators is not a function
```

推荐修复：保持 TS 4.7.4，将 `ts.getDecorators(node)` 替换为：

```ts
(node as any).decorators
```

### TS 5.x 的 `ts.sys` getter-only

如果升级到 TS 5.x，`CodeAnalyze.js` 中对 `ts.sys` 的赋值可能报：

```text
Cannot set property sys of #<Object> which has only a getter
```

因此在未完整适配 CodeAnalyze 前，不建议贸然升级 PuertsEditor TypeScript 到 5.x。

### PuertsEditor node_modules 未安装

如果：

```text
Content/JavaScript/PuertsEditor/node_modules/typescript/
```

不存在，`CodeAnalyze.js` 可能完全不运行。

修复：

```bash
cd Content/JavaScript/PuertsEditor
npm install
```

## 诊断建议

```text
无 "start watch" 日志 → PuertsEditor node_modules 未安装 / CodeAnalyze 未启动
有 "start watch" 但无 "gen blueprint" → tsconfig、TS 编译或类结构问题
有 "gen blueprint" 但紧跟 Error → UEMeta / TypeScript Compiler API 版本不兼容
有 "gen blueprint" 且无 Error → 检查 TS_BLUEPRINT_PATH 对应目录是否生成 .uasset
```

## 相关概念

- [[concept-puerts-ts-blueprint-binding|PuerTS TS 到蓝图自动绑定机制]]
- [[concept-puerts-blueprintfunctionlibrary|PuerTS TS BlueprintFunctionLibrary 模式]]
- [[source-2026-07-02-puerts-ts-blueprint-uclass-troubleshooting|调研与踩坑来源]]
