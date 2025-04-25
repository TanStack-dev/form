---
id: standardSchemaValidators
title: standardSchemaValidators
---

<!-- 请勿编辑：此页面是从类型注释自动生成的 -->

# 变量: standardSchemaValidators

```ts
const standardSchemaValidators: object;
```

定义于: [packages/form-core/src/standardSchemaValidator.ts:48](https://github.com/TanStack/form/blob/main/packages/form-core/src/standardSchemaValidator.ts#L48)

## 类型声明

### validate()

#### 参数

##### \_\_namedParameters

[`TStandardSchemaValidatorValue`](../type-aliases/tstandardschemavalidatorvalue.md)\<`unknown`\>

##### schema

[`StandardSchemaV1`](../type-aliases/standardschemav1.md)

#### 返回值

  \| `undefined`
  \| readonly [`StandardSchemaV1Issue`](../interfaces/standardschemav1issue.md)[]
  \| \{
  `fields`: \{\};
  `form`: \{\};
 \}

### validateAsync()

#### 参数

##### \_\_namedParameters

[`TStandardSchemaValidatorValue`](../type-aliases/tstandardschemavalidatorvalue.md)\<`unknown`\>

##### schema

[`StandardSchemaV1`](../type-aliases/standardschemav1.md)

#### 返回值

`Promise`\<
  \| `undefined`
  \| readonly [`StandardSchemaV1Issue`](../interfaces/standardschemav1issue.md)[]
  \| \{
  `fields`: \{\};
  `form`: \{\};
 \}\>
