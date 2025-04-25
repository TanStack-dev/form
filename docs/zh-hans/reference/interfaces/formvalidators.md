---
source-updated-at: '2025-03-16T18:32:04.000Z'
translation-updated-at: '2025-04-12T04:12:18.049Z'
id: FormValidators
title: 接口 / FormValidators
---
# 接口: FormValidators\<TFormData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync\>

定义于: [packages/form-core/src/FormApi.ts:150](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L150)

## 类型参数

• **TFormData**

• **TOnMount** *继承自* `undefined` \| `FormValidateOrFn`\<`TFormData`\>

• **TOnChange** *继承自* `undefined` \| `FormValidateOrFn`\<`TFormData`\>

• **TOnChangeAsync** *继承自* `undefined` \| `FormAsyncValidateOrFn`\<`TFormData`\>

• **TOnBlur** *继承自* `undefined` \| `FormValidateOrFn`\<`TFormData`\>

• **TOnBlurAsync** *继承自* `undefined` \| `FormAsyncValidateOrFn`\<`TFormData`\>

• **TOnSubmit** *继承自* `undefined` \| `FormValidateOrFn`\<`TFormData`\>

• **TOnSubmitAsync** *继承自* `undefined` \| `FormAsyncValidateOrFn`\<`TFormData`\>

## 属性

### onBlur?

```ts
optional onBlur: TOnBlur;
```

定义于: [packages/form-core/src/FormApi.ts:179](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L179)

可选函数，当字段失去焦点时验证表单数据，返回一个 `FormValidationError`

***

### onBlurAsync?

```ts
optional onBlurAsync: TOnBlurAsync;
```

定义于: [packages/form-core/src/FormApi.ts:183](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L183)

可选 onBlur 异步验证方法，当字段失去焦点时返回 `FormValidationError` 或 `Promise<FormValidationError>`

***

### onBlurAsyncDebounceMs?

```ts
optional onBlurAsyncDebounceMs: number;
```

定义于: [packages/form-core/src/FormApi.ts:187](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L187)

默认的毫秒数，如果设置为大于 0 的数字，将按此毫秒数对异步验证事件进行防抖处理。

***

### onChange?

```ts
optional onChange: TOnChange;
```

定义于: [packages/form-core/src/FormApi.ts:167](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L167)

可选函数，每当值变化时检查数据的有效性

***

### onChangeAsync?

```ts
optional onChangeAsync: TOnChangeAsync;
```

定义于: [packages/form-core/src/FormApi.ts:171](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L171)

可选的 onChange 异步对应方法，适用于可能涉及服务器请求的更复杂验证逻辑。

***

### onChangeAsyncDebounceMs?

```ts
optional onChangeAsyncDebounceMs: number;
```

定义于: [packages/form-core/src/FormApi.ts:175](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L175)

默认的毫秒数，如果设置为大于 0 的数字，将按此毫秒数对异步验证事件进行防抖处理。

***

### onMount?

```ts
optional onMount: TOnMount;
```

定义于: [packages/form-core/src/FormApi.ts:163](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L163)

可选函数，在组件挂载时立即触发。

***

### onSubmit?

```ts
optional onSubmit: TOnSubmit;
```

定义于: [packages/form-core/src/FormApi.ts:188](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L188)

***

### onSubmitAsync?

```ts
optional onSubmitAsync: TOnSubmitAsync;
```

定义于: [packages/form-core/src/FormApi.ts:189](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L189)
