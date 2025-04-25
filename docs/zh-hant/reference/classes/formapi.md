---
source-updated-at: '2025-04-25T12:58:29.000Z'
translation-updated-at: '2025-04-25T20:51:34.117Z'
id: FormApi
title: 類別 / FormApi
---
# 類別: FormApi\<TFormData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TOnServer, TSubmitMeta\>

定義於: [packages/form-core/src/FormApi.ts:765](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L765)

代表表單 API 的類別。負責處理表單狀態的邏輯與互動。

通常您不需要直接建立新的 `FormApi` 實例。相反地，您會使用框架提供的鉤子/函式如 `useForm` 或 `createForm` 來建立一個使用您框架反應式模型的新實例。然而，如果需要手動建立新實例，可以通過呼叫 `new FormApi` 建構函式來實現。

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

## 建構函式

### new FormApi()

```ts
new FormApi<TFormData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TOnServer, TSubmitMeta>(opts?): FormApi<TFormData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TOnServer, TSubmitMeta>
```

定義於: [packages/form-core/src/FormApi.ts:836](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L836)

使用給定的表單選項建構一個新的 `FormApi` 實例。

#### 參數

##### opts?

[`FormOptions`](../interfaces/formoptions.md)\<`TFormData`, `TOnMount`, `TOnChange`, `TOnChangeAsync`, `TOnBlur`, `TOnBlurAsync`, `TOnSubmit`, `TOnSubmitAsync`, `TOnServer`, `TSubmitMeta`\>

#### 回傳值

[`FormApi`](formapi.md)\<`TFormData`, `TOnMount`, `TOnChange`, `TOnChangeAsync`, `TOnBlur`, `TOnBlurAsync`, `TOnSubmit`, `TOnSubmitAsync`, `TOnServer`, `TSubmitMeta`\>

## 屬性

### baseStore

```ts
baseStore: Store<BaseFormState<TFormData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TOnServer>>;
```

定義於: [packages/form-core/src/FormApi.ts:792](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L792)

***

### fieldInfo

```ts
fieldInfo: Record<DeepKeys<TFormData>, FieldInfo<TFormData>>;
```

定義於: [packages/form-core/src/FormApi.ts:822](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L822)

表單中每個欄位的欄位資訊記錄。

***

### fieldMetaDerived

```ts
fieldMetaDerived: Derived<Record<DeepKeys<TFormData>, AnyFieldMeta>>;
```

定義於: [packages/form-core/src/FormApi.ts:805](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L805)

***

### options

```ts
options: FormOptions<TFormData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TOnServer, TSubmitMeta> = {};
```

定義於: [packages/form-core/src/FormApi.ts:780](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L780)

表單的選項。

***

### store

```ts
store: Derived<FormState<TFormData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TOnServer>>;
```

定義於: [packages/form-core/src/FormApi.ts:806](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L806)

## 存取器

### state

#### Get 簽名

```ts
get state(): FormState<TFormData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TOnServer>
```

定義於: [packages/form-core/src/FormApi.ts:824](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L824)

##### 回傳值

[`FormState`](../interfaces/formstate.md)\<`TFormData`, `TOnMount`, `TOnChange`, `TOnChangeAsync`, `TOnBlur`, `TOnBlurAsync`, `TOnSubmit`, `TOnSubmitAsync`, `TOnServer`\>

## 方法

### deleteField()

```ts
deleteField<TField>(field): void
```

定義於: [packages/form-core/src/FormApi.ts:1911](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L1911)

#### 型別參數

• **TField** *繼承自* `string`

#### 參數

##### field

`TField`

#### 回傳值

`void`

***

### getAllErrors()

```ts
getAllErrors(): object
```

定義於: [packages/form-core/src/FormApi.ts:2149](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L2149)

回傳表單與欄位層級的錯誤

#### 回傳值

`object`

##### fields

```ts
fields: Record<DeepKeys<TFormData>, {
  errorMap: ValidationErrorMap;
  errors: unknown[];
}>;
```

##### form

```ts
form: object;
```

###### form.errorMap

```ts
errorMap: FormValidationErrorMap<UnwrapFormValidateOrFn<TOnMount>, UnwrapFormValidateOrFn<TOnChange>, UnwrapFormAsyncValidateOrFn<TOnChangeAsync>, UnwrapFormValidateOrFn<TOnBlur>, UnwrapFormAsyncValidateOrFn<TOnBlurAsync>, UnwrapFormValidateOrFn<TOnSubmit>, UnwrapFormAsyncValidateOrFn<TOnSubmitAsync>, UnwrapFormAsyncValidateOrFn<TOnServer>>;
```

###### form.errors

```ts
errors: (
  | UnwrapFormValidateOrFn<TOnMount>
  | UnwrapFormValidateOrFn<TOnChange>
  | UnwrapFormAsyncValidateOrFn<TOnChangeAsync>
  | UnwrapFormValidateOrFn<TOnBlur>
  | UnwrapFormAsyncValidateOrFn<TOnBlurAsync>
  | UnwrapFormValidateOrFn<TOnSubmit>
  | UnwrapFormAsyncValidateOrFn<TOnSubmitAsync>
  | UnwrapFormAsyncValidateOrFn<TOnServer>)[];
```

***

### getFieldInfo()

```ts
getFieldInfo<TField>(field): FieldInfo<TFormData>
```

定義於: [packages/form-core/src/FormApi.ts:1825](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L1825)

取得指定欄位的欄位資訊。

#### 型別參數

• **TField** *繼承自* `string`

#### 參數

##### field

`TField`

#### 回傳值

[`FieldInfo`](../type-aliases/fieldinfo.md)\<`TFormData`\>

***

### getFieldMeta()

```ts
getFieldMeta<TField>(field): undefined | AnyFieldMeta
```

定義於: [packages/form-core/src/FormApi.ts:1816](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L1816)

取得指定欄位的中繼資料。

#### 型別參數

• **TField** *繼承自* `string`

#### 參數

##### field

`TField`

#### 回傳值

`undefined` \| [`AnyFieldMeta`](../type-aliases/anyfieldmeta.md)

***

### getFieldValue()

```ts
getFieldValue<TField>(field): DeepValue<TFormData, TField>
```

定義於: [packages/form-core/src/FormApi.ts:1809](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L1809)

取得指定欄位的值。

#### 型別參數

• **TField** *繼承自* `string`

#### 參數

##### field

`TField`

#### 回傳值

[`DeepValue`](../type-aliases/deepvalue.md)\<`TFormData`, `TField`\>

***

### handleSubmit()

#### 呼叫簽名

```ts
handleSubmit(): Promise<void>
```

定義於: [packages/form-core/src/FormApi.ts:1711](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L1711)

處理表單提交，執行驗證，並呼叫適當的 onSubmit 或 onSubmitInvalid 回呼函式。

##### 回傳值

`Promise`\<`void`\>

#### 呼叫簽名

```ts
handleSubmit(submitMeta): Promise<void>
```

定義於: [packages/form-core/src/FormApi.ts:1712](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L1712)

處理表單提交，執行驗證，並呼叫適當的 onSubmit 或 onSubmitInvalid 回呼函式。

##### 參數

###### submitMeta

`TSubmitMeta`

##### 回傳值

`Promise`\<`void`\>

***

### insertFieldValue()

```ts
insertFieldValue<TField>(
   field, 
   index, 
   value, 
opts?): Promise<void>
```

定義於: [packages/form-core/src/FormApi.ts:1950](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L1950)

#### 型別參數

• **TField** *繼承自* `string`

#### 參數

##### field

`TField`

##### index

`number`

##### value

[`DeepValue`](../type-aliases/deepvalue.md)\<`TFormData`, `TField`\> *繼承自* `any`[] ? `any`[] & [`DeepValue`](../type-aliases/deepvalue.md)\<`TFormData`, `TField`\>\[`number`\] : `never`

##### opts?

`UpdateMetaOptions`

#### 回傳值

`Promise`\<`void`\>

***

### mount()

```ts
mount(): () => void
```

定義於: [packages/form-core/src/FormApi.ts:1141](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L1141)

#### 回傳值

`Function`

##### 回傳值

`void`

***

### moveFieldValues()

```ts
moveFieldValues<TField>(
   field, 
   index1, 
   index2, 
   opts?): void
```

定義於: [packages/form-core/src/FormApi.ts:2074](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L2074)

將陣列欄位中第一個指定索引的值移動到第二個指定索引。

#### 型別參數

• **TField** *繼承自* `string`

#### 參數

##### field

`TField`

##### index1

`number`

##### index2

`number`

##### opts?

`UpdateMetaOptions`

#### 回傳值

`void`

***

### parseValuesWithSchema()

```ts
parseValuesWithSchema(schema): 
  | undefined
  | {
  fields: Record<string, StandardSchemaV1Issue[]>;
  form: Record<string, StandardSchemaV1Issue[]>;
}
```

定義於: [packages/form-core/src/FormApi.ts:2209](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L2209)

使用給定的標準結構描述解析表單值並回傳問題（如果有）。此方法不會設定任何內部錯誤。

#### 參數

##### schema

[`StandardSchemaV1`](../type-aliases/standardschemav1.md)\<`TFormData`, `unknown`\>

用來解析表單值的標準結構描述。

#### 回傳值

  \| `undefined`
  \| \{
  `fields`: `Record`\<`string`, [`StandardSchemaV1Issue`](../interfaces/standardschemav1issue.md)[]\>;
  `form`: `Record`\<`string`, [`StandardSchemaV1Issue`](../interfaces/standardschemav1issue.md)[]\>;
 \}

***

### parseValuesWithSchemaAsync()

```ts
parseValuesWithSchemaAsync(schema): Promise<
  | undefined
  | {
  fields: Record<string, StandardSchemaV1Issue[]>;
  form: Record<string, StandardSchemaV1Issue[]>;
}>
```

定義於: [packages/form-core/src/FormApi.ts:2221](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L2221)

使用給定的標準結構描述解析表單值並回傳問題（如果有）。此方法不會設定任何內部錯誤。

#### 參數

##### schema

[`StandardSchemaV1`](../type-aliases/standardschemav1.md)\<`TFormData`, `unknown`\>

用來解析表單值的標準結構描述。

#### 回傳值

`Promise`\<
  \| `undefined`
  \| \{
  `fields`: `Record`\<`string`, [`StandardSchemaV1Issue`](../interfaces/standardschemav1issue.md)[]\>;
  `form`: `Record`\<`string`, [`StandardSchemaV1Issue`](../interfaces/standardschemav1issue.md)[]\>;
 \}\>

***

### pushFieldValue()

```ts
pushFieldValue<TField>(
   field, 
   value, 
   opts?): void
```

定義於: [packages/form-core/src/FormApi.ts:1935](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L1935)

將值推入陣列欄位。

#### 型別參數

• **TField** *繼承自* `string`

#### 參數

##### field

`TField`

##### value

[`DeepValue`](../type-aliases/deepvalue.md)\<`TFormData`, `TField`\> *繼承自* `any`[] ? `any`[] & [`DeepValue`](../type-aliases/deepvalue.md)\<`TFormData`, `TField`\>\[`number`\] : `never`

##### opts?

`UpdateMetaOptions`

#### 回傳值

`void`

***

### removeFieldValue()

```ts
removeFieldValue<TField>(
   field, 
   index, 
opts?): Promise<void>
```

定義於: [packages/form-core/src/FormApi.ts:2008](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L2008)

從陣列欄位的指定索引移除值。

#### 型別參數

• **TField** *繼承自* `string`

#### 參數

##### field

`TField`

##### index

`number`

##### opts?

`UpdateMetaOptions`

#### 回傳值

`Promise`\<`void`\>

***

### replaceFieldValue()

```ts
replaceFieldValue<TField>(
   field, 
   index, 
   value, 
opts?): Promise<void>
```

定義於: [packages/form-core/src/FormApi.ts:1982](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L1982)

替換陣列欄位中指定索引的值。

#### 型別參數

• **TField** *繼承自* `string`

#### 參數

##### field

`TField`

##### index

`number`

##### value

[`DeepValue`](../type-aliases/deepvalue.md)\<`TFormData`, `TField`\> *繼承自* `any`[] ? `any`[] & [`DeepValue`](../type-aliases/deepvalue.md)\<`TFormData`, `TField`\>\[`number`\] : `never`

##### opts?

`UpdateMetaOptions`

#### 回傳值

`Promise`\<`void`\>

***

### reset()

```ts
reset(values?, opts?): void
```

定義於: [packages/form-core/src/FormApi.ts:1229](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L1229)

將表單狀態重置為預設值。
如果提供了值，表單將重置為這些值並
