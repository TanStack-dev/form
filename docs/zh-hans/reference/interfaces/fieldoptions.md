---
source-updated-at: '2025-03-16T18:32:04.000Z'
translation-updated-at: '2025-04-12T04:12:17.245Z'
id: FieldOptions
title: 接口 / FieldOptions
---
# 接口: FieldOptions\<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync\>

定义于: [packages/form-core/src/FieldApi.ts:362](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L362)

表示表单字段选项的对象类型。

## 继承自

- [`FieldApiOptions`](fieldapioptions.md)

## 类型参数

• **TParentData**

• **TName** *继承* [`DeepKeys`](../type-aliases/deepkeys.md)\<`TParentData`\>

• **TData** *继承* [`DeepValue`](../type-aliases/deepvalue.md)\<`TParentData`, `TName`\>

• **TOnMount** *继承* `undefined` \| `FieldValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnChange** *继承* `undefined` \| `FieldValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnChangeAsync** *继承* `undefined` \| `FieldAsyncValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnBlur** *继承* `undefined` \| `FieldValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnBlurAsync** *继承* `undefined` \| `FieldAsyncValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnSubmit** *继承* `undefined` \| `FieldValidateOrFn`\<`TParentData`, `TName`, `TData`\>

• **TOnSubmitAsync** *继承* `undefined` \| `FieldAsyncValidateOrFn`\<`TParentData`, `TName`, `TData`\>

## 属性

### asyncAlways?

```ts
optional asyncAlways: boolean;
```

定义于: [packages/form-core/src/FieldApi.ts:395](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L395)

如果为 `true`，即使同步验证期间出现错误，也始终运行异步验证。

***

### asyncDebounceMs?

```ts
optional asyncDebounceMs: number;
```

定义于: [packages/form-core/src/FieldApi.ts:391](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L391)

如果没有传递更具体的防抖时间，则默认用于异步验证的防抖时间。

***

### defaultMeta?

```ts
optional defaultMeta: Partial<FieldMeta<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, any, any, any, any, any, any, any>>;
```

定义于: [packages/form-core/src/FieldApi.ts:414](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L414)

字段的可选默认元数据对象。

***

### defaultValue?

```ts
optional defaultValue: NoInfer<TData>;
```

定义于: [packages/form-core/src/FieldApi.ts:387](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L387)

字段的可选默认值。

***

### disableErrorFlat?

```ts
optional disableErrorFlat: boolean;
```

定义于: [packages/form-core/src/FieldApi.ts:442](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L442)

禁用对 `field.errors` 的 `flat(1)` 操作。如果你想保持错误结构不变，这很有用。不建议大多数用例使用。

***

### listeners?

```ts
optional listeners: FieldListeners<TParentData, TName, TData>;
```

定义于: [packages/form-core/src/FieldApi.ts:438](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L438)

附加到相应事件的监听器列表

***

### name

```ts
name: TName;
```

定义于: [packages/form-core/src/FieldApi.ts:383](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L383)

字段名称。类型为 `DeepKeys<TParentData>`，以确保名称是父数据集的一个深层键。

***

### validators?

```ts
optional validators: FieldValidators<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync>;
```

定义于: [packages/form-core/src/FieldApi.ts:399](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L399)

传递给字段的验证器列表
