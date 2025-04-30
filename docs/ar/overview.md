---
source-updated-at: '2025-04-29T10:21:58.000Z'
translation-updated-at: '2025-04-30T22:04:54.331Z'
id: overview
title: نظرة عامة
---

# نظرة عامة

TanStack Form هو الحل الأمثل للتعامل مع النماذج في تطبيقات الويب، حيث يوفر نهجًا قويًا ومرنًا لإدارة النماذج. مصمم بدعم من الدرجة الأولى لـ TypeScript ومكونات واجهة مستخدم بلا رأس (headless UI) وتصميم مستقل عن إطار العمل، فهو يبسط التعامل مع النماذج ويضمن تجربة سلسة عبر أطر عمل الواجهة الأمامية المختلفة.

## الدافع

معظم أطر عمل الويب لا توفر حلاً شاملاً للتعامل مع النماذج، مما يدفع المطورين إلى إنشاء تطبيقات مخصصة خاصة بهم أو الاعتماد على مكتبات أقل كفاءة. يؤدي هذا غالبًا إلى نقص في الاتساق وضعف في الأداء وزيادة في وقت التطوير. يهدف TanStack Form إلى معالجة هذه التحديات من خلال توفير حل متكامل لإدارة النماذج يكون قويًا وسهل الاستخدام.

مع TanStack Form، يمكن للمطورين مواجهة التحديات الشائعة المتعلقة بالنماذج مثل:

- ربط البيانات التفاعلي (Reactive data binding) وإدارة الحالة (state management)
- التحقق المعقد من الصحة (Complex validation) والتعامل مع الأخطاء (error handling)
- إمكانية الوصول (Accessibility) والتصميم المتجاوب (responsive design)
- التدويل (Internationalization) والتوطين (localization)
- التوافق عبر المنصات (Cross-platform compatibility) والتخصيص التصميمي (custom styling)

من خلال توفير حل كامل لهذه التحديات، يمكن لـ TanStack Form تمكين المطورين من بناء نماذج قوية وسهلة الاستخدام بسهولة.

## كفى حديثًا، أرني بعض الأكواد!

في المثال أدناه، يمكنك رؤية TanStack Form في العمل مع محول إطار عمل React:

[فتح في CodeSandbox](https://codesandbox.io/s/github/tanstack/form/tree/main/examples/react/simple)

```tsx
import * as React from 'react'
import { createRoot } from 'react-dom/client'
import { useForm } from '@tanstack/react-form'
import type { AnyFieldApi } from '@tanstack/react-form'

function FieldInfo({ field }: { field: AnyFieldApi }) {
  return (
    <>
      {field.state.meta.isTouched && !field.state.meta.isValid ? (
        <em>{field.state.meta.errors.join(', ')}</em>
      ) : null}
      {field.state.meta.isValidating ? 'Validating...' : null}
    </>
  )
}

export default function App() {
  const form = useForm({
    defaultValues: {
      firstName: '',
      lastName: '',
    },
    onSubmit: async ({ value }) => {
      // Do something with form data
      console.log(value)
    },
  })

  return (
    <div>
      <h1>Simple Form Example</h1>
      <form
        onSubmit={(e) => {
          e.preventDefault()
          e.stopPropagation()
          form.handleSubmit()
        }}
      >
        <div>
          {/* A type-safe field component*/}
          <form.Field
            name="firstName"
            validators={{
              onChange: ({ value }) =>
                !value
                  ? 'A first name is required'
                  : value.length < 3
                    ? 'First name must be at least 3 characters'
                    : undefined,
              onChangeAsyncDebounceMs: 500,
              onChangeAsync: async ({ value }) => {
                await new Promise((resolve) => setTimeout(resolve, 1000))
                return (
                  value.includes('error') && 'No "error" allowed in first name'
                )
              },
            }}
            children={(field) => {
              // Avoid hasty abstractions. Render props are great!
              return (
                <>
                  <label htmlFor={field.name}>First Name:</label>
                  <input
                    id={field.name}
                    name={field.name}
                    value={field.state.value}
                    onBlur={field.handleBlur}
                    onChange={(e) => field.handleChange(e.target.value)}
                  />
                  <FieldInfo field={field} />
                </>
              )
            }}
          />
        </div>
        <div>
          <form.Field
            name="lastName"
            children={(field) => (
              <>
                <label htmlFor={field.name}>Last Name:</label>
                <input
                  id={field.name}
                  name={field.name}
                  value={field.state.value}
                  onBlur={field.handleBlur}
                  onChange={(e) => field.handleChange(e.target.value)}
                />
                <FieldInfo field={field} />
              </>
            )}
          />
        </div>
        <form.Subscribe
          selector={(state) => [state.canSubmit, state.isSubmitting]}
          children={([canSubmit, isSubmitting]) => (
            <button type="submit" disabled={!canSubmit}>
              {isSubmitting ? '...' : 'Submit'}
            </button>
          )}
        />
      </form>
    </div>
  )
}

const rootElement = document.getElementById('root')!

createRoot(rootElement).render(<App />)
```

## لقد أقنعتني، فماذا الآن؟

- تعلم TanStack Form بالسرعة التي تناسبك من خلال [دليل المشي التفصيلي](../installation) و[مرجع API](../reference/classes/formapi) الشامل لدينا
