---
source-updated-at: '2025-04-27T10:57:16.000Z'
translation-updated-at: '2025-05-06T20:30:11.180Z'
id: submission-handling
title: Submission handling
---

## تمرير بيانات إضافية إلى معالجة الإرسال

قد يكون لديك أنواع متعددة من سلوكيات الإرسال، على سبيل المثال، العودة إلى صفحة أخرى أو البقاء على النموذج. يمكنك تحقيق ذلك عن طريق تحديد خاصية `onSubmitMeta`. ستتم تمرير هذه البيانات الوصفية (meta data) إلى دالة `onSubmit`.

> ملاحظة: إذا تم استدعاء `form.handleSubmit()` بدون بيانات وصفية، فسيتم استخدام القيم الافتراضية المقدمة.

```angular-ts
import { Component } from '@angular/core';
import { injectForm } from '@tanstack/angular-form';


type FormMeta = {
  submitAction: 'continue' | 'backToMenu' | null;
};

// Metadata is not required to call form.handleSubmit().
// Specify what values to use as default if no meta is passed
const defaultMeta: FormMeta = {
  submitAction: null,
};

@Component({
  selector: 'app-root',
  template: `
    <form (submit)="handleSubmit($event)">
      <button type="submit" (click)="
        handleClick({ submitAction: 'continue' })
      ">Submit and continue</button>
      <button type="submit" (click)="
        handleClick({ submitAction: 'backToMenu' })
      ">Submit and back to menu</button>
    </form>
  `,
})
export class AppComponent {
  form = injectForm({
    defaultValues: {
      data: '',
    },
    // Define what meta values to expect on submission
    onSubmitMeta: defaultMeta,
    onSubmit: async ({ value, meta }) => {
      // Do something with the values passed via handleSubmit
      console.log(`Selected action - ${meta.submitAction}`, value);
    },
  });

  handleSubmit(event: SubmitEvent) {
    event.preventDefault();
    event.stopPropagation();
  }

  handleClick(meta: FormMeta) {
    // Overwrites the default specified in onSubmitMeta
    this.form.handleSubmit(meta);
  }
}
```

## تحويل البيانات باستخدام المخططات القياسية (Standard Schemas)

بينما يوفر Tanstack Form دعمًا [للمخططات القياسية (Standard Schema)](./validation.md) للتحقق من الصحة، إلا أنه لا يحتفظ ببيانات الإخراج الخاصة بالمخطط.

القيمة الممررة إلى دالة `onSubmit` ستكون دائمًا بيانات الإدخال. لاستقبال بيانات الإخراج من المخطط القياسي، يمكنك تحليلها في دالة `onSubmit`:

```tsx
import { z } from 'zod'
// ...

const schema = z.object({
  age: z.string().transform((age) => Number(age)),
})

// Tanstack Form uses the input type of Standard Schemas
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
      // Pass it through the schema to get the transformed value
      const result = schema.parse(value)
      const outputAge: number = result.age
    },
  })
}
```
