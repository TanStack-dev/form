---
source-updated-at: '2025-03-01T06:06:46.000Z'
translation-updated-at: '2025-04-12T04:11:32.565Z'
id: mergeForm
title: 函数 / mergeForm
---
# 函数: mergeForm()

```ts
function mergeForm<TFormData>(baseForm, state): FormApi<NoInfer<TFormData>, any, any, any, any, any, any, any, any, any>
```

定义于: [packages/form-core/src/mergeForm.ts:73](https://github.com/TanStack/form/blob/main/packages/form-core/src/mergeForm.ts#L73)

## 类型参数

• **TFormData**

## 参数

### baseForm

[`FormApi`](../classes/formapi.md)\<`NoInfer`\<`TFormData`\>, `any`, `any`, `any`, `any`, `any`, `any`, `any`, `any`, `any`\>

### state

`Partial`\<[`FormState`](../type-aliases/formstate.md)\<`TFormData`, `any`, `any`, `any`, `any`, `any`, `any`, `any`, `any`\>\>

## 返回值

[`FormApi`](../classes/formapi.md)\<`NoInfer`\<`TFormData`\>, `any`, `any`, `any`, `any`, `any`, `any`, `any`, `any`, `any`\>
