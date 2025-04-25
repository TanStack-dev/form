---
source-updated-at: '2025-03-01T06:06:46.000Z'
translation-updated-at: '2025-04-12T04:13:58.598Z'
id: createForm
title: 函数 / createForm
---
# 函数: createForm()

```ts
function createForm<TParentData, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync, TFormOnServer, TSubmitMeta>(opts?): FormApi<TParentData, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync, TFormOnServer, TSubmitMeta> & SolidFormApi<TParentData, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync, TFormOnServer, TSubmitMeta>
```

定义于: [packages/solid-form/src/createForm.tsx:115](https://github.com/TanStack/form/blob/main/packages/solid-form/src/createForm.tsx#L115)

## 类型参数

• **TParentData**

• **TFormOnMount** *继承自* `undefined` \| `FormValidateOrFn`\<`TParentData`\>

• **TFormOnChange** *继承自* `undefined` \| `FormValidateOrFn`\<`TParentData`\>

• **TFormOnChangeAsync** *继承自* `undefined` \| `FormAsyncValidateOrFn`\<`TParentData`\>

• **TFormOnBlur** *继承自* `undefined` \| `FormValidateOrFn`\<`TParentData`\>

• **TFormOnBlurAsync** *继承自* `undefined` \| `FormAsyncValidateOrFn`\<`TParentData`\>

• **TFormOnSubmit** *继承自* `undefined` \| `FormValidateOrFn`\<`TParentData`\>

• **TFormOnSubmitAsync** *继承自* `undefined` \| `FormAsyncValidateOrFn`\<`TParentData`\>

• **TFormOnServer** *继承自* `undefined` \| `FormAsyncValidateOrFn`\<`TParentData`\>

• **TSubmitMeta**

## 参数

### opts?

() => `FormOptions`\<`TParentData`, `TFormOnMount`, `TFormOnChange`, `TFormOnChangeAsync`, `TFormOnBlur`, `TFormOnBlurAsync`, `TFormOnSubmit`, `TFormOnSubmitAsync`, `TFormOnServer`, `TSubmitMeta`\>

## 返回值

`FormApi`\<`TParentData`, `TFormOnMount`, `TFormOnChange`, `TFormOnChangeAsync`, `TFormOnBlur`, `TFormOnBlurAsync`, `TFormOnSubmit`, `TFormOnSubmitAsync`, `TFormOnServer`, `TSubmitMeta`\> & [`SolidFormApi`](../interfaces/solidformapi.md)\<`TParentData`, `TFormOnMount`, `TFormOnChange`, `TFormOnChangeAsync`, `TFormOnBlur`, `TFormOnBlurAsync`, `TFormOnSubmit`, `TFormOnSubmitAsync`, `TFormOnServer`, `TSubmitMeta`\>
