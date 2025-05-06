---
source-updated-at: '2025-04-27T10:57:16.000Z'
translation-updated-at: '2025-05-06T20:28:45.484Z'
id: submission-handling
title: Submission handling
---

## تمرير بيانات إضافية إلى معالجة الإرسال

قد يكون لديك أنواع متعددة من سلوك الإرسال، على سبيل المثال، العودة إلى صفحة أخرى أو البقاء على النموذج.  
يمكنك تحقيق ذلك عن طريق تحديد خاصية `onSubmitMeta`. ستتم تمرير هذه البيانات الوصفية (meta data) إلى دالة `onSubmit`.

> ملاحظة: إذا تم استدعاء `form.handleSubmit()` بدون بيانات وصفية، فسيتم استخدام القيم الافتراضية المقدمة.

```vue
<script setup lang="ts">
import { useForm } from '@tanstack/vue-form'

type FormMeta = {
  submitAction: 'continue' | 'backToMenu' | null
}

// Metadata is not required to call form.handleSubmit().
// Specify what values to use as default if no meta is passed
const defaultMeta: FormMeta = {
  submitAction: null,
}

const form = useForm({
  defaultValues: {
    data: '',
  },
  // Define what meta values to expect on submission
  onSubmitMeta: defaultMeta,
  onSubmit: async ({ value, meta }) => {
    // Do something with the values passed via handleSubmit
    console.log(`Selected action - ${meta.submitAction}`, value)
  },
})
</script>

<template>
  <form @submit.prevent.stop="">
    <button
      type="submit"
      @click="
        () => {
          // Overwrites the default specified in onSubmitMeta
          form.handleSubmit({ submitAction: 'continue' })
        }
      "
    >
      Submit and continue
    </button>
    <button
      type="submit"
      @click="() => form.handleSubmit({ submitAction: 'backToMenu' })"
    >
      Submit and back to menu
    </button>
  </form>
</template>
```

## تحويل البيانات باستخدام المخططات القياسية (Standard Schemas)

بينما يوفر Tanstack Form [دعمًا للمخططات القياسية (Standard Schema)](./validation.md) للتحقق من الصحة، فإنه لا يحفظ بيانات الإخراج الخاصة بالمخطط.

القيمة التي يتم تمريرها إلى دالة `onSubmit` ستكون دائمًا بيانات الإدخال. لاستقبال بيانات الإخراج من المخطط القياسي، قم بتحليلها داخل دالة `onSubmit`:

```vue
<script setup lang="ts">
import { z } from 'zod'
import { useForm } from '@tanstack/vue-form'

const schema = z.object({
  age: z.string().transform((age) => Number(age)),
})

// Tanstack form uses the input type of Standard Schemas
const defaultValues: z.input<typeof schema> = {
  age: '13',
}

const form = useForm({
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
</script>
<template>
  <!-- ... -->
</template>
```
