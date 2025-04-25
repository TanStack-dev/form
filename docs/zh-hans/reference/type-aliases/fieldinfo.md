---
source-updated-at: '2025-03-16T18:32:04.000Z'
translation-updated-at: '2025-04-12T04:12:18.668Z'
id: FieldInfo
title: 类型 / FieldInfo
---
<!-- 请勿编辑：本页面内容由类型注释自动生成 -->

# 类型别名：FieldInfo\<TFormData\>

```ts
type FieldInfo<TFormData> = object;
```

定义于：[packages/form-core/src/FormApi.ts:361](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L361)

表示表单中特定字段信息的对象。

## 类型参数

• **TFormData**

## 类型声明

### instance

```ts
instance: 
  | FieldApi<TFormData, any, any, any, any, any, any, any, any, any, any, any, any, any, any, any, any, any, any>
  | null;
```

FieldAPI 的实例。

### validationMetaMap

```ts
validationMetaMap: Record<ValidationErrorMapKeys, ValidationMeta | undefined>;
```

字段验证内部处理的记录。
