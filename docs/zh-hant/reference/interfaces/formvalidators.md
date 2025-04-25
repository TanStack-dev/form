---
source-updated-at: '2025-04-25T12:58:29.000Z'
translation-updated-at: '2025-04-25T20:52:38.470Z'
id: FormValidators
title: 介面 / FormValidators
---
# 介面: FormValidators\<TFormData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync\>

定義於: [packages/form-core/src/FormApi.ts:156](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L156)

## 型別參數

• **TFormData**

• **TOnMount** *繼承自* `undefined` \| `FormValidateOrFn`\<`TFormData`\>

• **TOnChange** *繼承自* `undefined` \| `FormValidateOrFn`\<`TFormData`\>

• **TOnChangeAsync** *繼承自* `undefined` \| `FormAsyncValidateOrFn`\<`TFormData`\>

• **TOnBlur** *繼承自* `undefined` \| `FormValidateOrFn`\<`TFormData`\>

• **TOnBlurAsync** *繼承自* `undefined` \| `FormAsyncValidateOrFn`\<`TFormData`\>

• **TOnSubmit** *繼承自* `undefined` \| `FormValidateOrFn`\<`TFormData`\>

• **TOnSubmitAsync** *繼承自* `undefined` \| `FormAsyncValidateOrFn`\<`TFormData`\>

## 屬性

### onBlur?

```ts
optional onBlur: TOnBlur;
```

定義於: [packages/form-core/src/FormApi.ts:185](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L185)

選擇性函式，當欄位失去焦點時驗證表單資料，回傳 `FormValidationError`

***

### onBlurAsync?

```ts
optional onBlurAsync: TOnBlurAsync;
```

定義於: [packages/form-core/src/FormApi.ts:189](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L189)

選擇性的 onBlur 非同步驗證方法，當欄位失去焦點時回傳 `FormValidationError` 或 `Promise<FormValidationError>`

***

### onBlurAsyncDebounceMs?

```ts
optional onBlurAsyncDebounceMs: number;
```

定義於: [packages/form-core/src/FormApi.ts:193](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L193)

預設的毫秒數，若設定為大於 0 的數字，會以此毫秒數延遲非同步驗證事件。

***

### onChange?

```ts
optional onChange: TOnChange;
```

定義於: [packages/form-core/src/FormApi.ts:173](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L173)

選擇性函式，當值變更時檢查資料的有效性

***

### onChangeAsync?

```ts
optional onChangeAsync: TOnChangeAsync;
```

定義於: [packages/form-core/src/FormApi.ts:177](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L177)

選擇性的 onChange 非同步對應方法，適用於更複雜的驗證邏輯，可能包含伺服器請求。

***

### onChangeAsyncDebounceMs?

```ts
optional onChangeAsyncDebounceMs: number;
```

定義於: [packages/form-core/src/FormApi.ts:181](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L181)

預設的毫秒數，若設定為大於 0 的數字，會以此毫秒數延遲非同步驗證事件。

***

### onMount?

```ts
optional onMount: TOnMount;
```

定義於: [packages/form-core/src/FormApi.ts:169](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L169)

選擇性函式，在元件掛載時立即觸發。

***

### onSubmit?

```ts
optional onSubmit: TOnSubmit;
```

定義於: [packages/form-core/src/FormApi.ts:194](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L194)

***

### onSubmitAsync?

```ts
optional onSubmitAsync: TOnSubmitAsync;
```

定義於: [packages/form-core/src/FormApi.ts:195](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L195)
