---
source-updated-at: '2025-04-25T12:58:29.000Z'
translation-updated-at: '2025-04-25T20:52:58.829Z'
id: FormOptions
title: 介面 / FormOptions
---
# 介面: FormOptions\<TFormData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TOnServer, TSubmitMeta\>

定義於: [packages/form-core/src/FormApi.ts:319](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L319)

代表表單選項的物件。

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

• **TSubmitMeta** = `never`

## 屬性

### asyncAlways?

```ts
optional asyncAlways: boolean;
```

定義於: [packages/form-core/src/FormApi.ts:354](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L354)

若為 true，即使同步驗證已產生錯誤，仍會執行非同步驗證。預設為 undefined。

***

### asyncDebounceMs?

```ts
optional asyncDebounceMs: number;
```

定義於: [packages/form-core/src/FormApi.ts:358](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L358)

可選的毫秒數，用於在觸發非同步動作前引入延遲。

***

### canSubmitWhenInvalid?

```ts
optional canSubmitWhenInvalid: boolean;
```

定義於: [packages/form-core/src/FormApi.ts:362](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L362)

若為 true，允許表單在無效狀態下提交，即 canSubmit 將保持為 true，無論驗證錯誤為何。預設為 undefined。

***

### defaultState?

```ts
optional defaultState: Partial<FormState<TFormData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TOnServer>>;
```

定義於: [packages/form-core/src/FormApi.ts:338](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L338)

表單的預設狀態。

***

### defaultValues?

```ts
optional defaultValues: TFormData;
```

定義於: [packages/form-core/src/FormApi.ts:334](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L334)

設定表單的初始值。

***

### listeners?

```ts
optional listeners: FormListeners<TFormData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TOnServer, TSubmitMeta>;
```

定義於: [packages/form-core/src/FormApi.ts:385](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L385)

表單層級的監聽器

***

### onSubmit()?

```ts
optional onSubmit: (props) => any;
```

定義於: [packages/form-core/src/FormApi.ts:401](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L401)

當表單提交時呼叫的函式，定義使用者提交有效表單後應執行的動作，回傳 `any` 或 Promise `Promise<any>`

#### 參數

##### props

###### formApi

[`FormApi`](../classes/formapi.md)\<`TFormData`, `TOnMount`, `TOnChange`, `TOnChangeAsync`, `TOnBlur`, `TOnBlurAsync`, `TOnSubmit`, `TOnSubmitAsync`, `TOnServer`, `TSubmitMeta`\>

###### meta

`TSubmitMeta`

###### value

`TFormData`

#### 回傳值

`any`

***

### onSubmitInvalid()?

```ts
optional onSubmitInvalid: (props) => void;
```

定義於: [packages/form-core/src/FormApi.ts:420](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L420)

指定當使用者嘗試提交無效表單時應執行的動作。

#### 參數

##### props

###### formApi

[`FormApi`](../classes/formapi.md)\<`TFormData`, `TOnMount`, `TOnChange`, `TOnChangeAsync`, `TOnBlur`, `TOnBlurAsync`, `TOnSubmit`, `TOnSubmitAsync`, `TOnServer`, `TSubmitMeta`\>

###### value

`TFormData`

#### 回傳值

`void`

***

### onSubmitMeta?

```ts
optional onSubmitMeta: TSubmitMeta;
```

定義於: [packages/form-core/src/FormApi.ts:380](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L380)

onSubmitMeta，從 handleSubmit 處理函式傳遞給 onSubmit 函式 props 的資料

***

### transform?

```ts
optional transform: FormTransform<NoInfer<TFormData>, NoInfer<TOnMount>, NoInfer<TOnChange>, NoInfer<TOnChangeAsync>, NoInfer<TOnBlur>, NoInfer<TOnBlurAsync>, NoInfer<TOnSubmit>, NoInfer<TOnSubmitAsync>, NoInfer<TOnServer>, NoInfer<TSubmitMeta>>;
```

定義於: [packages/form-core/src/FormApi.ts:435](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L435)

***

### validators?

```ts
optional validators: FormValidators<TFormData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync>;
```

定義於: [packages/form-core/src/FormApi.ts:366](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L366)

傳遞給表單的驗證器列表
