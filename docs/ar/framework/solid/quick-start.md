---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T22:04:20.609Z'
id: quick-start
title: بدء سريع
---

الحد الأدنى للبدء مع TanStack Form هو إنشاء نموذج وإضافة حقل. ضع في اعتبارك أن هذا المثال لا يتضمن أي تحقق من الصحة أو معالجة للأخطاء... حتى الآن.

```tsx
import { createForm } from '@tanstack/solid-form'

function App() {
  const form = createForm(() => ({
    defaultValues: {
      fullName: '',
    },
    onSubmit: async ({ value }) => {
      // افعل شيئًا ببيانات النموذج
      console.log(value)
    },
  }))

  return (
    <div>
      <h1>مثال نموذج بسيط</h1>
      <form
        onSubmit={(e) => {
          e.preventDefault()
          e.stopPropagation()
          form.handleSubmit()
        }}
      >
        <div>
          <form.Field
            name="fullName"
            children={(field) => (
              <input
                name={field().name}
                value={field().state.value}
                onBlur={field().handleBlur}
                onInput={(e) => field().handleChange(e.target.value)}
              />
            )}
          />
        </div>
        <button type="submit">إرسال</button>
      </form>
    </div>
  )
}
```

من هنا، ستكون جاهزًا لاستكشاف جميع الميزات الأخرى لـ TanStack Form!
