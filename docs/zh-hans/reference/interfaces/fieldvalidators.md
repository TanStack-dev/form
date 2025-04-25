---
source-updated-at: '2025-03-16T18:32:04.000Z'
translation-updated-at: '2025-04-12T04:12:39.423Z'
id: FieldValidators
title: 接口 / FieldValidators
---
# 接口: FieldValidators\<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync\>

定义于: [packages/form-core/src/FieldApi.ts:267](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L267)

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

## 属性

### onBlur?

```ts
optional onBlur: TOnBlur;
```

定义于: [packages/form-core/src/FieldApi.ts:316](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L316)

一个可选函数，在输入框的 blur 事件时运行。

#### 示例

```ts
z.string().min(1)
```

***

### onBlurAsync?

```ts
optional onBlurAsync: TOnBlurAsync;
```

定义于: [packages/form-core/src/FieldApi.ts:322](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L322)

一个可选属性，类似于 `onBlur` 但用于异步验证。

#### 示例

```ts
z.string().refine(async (val) => val.length > 3, { message: 'Testing 123' })
```

***

### onBlurAsyncDebounceMs?

```ts
optional onBlurAsyncDebounceMs: number;
```

定义于: [packages/form-core/src/FieldApi.ts:329](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L329)

一个可选数字，表示 `onBlurAsync` 在运行前应等待的时间。

如果设置为大于 0 的数字，将以毫秒为单位对异步验证事件进行防抖处理。

***

### onBlurListenTo?

```ts
optional onBlurListenTo: unknown extends TParentData ? string : TParentData extends readonly any[] & IsTuple<TParentData> ? PrefixTupleAccessor<TParentData<TParentData>, AllowedIndexes<TParentData<TParentData>, never>, []> : TParentData extends any[] ? PrefixArrayAccessor<TParentData<TParentData>, [any]> : TParentData extends Date ? never : TParentData extends object ? PrefixObjectAccessor<TParentData<TParentData>, []> : TParentData extends string | number | bigint | boolean ? "" : never[];
```

定义于: [packages/form-core/src/FieldApi.ts:333](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L333)

一个可选的字段名称列表，当其值发生变化时，应触发此字段的 `onBlur` 和 `onBlurAsync` 事件。

***

### onChange?

```ts
optional onChange: TOnChange;
```

定义于: [packages/form-core/src/FieldApi.ts:294](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L294)

一个可选函数，在输入框的 change 事件时运行。

#### 示例

```ts
z.string().min(1)
```

***

### onChangeAsync?

```ts
optional onChangeAsync: TOnChangeAsync;
```

定义于: [packages/form-core/src/FieldApi.ts:300](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L300)

一个可选属性，类似于 `onChange` 但用于异步验证。

#### 示例

```ts
z.string().refine(async (val) => val.length > 3, { message: 'Testing 123' })
```

***

### onChangeAsyncDebounceMs?

```ts
optional onChangeAsyncDebounceMs: number;
```

定义于: [packages/form-core/src/FieldApi.ts:306](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L306)

一个可选数字，表示 `onChangeAsync` 在运行前应等待的时间。

如果设置为大于 0 的数字，将以毫秒为单位对异步验证事件进行防抖处理。

***

### onChangeListenTo?

```ts
optional onChangeListenTo: unknown extends TParentData ? string : TParentData extends readonly any[] & IsTuple<TParentData> ? PrefixTupleAccessor<TParentData<TParentData>, AllowedIndexes<TParentData<TParentData>, never>, []> : TParentData extends any[] ? PrefixArrayAccessor<TParentData<TParentData>, [any]> : TParentData extends Date ? never : TParentData extends object ? PrefixObjectAccessor<TParentData<TParentData>, []> : TParentData extends string | number | bigint | boolean ? "" : never[];
```

定义于: [packages/form-core/src/FieldApi.ts:310](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L310)

一个可选的字段名称列表，当其值发生变化时，应触发此字段的 `onChange` 和 `onChangeAsync` 事件。

***

### onMount?

```ts
optional onMount: TOnMount;
```

定义于: [packages/form-core/src/FieldApi.ts:288](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L288)

一个可选函数，在输入框的 mount 事件时运行。

***

### onSubmit?

```ts
optional onSubmit: TOnSubmit;
```

定义于: [packages/form-core/src/FieldApi.ts:339](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L339)

一个可选函数，在表单的 submit 事件时运行。

#### 示例

```ts
z.string().min(1)
```

***

### onSubmitAsync?

```ts
optional onSubmitAsync: TOnSubmitAsync;
```

定义于: [packages/form-core/src/FieldApi.ts:345](https://github.com/TanStack/form/blob/main/packages/form-core/src/FieldApi.ts#L345)

一个可选属性，类似于 `onSubmit` 但用于异步验证。

#### 示例

```ts
z.string().refine(async (val) => val.length > 3, { message: 'Testing 123' })
```
