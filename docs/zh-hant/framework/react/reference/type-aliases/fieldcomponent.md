---
source-updated-at: '2025-04-11T15:48:08.000Z'
translation-updated-at: '2025-04-25T20:54:09.782Z'
id: FieldComponent
title: 類型 / FieldComponent
---
<!-- 請勿編輯：此頁面由類型註解自動生成 -->

# 類型別名：FieldComponent()\<TParentData, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync, TFormOnServer, TPatentSubmitMeta, ExtendedApi\>

```ts
type FieldComponent<TParentData, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync, TFormOnServer, TPatentSubmitMeta, ExtendedApi> = <TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync>({
  children,
  ...fieldOptions
}) => ReactNode;
```

定義於：[packages/react-form/src/useField.tsx:363](https://github.com/TanStack/form/blob/main/packages/react-form/src/useField.tsx#L363)

一個表示特定表單資料類型的欄位元件類型別名。

## 類型參數

• **TParentData**

• **TFormOnMount** *繼承自* `undefined` \| `FormValidateOrFn`\<`TParentData`\>

• **TFormOnChange** *繼承自* `undefined` \| `FormValidateOrFn`\<`TParentData`\>

• **TFormOnChangeAsync** *繼承自* `undefined` \| `FormAsyncValidateOrFn`\<`TParentData`\>

• **TFormOnBlur** *繼承自* `undefined` \| `FormValidateOrFn`\<`TParentData`\>

• **TFormOnBlurAsync** *繼承自* `undefined` \| `FormAsyncValidateOrFn`\<`TParentData`\>

• **TFormOnSubmit** *繼承自* `undefined` \| `FormValidateOrFn`\<`TParentData`\>

• **TFormOnSubmitAsync** *繼承自* `undefined` \| `FormAsyncValidateOrFn`\<`TParentData`\>

• **TFormOnServer** *繼承自* `undefined` \| `FormAsyncValidateOrFn`\<`TParentData`\>

• **TPatentSubmitMeta**

• **ExtendedApi** = \{\}

## 類型參數

• **TName** *繼承自* `DeepKeys`\<`TParentData`\>

• **TData** *繼承自* `DeepValue`\<`TParentData`, `TName`\>

• **TOnMount** *繼承自* `undefined` \| `FieldValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnChange** *繼承自* `undefined` \| `FieldValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnChangeAsync** *繼承自* `undefined` \| `FieldAsyncValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnBlur** *繼承自* `undefined` \| `FieldValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnBlurAsync** *繼承自* `undefined` \| `FieldAsyncValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnSubmit** *繼承自* `undefined` \| `FieldValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnSubmitAsync** *繼承自* `undefined` \| `FieldAsyncValidateOrFn`\<`TParentData`, `TName`, `TData`\>

## 參數

### \{
  children,
  ...fieldOptions
\}

`FieldComponentBoundProps`\<`TParentData`, `TName`, `TData`, `TOnMount`, `TOnChange`, `TOnChangeAsync`, `TOnBlur`, `TOnBlurAsync`, `TOnSubmit`, `TOnSubmitAsync`, `TFormOnMount`, `TFormOnChange`, `TFormOnChangeAsync`, `TFormOnBlur`, `TFormOnBlurAsync`, `TFormOnSubmit`, `TFormOnSubmitAsync`, `TFormOnServer`, `TPatentSubmitMeta`, `ExtendedApi`\>

## 返回值

`ReactNode`
