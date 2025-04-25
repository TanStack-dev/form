---
source-updated-at: '2025-03-16T18:32:04.000Z'
translation-updated-at: '2025-04-12T04:12:38.958Z'
id: FieldApiOptions
title: 接口 / FieldApiOptions
---
# 接口: FieldApiOptions\<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync, TFormOnServer, TParentSubmitMeta\>

定义于: [packages/form-core/src/FieldApi.ts:448](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L448)

表示 FieldApi 类所需选项的对象类型。

## 继承

- [`FieldOptions`](fieldoptions.md)\<`TParentData`, `TName`, `TData`, `TOnMount`, `TOnChange`, `TOnChangeAsync`, `TOnBlur`, `TOnBlurAsync`, `TOnSubmit`, `TOnSubmitAsync`\>

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

## 属性

### asyncAlways?

```ts
optional asyncAlways: boolean;
```

定义于: [packages/form-core/src/FieldApi.ts:395](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L395)

如果为 `true`，即使同步验证期间出现错误，也始终运行异步验证。

#### 继承自

[`FieldOptions`](fieldoptions.md).[`asyncAlways`](FieldOptions.md#asyncalways)

***

### asyncDebounceMs?

```ts
optional asyncDebounceMs: number;
```

定义于: [packages/form-core/src/FieldApi.ts:391](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L391)

如果没有传递更具体的防抖时间，则作为异步验证的默认防抖时间。

#### 继承自

[`FieldOptions`](fieldoptions.md).[`asyncDebounceMs`](FieldOptions.md#asyncdebouncems)

***

### defaultMeta?

```ts
optional defaultMeta: Partial<FieldMeta<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, any, any, any, any, any, any, any>>;
```

定义于: [packages/form-core/src/FieldApi.ts:414](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L414)

一个可选对象，包含字段的默认元数据。

#### 继承自

[`FieldOptions`](fieldoptions.md).[`defaultMeta`](FieldOptions.md#defaultmeta)

***

### defaultValue?

```ts
optional defaultValue: NoInfer<TData>;
```

定义于: [packages/form-core/src/FieldApi.ts:387](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L387)

字段的可选默认值。

#### 继承自

[`FieldOptions`](fieldoptions.md).[`defaultValue`](FieldOptions.md#defaultvalue)

***

### disableErrorFlat?

```ts
optional disableErrorFlat: boolean;
```

定义于: [packages/form-core/src/FieldApi.ts:442](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L442)

禁用对 `field.errors` 的 `flat(1)` 操作。如果你想保持错误结构不变，这很有用。不建议用于大多数场景。

#### 继承自

[`FieldOptions`](fieldoptions.md).[`disableErrorFlat`](FieldOptions.md#disableerrorflat)

***

### form

```ts
form: FormApi<TParentData, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync, TFormOnServer, TParentSubmitMeta>;
```

定义于: [packages/form-core/src/FieldApi.ts:486](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L486)

***

### listeners?

```ts
optional listeners: FieldListeners<TParentData, TName, TData>;
```

定义于: [packages/form-core/src/FieldApi.ts:438](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L438)

监听器列表，用于附加到相应事件

#### 继承自

[`FieldOptions`](fieldoptions.md).[`listeners`](FieldOptions.md#listeners)

***

### name

```ts
name: TName;
```

定义于: [packages/form-core/src/FieldApi.ts:383](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L383)

字段名称。类型为 `DeepKeys<TParentData>`，以确保名称是父数据集的深层键。

#### 继承自

[`FieldOptions`](fieldoptions.md).[`name`](FieldOptions.md#name)

***

### validators?

```ts
optional validators: FieldValidators<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync>;
```

定义于: [packages/form-core/src/FieldApi.ts:399](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L399)

传递给字段的验证器列表

#### 继承自

[`FieldOptions`](fieldoptions.md).[`validators`](FieldOptions.md#validators)
