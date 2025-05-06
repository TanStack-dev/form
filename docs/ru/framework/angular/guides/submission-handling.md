---
source-updated-at: '2025-04-27T10:57:16.000Z'
translation-updated-at: '2025-05-06T20:26:45.811Z'
id: submission-handling
title: Submission handling
---

## Передача дополнительных данных при обработке отправки формы

У вас могут быть различные типы поведения при отправке формы, например, возврат на другую страницу или оставаться на форме.  
Это можно реализовать, указав свойство `onSubmitMeta`. Эти метаданные будут переданы в функцию `onSubmit`.

> Примечание: если `form.handleSubmit()` вызывается без метаданных, будут использованы указанные значения по умолчанию.

```angular-ts
import { Component } from '@angular/core';
import { injectForm } from '@tanstack/angular-form';


type FormMeta = {
  submitAction: 'continue' | 'backToMenu' | null;
};

// Метаданные не обязательны для вызова form.handleSubmit().
// Укажите значения по умолчанию, если метаданные не переданы
const defaultMeta: FormMeta = {
  submitAction: null,
};

@Component({
  selector: 'app-root',
  template: `
    <form (submit)="handleSubmit($event)">
      <button type="submit" (click)="
        handleClick({ submitAction: 'continue' })
      ">Отправить и продолжить</button>
      <button type="submit" (click)="
        handleClick({ submitAction: 'backToMenu' })
      ">Отправить и вернуться в меню</button>
    </form>
  `,
})
export class AppComponent {
  form = injectForm({
    defaultValues: {
      data: '',
    },
    // Определите, какие метаданные ожидать при отправке
    onSubmitMeta: defaultMeta,
    onSubmit: async ({ value, meta }) => {
      // Сделайте что-то с переданными значениями через handleSubmit
      console.log(`Выбранное действие - ${meta.submitAction}`, value);
    },
  });

  handleSubmit(event: SubmitEvent) {
    event.preventDefault();
    event.stopPropagation();
  }

  handleClick(meta: FormMeta) {
    // Переопределяет значение по умолчанию, указанное в onSubmitMeta
    this.form.handleSubmit(meta);
  }
}
```

## Преобразование данных с помощью стандартных схем (Standard Schemas)

Хотя Tanstack Form предоставляет [поддержку стандартных схем (Standard Schema)](./validation.md) для валидации, он не сохраняет выходные данные схемы.

Значение, передаваемое в функцию `onSubmit`, всегда будет входными данными. Чтобы получить выходные данные стандартной схемы, обработайте их в функции `onSubmit`:

```tsx
import { z } from 'zod'
// ...

const schema = z.object({
  age: z.string().transform((age) => Number(age)),
})

// Tanstack Form использует входной тип стандартных схем
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
      // Обработайте данные через схему, чтобы получить преобразованное значение
      const result = schema.parse(value)
      const outputAge: number = result.age
    },
  })
}
```
