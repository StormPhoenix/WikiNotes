# PuerTS TS 生成类 StaticClass() 返回父类 UClass 踩坑记录

> 来源：CozyLifeProject 实际开发中遇到的踩坑现象，以及通过 DSGame PuerTS 插件源码调研确认的根因。
> 时间：2026-07-03
> 关联文档：
> - `G:/DevRS/CozyLifeDevShell/CozyLifeProject/TypeScript/Docs/Tech/puerts-ts-staticclass-pitfall.md`
> - `G:/DevRS/CozyLifeDevShell/CozyLifeProject/TypeScript/Docs/Tech/puerts-uclass-extends-troubleshooting.md`

## 1. 现象

在 TS 中通过 PuerTS 继承 `UE.ActorComponent` 编写 `DialogueComponent`，然后在另一段 TS 代码中调用：

```typescript
const DialogueComp = (NPC as UE.Actor).GetComponentByClass(
    (require('../Dialogue/DialogueComponent').default as any).StaticClass()
);
```

实际返回的组件是 `SkeletalMeshComponent`，而非 `DialogueComponent`：

```
[Interaction] TryInteract: DialogueComp info [ClassName: SkeletalMeshComponent]
[Interaction] TryInteract: DialogueAsset property is null [HasProperty: false]
```

## 2. 根因：JS 原型链 + PuerTS 未注入生成类函数模板

### 2.1 JS/TS 类的静态继承

TS 中：

```ts
class DialogueComponent extends UE.ActorComponent { }
```

编译后等价于：

```js
DialogueComponent.__proto__ = UE.ActorComponent
```

这是 ES6 class 的静态继承语义——子类构造函数的 `__proto__` 指向父类构造函数。

### 2.2 PuerTS 的 StaticClass() 实现

PuerTS 在 `StructWrapper.cpp` 中为每个 UE 类型创建 V8 函数模板，并挂载 `StaticClass()` 函数：

```cpp
// StructWrapper.cpp:344-345
Result->Set(FV8Utils::InternalString(Isolate, "StaticClass"),
    v8::FunctionTemplate::New(Isolate, StaticClass, v8::External::New(Isolate, this)));
```

回调时引用当前 `FStructWrapper` 持有的 `UStruct`：

```cpp
// StructWrapper.cpp:464-478
void FStructWrapper::StaticClass(const v8::FunctionCallbackInfo<v8::Value>& Info) {
    FStructWrapper* This = reinterpret_cast<FStructWrapper*>(
        (v8::Local<v8::External>::Cast(Info.Data()))->Value());
    auto Result = FV8Utils::IsolateData<IObjectMapper>(Isolate)->FindOrAdd(
        Isolate, Context, This->Struct->GetClass(), This->Struct.Get());
    Info.GetReturnValue().Set(Result);
}
```

`This->Struct.Get()` 返回创建 `FStructWrapper` 时绑定的 UStruct。

对于 `UE.ActorComponent` 的函数模板，`This->Struct` = `UActorComponent::StaticClass()`。

### 2.3 原型链决议过程

```text
DialogueComponent.StaticClass()
  → DialogueComponent 自身没有 StaticClass 属性
  → 沿 __proto__ 向上查找
  → DialogueComponent.__proto__ = UE.ActorComponent
  → UE.ActorComponent.StaticClass()  ← 找到了
  → FStructWrapper(UActorComponent) 的 StaticClass()
  → 返回 UActorComponent::StaticClass()
```

### 2.4 PuerTS 已为生成类准备了正确的函数模板，但未注入到原型链

`JsEnvImpl.cpp::GetJsClass(UTypeScriptGeneratedClass)` 会为生成的 `UTypeScriptGeneratedClass(DialogueComponent_C)` 创建 V8 函数模板，上面也有正确的 `StaticClass()`。但 `MakeSureInject()` 中主要关注 instance prototype 链：

```cpp
// JsEnvImpl.cpp:1375-1400
// 构建了 Instance → NativeProto → Proto → Super Proto 的实例链
// 但没有将 GetJsClass(TypeScriptGeneratedClass) 注入到
// TS 构造函数 DialogueComponent 的 __proto__ 链上
```

因此：

```text
DialogueComponent.StaticClass()
  → 找到 UE.ActorComponent.StaticClass  ← 父类（❌）

正确路径本应是：
DialogueComponent.StaticClass()
  → 找到 UTypeScriptGeneratedClass(DialogueComponent_C).StaticClass()  ← 生成类（✅）
```

### 2.5 蓝图侧为什么正常？

蓝图编辑器通过 UE 资产系统直接拿到 `UTypeScriptGeneratedClass(DialogueComponent_C)` 的引用，不经过 JS 原型链。问题仅出现在 TS 侧。

## 3. 解决方案与通用规则

### 方案 A（采用）：使用 UE.Class.Load() 通过蓝图资产路径加载

```typescript
const DialogueCompClass = UE.Class.Load(
    '/Game/Blueprints/TypeScript/Game/Dialogue/DialogueComponent.DialogueComponent_C'
);
const DialogueComp = (NPC as UE.Actor).GetComponentByClass(DialogueCompClass);
```

### 通用规则

- **禁止** 在 TS 侧使用 `TsClass.StaticClass()` 获取 PuerTS 生成类的 UClass
- **应使用** `UE.Class.Load(blueprintPath)` 通过蓝图资产路径加载
- 蓝图侧 `GetComponentByClass` 节点无此限制

## 4. 关键源码位置

| 文件 | 行号 | 说明 |
|---|---|---|
| `StructWrapper.cpp` | 344-345 | `StaticClass` 挂载到 V8 函数模板 |
| `StructWrapper.cpp` | 464-478 | `StaticClass()` 回调实现，返回 `This->Struct.Get()` |
| `DeclarationGenerator.cpp` | 1443 | 仅生成 `static StaticClass(): Class;` 声明 |
| `JsEnvImpl.cpp` | 1200-1270 | `MakeSureInject` — TS 类 default export 获取与模块加载 |
| `JsEnvImpl.cpp` | 1371 | `NativeCtor = GetJsClass(TypeScriptGeneratedClass, Context)` |
| `JsEnvImpl.cpp` | 1375-1400 | instance prototype 链构建（不涉及构造函数静态方法） |
| `JsEnvImpl.cpp` | 3178 | `GetJsClass` — 为 `UTypeScriptGeneratedClass` 创建 V8 函数模板 |

所有文件来自 DSGame PuerTS v1.0.5 插件源码：
`D:/DSProjectDev/DSGame/Plugins/Puerts/Source/`
