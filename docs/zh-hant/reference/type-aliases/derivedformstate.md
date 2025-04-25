---
source-updated-at: '2025-04-25T12:58:29.000Z'
translation-updated-at: '2025-04-25T20:52:33.990Z'
id: DerivedFormState
title: 類型 / DerivedFormState
---
# 型別別名：DerivedFormState\<TFormData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TOnServer\>

```ts
type DerivedFormState<TFormData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TOnServer> = object;
```

定義於：[packages/form-core/src/FormApi.ts:571](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L571)

## 型別參數

• **TFormData**

• **TOnMount** *繼承自* `undefined` \| `FormValidateOrFn`\<`TFormData`\>

• **TOnChange** *繼承自* `undefined` \| `FormValidateOrFn`\<`TFormData`\>

• **TOnChangeAsync** *繼承自* `undefined` \| `FormAsyncValidateOrFn`\<`TFormData`\>

• **TOnBlur** *繼承自* `undefined` \| `FormValidateOrFn`\<`TFormData`\>

• **TOnBlurAsync** *繼承自* `undefined` \| `FormAsyncValidateOrFn`\<`TFormData`\>

• **TOnSubmit** *繼承自* `undefined` \| `FormValidateOrFn`\<`TFormData`\>

• **TOnSubmitAsync** *繼承自* `undefined` \| `FormAsyncValidateOrFn`\<`TFormData`\>

• **TOnServer** *繼承自* `undefined` \| `FormAsyncValidateOrFn`\<`TFormData`\>

## 型別宣告

### canSubmit

```ts
canSubmit: boolean;
```

表示表單是否可根據當前狀態提交的布林值。

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

表單本身的錯誤陣列。

### fieldMeta

```ts
fieldMeta: Record<DeepKeys<TFormData>, AnyFieldMeta>;
```

表單中每個欄位的欄位元資料記錄。

### isBlurred

```ts
isBlurred: boolean;
```

表示表單中是否有任何欄位已失去焦點的布林值。

### isDirty

```ts
isDirty: boolean;
```

表示表單中是否有任何欄位的值已被使用者修改的布林值。若使用者修改了至少一個欄位則為 `true`。與 `isPristine` 相反。

### isFieldsValid

```ts
isFieldsValid: boolean;
```

表示表單中所有欄位是否有效的布林值。

### isFieldsValidating

```ts
isFieldsValidating: boolean;
```

表示表單中是否有任何欄位正在驗證的布林值。

### isFormValid

```ts
isFormValid: boolean;
```

表示表單是否有效的布林值。

### isFormValidating

```ts
isFormValidating: boolean;
```

表示表單是否正在驗證的布林值。

### isPristine

```ts
isPristine: boolean;
```

表示表單中所有欄位的值是否未被使用者修改的布林值。若使用者未修改任何欄位則為 `true`。與 `isDirty` 相反。

### isTouched

```ts
isTouched: boolean;
```

表示表單中是否有任何欄位已被觸碰的布林值。

### isValid

```ts
isValid: boolean;
```

表示表單及其所有欄位是否有效的布林值。
