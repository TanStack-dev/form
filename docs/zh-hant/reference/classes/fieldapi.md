---
source-updated-at: '2025-04-20T06:54:03.000Z'
translation-updated-at: '2025-04-25T20:51:32.206Z'
id: FieldApi
title: 類別 / FieldApi
---
# 類別: FieldApi\<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync, TFormOnServer, TParentSubmitMeta\>

定義於: [packages/form-core/src/FieldApi.ts:855](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L855)

一個用於管理表單欄位的 API 類別。

通常您不需要直接建立新的 `FieldApi` 實例。  
相反地，您會使用框架的鉤子/函數如 `useField` 或 `createField`  
來為您建立一個使用您框架反應式模型的新實例。  
然而，如果需要手動建立新實例，您可以通過呼叫  
`new FieldApi` 建構函式來實現。

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

• **TFormOnMount** *繼承自* `undefined` \| `FormValidateOrFn`\<`TParentData`\>

• **TFormOnChange** *繼承自* `undefined` \| `FormValidateOrFn`\<`TParentData`\>

• **TFormOnChangeAsync** *繼承自* `undefined` \| `FormAsyncValidateOrFn`\<`TParentData`\>

• **TFormOnBlur** *繼承自* `undefined` \| `FormValidateOrFn`\<`TParentData`\>

• **TFormOnBlurAsync** *繼承自* `undefined` \| `FormAsyncValidateOrFn`\<`TParentData`\>

• **TFormOnSubmit** *繼承自* `undefined` \| `FormValidateOrFn`\<`TParentData`\>

• **TFormOnSubmitAsync** *繼承自* `undefined` \| `FormAsyncValidateOrFn`\<`TParentData`\>

• **TFormOnServer** *繼承自* `undefined` \| `FormAsyncValidateOrFn`\<`TParentData`\>

• **TParentSubmitMeta**

## 建構函式

### new FieldApi()

```ts
new FieldApi<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync, TFormOnServer, TParentSubmitMeta>(opts): FieldApi<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync, TFormOnServer, TParentSubmitMeta>
```

定義於: [packages/form-core/src/FieldApi.ts:986](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L986)

初始化一個新的 `FieldApi` 實例。

#### 參數

##### opts

[`FieldApiOptions`](../interfaces/fieldapioptions.md)\<`TParentData`, `TName`, `TData`, `TOnMount`, `TOnChange`, `TOnChangeAsync`, `TOnBlur`, `TOnBlurAsync`, `TOnSubmit`, `TOnSubmitAsync`, `TFormOnMount`, `TFormOnChange`, `TFormOnChangeAsync`, `TFormOnBlur`, `TFormOnBlurAsync`, `TFormOnSubmit`, `TFormOnSubmitAsync`, `TFormOnServer`, `TParentSubmitMeta`\>

#### 回傳

[`FieldApi`](fieldapi.md)\<`TParentData`, `TName`, `TData`, `TOnMount`, `TOnChange`, `TOnChangeAsync`, `TOnBlur`, `TOnBlurAsync`, `TOnSubmit`, `TOnSubmitAsync`, `TFormOnMount`, `TFormOnChange`, `TFormOnChangeAsync`, `TFormOnBlur`, `TFormOnBlurAsync`, `TFormOnSubmit`, `TFormOnSubmitAsync`, `TFormOnServer`, `TParentSubmitMeta`\>

## 屬性

### form

```ts
form: FormApi<TParentData, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync, TFormOnServer, TParentSubmitMeta>;
```

定義於: [packages/form-core/src/FieldApi.ts:899](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L899)

對表單 API 實例的參考。

***

### name

```ts
name: DeepKeys<TParentData>;
```

定義於: [packages/form-core/src/FieldApi.ts:923](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L923)

欄位名稱。

***

### options

```ts
options: FieldApiOptions<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync, TFormOnServer, TParentSubmitMeta>;
```

定義於: [packages/form-core/src/FieldApi.ts:927](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L927)

欄位選項。

***

### store

```ts
store: Derived<FieldState<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync>>;
```

定義於: [packages/form-core/src/FieldApi.ts:951](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L951)

欄位狀態儲存。

***

### timeoutIds

```ts
timeoutIds: object;
```

定義於: [packages/form-core/src/FieldApi.ts:978](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L978)

#### listeners

```ts
listeners: Record<ListenerCause, null | Timeout>;
```

#### validations

```ts
validations: Record<ValidationCause, null | Timeout>;
```

## 存取器

### state

#### Get 簽名

```ts
get state(): FieldState<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync>
```

定義於: [packages/form-core/src/FieldApi.ts:975](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L975)

當前欄位狀態。

##### 回傳

[`FieldState`](../type-aliases/fieldstate.md)\<`TParentData`, `TName`, `TData`, `TOnMount`, `TOnChange`, `TOnChangeAsync`, `TOnBlur`, `TOnBlurAsync`, `TOnSubmit`, `TOnSubmitAsync`, `TFormOnMount`, `TFormOnChange`, `TFormOnChangeAsync`, `TFormOnBlur`, `TFormOnBlurAsync`, `TFormOnSubmit`, `TFormOnSubmitAsync`\>

## 方法

### getInfo()

```ts
getInfo(): FieldInfo<TParentData>
```

定義於: [packages/form-core/src/FieldApi.ts:1239](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L1239)

取得欄位資訊物件。

#### 回傳

[`FieldInfo`](../type-aliases/fieldinfo.md)\<`TParentData`\>

***

### getMeta()

```ts
getMeta(): FieldMeta<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync>
```

定義於: [packages/form-core/src/FieldApi.ts:1207](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L1207)

#### 回傳

[`FieldMeta`](../type-aliases/fieldmeta.md)\<`TParentData`, `TName`, `TData`, `TOnMount`, `TOnChange`, `TOnChangeAsync`, `TOnBlur`, `TOnBlurAsync`, `TOnSubmit`, `TOnSubmitAsync`, `TFormOnMount`, `TFormOnChange`, `TFormOnChangeAsync`, `TFormOnBlur`, `TFormOnBlurAsync`, `TFormOnSubmit`, `TFormOnSubmitAsync`\>

***

### ~~getValue()~~

```ts
getValue(): TData
```

定義於: [packages/form-core/src/FieldApi.ts:1192](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L1192)

取得當前欄位值。

#### 回傳

`TData`

#### 已棄用

請改用 `field.state.value`。

***

### handleBlur()

```ts
handleBlur(): void
```

定義於: [packages/form-core/src/FieldApi.ts:1651](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L1651)

處理 blur 事件。

#### 回傳

`void`

***

### handleChange()

```ts
handleChange(updater): void
```

定義於: [packages/form-core/src/FieldApi.ts:1644](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L1644)

處理 change 事件。

#### 參數

##### updater

[`Updater`](../type-aliases/updater.md)\<`TData`\>

#### 回傳

`void`

***

### insertValue()

```ts
insertValue(
   index, 
   value, 
   opts?): void
```

定義於: [packages/form-core/src/FieldApi.ts:1256](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L1256)

在指定索引處插入值，並將後續值向右移動。

#### 參數

##### index

`number`

##### value

`TData` *繼承自* `any`[] ? `TData`\<`TData`\>\[`number`\] : `never`

##### opts?

`UpdateMetaOptions`

#### 回傳

`void`

***

### mount()

```ts
mount(): () => void
```

定義於: [packages/form-core/src/FieldApi.ts:1082](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L1082)

將欄位實例掛載到表單。

#### 回傳

`Function`

##### 回傳

`void`

***

### moveValue()

```ts
moveValue(
   aIndex, 
   bIndex, 
   opts?): void
```

定義於: [packages/form-core/src/FieldApi.ts:1300](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L1300)

將第一個指定索引處的值移動到第二個指定索引處。

#### 參數

##### aIndex

`number`

##### bIndex

`number`

##### opts?

`UpdateMetaOptions`

#### 回傳

`void`

***

### parseValueWithSchema()

```ts
parseValueWithSchema(schema): 
  | undefined
  | StandardSchemaV1Issue[]
```

定義於: [packages/form-core/src/FieldApi.ts:1686](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L1686)

使用給定的 schema 解析欄位值並回傳  
問題（如果有的話）。此方法不會設定任何內部錯誤。

#### 參數

##### schema

[`StandardSchemaV1`](../type-aliases/standardschemav1.md)\<`TData`, `unknown`\>

用於解析此欄位值的標準 schema。

#### 回傳

  \| `undefined`
  \| [`StandardSchemaV1Issue`](../interfaces/standardschemav1issue.md)[]

***

### parseValueWithSchemaAsync()

```ts
parseValueWithSchemaAsync(schema): Promise<
  | undefined
| StandardSchemaV1Issue[]>
```

定義於: [packages/form-core/src/FieldApi.ts:1698](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L1698)

使用給定的 schema 解析欄位值並回傳  
問題（如果有的話）。此方法不會設定任何內部錯誤。

#### 參數

##### schema

[`StandardSchemaV1`](../type-aliases/standardschemav1.md)\<`TData`, `unknown`\>

用於解析此欄位值的標準 schema。

#### 回傳

`Promise`\<
  \| `undefined`
  \| [`StandardSchemaV1Issue`](../interfaces/standardschemav1issue.md)[]\>

***

### pushValue()

```ts
pushValue(value, opts?): void
```

定義於: [packages/form-core/src/FieldApi.ts:1244](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L1244)

將新值推入欄位。

#### 參數

##### value

`TData` *繼承自* `any`[] ? `TData`\<`TData`\>\[`number`\] : `never`

##### opts?

`UpdateMetaOptions`

#### 回傳

`void`

***

### removeValue()

```ts
removeValue(index, opts?): void
```

定義於: [packages/form-core/src/FieldApi.ts:1282](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L1282)

移除指定索引處的值。

#### 參數

##### index

`number`

##### opts?

`UpdateMetaOptions`

#### 回傳

`void`

***

### replaceValue()

```ts
replaceValue(
   index, 
   value, 
   opts?): void
```

定義於: [packages/form-core/src/FieldApi.ts:1269](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L1269)

替換指定索引處的值。

#### 參數

##### index

`number`

##### value

`TData` *繼承自* `any`[] ? `TData`\<`TData`\>\[`number`\] : `never`

##### opts?

`UpdateMetaOptions`

#### 回傳

`void`

***

### setErrorMap()

```ts
setErrorMap(errorMap): void
```

定義於: [packages/form-core/src/FieldApi.ts:1668](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L1668)

更新欄位的 errorMap

#### 參數

##### errorMap

`ValidationErrorMap`

#### 回傳

`void`

***

### setMeta()

```ts
setMeta(updater): void
```

定義於: [packages/form-core/src/FieldApi.ts:1212](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L1212)

設定欄位元數據。

#### 參數

##### updater

[`Updater`](../type-aliases/updater.md)\
