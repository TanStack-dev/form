---
source-updated-at: '2025-03-16T18:32:04.000Z'
translation-updated-at: '2025-04-12T04:12:31.859Z'
id: FormOptions
title: 接口 / FormOptions
---
<!-- 请勿编辑：此页面由类型注释自动生成 -->

# 接口: FormOptions\<TFormData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TOnServer, TSubmitMeta\>

定义于: [packages/form-core/src/FormApi.ts:238](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L238)

表示表单配置选项的对象。

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

## 属性

### asyncAlways?

```ts
optional asyncAlways: boolean;
```

定义于: [packages/form-core/src/FormApi.ts:273](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L273)

如果为 true，即使同步验证已产生错误，也始终运行异步验证。默认为 undefined。

***

### asyncDebounceMs?

```ts
optional asyncDebounceMs: number;
```

定义于: [packages/form-core/src/FormApi.ts:277](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L277)

可选的时间（毫秒），用于在触发异步操作前引入延迟。

***

### defaultState?

```ts
optional defaultState: Partial<FormState<TFormData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TOnServer>>;
```

定义于: [packages/form-core/src/FormApi.ts:257](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L257)

表单的默认状态。

***

### defaultValues?

```ts
optional defaultValues: TFormData;
```

定义于: [packages/form-core/src/FormApi.ts:253](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L253)

设置表单的初始值。

***

### onSubmit()?

```ts
optional onSubmit: (props) => any;
```

定义于: [packages/form-core/src/FormApi.ts:300](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L300)

表单提交时调用的函数，当用户提交有效表单后应执行的操作，返回 `any` 或 Promise `Promise<any>`

#### 参数

##### props

###### formApi

[`FormApi`](../classes/formapi.md)\<`TFormData`, `TOnMount`, `TOnChange`, `TOnChangeAsync`, `TOnBlur`, `TOnBlurAsync`, `TOnSubmit`, `TOnSubmitAsync`, `TOnServer`, `TSubmitMeta`\>

###### meta

`TSubmitMeta`

###### value

`TFormData`

#### 返回值

`any`

***

### onSubmitInvalid()?

```ts
optional onSubmitInvalid: (props) => void;
```

定义于: [packages/form-core/src/FormApi.ts:319](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L319)

指定当用户尝试提交无效表单时的操作。

#### 参数

##### props

###### formApi

[`FormApi`](../classes/formapi.md)\<`TFormData`, `TOnMount`, `TOnChange`, `TOnChangeAsync`, `TOnBlur`, `TOnBlurAsync`, `TOnSubmit`, `TOnSubmitAsync`, `TOnServer`, `TSubmitMeta`\>

###### value

`TFormData`

#### 返回值

`void`

***

### onSubmitMeta?

```ts
optional onSubmitMeta: TSubmitMeta;
```

定义于: [packages/form-core/src/FormApi.ts:295](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L295)

onSubmitMeta，从 handleSubmit 处理器传递到 onSubmit 函数 props 的数据

***

### transform?

```ts
optional transform: FormTransform<NoInfer<TFormData>, NoInfer<TOnMount>, NoInfer<TOnChange>, NoInfer<TOnChangeAsync>, NoInfer<TOnBlur>, NoInfer<TOnBlurAsync>, NoInfer<TOnSubmit>, NoInfer<TOnSubmitAsync>, NoInfer<TOnServer>, NoInfer<TSubmitMeta>>;
```

定义于: [packages/form-core/src/FormApi.ts:334](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L334)

***

### validators?

```ts
optional validators: FormValidators<TFormData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync>;
```

定义于: [packages/form-core/src/FormApi.ts:281](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L281)

传递给表单的验证器列表
