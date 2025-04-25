---
id: StandardSchemaV1Issue
title: StandardSchemaV1Issue
---

<!-- 请勿编辑：此页面是从类型注释自动生成的 -->

# 接口: StandardSchemaV1Issue

定义于: [packages/form-core/src/standardSchemaValidator.ts:144](https://github.com/TanStack/form/blob/main/packages/form-core/src/standardSchemaValidator.ts#L144)

失败输出的问题接口。

## 属性

### message

```ts
readonly message: string;
```

定义于: [packages/form-core/src/standardSchemaValidator.ts:148](https://github.com/TanStack/form/blob/main/packages/form-core/src/standardSchemaValidator.ts#L148)

问题的错误消息。

***

### path?

```ts
readonly optional path: readonly (PropertyKey | StandardSchemaV1PathSegment)[];
```

定义于: [packages/form-core/src/standardSchemaValidator.ts:152](https://github.com/TanStack/form/blob/main/packages/form-core/src/standardSchemaValidator.ts#L152)

问题的路径（如果有）。
