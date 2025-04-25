---
id: FieldMetaBase
title: FieldMetaBase
---

<!-- 请勿编辑：此页面是从类型注释自动生成的 -->

# 类型别名: FieldMetaBase\<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync\>

```ts
type FieldMetaBase<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync> = object;
```

定义于: [packages/form-core/src/FieldApi.ts:500](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L500)

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

### errorMap

```ts
errorMap: ValidationErrorMap<UnwrapFieldValidateOrFn<TParentData, TName, TOnMount, TFormOnMount>, UnwrapFieldValidateOrFn<TParentData, TName, TOnChange, TFormOnChange>, UnwrapFieldAsyncValidateOrFn<TParentData, TName, TOnChangeAsync, TFormOnChangeAsync>, UnwrapFieldValidateOrFn<TParentData, TName, TOnBlur, TFormOnBlur>, UnwrapFieldAsyncValidateOrFn<TParentData, TName, TOnBlurAsync, TFormOnBlurAsync>, UnwrapFieldValidateOrFn<TParentData, TName, TOnSubmit, TFormOnSubmit>, UnwrapFieldAsyncValidateOrFn<TParentData, TName, TOnSubmitAsync, TFormOnSubmitAsync>>;
```

与字段值相关的错误映射。

### isBlurred

```ts
isBlurred: boolean;
```

指示字段是否已失焦的标志。

### isDirty

```ts
isDirty: boolean;
```

如果字段的值已被用户修改，则为 `true` 的标志。与 `isPristine` 相反。

### isTouched

```ts
isTouched: boolean;
```

指示字段是否已被触碰的标志。

### isValidating

```ts
isValidating: boolean;
```

指示字段当前是否正在验证的标志。
