---
id: FieldMetaDerived
title: FieldMetaDerived
---

<!-- 请勿编辑：此页面是从类型注释自动生成的 -->

# 类型别名: FieldMetaDerived\<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync\>

```ts
type FieldMetaDerived<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync> = object;
```

定义于: [packages/form-core/src/FieldApi.ts:590](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L590)

## 类型参数

• **TParentData**

• **TName** *extends* [`DeepKeys`](deepkeys.md)\<`TParentData`\>

• **TData** *extends* [`DeepValue`](deepvalue.md)\<`TParentData`, `TName`\>

• **TOnMount** *extends* `undefined` \| `FieldValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnChange** *extends* `undefined` \| `FieldValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnChangeAsync** *extends* `undefined` \| `FieldAsyncValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnBlur** *extends* `undefined` \| `FieldValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnBlurAsync** *extends* `undefined` \| `FieldAsyncValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnSubmit** *extends* `undefined` \| `FieldValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnSubmitAsync** *extends* `undefined` \| `FieldAsyncValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TFormOnMount** *extends* `undefined` \| `FormValidateOrFn`\<`TParentData`\>

• **TFormOnChange** *extends* `undefined` \| `FormValidateOrFn`\<`TParentData`\>

• **TFormOnChangeAsync** *extends* `undefined` \| `FormAsyncValidateOrFn`\<`TParentData`\>

• **TFormOnBlur** *extends* `undefined` \| `FormValidateOrFn`\<`TParentData`\>

• **TFormOnBlurAsync** *extends* `undefined` \| `FormAsyncValidateOrFn`\<`TParentData`\>

• **TFormOnSubmit** *extends* `undefined` \| `FormValidateOrFn`\<`TParentData`\>

• **TFormOnSubmitAsync** *extends* `undefined` \| `FormAsyncValidateOrFn`\<`TParentData`\>

## 类型声明

### errors

```ts
errors: (
  | UnwrapOneLevelOfArray<UnwrapFieldValidateOrFn<TParentData, TName, TOnMount, TFormOnMount>>
  | UnwrapOneLevelOfArray<UnwrapFieldValidateOrFn<TParentData, TName, TOnChange, TFormOnChange>>
  | UnwrapOneLevelOfArray<UnwrapFieldAsyncValidateOrFn<TParentData, TName, TOnChangeAsync, TFormOnChangeAsync>>
  | UnwrapOneLevelOfArray<UnwrapFieldValidateOrFn<TParentData, TName, TOnBlur, TFormOnBlur>>
  | UnwrapOneLevelOfArray<UnwrapFieldAsyncValidateOrFn<TParentData, TName, TOnBlurAsync, TFormOnBlurAsync>>
  | UnwrapOneLevelOfArray<UnwrapFieldValidateOrFn<TParentData, TName, TOnSubmit, TFormOnSubmit>>
  | UnwrapOneLevelOfArray<UnwrapFieldAsyncValidateOrFn<TParentData, TName, TOnSubmitAsync, TFormOnSubmitAsync>>)[];
```

与字段值相关的错误数组。

### isPristine

```ts
isPristine: boolean;
```

如果字段的值未被用户修改，则为 `true` 的标志。与 `isDirty` 相反。
