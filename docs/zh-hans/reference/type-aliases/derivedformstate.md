---
id: DerivedFormState
title: DerivedFormState
---

<!-- 请勿编辑：此页面是从类型注释自动生成的 -->

# 类型别名: DerivedFormState\<TFormData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TOnServer\>

```ts
type DerivedFormState<TFormData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TOnServer> = object;
```

定义于: [packages/form-core/src/FormApi.ts:470](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L470)

## 类型参数

• **TFormData**

• **TOnMount** *extends* `undefined` \| `FormValidateOrFn`\<`TFormData`\>

• **TOnChange** *extends* `undefined` \| `FormValidateOrFn`\<`TFormData`\>

• **TOnChangeAsync** *extends* `undefined` \| `FormAsyncValidateOrFn`\<`TFormData`\>

• **TOnBlur** *extends* `undefined` \| `FormValidateOrFn`\<`TFormData`\>

• **TOnBlurAsync** *extends* `undefined` \| `FormAsyncValidateOrFn`\<`TFormData`\>

• **TOnSubmit** *extends* `undefined` \| `FormValidateOrFn`\<`TFormData`\>

• **TOnSubmitAsync** *extends* `undefined` \| `FormAsyncValidateOrFn`\<`TFormData`\>

• **TOnServer** *extends* `undefined` \| `FormAsyncValidateOrFn`\<`TFormData`\>

## 类型声明

### canSubmit

```ts
canSubmit: boolean;
```

指示表单是否可以根据其当前状态提交的布尔值。

### errors

```ts
errors: (
  | UnwrapFormValidateOrFn<TOnMount>
  | UnwrapFormValidateOrFn<TOnChange>
  | UnwrapFormAsyncValidateOrFn<TOnChangeAsync>
  | UnwrapFormValidateOrFn<TOnBlur>
  | UnwrapFormAsyncValidateOrFn<TOnBlurAsync>
  | UnwrapFormValidateOrFn<TOnSubmit>
  | UnwrapFormAsyncValidateOrFn<TOnSubmitAsync>
  | UnwrapFormAsyncValidateOrFn<TOnServer>)[];
```

表单本身的错误数组。

### fieldMeta

```ts
fieldMeta: Record<DeepKeys<TFormData>, AnyFieldMeta>;
```

表单中每个字段的元数据记录。

### isBlurred

```ts
isBlurred: boolean;
```

指示表单的任何字段是否已失焦的布尔值。

### isDirty

```ts
isDirty: boolean;
```

指示表单的任何字段值是否已被用户修改的布尔值。如果用户至少修改了一个字段，则为 `true`。与 `isPristine` 相反。

### isFieldsValid

```ts
isFieldsValid: boolean;
```

指示所有表单字段是否有效的布尔值。

### isFieldsValidating

```ts
isFieldsValidating: boolean;
```

指示表单的任何字段当前是否正在验证的布尔值。

### isFormValid

```ts
isFormValid: boolean;
```

指示表单是否有效的布尔值。

### isFormValidating

```ts
isFormValidating: boolean;
```

指示表单当前是否正在验证的布尔值。

### isPristine

```ts
isPristine: boolean;
```

指示表单的所有字段值是否都未被用户修改的布尔值。如果用户没有修改任何字段，则为 `true`。与 `isDirty` 相反。

### isTouched

```ts
isTouched: boolean;
```

指示表单的任何字段是否已被触碰的布尔值。

### isValid

```ts
isValid: boolean;
```

指示表单及其所有字段是否有效的布尔值。
