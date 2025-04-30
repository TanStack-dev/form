---
source-updated-at: '2025-03-31T17:05:18.000Z'
translation-updated-at: '2025-04-30T22:04:20.046Z'
id: quick-start
title: بدء سريع
---

# البداية السريعة

الحد الأدنى للبدء مع TanStack Form هو إنشاء نموذج وإضافة حقل. ضع في الاعتبار أن هذا المثال لا يتضمن أي تحقق من الصحة أو معالجة للأخطاء... حتى الآن.

```vue
<!-- App.vue -->
<script setup>
import { useForm } from '@tanstack/vue-form'

const form = useForm({
  defaultValues: {
    fullName: '',
  },
  onSubmit: async ({ value }) => {
    // Do something with form data
    console.log(value)
  },
})
</script>

<template>
  <div>
    <form @submit.prevent.stop="form.handleSubmit">
      <div>
        <form.Field name="fullName">
          <template v-slot="{ field }">
            <input
              :name="field.name"
              :value="field.state.value"
              @blur="field.handleBlur"
              @input="(e) => field.handleChange((e.target as HTMLInputElement).value)"
            />
          </template>
        </form.Field>
      </div>
      <button type="submit">Submit</button>
    </form>
  </div>
</template>
```

من هنا، ستكون جاهزًا لاستكشاف جميع الميزات الأخرى لـ TanStack Form!
