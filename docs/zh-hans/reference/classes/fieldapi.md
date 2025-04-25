---
source-updated-at: '2025-03-31T17:13:49.000Z'
translation-updated-at: '2025-04-12T04:13:53.652Z'
id: FieldApi
title: 类 / FieldApi
---
# 类: FieldApi\<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync, TFormOnServer, TParentSubmitMeta\>

定义于: [packages/form-core/src/FieldApi.ts:860](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L860)

用于管理表单字段 API 的类。

通常您不需要直接创建新的 `FieldApi` 实例。
相反，您可以使用框架钩子/函数如 `useField` 或 `createField`
来为您创建一个使用框架响应式模型的新实例。
但如果需要手动创建新实例，可以通过调用
`new FieldApi` 构造函数来实现。

## 类型参数

• **TParentData**

• **TName** *继承自* [`DeepKeys`](../type-aliases/deepkeys.md)\<`TParentData`\>

• **TData** *继承自* [`DeepValue`](../type-aliases/deepvalue.md)\<`TParentData`, `TName`\>

• **TOnMount** *继承自* `undefined` \| `FieldValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnChange** *继承自* `undefined` \| `FieldValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnChangeAsync** *继承自* `undefined` \| `FieldAsyncValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnBlur** *继承自* `undefined` \| `FieldValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnBlurAsync** *继承自* `undefined` \| `FieldAsyncValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnSubmit** *继承自* `undefined` \| `FieldValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnSubmitAsync** *继承自* `undefined` \| `FieldAsyncValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TFormOnMount** *继承自* `undefined` \| `FormValidateOrFn`\<`TParentData`\>

• **TFormOnChange** *继承自* `undefined` \| `FormValidateOrFn`\<`TParentData`\>

• **TFormOnChangeAsync** *继承自* `undefined` \| `FormAsyncValidateOrFn`\<`TParentData`\>

• **TFormOnBlur** *继承自* `undefined` \| `FormValidateOrFn`\<`TParentData`\>

• **TFormOnBlurAsync** *继承自* `undefined` \| `FormAsyncValidateOrFn`\<`TParentData`\>

• **TFormOnSubmit** *继承自* `undefined` \| `FormValidateOrFn`\<`TParentData`\>

• **TFormOnSubmitAsync** *继承自* `undefined` \| `FormAsyncValidateOrFn`\<`TParentData`\>

• **TFormOnServer** *继承自* `undefined` \| `FormAsyncValidateOrFn`\<`TParentData`\>

• **TParentSubmitMeta**

## 构造函数

### new FieldApi()

```ts
new FieldApi<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync, TFormOnServer, TParentSubmitMeta>(opts): FieldApi<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync, TFormOnServer, TParentSubmitMeta>
```

定义于: [packages/form-core/src/FieldApi.ts:974](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L974)

初始化一个新的 `FieldApi` 实例。

#### 参数

##### opts

[`FieldApiOptions`](../interfaces/fieldapioptions.md)\<`TParentData`, `TName`, `TData`, `TOnMount`, `TOnChange`, `TOnChangeAsync`, `TOnBlur`, `TOnBlurAsync`, `TOnSubmit`, `TOnSubmitAsync`, `TFormOnMount`, `TFormOnChange`, `TFormOnChangeAsync`, `TFormOnBlur`, `TFormOnBlurAsync`, `TFormOnSubmit`, `TFormOnSubmitAsync`, `TFormOnServer`, `TParentSubmitMeta`\>

#### 返回

[`FieldApi`](fieldapi.md)\<`TParentData`, `TName`, `TData`, `TOnMount`, `TOnChange`, `TOnChangeAsync`, `TOnBlur`, `TOnBlurAsync`, `TOnSubmit`, `TOnSubmitAsync`, `TFormOnMount`, `TFormOnChange`, `TFormOnChangeAsync`, `TFormOnBlur`, `TFormOnBlurAsync`, `TFormOnSubmit`, `TFormOnSubmitAsync`, `TFormOnServer`, `TParentSubmitMeta`\>

## 属性

### form

```ts
form: FormApi<TParentData, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync, TFormOnServer, TParentSubmitMeta>;
```

定义于: [packages/form-core/src/FieldApi.ts:890](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L890)

对表单 API 实例的引用。

***

### name

```ts
name: unknown extends TParentData ? string : TParentData extends readonly any[] & IsTuple<TParentData> ? PrefixTupleAccessor<TParentData<TParentData>, AllowedIndexes<TParentData<TParentData>, never>, []> : TParentData extends any[] ? PrefixArrayAccessor<TParentData<TParentData>, [any]> : TParentData extends Date ? never : TParentData extends object ? PrefixObjectAccessor<TParentData<TParentData>, []> : TParentData extends string | number | bigint | boolean ? "" : never;
```

定义于: [packages/form-core/src/FieldApi.ts:914](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L914)

字段名称。

***

### options

```ts
options: FieldApiOptions<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync, TFormOnServer, TParentSubmitMeta>;
```

定义于: [packages/form-core/src/FieldApi.ts:918](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L918)

字段选项。

***

### store

```ts
store: Derived<FieldState<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync>>;
```

定义于: [packages/form-core/src/FieldApi.ts:942](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L942)

字段状态存储。

***

### timeoutIds

```ts
timeoutIds: Record<ValidationCause, null | Timeout>;
```

定义于: [packages/form-core/src/FieldApi.ts:969](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L969)

## 访问器

### state

#### Get 签名

```ts
get state(): FieldState<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync>
```

定义于: [packages/form-core/src/FieldApi.ts:966](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L966)

当前字段状态。

##### 返回

[`FieldState`](../type-aliases/fieldstate.md)\<`TParentData`, `TName`, `TData`, `TOnMount`, `TOnChange`, `TOnChangeAsync`, `TOnBlur`, `TOnBlurAsync`, `TOnSubmit`, `TOnSubmitAsync`, `TFormOnMount`, `TFormOnChange`, `TFormOnChangeAsync`, `TFormOnBlur`, `TFormOnBlurAsync`, `TFormOnSubmit`, `TFormOnSubmitAsync`\>

## 方法

### getInfo()

```ts
getInfo(): FieldInfo<TParentData>
```

定义于: [packages/form-core/src/FieldApi.ts:1222](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L1222)

获取字段信息对象。

#### 返回

[`FieldInfo`](../type-aliases/fieldinfo.md)\<`TParentData`\>

***

### getMeta()

```ts
getMeta(): FieldMeta<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync>
```

定义于: [packages/form-core/src/FieldApi.ts:1190](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L1190)

#### 返回

[`FieldMeta`](../type-aliases/fieldmeta.md)\<`TParentData`, `TName`, `TData`, `TOnMount`, `TOnChange`, `TOnChangeAsync`, `TOnBlur`, `TOnBlurAsync`, `TOnSubmit`, `TOnSubmitAsync`, `TFormOnMount`, `TFormOnChange`, `TFormOnChangeAsync`, `TFormOnBlur`, `TFormOnBlurAsync`, `TFormOnSubmit`, `TFormOnSubmitAsync`\>

***

### ~~getValue()~~

```ts
getValue(): TData
```

定义于: [packages/form-core/src/FieldApi.ts:1172](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L1172)

获取当前字段值。

#### 返回

`TData`

#### 弃用

改用 `field.state.value`。

***

### handleBlur()

```ts
handleBlur(): void
```

定义于: [packages/form-core/src/FieldApi.ts:1632](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L1632)

处理 blur 事件。

#### 返回

`void`

***

### handleChange()

```ts
handleChange(updater): void
```

定义于: [packages/form-core/src/FieldApi.ts:1625](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L1625)

处理 change 事件。

#### 参数

##### updater

[`Updater`](../type-aliases/updater.md)\<`TData`\>

#### 返回

`void`

***

### insertValue()

```ts
insertValue(
   index, 
   value, 
   opts?): void
```

定义于: [packages/form-core/src/FieldApi.ts:1242](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L1242)

在指定索引处插入值，并将后续值向右移动。

#### 参数

##### index

`number`

##### value

`TData` *继承自* `any`[] ? `TData`\<`TData`\>\[`number`\] : `never`

##### opts?

`UpdateMetaOptions`

#### 返回

`void`

***

### mount()

```ts
mount(): () => void
```

定义于: [packages/form-core/src/FieldApi.ts:1067](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L1067)

将字段实例挂载到表单。

#### 返回

`Function`

##### 返回

`void`

***

### moveValue()

```ts
moveValue(
   aIndex, 
   bIndex, 
   opts?): void
```

定义于: [packages/form-core/src/FieldApi.ts:1298](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L1298)

将第一个指定索引处的值移动到第二个指定索引处。

#### 参数

##### aIndex

`number`

##### bIndex

`number`

##### opts?

`UpdateMetaOptions`

#### 返回

`void`

***

### pushValue()

```ts
pushValue(value, opts?): void
```

定义于: [packages/form-core/src/FieldApi.ts:1227](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L1227)

向字段推送新值。

#### 参数

##### value

`TData` *继承自* `any`[] ? `TData`\<`TData`\>\[`number`\] : `never`

##### opts?

`UpdateMetaOptions`

#### 返回

`void`

***

### removeValue()

```ts
removeValue(index, opts?): void
```

定义于: [packages/form-core/src/FieldApi.ts:1274](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L1274)

移除指定索引处的值。

#### 参数

##### index

`number`

##### opts?

`UpdateMetaOptions`

#### 返回

`void`

***

### replaceValue()

```ts
replaceValue(
   index, 
   value, 
   opts?): void
```

定义于: [packages/form-core/src/FieldApi.ts:1258](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L1258)

替换指定索引处的值。

#### 参数

##### index

`number`

##### value

`TData` *继承自* `any`[] ? `TData`\<`TData`\>\[`number`\] : `never`

##### opts?

`UpdateMetaOptions`

#### 返回

`void`

***

### setErrorMap()

```ts
setErrorMap(errorMap): void
```

定义于: [packages/form-core/src/FieldApi.ts:1652](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L1652)

更新字段的 errorMap

#### 参数

##### errorMap

`ValidationErrorMap`

#### 返回

`void`

***

### setMeta()

```ts
setMeta(updater): void
```

定义于: [packages/form-core/src/FieldApi.ts:1195](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L1195)

设置字段元数据。

#### 参数

##### updater

[`Updater`](../type-aliases/updater.md)\<[`FieldMeta`](../type-aliases/fieldmeta.md)\<`TParentData`, `TName`, `TData`, `TOnMount`, `TOnChange`, `TOnChangeAsync`, `TOnBlur`, `TOnBlurAsync`, `TOnSubmit`, `TOnSubmitAsync`, `TFormOnMount`, `TFormOnChange`, `TFormOnChangeAsync`, `TFormOnBlur`, `TFormOnBlurAsync`, `TFormOnSubmit`, `TFormOnSubmitAsync`\>\>

#### 返回

`void`

***

### setValue()

```ts
setValue(updater, options?): void
```

定义于: [packages/form-core/src/FieldApi.ts:1179](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L1179)

设置字段值并运行 `change` 验证器。

#### 参数

##### updater

[`Updater`](../type-aliases/updater.md)\<`TData`\>

##### options?

`UpdateMetaOptions`

#### 返回

`void`

***

### swapValues()

```ts
swapValues(
   aIndex, 
   bIndex, 
   opts?): void
```

定义于: [packages/form-core/src/FieldApi.ts:1286](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L1286)

交换指定索引处的值。

#### 参数

##### aIndex

`number`

##### bIndex

`number`

##### opts?

`UpdateMetaOptions`

#### 返回

`void`

***

### update()

```ts
update(opts): void
```

定义于: [packages/form-core/src/FieldApi.ts:1115](https://github
