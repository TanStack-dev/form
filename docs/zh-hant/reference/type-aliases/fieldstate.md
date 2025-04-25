---
source-updated-at: '2025-04-20T06:54:03.000Z'
translation-updated-at: '2025-04-25T20:52:34.687Z'
id: FieldState
title: 類型 / FieldState
---
# 類型別名: FieldState\<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync\>

```ts
type FieldState<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync> = object;
```

定義於: [packages/form-core/src/FieldApi.ts:766](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L766)

表示欄位狀態的物件類型。

## 類型參數

• **TParentData**

• **TName** *繼承自* [`DeepKeys`](deepkeys.md)\<`TParentData`\>

• **TData** *繼承自* [`DeepValue`](deepvalue.md)\<`TParentData`, `TName`\>

• **TOnMount** *繼承自* `undefined` \| `FieldValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnChange** *繼承自* `undefined` \| `FieldValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnChangeAsync** *繼承自* `undefined` \| `FieldAsyncValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnBlur** *繼承自* `undefined` \| `FieldValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnBlurAsync** *繼承自* `undefined` \| `FieldAsyncValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnSubmit** *繼承自* `undefined` \| `FieldValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnSubmitAsync** *繼承自* `undefined` \| `FieldAsyncValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TFormOnMount** *繼承自* `undefined` \| `FormValidateOrFn`\<`TParentData`\>

• **TFormOnChange** *繼承自* `undefined` \| `FormValidateOrFn`\<`TParentData`\>

• **TFormOnChangeAsync** *繼承自* `undefined` \| `FormAsyncValidateOrFn`\<`TParentData`\>

• **TFormOnBlur** *繼承自* `undefined` \| `FormValidateOrFn`\<`TParentData`\>

• **TFormOnBlurAsync** *繼承自* `undefined` \| `FormAsyncValidateOrFn`\<`TParentData`\>

• **TFormOnSubmit** *繼承自* `undefined` \| `FormValidateOrFn`\<`TParentData`\>

• **TFormOnSubmitAsync** *繼承自* `undefined` \| `FormAsyncValidateOrFn`\<`TParentData`\>

## 類型宣告

### meta

```ts
meta: FieldMeta<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync>;
```

欄位當前的元數據 (metadata)。

### value

```ts
value: TData;
```

欄位當前的值。
