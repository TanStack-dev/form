---
source-updated-at: '2025-02-25T06:59:48.000Z'
translation-updated-at: '2025-04-25T20:53:54.275Z'
id: useForm
title: 函式 / useForm
---
# 函式: useForm()

```ts
function useForm<TFormData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TOnServer, TSubmitMeta>(opts?): ReactFormExtendedApi<TFormData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TOnServer, TSubmitMeta>
```

定義於: [packages/react-form/src/useForm.tsx:142](https://github.com/TanStack/form/blob/main/packages/react-form/src/useForm.tsx#L142)

一個自訂的 React Hook，回傳 `FormApi` 類別的擴充實例。

此 API 封裝了所有與表單相關的必要功能。它讓您可以管理表單狀態、處理提交以及與表單欄位互動。

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

• **TSubmitMeta**

## 參數

### opts?

`FormOptions`\<`TFormData`, `TOnMount`, `TOnChange`, `TOnChangeAsync`, `TOnBlur`, `TOnBlurAsync`, `TOnSubmit`, `TOnSubmitAsync`, `TOnServer`, `TSubmitMeta`\>

## 回傳值

[`ReactFormExtendedApi`](../type-aliases/reactformextendedapi.md)\<`TFormData`, `TOnMount`, `TOnChange`, `TOnChangeAsync`, `TOnBlur`, `TOnBlurAsync`, `TOnSubmit`, `TOnSubmitAsync`, `TOnServer`, `TSubmitMeta`\>
