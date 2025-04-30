---
source-updated-at: '2025-04-24T18:50:42.000Z'
translation-updated-at: '2025-04-30T22:04:36.269Z'
id: quick-start
title: بدء سريع
---

# بدء سريع

الحد الأدنى للبدء مع TanStack Form هو إنشاء نموذج وإضافة حقل. ضع في الاعتبار أن هذا المثال لا يتضمن أي تحقق من الصحة أو معالجة للأخطاء... حتى الآن.

```svelte
<script>
  import { createForm } from '@tanstack/svelte-form'

  const form = createForm(() => ({
    defaultValues: {
      fullName: '',
    },
    onSubmit: async ({ value }) => {
      // Do something with form data
      console.log(value)
    },
  }))
</script>

<div>
  <h1>Simple Form Example</h1>
  <form
    onsubmit={(e) => {
      e.preventDefault()
      e.stopPropagation()
      form.handleSubmit()
    }}
  >
    <div>
      <form.Field name="fullName">
        {#snippet children(field)}
          <input
            name={field.name}
            value={field.state.value}
            onblur={field.handleBlur}
            oninput={(e) => field.handleChange(e.target.value)}
          />
        {/snippet}
      </form.Field>
    </div>
    <button type="submit">Submit</button>
  </form>
</div>
```

من هنا، ستكون جاهزًا لاستكشاف جميع الميزات الأخرى لـ TanStack Form!
