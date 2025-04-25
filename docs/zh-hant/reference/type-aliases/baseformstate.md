---
source-updated-at: '2025-04-25T12:58:29.000Z'
translation-updated-at: '2025-04-25T20:52:36.068Z'
id: BaseFormState
title: 類型 / BaseFormState
---
# 類型別名: BaseFormState\<TFormData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TOnServer\>

```ts
type BaseFormState<TFormData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TOnServer> = object;
```

定義於: [packages/form-core/src/FormApi.ts:496](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L496)

代表表單當前狀態的物件。

## 類型參數

• **TFormData**

• **TOnMount** *繼承自* `undefined` \| `FormValidateOrFn`\<`TFormData`\>

• **TOnChange** *繼承自* `undefined` \| `FormValidateOrFn`\<`TFormData`\>

• **TOnChangeAsync** *繼承自* `undefined` \| `FormAsyncValidateOrFn`\<`TFormData`\>

• **TOnBlur** *繼承自* `undefined` \| `FormValidateOrFn`\<`TFormData`\>

• **TOnBlurAsync** *繼承自* `undefined` \| `FormAsyncValidateOrFn`\<`TFormData`\>

• **TOnSubmit** *繼承自* `undefined` \| `FormValidateOrFn`\<`TFormData`\>

• **TOnSubmitAsync** *繼承自* `undefined` \| `FormAsyncValidateOrFn`\<`TFormData`\>

• **TOnServer** *繼承自* `undefined` \| `FormAsyncValidateOrFn`\<`TFormData`\>

## 類型宣告

### \_force\_re\_eval?

```ts
optional _force_re_eval: boolean;
```

@private, 用於在選項變更時強制重新評估表單狀態

### errorMap

```ts
errorMap: FormValidationErrorMap<UnwrapFormValidateOrFn<TOnMount>, UnwrapFormValidateOrFn<TOnChange>, UnwrapFormAsyncValidateOrFn<TOnChangeAsync>, UnwrapFormValidateOrFn<TOnBlur>, UnwrapFormAsyncValidateOrFn<TOnBlurAsync>, UnwrapFormValidateOrFn<TOnSubmit>, UnwrapFormAsyncValidateOrFn<TOnSubmitAsync>, UnwrapFormAsyncValidateOrFn<TOnServer>>;
```

表單本身的錯誤映射。

### fieldMetaBase

```ts
fieldMetaBase: Record<DeepKeys<TFormData>, AnyFieldMetaBase>;
```

表單中每個欄位的元資料記錄，不包含衍生屬性，如 `errors` 等

### isSubmitSuccessful

```ts
isSubmitSuccessful: boolean;
```

表示最後一次提交是否成功的布林值。

### isSubmitted

```ts
isSubmitted: boolean;
```

表示 `onSubmit` 函式是否已成功完成的布林值。

每次新的提交嘗試時會重置為 `false`。

注意: 您可以使用 isSubmitting 來檢查表單當前是否正在提交。

### isSubmitting

```ts
isSubmitting: boolean;
```

表示在呼叫 `handleSubmit` 後表單當前是否正在提交的布林值。

在以下情況之一完成時會重置為 `false`:
- 驗證步驟返回錯誤。
- `onSubmit` 函式已完成。

注意: 如果您在 `onSubmit` 函式中執行非同步操作，請確保等待它們完成，以確保 `isSubmitting` 僅在非同步操作完成時設為 `false`。

這對於在提交期間顯示載入指示器或禁用表單輸入非常有用。

### isValidating

```ts
isValidating: boolean;
```

表示表單或其任何欄位當前是否正在驗證的布林值。

### submissionAttempts

```ts
submissionAttempts: number;
```

追蹤提交嘗試次數的計數器。

### validationMetaMap

```ts
validationMetaMap: Record<ValidationErrorMapKeys, ValidationMeta | undefined>;
```

用於追蹤表單中驗證邏輯的內部機制。

### values

```ts
values: TFormData;
```

表單欄位的當前值。
