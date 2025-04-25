---
source-updated-at: '2025-03-01T06:06:46.000Z'
translation-updated-at: '2025-04-12T04:11:48.710Z'
id: DeepKeys
title: 类型 / DeepKeys
---
# 类型别名：DeepKeys\<T, TDepth\>

```ts
type DeepKeys<T, TDepth> = TDepth["length"] extends 5 ? never : unknown extends T ? PrefixFromDepth<string, TDepth> : T extends readonly any[] & IsTuple<T> ? PrefixTupleAccessor<T, AllowedIndexes<T>, TDepth> : T extends any[] ? PrefixArrayAccessor<T, [...TDepth, any]> : T extends Date ? never : T extends object ? PrefixObjectAccessor<T, TDepth> : T extends string | number | boolean | bigint ? "" : never;
```

定义于：[packages/form-core/src/util-types.ts:70](https://github.com/TanStack/form/blob/main/packages/form-core/src/util-types.ts#L70)

用于获取对象或数组的深层嵌套键名。

## 类型参数

• **T**

• **TDepth** *继承自* `any`[] = \[\]
