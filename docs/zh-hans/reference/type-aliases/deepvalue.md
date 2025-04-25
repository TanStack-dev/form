---
source-updated-at: '2025-03-01T06:06:46.000Z'
translation-updated-at: '2025-04-12T04:12:08.975Z'
id: DeepValue
title: 类型 / DeepValue
---
<!-- 请勿编辑：此页面由类型注释自动生成 -->

# 类型别名：DeepValue\<TValue, TAccessor, TDepth\>

```ts
type DeepValue<TValue, TAccessor, TDepth> = unknown extends TValue ? TValue : TDepth["length"] extends 10 ? unknown : TValue extends ReadonlyArray<any> ? TAccessor extends `[${infer TBrackets}].${infer TAfter}` ? DeepValue<DeepValue<TValue, TBrackets, [...TDepth, any]>, TAfter, [...TDepth, any]> : TAccessor extends `[${infer TBrackets}]` ? DeepValue<TValue, TBrackets, [...TDepth, any]> : TAccessor extends keyof TValue ? TValue[TAccessor] : TValue[TAccessor & number] : TAccessor extends `${infer TBefore}[${infer TEverythingElse}` ? DeepValue<DeepValue<TValue, TBefore, [...TDepth, any]>, `[${TEverythingElse}`, [...TDepth, any]> : TAccessor extends `[${infer TBrackets}]` ? DeepValue<TValue, TBrackets, [...TDepth, any]> : TAccessor extends `${infer TBefore}.${infer TAfter}` ? DeepValue<DeepValue<TValue, TBefore, [...TDepth, any]>, TAfter, [...TDepth, any]> : TAccessor extends string ? 
  | Get<TValue, TAccessor>
  | ApplyNull<...> | ApplyUndefined<...> : never;
```

定义于：[packages/form-core/src/util-types.ts:104](https://github.com/TanStack/form/blob/main/packages/form-core/src/util-types.ts#L104)

推断对象或数组中深层嵌套属性的类型。

## 类型参数

• **TValue**

• **TAccessor**

• **TDepth** *继承自* `ReadonlyArray`\<`any`\> = \[\]
