# Ingest Log: PuerTS TS Blueprint UClass Troubleshooting

- time: 2026-07-02T18:40:14+08:00
- kind: ingest

## Sources

- `raw/2026-07-02-puerts-ts-blueprint-uclass-troubleshooting.md`

## Created Pages

- `wiki/source-2026-07-02-puerts-ts-blueprint-uclass-troubleshooting.md`
- `wiki/concept-puerts-ts-blueprint-binding.md`
- `wiki/concept-puerts-blueprintfunctionlibrary.md`
- `wiki/concept-puerts-typescript-version-compatibility.md`

## Updated Pages

- `wiki/index.md`

## Summary

摄入关于 PuerTS 中 TypeScript 继承 UE 类、自动生成蓝图资产、蓝图发现并调用 TS 函数的调研资料。资料重点包括：

- `CodeAnalyze.js` 解析 TS AST；
- `UPEBlueprintAsset` 创建 / 更新蓝图资产；
- `UTypeScriptGeneratedClass` 继承 `UBlueprintGeneratedClass` 并通过 `execCallJS` / `execLazyLoadCallJS` 转发蓝图调用；
- `BlueprintFunctionLibrary` static 方法模式；
- `TS_BLUEPRINT_PATH` 路径来源；
- `useDefineForClassFields: false`、TypeScript 4.7.4、`ts.getDecorators`、`ts.sys`、PuertsEditor node_modules 等踩坑点。

## Conflicts

None.
