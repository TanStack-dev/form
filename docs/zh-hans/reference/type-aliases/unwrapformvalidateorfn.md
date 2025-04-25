---
id: UnwrapFormValidateOrFn
title: UnwrapFormValidateOrFn
---

<!-- 请勿编辑：此页面是从类型注释自动生成的 -->

# 类型别名: UnwrapFormValidateOrFn\<TValidateOrFn\>

```ts
type UnwrapFormValidateOrFn<TValidateOrFn> = [TValidateOrFn] extends [FormValidateFn<any>] ? ReturnType<TValidateOrFn> : [TValidateOrFn] extends [StandardSchemaV1<any, any>] ? Record<string, StandardSchemaV1Issue[]> : undefined;
```

定义于: [packages/form-core/src/FormApi.ts:90](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L90)

## 类型参数

• **TValidateOrFn** *extends* `undefined` \| `FormValidateOrFn`\<`any`\>
