---
source-updated-at: '2025-02-25T06:59:48.000Z'
translation-updated-at: '2025-04-25T20:55:08.950Z'
id: injectStore
title: 函式 / injectStore
---
<!-- 請勿編輯：此頁面由類型註解自動生成 -->

# 函式: injectStore()

```ts
function injectStore<TFormData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TOnServer, TSubmitMeta, TSelected>(form, selector?): Signal<TSelected>
```

定義於: [inject-store.ts:9](https://github.com/TanStack/form/blob/main/packages/angular-form/src/inject-store.ts#L9)

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

• **TSubmitMeta**

• **TSelected** = `NoInfer`\<`FormState`\<`TFormData`, `TOnMount`, `TOnChange`, `TOnChangeAsync`, `TOnBlur`, `TOnBlurAsync`, `TOnSubmit`, `TOnSubmitAsync`, `TOnServer`\>\>

## 參數

### form

`FormApi`\<`TFormData`, `TOnMount`, `TOnChange`, `TOnChangeAsync`, `TOnBlur`, `TOnBlurAsync`, `TOnSubmit`, `TOnSubmitAsync`, `TOnServer`, `TSubmitMeta`\>

### selector?

(`state`) => `TSelected`

## 回傳值

`Signal`\<`TSelected`\>
