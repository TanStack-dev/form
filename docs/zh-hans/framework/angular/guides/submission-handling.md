---
source-updated-at: '2025-04-27T10:57:16.000Z'
translation-updated-at: '2025-05-06T20:17:14.989Z'
id: submission-handling
title: Submission handling
---

## 向提交处理传递额外数据

您可能需要处理多种提交行为类型，例如返回其他页面或停留在当前表单。可以通过指定 `onSubmitMeta` 属性来实现这一需求，该元数据将被传递到 `onSubmit` 函数中。

> 注意：如果调用 `form.handleSubmit()` 时未提供元数据，则会使用指定的默认值。

```angular-ts
import { Component } from '@angular/core';
import { injectForm } from '@tanstack/angular-form';


type FormMeta = {
  submitAction: 'continue' | 'backToMenu' | null;
};

// 调用 form.handleSubmit() 时元数据不是必需的
// 指定未传递元数据时使用的默认值
const defaultMeta: FormMeta = {
  submitAction: null,
};

@Component({
  selector: 'app-root',
  template: `
    <form (submit)="handleSubmit($event)">
      <button type="submit" (click)="
        handleClick({ submitAction: 'continue' })
      ">提交并继续</button>
      <button type="submit" (click)="
        handleClick({ submitAction: 'backToMenu' })
      ">提交并返回菜单</button>
    </form>
  `,
})
export class AppComponent {
  form = injectForm({
    defaultValues: {
      data: '',
    },
    // 定义提交时期望的元数据值
    onSubmitMeta: defaultMeta,
    onSubmit: async ({ value, meta }) => {
      // 处理通过 handleSubmit 传递的值
      console.log(`选择的操作 - ${meta.submitAction}`, value);
    },
  });

  handleSubmit(event: SubmitEvent) {
    event.preventDefault();
    event.stopPropagation();
  }

  handleClick(meta: FormMeta) {
    // 覆盖 onSubmitMeta 中指定的默认值
    this.form.handleSubmit(meta);
  }
}
```

## 使用标准模式 (Standard Schema) 转换数据

虽然 Tanstack Form 提供了 [标准模式支持 (Standard Schema)](./validation.md) 用于验证，但不会保留模式输出的数据。

传递给 `onSubmit` 函数的值始终是输入数据。要获取标准模式的输出数据，请在 `onSubmit` 函数中解析它：

```tsx
import { z } from 'zod'
// ...

const schema = z.object({
  age: z.string().transform((age) => Number(age)),
})

// Tanstack Form 使用标准模式的输入类型
const defaultValues: z.input<typeof schema> = {
  age: '13',
}

// ...

@Component({
  // ...
})
export class AppComponent {
  form = injectForm({
    defaultValues,
    validators: {
      onChange: schema,
    },
    onSubmit: ({ value }) => {
      const inputAge: string = value.age
      // 通过模式解析以获取转换后的值
      const result = schema.parse(value)
      const outputAge: number = result.age
    },
  })
}
```
