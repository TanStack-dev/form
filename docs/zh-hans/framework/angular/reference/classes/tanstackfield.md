---
source-updated-at: '2025-03-01T06:06:46.000Z'
translation-updated-at: '2025-04-12T04:15:46.573Z'
id: TanStackField
title: 类 / TanStackField
---
# 类: TanStackField\<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync, TFormOnServer, TSubmitMeta\>

定义于: [tanstack-field.directive.ts:31](https://github.com/TanStack/form/blob/main/packages/angular-form/src/tanstack-field.directive.ts#L31)

## 类型参数

• **TParentData**

• **TName** *继承自* `DeepKeys`\<`TParentData`\>

• **TData** *继承自* `DeepValue`\<`TParentData`, `TName`\>

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

• **TSubmitMeta**

## 实现接口

- `OnInit`
- `OnChanges`
- `OnDestroy`
- `FieldOptions`\<`TParentData`, `TName`, `TData`, `TOnMount`, `TOnChange`, `TOnChangeAsync`, `TOnBlur`, `TOnBlurAsync`, `TOnSubmit`, `TOnSubmitAsync`\>

## 构造函数

### new TanStackField()

```ts
new TanStackField<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync, TFormOnServer, TSubmitMeta>(): TanStackField<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync, TFormOnServer, TSubmitMeta>
```

#### 返回值

[`TanStackField`](tanstackfield.md)\<`TParentData`, `TName`, `TData`, `TOnMount`, `TOnChange`, `TOnChangeAsync`, `TOnBlur`, `TOnBlurAsync`, `TOnSubmit`, `TOnSubmitAsync`, `TFormOnMount`, `TFormOnChange`, `TFormOnChangeAsync`, `TFormOnBlur`, `TFormOnBlurAsync`, `TFormOnSubmit`, `TFormOnSubmitAsync`, `TFormOnServer`, `TSubmitMeta`\>

## 属性

### api

```ts
api: FieldApi<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync, TFormOnServer, TSubmitMeta>;
```

定义于: [tanstack-field.directive.ts:129](https://github.com/TanStack/form/blob/main/packages/angular-form/src/tanstack-field.directive.ts#L129)

***

### asyncAlways?

```ts
optional asyncAlways: boolean;
```

定义于: [tanstack-field.directive.ts:78](https://github.com/TanStack/form/blob/main/packages/angular-form/src/tanstack-field.directive.ts#L78)

如果为 `true`，即使同步验证期间出现错误，也会始终运行异步验证。

#### 实现接口

```ts
FieldOptions.asyncAlways
```

***

### asyncDebounceMs?

```ts
optional asyncDebounceMs: number;
```

定义于: [tanstack-field.directive.ts:77](https://github.com/TanStack/form/blob/main/packages/angular-form/src/tanstack-field.directive.ts#L77)

如果没有传递更具体的防抖时间，则作为异步验证的默认防抖时间。

#### 实现接口

```ts
FieldOptions.asyncDebounceMs
```

***

### defaultMeta?

```ts
optional defaultMeta: Partial<FieldMeta<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync>>;
```

定义于: [tanstack-field.directive.ts:106](https://github.com/TanStack/form/blob/main/packages/angular-form/src/tanstack-field.directive.ts#L106)

一个可选对象，包含字段的默认元数据。

#### 实现接口

```ts
FieldOptions.defaultMeta
```

***

### defaultValue?

```ts
optional defaultValue: NoInfer<TData>;
```

定义于: [tanstack-field.directive.ts:76](https://github.com/TanStack/form/blob/main/packages/angular-form/src/tanstack-field.directive.ts#L76)

字段的可选默认值。

#### 实现接口

```ts
FieldOptions.defaultValue
```

***

### disableErrorFlat?

```ts
optional disableErrorFlat: boolean;
```

定义于: [tanstack-field.directive.ts:127](https://github.com/TanStack/form/blob/main/packages/angular-form/src/tanstack-field.directive.ts#L127)

禁用 `field.errors` 上的 `flat(1)` 操作。如果您希望保持错误结构不变，这很有用。不建议用于大多数用例。

#### 实现接口

```ts
FieldOptions.disableErrorFlat
```

***

### listeners?

```ts
optional listeners: NoInfer<FieldListeners<TParentData, TName, TData>>;
```

定义于: [tanstack-field.directive.ts:105](https://github.com/TanStack/form/blob/main/packages/angular-form/src/tanstack-field.directive.ts#L105)

附加到相应事件的监听器列表

#### 实现接口

```ts
FieldOptions.listeners
```

***

### name

```ts
name: TName;
```

定义于: [tanstack-field.directive.ts:75](https://github.com/TanStack/form/blob/main/packages/angular-form/src/tanstack-field.directive.ts#L75)

字段名称。类型为 `DeepKeys<TParentData>`，以确保名称是父数据集的一个深层键。

#### 实现接口

```ts
FieldOptions.name
```

***

### tanstackField

```ts
tanstackField: FormApi<TParentData, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync, TFormOnServer, TSubmitMeta>;
```

定义于: [tanstack-field.directive.ts:79](https://github.com/TanStack/form/blob/main/packages/angular-form/src/tanstack-field.directive.ts#L79)

***

### unmount()?

```ts
optional unmount: () => void;
```

定义于: [tanstack-field.directive.ts:185](https://github.com/TanStack/form/blob/main/packages/angular-form/src/tanstack-field.directive.ts#L185)

#### 返回值

`void`

***

### validators?

```ts
optional validators: NoInfer<FieldValidators<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync>>;
```

定义于: [tanstack-field.directive.ts:91](https://github.com/TanStack/form/blob/main/packages/angular-form/src/tanstack-field.directive.ts#L91)

传递给字段的验证器列表

#### 实现接口

```ts
FieldOptions.validators
```

## 方法

### ngOnChanges()

```ts
ngOnChanges(): void
```

定义于: [tanstack-field.directive.ts:197](https://github.com/TanStack/form/blob/main/packages/angular-form/src/tanstack-field.directive.ts#L197)

一个回调方法，在默认变更检测器检查数据绑定属性后立即调用（如果至少有一个属性已更改），并且在视图和内容子项检查之前调用。

#### 返回值

`void`

#### 实现接口

```ts
OnChanges.ngOnChanges
```

***

### ngOnDestroy()

```ts
ngOnDestroy(): void
```

定义于: [tanstack-field.directive.ts:193](https://github.com/TanStack/form/blob/main/packages/angular-form/src/tanstack-field.directive.ts#L193)

一个执行自定义清理的回调方法，在指令、管道或服务实例销毁之前立即调用。

#### 返回值

`void`

#### 实现接口

```ts
OnDestroy.ngOnDestroy
```

***

### ngOnInit()

```ts
ngOnInit(): void
```

定义于: [tanstack-field.directive.ts:187](https://github.com/TanStack/form/blob/main/packages/angular-form/src/tanstack-field.directive.ts#L187)

一个回调方法，在默认变更检测器首次检查指令的数据绑定属性后立即调用，并且在任何视图或内容子项检查之前调用。仅在指令实例化时调用一次。

#### 返回值

`void`

#### 实现接口

```ts
OnInit.ngOnInit
```
