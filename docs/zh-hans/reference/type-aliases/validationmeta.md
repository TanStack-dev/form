---
source-updated-at: '2025-03-16T18:32:04.000Z'
translation-updated-at: '2025-04-12T04:12:37.247Z'
id: ValidationMeta
title: 类型 / ValidationMeta
---
<!-- 请勿编辑：本页面内容由类型注释自动生成 -->

# 类型别名：ValidationMeta

```ts
type ValidationMeta = object;
```

定义于：[packages/form-core/src/FormApi.ts:351](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L351)

表示字段验证元数据的对象。不适用于公开使用。

## 类型声明

### lastAbortController

```ts
lastAbortController: AbortController;
```

存储在内存中的中止控制器 (AbortController)，用于取消先前的异步验证尝试。
