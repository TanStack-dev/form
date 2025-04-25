---
source-updated-at: '2025-04-02T16:32:51.000Z'
translation-updated-at: '2025-04-25T20:49:09.473Z'
id: mergeForm
title: 函式 / mergeForm
---
<!-- 請勿編輯：此頁面由類型註解自動生成 -->

# 函式: mergeForm()

```ts
function mergeForm<TFormData>(baseForm, state): FormApi<NoInfer<TFormData>, any, any, any, any, any, any, any, any, any>
```

定義於: [packages/form-core/src/mergeForm.ts:73](https://github.com/TanStack/form/blob/main/packages/form-core/src/mergeForm.ts#L73)

## 型別參數

• **TFormData**

## 參數

### baseForm

[`FormApi`](../classes/formapi.md)\<`NoInfer`\<`TFormData`\>, `any`, `any`, `any`, `any`, `any`, `any`, `any`, `any`, `any`\>

### state

`Partial`\<[`FormState`](../interfaces/formstate.md)\<`TFormData`, `any`, `any`, `any`, `any`, `any`, `any`, `any`, `any`\>\>

## 回傳值

[`FormApi`](../classes/formapi.md)\<`NoInfer`\<`TFormData`\>, `any`, `any`, `any`, `any`, `any`, `any`, `any`, `any`, `any`\>
