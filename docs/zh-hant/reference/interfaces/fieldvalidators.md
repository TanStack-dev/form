---
source-updated-at: '2025-04-20T06:54:03.000Z'
translation-updated-at: '2025-04-25T20:52:57.035Z'
id: FieldValidators
title: 介面 / FieldValidators
---
# 介面: FieldValidators\<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync\>

定義於: [packages/form-core/src/FieldApi.ts:272](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L272)

## 型別參數

• **TParentData**

• **TName** *extends* [`DeepKeys`](../type-aliases/deepkeys.md)\<`TParentData`\>

• **TData** *extends* [`DeepValue`](../type-aliases/deepvalue.md)\<`TParentData`, `TName`\>

• **TOnMount** *extends* `undefined` \| `FieldValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnChange** *extends* `undefined` \| `FieldValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnChangeAsync** *extends* `undefined` \| `FieldAsyncValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnBlur** *extends* `undefined` \| `FieldValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnBlurAsync** *extends* `undefined` \| `FieldAsyncValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnSubmit** *extends* `undefined` \| `FieldValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnSubmitAsync** *extends* `undefined` \| `FieldAsyncValidateOrFn`\<`TParentData`, `TName`, `TData`\>

## 屬性

### onBlur?

```ts
optional onBlur: TOnBlur;
```

定義於: [packages/form-core/src/FieldApi.ts:321](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L321)

一個可選函式，會在輸入框的 blur 事件時執行。

#### 範例

```ts
z.string().min(1)
```

***

### onBlurAsync?

```ts
optional onBlurAsync: TOnBlurAsync;
```

定義於: [packages/form-core/src/FieldApi.ts:327](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L327)

一個可選屬性，類似 `onBlur` 但為非同步驗證。

#### 範例

```ts
z.string().refine(async (val) => val.length > 3, { message: 'Testing 123' })
```

***

### onBlurAsyncDebounceMs?

```ts
optional onBlurAsyncDebounceMs: number;
```

定義於: [packages/form-core/src/FieldApi.ts:334](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L334)

一個可選數字，表示 `onBlurAsync` 在執行前應等待的時間

如果設為大於 0 的數字，將會以毫秒為單位對非同步驗證事件進行防抖處理

***

### onBlurListenTo?

```ts
optional onBlurListenTo: DeepKeys<TParentData>[];
```

定義於: [packages/form-core/src/FieldApi.ts:338](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L338)

一個可選欄位名稱列表，當這些欄位的值變更時，會觸發此欄位的 `onBlur` 和 `onBlurAsync` 事件

***

### onChange?

```ts
optional onChange: TOnChange;
```

定義於: [packages/form-core/src/FieldApi.ts:299](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L299)

一個可選函式，會在輸入框的 change 事件時執行。

#### 範例

```ts
z.string().min(1)
```

***

### onChangeAsync?

```ts
optional onChangeAsync: TOnChangeAsync;
```

定義於: [packages/form-core/src/FieldApi.ts:305](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L305)

一個可選屬性，類似 `onChange` 但為非同步驗證

#### 範例

```ts
z.string().refine(async (val) => val.length > 3, { message: 'Testing 123' })
```

***

### onChangeAsyncDebounceMs?

```ts
optional onChangeAsyncDebounceMs: number;
```

定義於: [packages/form-core/src/FieldApi.ts:311](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L311)

一個可選數字，表示 `onChangeAsync` 在執行前應等待的時間

如果設為大於 0 的數字，將會以毫秒為單位對非同步驗證事件進行防抖處理

***

### onChangeListenTo?

```ts
optional onChangeListenTo: DeepKeys<TParentData>[];
```

定義於: [packages/form-core/src/FieldApi.ts:315](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L315)

一個可選欄位名稱列表，當這些欄位的值變更時，會觸發此欄位的 `onChange` 和 `onChangeAsync` 事件

***

### onMount?

```ts
optional onMount: TOnMount;
```

定義於: [packages/form-core/src/FieldApi.ts:293](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L293)

一個可選函式，會在輸入框的 mount 事件時執行。

***

### onSubmit?

```ts
optional onSubmit: TOnSubmit;
```

定義於: [packages/form-core/src/FieldApi.ts:344](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L344)

一個可選函式，會在表單的 submit 事件時執行。

#### 範例

```ts
z.string().min(1)
```

***

### onSubmitAsync?

```ts
optional onSubmitAsync: TOnSubmitAsync;
```

定義於: [packages/form-core/src/FieldApi.ts:350](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L350)

一個可選屬性，類似 `onSubmit` 但為非同步驗證。

#### 範例

```ts
z.string().refine(async (val) => val.length > 3, { message: 'Testing 123' })
```
