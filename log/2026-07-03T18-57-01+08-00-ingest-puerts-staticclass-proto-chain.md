# Ingest Log: PuerTS StaticClass Proto Chain Pitfall

- time: 2026-07-03T18:57:01+08:00
- kind: ingest

## Sources

- `raw/2026-07-03-puerts-static-class-proto-chain-pitfall.md`

## Created Pages

- `wiki/source-2026-07-03-puerts-static-class-proto-chain-pitfall.md`
- `wiki/concept-puerts-staticclass-proto-chain.md`
- `wiki/concept-puerts-ue-class-load.md`

## Updated Pages

- `wiki/index.md`

## Summary

摄入 CozyLifeProject 实际开发中遇到的 PuerTS TS 生成类 `StaticClass()` 返回父类 UClass 的踩坑现象。核心根因是 JS class extends 导致构造函数 `__proto__` 链接到父类，PuerTS 未将生成类 `UTypeScriptGeneratedClass` 的 V8 函数模板注入到 TS 构造函数的静态方法解析链上。源码证据来自 `StructWrapper.cpp:344-478`、`JsEnvImpl.cpp:1371-1400`、`DeclarationGenerator.cpp:1443`。解决方案为使用 `UE.Class.Load('/Game/..._C')` 替代 `StaticClass()`。

## Conflicts

None.
