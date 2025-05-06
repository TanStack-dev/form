---
source-updated-at: '2025-04-27T10:57:16.000Z'
translation-updated-at: '2025-05-06T20:18:01.104Z'
id: submission-handling
title: Submission handling
---

## 傳遞額外資料至提交處理

您可能會有多種提交行為類型，例如返回其他頁面或停留在表單上。您可以透過指定 `onSubmitMeta` 屬性來實現此功能。這些元資料將會傳遞至 `onSubmit` 函式。

> 注意：若 `form.handleSubmit()` 在沒有元資料的情況下被呼叫，將會使用預設提供的值。

```angular-ts
import { Component } from '@angular/core';
import { injectForm } from '@tanstack/angular-form';


type FormMeta = {
  submitAction: 'continue' | 'backToMenu' | null;
};

// 呼叫 form.handleSubmit() 時不需要元資料。
// 指定在未傳遞元資料時使用的預設值
const defaultMeta: FormMeta = {
  submitAction: null,
};

@Component({
  selector: 'app-root',
  template: `
    <form (submit)="handleSubmit($event)">
      <button type="submit" (click)="
        handleClick({ submitAction: 'continue' })
      ">提交並繼續</button>
      <button type="submit" (click)="
        handleClick({ submitAction: 'backToMenu' })
      ">提交並返回選單</button>
    </form>
  `,
})
export class AppComponent {
  form = injectForm({
    defaultValues: {
      data: '',
    },
    // 定義提交時預期的元資料值
    onSubmitMeta: defaultMeta,
    onSubmit: async ({ value, meta }) => {
      // 使用透過 handleSubmit 傳遞的值進行操作
      console.log(`選擇的操作 - ${meta.submitAction}`, value);
    },
  });

  handleSubmit(event: SubmitEvent) {
    event.preventDefault();
    event.stopPropagation();
  }

  handleClick(meta: FormMeta) {
    // 覆寫 onSubmitMeta 中指定的預設值
    this.form.handleSubmit(meta);
  }
}
```

## 使用標準結構描述 (Standard Schemas) 轉換資料

雖然 Tanstack Form 提供了[標準結構描述支援 (Standard Schema support)](./validation.md) 進行驗證，但並不會保留結構描述的輸出資料。

傳遞至 `onSubmit` 函式的值始終會是輸入資料。若要接收標準結構描述的輸出資料，請在 `onSubmit` 函式中進行解析：

```tsx
import { z } from 'zod'
// ...

const schema = z.object({
  age: z.string().transform((age) => Number(age)),
})

// Tanstack Form 使用標準結構描述的輸入類型
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
      // 透過結構描述解析以取得轉換後的值
      const result = schema.parse(value)
      const outputAge: number = result.age
    },
  })
}
```
