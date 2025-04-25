---
source-updated-at: '2025-04-20T06:54:03.000Z'
translation-updated-at: '2025-04-25T20:52:22.471Z'
id: FieldOptions
title: 介面 / FieldOptions
---
# 介面: FieldOptions\<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync\>

定義於: [packages/form-core/src/FieldApi.ts:369](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L369)

表示表單欄位選項的物件類型。

## 擴展自

- [`FieldApiOptions`](fieldapioptions.md)

## 型別參數

• **TParentData**

• **TName** *繼承自* [`DeepKeys`](../type-aliases/deepkeys.md)\<`TParentData`\>

• **TData** *繼承自* [`DeepValue`](../type-aliases/deepvalue.md)\<`TParentData`, `TName`\>

• **TOnMount** *繼承自* `undefined` \| `FieldValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnChange** *繼承自* `undefined` \| `FieldValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnChangeAsync** *繼承自* `undefined` \| `FieldAsyncValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnBlur** *繼承自* `undefined` \| `FieldValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnBlurAsync** *繼承自* `undefined` \| `FieldAsyncValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnSubmit** *繼承自* `undefined` \| `FieldValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnSubmitAsync** *繼承自* `undefined` \| `FieldAsyncValidateOrFn`\<`TParentData`, `TName`, `TData`\>

## 屬性

### asyncAlways?

```ts
optional asyncAlways: boolean;
```

定義於: [packages/form-core/src/FieldApi.ts:402](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L402)

若為 `true`，則總是執行非同步驗證，即使在同步驗證期間已產生錯誤。

***

### asyncDebounceMs?

```ts
optional asyncDebounceMs: number;
```

定義於: [packages/form-core/src/FieldApi.ts:398](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L398)

若未傳入更具體的防抖時間，則此為非同步驗證的預設防抖時間。

***

### defaultMeta?

```ts
optional defaultMeta: Partial<FieldMeta<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, any, any, any, any, any, any, any>>;
```

定義於: [packages/form-core/src/FieldApi.ts:421](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L421)

用於欄位的預設元資料選用物件。

***

### defaultValue?

```ts
optional defaultValue: NoInfer<TData>;
```

定義於: [packages/form-core/src/FieldApi.ts:394](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L394)

欄位的選用預設值。

***

### disableErrorFlat?

```ts
optional disableErrorFlat: boolean;
```

定義於: [packages/form-core/src/FieldApi.ts:449](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L449)

停用 `field.errors` 上的 `flat(1)` 操作。這在您希望保持錯誤結構不變時很有用。不建議用於大多數情況。

***

### listeners?

```ts
optional listeners: FieldListeners<TParentData, TName, TData>;
```

定義於: [packages/form-core/src/FieldApi.ts:445](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L445)

附加到相應事件的監聽器列表

***

### name

```ts
name: TName;
```

定義於: [packages/form-core/src/FieldApi.ts:390](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L390)

欄位名稱。型別為 `DeepKeys<TParentData>` 以確保您的名稱是父資料集的深層鍵。

***

### validators?

```ts
optional validators: FieldValidators<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync>;
```

定義於: [packages/form-core/src/FieldApi.ts:406](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L406)

傳遞給欄位的驗證器列表
