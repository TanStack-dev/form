---
id: shallow
title: shallow
---

<!-- 请勿编辑：此页面是从类型注释自动生成的 -->

# 函数: shallow()

```ts
function shallow<T>(objA, objB): boolean
```

定义于: [packages/form-core/src/utils.ts:334](https://github.com/TanStack/form/blob/main/packages/form-core/src/utils.ts#L334)

执行两个对象之间的浅层比较，判断它们是否相等。

## 类型参数

• **T**

## 参数

### objA

`T`

要比较的第一个对象。

### objB

`T`

要比较的第二个对象。

## 返回值

`boolean`

如果两个对象浅层相等则返回`true`，否则返回`false`。
