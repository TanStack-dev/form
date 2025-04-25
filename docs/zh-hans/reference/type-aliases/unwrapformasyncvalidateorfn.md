---
id: UnwrapFormAsyncValidateOrFn
title: UnwrapFormAsyncValidateOrFn
---

<!-- 请勿编辑：此页面是从类型注释自动生成的 -->

# 类型别名: UnwrapFormAsyncValidateOrFn\<TValidateOrFn\>

```ts
type UnwrapFormAsyncValidateOrFn<TValidateOrFn> = [TValidateOrFn] extends [FormValidateAsyncFn<any>] ? Awaited<ReturnType<TValidateOrFn>> : [TValidateOrFn] extends [StandardSchemaV1<any, any>] ? Record<string, StandardSchemaV1Issue[]> : undefined;
```

定义于: [packages/form-core/src/FormApi.ts:142](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L142)

## 类型参数

• **TValidateOrFn** *extends* `undefined` \| `FormAsyncValidateOrFn`\<`any`\>
