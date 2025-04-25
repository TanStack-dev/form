---
source-updated-at: '2025-03-27T10:36:50.000Z'
translation-updated-at: '2025-04-12T04:13:59.633Z'
id: FormApi
title: 类 / FormApi
---
# 类: FormApi\<TFormData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TOnServer, TSubmitMeta\>

定义于: [packages/form-core/src/FormApi.ts:664](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L664)

表示表单 API 的类。它处理与表单状态相关的逻辑和交互。

通常情况下，您不需要直接创建新的 `FormApi` 实例。相反，您可以使用框架提供的钩子/函数（如 `useForm` 或 `createForm`）来创建一个使用您框架响应式模型的新实例。但是，如果需要手动创建新实例，可以通过调用 `new FormApi` 构造函数来实现。

## 类型参数

• **TFormData**

• **TOnMount** *继承自* `undefined` \| `FormValidateOrFn`\<`TFormData`\>

• **TOnChange** *继承自* `undefined` \| `FormValidateOrFn`\<`TFormData`\>

• **TOnChangeAsync** *继承自* `undefined` \| `FormAsyncValidateOrFn`\<`TFormData`\>

• **TOnBlur** *继承自* `undefined` \| `FormValidateOrFn`\<`TFormData`\>

• **TOnBlurAsync** *继承自* `undefined` \| `FormAsyncValidateOrFn`\<`TFormData`\>

• **TOnSubmit** *继承自* `undefined` \| `FormValidateOrFn`\<`TFormData`\>

• **TOnSubmitAsync** *继承自* `undefined` \| `FormAsyncValidateOrFn`\<`TFormData`\>

• **TOnServer** *继承自* `undefined` \| `FormAsyncValidateOrFn`\<`TFormData`\>

• **TSubmitMeta** = `never`

## 构造函数

### new FormApi()

```ts
new FormApi<TFormData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TOnServer, TSubmitMeta>(opts?): FormApi<TFormData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TOnServer, TSubmitMeta>
```

定义于: [packages/form-core/src/FormApi.ts:751](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L751)

使用给定的表单选项构造一个新的 `FormApi` 实例。

#### 参数

##### opts?

[`FormOptions`](../interfaces/formoptions.md)\<`TFormData`, `TOnMount`, `TOnChange`, `TOnChangeAsync`, `TOnBlur`, `TOnBlurAsync`, `TOnSubmit`, `TOnSubmitAsync`, `TOnServer`, `TSubmitMeta`\>

#### 返回值

[`FormApi`](formapi.md)\<`TFormData`, `TOnMount`, `TOnChange`, `TOnChangeAsync`, `TOnBlur`, `TOnBlurAsync`, `TOnSubmit`, `TOnSubmitAsync`, `TOnServer`, `TSubmitMeta`\>

## 属性

### baseStore

```ts
baseStore: Store<BaseFormState<TFormData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TOnServer>>;
```

定义于: [packages/form-core/src/FormApi.ts:691](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L691)

***

### fieldInfo

```ts
fieldInfo: Record<unknown extends TFormData ? string : TFormData extends readonly any[] & IsTuple<TFormData> ? PrefixTupleAccessor<TFormData<TFormData>, AllowedIndexes<TFormData<TFormData>, never>, []> : TFormData extends any[] ? PrefixArrayAccessor<TFormData<TFormData>, [any]> : TFormData extends Date ? never : TFormData extends object ? PrefixObjectAccessor<TFormData<TFormData>, []> : TFormData extends string | number | bigint | boolean ? "" : never, FieldInfo<TFormData>>;
```

定义于: [packages/form-core/src/FormApi.ts:721](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L721)

表单中每个字段的信息记录。

***

### fieldMetaDerived

```ts
fieldMetaDerived: Derived<Record<unknown extends TFormData ? string : TFormData extends readonly any[] & IsTuple<TFormData> ? PrefixTupleAccessor<TFormData<TFormData>, AllowedIndexes<TFormData<TFormData>, never>, []> : TFormData extends any[] ? PrefixArrayAccessor<TFormData<TFormData>, [any]> : TFormData extends Date ? never : TFormData extends object ? PrefixObjectAccessor<TFormData<TFormData>, []> : TFormData extends string | number | bigint | boolean ? "" : never, AnyFieldMeta>>;
```

定义于: [packages/form-core/src/FormApi.ts:704](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L704)

***

### options

```ts
options: FormOptions<TFormData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TOnServer, TSubmitMeta> = {};
```

定义于: [packages/form-core/src/FormApi.ts:679](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L679)

表单的选项。

***

### store

```ts
store: Derived<FormState<TFormData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TOnServer>>;
```

定义于: [packages/form-core/src/FormApi.ts:705](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L705)

## 访问器

### state

#### Get 签名

```ts
get state(): FormState<TFormData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TOnServer>
```

定义于: [packages/form-core/src/FormApi.ts:723](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L723)

##### 返回值

[`FormState`](../type-aliases/formstate.md)\<`TFormData`, `TOnMount`, `TOnChange`, `TOnChangeAsync`, `TOnBlur`, `TOnBlurAsync`, `TOnSubmit`, `TOnSubmitAsync`, `TOnServer`\>

## 方法

### deleteField()

```ts
deleteField<TField>(field): void
```

定义于: [packages/form-core/src/FormApi.ts:1772](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L1772)

#### 类型参数

• **TField** *继承自* `string` \| `number`

#### 参数

##### field

`TField`

#### 返回值

`void`

***

### getAllErrors()

```ts
getAllErrors(): object
```

定义于: [packages/form-core/src/FormApi.ts:2010](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L2010)

返回表单和字段级别的错误

#### 返回值

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

定义于: [packages/form-core/src/FormApi.ts:1686](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L1686)

获取指定字段的字段信息。

#### 类型参数

• **TField** *继承自* `string` \| `number`

#### 参数

##### field

`TField`

#### 返回值

[`FieldInfo`](../type-aliases/fieldinfo.md)\<`TFormData`\>

***

### getFieldMeta()

```ts
getFieldMeta<TField>(field): undefined | AnyFieldMeta
```

定义于: [packages/form-core/src/FormApi.ts:1677](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L1677)

获取指定字段的元数据。

#### 类型参数

• **TField** *继承自* `string` \| `number`

#### 参数

##### field

`TField`

#### 返回值

`undefined` \| [`AnyFieldMeta`](../type-aliases/anyfieldmeta.md)

***

### getFieldValue()

```ts
getFieldValue<TField>(field): DeepValue<TFormData, TField>
```

定义于: [packages/form-core/src/FormApi.ts:1670](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L1670)

获取指定字段的值。

#### 类型参数

• **TField** *继承自* `string` \| `number`

#### 参数

##### field

`TField`

#### 返回值

[`DeepValue`](../type-aliases/deepvalue.md)\<`TFormData`, `TField`\>

***

### handleSubmit()

#### 调用签名

```ts
handleSubmit(): Promise<void>
```

定义于: [packages/form-core/src/FormApi.ts:1586](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L1586)

处理表单提交，执行验证，并调用适当的 onSubmit 或 onInvalidSubmit 回调。

##### 返回值

`Promise`\<`void`\>

#### 调用签名

```ts
handleSubmit(submitMeta): Promise<void>
```

定义于: [packages/form-core/src/FormApi.ts:1587](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L1587)

处理表单提交，执行验证，并调用适当的 onSubmit 或 onInvalidSubmit 回调。

##### 参数

###### submitMeta

`TSubmitMeta`

##### 返回值

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

定义于: [packages/form-core/src/FormApi.ts:1811](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L1811)

#### 类型参数

• **TField** *继承自* `string` \| `number`

#### 参数

##### field

`TField`

##### index

`number`

##### value

[`DeepValue`](../type-aliases/deepvalue.md)\<`TFormData`, `TField`, \[\]\> *继承自* `any`[] ? `any`[] & [`DeepValue`](../type-aliases/deepvalue.md)\<`TFormData`, `TField`, \[\]\>\[`number`\] : `never`

##### opts?

`UpdateMetaOptions`

#### 返回值

`Promise`\<`void`\>

***

### mount()

```ts
mount(): () => void
```

定义于: [packages/form-core/src/FormApi.ts:1054](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L1054)

#### 返回值

`Function`

##### 返回值

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

定义于: [packages/form-core/src/FormApi.ts:1935](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L1935)

将数组字段中第一个指定索引处的值移动到第二个指定索引处。

#### 类型参数

• **TField** *继承自* `string` \| `number`

#### 参数

##### field

`TField`

##### index1

`number`

##### index2

`number`

##### opts?

`UpdateMetaOptions`

#### 返回值

`void`

***

### pushFieldValue()

```ts
pushFieldValue<TField>(
   field, 
   value, 
   opts?): void
```

定义于: [packages/form-core/src/FormApi.ts:1796](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L1796)

将值推入数组字段。

#### 类型参数

• **TField** *继承自* `string` \| `number`

#### 参数

##### field

`TField`

##### value

[`DeepValue`](../type-aliases/deepvalue.md)\<`TFormData`, `TField`, \[\]\> *继承自* `any`[] ? `any`[] & [`DeepValue`](../type-aliases/deepvalue.md)\<`TFormData`, `TField`, \[\]\>\[`number`\] : `never`

##### opts?

`UpdateMetaOptions`

#### 返回值

`void`

***

### removeFieldValue()

```ts
removeFieldValue<TField>(
   field, 
   index, 
opts?): Promise<void>
```

定义于: [packages/form-core/src/FormApi.ts:1869](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L1869)

从数组字段中移除指定索引处的值。

#### 类型参数

• **TField** *继承自* `string` \| `number`

#### 参数

##### field

`TField`

##### index

`number`

##### opts?

`UpdateMetaOptions`

#### 返回值

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

定义于: [packages/form-core/src/FormApi.ts:1843](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L1843)

替换数组字段中指定索引处的值。

#### 类型参数

• **TField** *继承自* `string` \| `number`

#### 参数

##### field

`TField`

##### index

`number`

##### value

[`DeepValue`](../type-aliases/deepvalue.md)\<`TFormData`, `TField`, \[\]\> *继承自* `any`[] ? `any`[] & [`DeepValue`](../type-aliases/deepvalue.md)\<`TFormData`, `TField`, \[\]\>\[`number`\] : `never`

##### opts?

`UpdateMetaOptions`

#### 返回值

`Promise`\<`void`\>

***

### reset()

```ts
reset(values?, opts?): void
```

定义于: [packages/form-core/src/FormApi.ts:1139](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L1139)

将表单状态重置为默认值。
如果提供了值，表单将被重置为这些值，并且默认值将被更新。

#### 参数

##### values?

`TFormData`

用于重置表单的可选值。

##### opts?

控制重置行为的可选选项。

###### keepDefaultValues?

`boolean`

#### 返回值

`void`

***

### resetField()

```ts
resetField<TField>(field): void
```

定义于: [packages/form-core/src/FormApi.ts:1963](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L1963)

将字段值和元数据重置为默认状态

#### 类型参数

• **TField** *继承自* `string` \| `number`

#### 参数

##### field

`TField`

#### 返回值

`void`

***

### resetFieldMeta()

```ts
resetFieldMeta<TField>(fieldMeta): Record<TField, AnyFieldMeta>
```

定义于: [packages/form-core/src/FormApi.ts:1726](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L1726)

重置每个字段的元数据

#### 类型参数

• **TField** *继承自* `string` \| `number`

#### 参数

##### fieldMeta

`Record`\<`TField`, [`AnyFieldMeta`](../type-aliases/anyfieldmeta.md)\>

#### 返回值

`Record`\<`TField`, [`AnyFieldMeta`](../type-aliases/anyfieldmeta.md)\>

***

### setErrorMap()

```ts
setErrorMap(errorMap): void
```

定义于: [packages/form-core/src/FormApi.ts:1984](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L1984
