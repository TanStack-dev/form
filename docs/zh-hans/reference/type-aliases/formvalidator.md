---
id: FormValidator
title: FormValidator
---

<!-- 请勿编辑：此页面是从类型注释自动生成的 -->

# 类型别名: FormValidator\<TFormData, TType, TFn\>

```ts
type FormValidator<TFormData, TType, TFn> = object;
```

定义于: [packages/form-core/src/FormApi.ts:120](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L120)

## 类型参数

• **TFormData**

• **TType**

• **TFn** = `unknown`

## 类型声明

### validate()

#### 参数

##### options

###### value

`TType`

##### fn

`TFn`

#### 返回值

`unknown`

### validateAsync()

#### 参数

##### options

###### value

`TType`

##### fn

`TFn`

#### 返回值

`Promise`\<`unknown`\>
