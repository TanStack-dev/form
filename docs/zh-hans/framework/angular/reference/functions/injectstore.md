---
source-updated-at: '2025-02-25T06:59:48.000Z'
translation-updated-at: '2025-04-12T04:14:24.688Z'
id: injectStore
title: 函数 / injectStore
---
<!-- 请勿编辑：此页面根据类型注释自动生成 -->

# 函数: injectStore()

```ts
function injectStore<TFormData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TOnServer, TSubmitMeta, TSelected>(form, selector?): Signal<TSelected>
```

定义于: [inject-store.ts:9](https://github.com/TanStack/form/blob/main/packages/angular-form/src/inject-store.ts#L9)

## 类型参数

• **TFormData**

• **TOnMount** *继承自* `undefined` \| `FormValidateOrFn`\<`TFormData`\>

• **TOnChange** *继承自* `undefined` \| `FormValidateOrFn`\<`TFormData`\>

• **TOnChangeAsync** *继承自* `undefined` \| `FormAsyncValidateOrFn`\<`TFormData`\>

• **TOnBlur** *继承自* `undefined` \| `FormValidateOrFn`\<`TFormData`\>

• **TOnBlurAsync** *继承自* `undefined` \| `FormAsyncValidateOrFn`\<`TFormData`\>

• **TOnSubmit** *继承自* `undefined` \| `FormValidateOrFn`\<`TFormData`\>

• **TOnSubmitAsync** *继承自* `undefined` \| `FormAsyncValidateOrFn`\<`TFormData`\>

• **TOnServer** *继承自* `undefined` \| `FormAsyncValidateOrFn`\<`TFormData`\>

• **TSubmitMeta**

• **TSelected** = `NoInfer`\<`FormState`\<`TFormData`, `TOnMount`, `TOnChange`, `TOnChangeAsync`, `TOnBlur`, `TOnBlurAsync`, `TOnSubmit`, `TOnSubmitAsync`, `TOnServer`\>\>

## 参数

### form

`FormApi`\<`TFormData`, `TOnMount`, `TOnChange`, `TOnChangeAsync`, `TOnBlur`, `TOnBlurAsync`, `TOnSubmit`, `TOnSubmitAsync`, `TOnServer`, `TSubmitMeta`\>

### selector?

(`state`) => `TSelected`

## 返回值

`Signal`\<`TSelected`\>
