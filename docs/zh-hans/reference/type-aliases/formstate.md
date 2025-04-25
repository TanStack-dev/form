---
source-updated-at: '2025-03-27T10:36:50.000Z'
translation-updated-at: '2025-04-12T04:12:40.775Z'
id: FormState
title: 类型 / FormState
---
# 类型别名：FormState\<TFormData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TOnServer\>

```ts
type FormState<TFormData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TOnServer> = BaseFormState<TFormData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TOnServer> & DerivedFormState<TFormData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TOnServer>;
```

定义于：[packages/form-core/src/FormApi.ts:540](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L540)

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
