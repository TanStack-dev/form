---
id: UnwrapFieldAsyncValidateOrFn
title: UnwrapFieldAsyncValidateOrFn
---

<!-- 请勿编辑：此页面是从类型注释自动生成的 -->

# 类型别名: UnwrapFieldAsyncValidateOrFn\<TParentData, TName, TValidateOrFn, TFormValidateOrFn\>

```ts
type UnwrapFieldAsyncValidateOrFn<TParentData, TName, TValidateOrFn, TFormValidateOrFn> =
  | [TFormValidateOrFn] extends [StandardSchemaV1<any, infer TStandardOut>] ? TName extends keyof TStandardOut ? StandardSchemaV1Issue[] : undefined : undefined
  | UnwrapFormAsyncValidateOrFnForInner<TFormValidateOrFn> extends infer TFormValidateVal ? TFormValidateVal extends object ? [DeepValue<TFormValidateVal, TName>] extends [never] ? undefined : StandardSchemaV1Issue[] : TFormValidateVal extends object ? TName extends keyof TFormValidateVal["fields"] ? TFormValidateVal["fields"][TName] : undefined : undefined : never
  | [TValidateOrFn] extends [FieldValidateAsyncFn<any, any, any>] ? Awaited<ReturnType<TValidateOrFn>> : [TValidateOrFn] extends [StandardSchemaV1<any, any>] ? StandardSchemaV1Issue[] : undefined;
```

定义于: [packages/form-core/src/FieldApi.ts:204](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L204)

## 类型参数

• **TParentData**

• **TName** *extends* [`DeepKeys`](deepkeys.md)\<`TParentData`\>

• **TValidateOrFn** *extends* `undefined` \| `FieldAsyncValidateOrFn`\<`any`, `any`, `any`\>

• **TFormValidateOrFn** *extends* `undefined` \| `FormAsyncValidateOrFn`\<`any`\>
