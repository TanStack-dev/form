---
source-updated-at: '2025-04-20T06:54:03.000Z'
translation-updated-at: '2025-04-25T20:50:09.855Z'
id: FieldApiOptions
title: 介面 / FieldApiOptions
---
<!-- 請勿編輯：此頁面由類型註解自動生成 -->

# 介面: FieldApiOptions\<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync, TFormOnServer, TParentSubmitMeta\>

定義於: [packages/form-core/src/FieldApi.ts:455](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L455)

表示 FieldApi 類別所需選項的物件類型。

## 擴展

- [`FieldOptions`](fieldoptions.md)\<`TParentData`, `TName`, `TData`, `TOnMount`, `TOnChange`, `TOnChangeAsync`, `TOnBlur`, `TOnBlurAsync`, `TOnSubmit`, `TOnSubmitAsync`\>

## 類型參數

• **TParentData**

• **TName** *繼承* [`DeepKeys`](../type-aliases/deepkeys.md)\<`TParentData`\>

• **TData** *繼承* [`DeepValue`](../type-aliases/deepvalue.md)\<`TParentData`, `TName`\>

• **TOnMount** *繼承* `undefined` \| `FieldValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnChange** *繼承* `undefined` \| `FieldValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnChangeAsync** *繼承* `undefined` \| `FieldAsyncValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnBlur** *繼承* `undefined` \| `FieldValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnBlurAsync** *繼承* `undefined` \| `FieldAsyncValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnSubmit** *繼承* `undefined` \| `FieldValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnSubmitAsync** *繼承* `undefined` \| `FieldAsyncValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TFormOnMount** *繼承* `undefined` \| `FormValidateOrFn`\<`TParentData`\>

• **TFormOnChange** *繼承* `undefined` \| `FormValidateOrFn`\<`TParentData`\>

• **TFormOnChangeAsync** *繼承* `undefined` \| `FormAsyncValidateOrFn`\<`TParentData`\>

• **TFormOnBlur** *繼承* `undefined` \| `FormValidateOrFn`\<`TParentData`\>

• **TFormOnBlurAsync** *繼承* `undefined` \| `FormAsyncValidateOrFn`\<`TParentData`\>

• **TFormOnSubmit** *繼承* `undefined` \| `FormValidateOrFn`\<`TParentData`\>

• **TFormOnSubmitAsync** *繼承* `undefined` \| `FormAsyncValidateOrFn`\<`TParentData`\>

• **TFormOnServer** *繼承* `undefined` \| `FormAsyncValidateOrFn`\<`TParentData`\>

• **TParentSubmitMeta**

## 屬性

### asyncAlways?

```ts
optional asyncAlways: boolean;
```

定義於: [packages/form-core/src/FieldApi.ts:402](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L402)

若為 `true`，即使同步驗證過程中出現錯誤，仍會執行非同步驗證。

#### 繼承自

[`FieldOptions`](fieldoptions.md).[`asyncAlways`](FieldOptions.md#asyncalways)

***

### asyncDebounceMs?

```ts
optional asyncDebounceMs: number;
```

定義於: [packages/form-core/src/FieldApi.ts:398](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L398)

若未傳入更特定的防抖時間，則為非同步驗證的預設防抖時間。

#### 繼承自

[`FieldOptions`](fieldoptions.md).[`asyncDebounceMs`](FieldOptions.md#asyncdebouncems)

***

### defaultMeta?

```ts
optional defaultMeta: Partial<FieldMeta<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, any, any, any, any, any, any, any>>;
```

定義於: [packages/form-core/src/FieldApi.ts:421](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L421)

欄位的預設元資料 (metadata) 物件，可選。

#### 繼承自

[`FieldOptions`](fieldoptions.md).[`defaultMeta`](FieldOptions.md#defaultmeta)

***

### defaultValue?

```ts
optional defaultValue: NoInfer<TData>;
```

定義於: [packages/form-core/src/FieldApi.ts:394](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L394)

欄位的預設值，可選。

#### 繼承自

[`FieldOptions`](fieldoptions.md).[`defaultValue`](FieldOptions.md#defaultvalue)

***

### disableErrorFlat?

```ts
optional disableErrorFlat: boolean;
```

定義於: [packages/form-core/src/FieldApi.ts:449](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L449)

禁用 `field.errors` 上的 `flat(1)` 操作。這適用於您希望保持錯誤結構不變的情況。不建議用於大多數使用場景。

#### 繼承自

[`FieldOptions`](fieldoptions.md).[`disableErrorFlat`](FieldOptions.md#disableerrorflat)

***

### form

```ts
form: FormApi<TParentData, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync, TFormOnServer, TParentSubmitMeta>;
```

定義於: [packages/form-core/src/FieldApi.ts:507](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L507)

***

### listeners?

```ts
optional listeners: FieldListeners<TParentData, TName, TData>;
```

定義於: [packages/form-core/src/FieldApi.ts:445](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L445)

附加到相應事件的監聽器列表

#### 繼承自

[`FieldOptions`](fieldoptions.md).[`listeners`](FieldOptions.md#listeners)

***

### name

```ts
name: TName;
```

定義於: [packages/form-core/src/FieldApi.ts:390](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L390)

欄位名稱。類型為 `DeepKeys<TParentData>`，以確保您的名稱是父資料集的深層鍵名。

#### 繼承自

[`FieldOptions`](fieldoptions.md).[`name`](FieldOptions.md#name)

***

### validators?

```ts
optional validators: FieldValidators<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync>;
```

定義於: [packages/form-core/src/FieldApi.ts:406](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L406)

傳遞給欄位的驗證器列表

#### 繼承自

[`FieldOptions`](fieldoptions.md).[`validators`](FieldOptions.md#validators)
