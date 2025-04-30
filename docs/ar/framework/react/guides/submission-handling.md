---
source-updated-at: '2025-04-27T10:57:16.000Z'
translation-updated-at: '2025-04-30T22:05:19.866Z'
id: submission-handling
title: معالجة الإرسال
---

## تمرير بيانات إضافية إلى معالجة الإرسال

قد يكون لديك أنواع متعددة من سلوكيات الإرسال، على سبيل المثال، العودة إلى صفحة أخرى أو البقاء على النموذج. يمكنك تحقيق ذلك عن طريق تحديد خاصية `onSubmitMeta`. ستتم تمرير هذه البيانات الوصفية (meta data) إلى دالة `onSubmit`.

> ملاحظة: إذا تم استدعاء `form.handleSubmit()` بدون بيانات وصفية، فسيتم استخدام القيم الافتراضية المقدمة.

```tsx
import { useForm } from '@tanstack/react-form'

type FormMeta = {
  submitAction: 'continue' | 'backToMenu' | null
}

// Metadata is not required to call form.handleSubmit().
// Specify what values to use as default if no meta is passed
const defaultMeta: FormMeta = {
  submitAction: null,
}

function App() {
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

بينما يوفر Tanstack Form دعمًا [لمخططات قياسية (Standard Schemas)](./validation.md) للتحقق من الصحة، إلا أنه لا يحفظ بيانات الإخراج الخاصة بالمخطط.

القيمة التي يتم تمريرها إلى دالة `onSubmit` ستكون دائمًا بيانات الإدخال. لاستقبال بيانات الإخراج من مخطط قياسي، قم بتحليلها في دالة `onSubmit`:

```tsx
const schema = z.object({
  age: z.string().transform((age) => Number(age)),
})

// Tanstack Form uses the input type of Standard Schemas
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
```
