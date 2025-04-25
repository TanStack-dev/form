---
id: BaseFormState
title: BaseFormState
---

<!-- 请勿编辑：此页面是从类型注释自动生成的 -->

# 类型别名: BaseFormState\<TFormData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TOnServer\>

```ts
type BaseFormState<TFormData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TOnServer> = object;
```

定义于: [packages/form-core/src/FormApi.ts:395](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L395)

表示表单当前状态的对象。

## 类型参数

• **TFormData**

• **TOnMount** *extends* `undefined` \| `FormValidateOrFn`\<`TFormData`\>

• **TOnChange** *extends* `undefined` \| `FormValidateOrFn`\<`TFormData`\>

• **TOnChangeAsync** *extends* `undefined` \| `FormAsyncValidateOrFn`\<`TFormData`\>

• **TOnBlur** *extends* `undefined` \| `FormValidateOrFn`\<`TFormData`\>

• **TOnBlurAsync** *extends* `undefined` \| `FormAsyncValidateOrFn`\<`TFormData`\>

• **TOnSubmit** *extends* `undefined` \| `FormValidateOrFn`\<`TFormData`\>

• **TOnSubmitAsync** *extends* `undefined` \| `FormAsyncValidateOrFn`\<`TFormData`\>

• **TOnServer** *extends* `undefined` \| `FormAsyncValidateOrFn`\<`TFormData`\>

## 类型声明

### \_force\_re\_eval?

```ts
optional _force_re_eval: boolean;
```

@private，用于在选项更改时强制重新评估表单状态

### errorMap

```ts
errorMap: FormValidationErrorMap<UnwrapFormValidateOrFn<TOnMount>, UnwrapFormValidateOrFn<TOnChange>, UnwrapFormAsyncValidateOrFn<TOnChangeAsync>, UnwrapFormValidateOrFn<TOnBlur>, UnwrapFormAsyncValidateOrFn<TOnBlurAsync>, UnwrapFormValidateOrFn<TOnSubmit>, UnwrapFormAsyncValidateOrFn<TOnSubmitAsync>, UnwrapFormAsyncValidateOrFn<TOnServer>>;
```

表单本身的错误映射。

### fieldMetaBase

```ts
fieldMetaBase: Record<DeepKeys<TFormData>, AnyFieldMetaBase>;
```

表单中每个字段的元数据记录，不包括派生属性，如 `errors` 等

### isSubmitSuccessful

```ts
isSubmitSuccessful: boolean;
```

指示最后一次提交是否成功的布尔值。

### isSubmitted

```ts
isSubmitted: boolean;
```

指示 `onSubmit` 函数是否已成功完成的布尔值。

在每次新的提交尝试时返回到 `false`。

注意：您可以使用 isSubmitting 来检查表单当前是否正在提交。

### isSubmitting

```ts
isSubmitting: boolean;
```

指示在调用 `handleSubmit` 后表单当前是否正在提交过程中的布尔值。

在提交完成时因以下原因之一返回到 `false`：
- 验证步骤返回了错误。
- `onSubmit` 函数已完成。

注意：如果您在 `onSubmit` 函数中运行异步操作，请确保等待它们完成，以确保 `isSubmitting` 仅在异步操作完成时设置为 `false`。

这对于在提交过程中显示加载指示器或禁用表单输入很有用。

### isValidating

```ts
isValidating: boolean;
```

指示表单或其任何字段当前是否正在验证的布尔值。

### submissionAttempts

```ts
submissionAttempts: number;
```

用于跟踪提交尝试次数的计数器。

### validationMetaMap

```ts
validationMetaMap: Record<ValidationErrorMapKeys, ValidationMeta | undefined>;
```

用于跟踪表单中验证逻辑的内部机制。

### values

```ts
values: TFormData;
```

表单字段的当前值。
