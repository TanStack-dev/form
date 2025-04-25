---
source-updated-at: '2025-03-01T06:06:46.000Z'
translation-updated-at: '2025-04-25T20:56:31.560Z'
id: TanStackField
title: 類別 / TanStackField
---
# 類別: TanStackField\<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync, TFormOnServer, TSubmitMeta\>

定義於: [tanstack-field.directive.ts:31](https://github.com/TanStack/form/blob/main/packages/angular-form/src/tanstack-field.directive.ts#L31)

## 型別參數

• **TParentData**

• **TName** *繼承自* `DeepKeys`\<`TParentData`\>

• **TData** *繼承自* `DeepValue`\<`TParentData`, `TName`\>

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

• **TFormOnServer** *繼承自* `undefined` \| `FormAsyncValidateOrFn`\<`TParentData`\>

• **TSubmitMeta**

## 實作

- `OnInit`
- `OnChanges`
- `OnDestroy`
- `FieldOptions`\<`TParentData`, `TName`, `TData`, `TOnMount`, `TOnChange`, `TOnChangeAsync`, `TOnBlur`, `TOnBlurAsync`, `TOnSubmit`, `TOnSubmitAsync`\>

## 建構函式

### new TanStackField()

```ts
new TanStackField<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync, TFormOnServer, TSubmitMeta>(): TanStackField<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync, TFormOnServer, TSubmitMeta>
```

#### 回傳

[`TanStackField`](tanstackfield.md)\<`TParentData`, `TName`, `TData`, `TOnMount`, `TOnChange`, `TOnChangeAsync`, `TOnBlur`, `TOnBlurAsync`, `TOnSubmit`, `TOnSubmitAsync`, `TFormOnMount`, `TFormOnChange`, `TFormOnChangeAsync`, `TFormOnBlur`, `TFormOnBlurAsync`, `TFormOnSubmit`, `TFormOnSubmitAsync`, `TFormOnServer`, `TSubmitMeta`\>

## 屬性

### api

```ts
api: FieldApi<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync, TFormOnServer, TSubmitMeta>;
```

定義於: [tanstack-field.directive.ts:129](https://github.com/TanStack/form/blob/main/packages/angular-form/src/tanstack-field.directive.ts#L129)

***

### asyncAlways?

```ts
optional asyncAlways: boolean;
```

定義於: [tanstack-field.directive.ts:78](https://github.com/TanStack/form/blob/main/packages/angular-form/src/tanstack-field.directive.ts#L78)

若為 `true`，即使同步驗證期間出現錯誤，仍會執行非同步驗證。

#### 實作於

```ts
FieldOptions.asyncAlways
```

***

### asyncDebounceMs?

```ts
optional asyncDebounceMs: number;
```

定義於: [tanstack-field.directive.ts:77](https://github.com/TanStack/form/blob/main/packages/angular-form/src/tanstack-field.directive.ts#L77)

若未傳入更特定的防抖時間，則為非同步驗證的預設防抖時間。

#### 實作於

```ts
FieldOptions.asyncDebounceMs
```

***

### defaultMeta?

```ts
optional defaultMeta: Partial<FieldMeta<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync>>;
```

定義於: [tanstack-field.directive.ts:106](https://github.com/TanStack/form/blob/main/packages/angular-form/src/tanstack-field.directive.ts#L106)

欄位的預設中繼資料物件（可選）。

#### 實作於

```ts
FieldOptions.defaultMeta
```

***

### defaultValue?

```ts
optional defaultValue: NoInfer<TData>;
```

定義於: [tanstack-field.directive.ts:76](https://github.com/TanStack/form/blob/main/packages/angular-form/src/tanstack-field.directive.ts#L76)

欄位的預設值（可選）。

#### 實作於

```ts
FieldOptions.defaultValue
```

***

### disableErrorFlat?

```ts
optional disableErrorFlat: boolean;
```

定義於: [tanstack-field.directive.ts:127](https://github.com/TanStack/form/blob/main/packages/angular-form/src/tanstack-field.directive.ts#L127)

停用 `field.errors` 上的 `flat(1)` 操作。這在您希望保持錯誤結構不變時很有用。不建議大多數使用情境使用。

#### 實作於

```ts
FieldOptions.disableErrorFlat
```

***

### listeners?

```ts
optional listeners: NoInfer<FieldListeners<TParentData, TName, TData>>;
```

定義於: [tanstack-field.directive.ts:105](https://github.com/TanStack/form/blob/main/packages/angular-form/src/tanstack-field.directive.ts#L105)

附加到相應事件的監聽器列表

#### 實作於

```ts
FieldOptions.listeners
```

***

### name

```ts
name: TName;
```

定義於: [tanstack-field.directive.ts:75](https://github.com/TanStack/form/blob/main/packages/angular-form/src/tanstack-field.directive.ts#L75)

欄位名稱。型別為 `DeepKeys<TParentData>`，以確保您的名稱是父資料集的深層鍵。

#### 實作於

```ts
FieldOptions.name
```

***

### tanstackField

```ts
tanstackField: FormApi<TParentData, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync, TFormOnServer, TSubmitMeta>;
```

定義於: [tanstack-field.directive.ts:79](https://github.com/TanStack/form/blob/main/packages/angular-form/src/tanstack-field.directive.ts#L79)

***

### unmount()?

```ts
optional unmount: () => void;
```

定義於: [tanstack-field.directive.ts:185](https://github.com/TanStack/form/blob/main/packages/angular-form/src/tanstack-field.directive.ts#L185)

#### 回傳

`void`

***

### validators?

```ts
optional validators: NoInfer<FieldValidators<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync>>;
```

定義於: [tanstack-field.directive.ts:91](https://github.com/TanStack/form/blob/main/packages/angular-form/src/tanstack-field.directive.ts#L91)

傳遞給欄位的驗證器列表

#### 實作於

```ts
FieldOptions.validators
```

## 方法

### ngOnChanges()

```ts
ngOnChanges(): void
```

定義於: [tanstack-field.directive.ts:197](https://github.com/TanStack/form/blob/main/packages/angular-form/src/tanstack-field.directive.ts#L197)

一個回呼方法，在預設變更檢測器檢查資料綁定屬性（若至少有一個已變更）後立即呼叫，並在檢查視圖和內容子項之前呼叫。

#### 回傳

`void`

#### 實作於

```ts
OnChanges.ngOnChanges
```

***

### ngOnDestroy()

```ts
ngOnDestroy(): void
```

定義於: [tanstack-field.directive.ts:193](https://github.com/TanStack/form/blob/main/packages/angular-form/src/tanstack-field.directive.ts#L193)

一個執行自訂清理的回呼方法，在指令、管道或服務實例被銷毀前立即呼叫。

#### 回傳

`void`

#### 實作於

```ts
OnDestroy.ngOnDestroy
```

***

### ngOnInit()

```ts
ngOnInit(): void
```

定義於: [tanstack-field.directive.ts:187](https://github.com/TanStack/form/blob/main/packages/angular-form/src/tanstack-field.directive.ts#L187)

一個回呼方法，在預設變更檢測器首次檢查指令的資料綁定屬性後立即呼叫，並在檢查任何視圖或內容子項之前呼叫。僅在指令實例化時呼叫一次。

#### 回傳

`void`

#### 實作於

```ts
OnInit.ngOnInit
```
