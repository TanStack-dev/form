---
source-updated-at: '2025-04-27T10:57:16.000Z'
translation-updated-at: '2025-05-06T20:29:16.707Z'
id: submission-handling
title: Submission handling
---

## تمرير بيانات إضافية إلى معالجة الإرسال

قد يكون لديك أنواع متعددة من سلوكيات الإرسال، على سبيل المثال، العودة إلى صفحة أخرى أو البقاء في النموذج. يمكنك تحقيق ذلك عن طريق تحديد خاصية `onSubmitMeta`. سيتم تمرير هذه البيانات الوصفية (meta data) إلى دالة `onSubmit`.

> ملاحظة: إذا تم استدعاء `form.handleSubmit()` بدون بيانات وصفية، فسيتم استخدام القيم الافتراضية المقدمة.

```tsx
import { createForm } from '@tanstack/solid-form'

type FormMeta = {
  submitAction: 'continue' | 'backToMenu' | null
}

// Metadata is not required to call form.handleSubmit().
// Specify what values to use as default if no meta is passed
const defaultMeta: FormMeta = {
  submitAction: null,
}

export default function App() {
  const form = createForm(() => ({
    defaultValues: {
      data: '',
    },
    // Define what meta values to expect on submission
    onSubmitMeta: defaultMeta,
    onSubmit: async ({ value, meta }) => {
      // Do something with the values passed via handleSubmit
      console.log(`Selected action - ${meta.submitAction}`, value)
    },
  }))

  return (
    <form
      onSubmit={(e) => {
        e.preventDefault()
        e.stopPropagation()
      }}
    >
      {/* ... */}
      <button
        type="submit"
        // Overwrites the default specified in onSubmitMeta
        onClick={() => form.handleSubmit({ submitAction: 'continue' })}
      >
        Submit and continue
      </button>
      <button
        type="submit"
        onClick={() => form.handleSubmit({ submitAction: 'backToMenu' })}
      >
        Submit and back to menu
      </button>
    </form>
  )
}
```

## تحويل البيانات باستخدام مخططات قياسية (Standard Schemas)

بينما يوفر Tanstack Form دعمًا [لمخططات قياسية (Standard Schema)](./validation.md) للتحقق من الصحة، فإنه لا يحافظ على بيانات الإخراج الخاصة بالمخطط.

القيمة التي يتم تمريرها إلى دالة `onSubmit` ستكون دائمًا بيانات الإدخال. لاستقبال بيانات الإخراج من مخطط قياسي، يمكنك تحليلها في دالة `onSubmit`:

```tsx
import { createForm } from '@tanstack/solid-form'
import { z } from 'zod'

// ...

const schema = z.object({
  age: z.string().transform((age) => Number(age)),
})

// Tanstack Form uses the input type of Standard Schemas
const defaultValues: z.input<typeof schema> = {
  age: '13',
}

const form = createForm(() => ({
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
}))
```
