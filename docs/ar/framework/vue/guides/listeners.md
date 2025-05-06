---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-05-06T20:29:39.017Z'
id: listeners
title: Side effects for event triggers
---

في الحالات التي ترغب فيها "بالتأثير" أو "الاستجابة" للمحفزات، يوجد واجهة برمجة التطبيقات الخاصة بالمستمع (listener API). على سبيل المثال، إذا كنت كمطور ترغب في إعادة تعيين حقل نموذج نتيجة لتغيير حقل آخر، فستستخدم واجهة برمجة التطبيقات الخاصة بالمستمع.

تخيل سير العمل التالي للمستخدم:

- يختار المستخدم دولة من القائمة المنسدلة.
- ثم يختار محافظة من قائمة منسدلة أخرى.
- يقوم المستخدم بتغيير الدولة المختارة إلى دولة مختلفة.

في هذا المثال، عندما يقوم المستخدم بتغيير الدولة، يجب إعادة تعيين المحافظة المختارة لأنها لم تعد صالحة. باستخدام واجهة برمجة التطبيقات الخاصة بالمستمع، يمكننا الاشتراك في حدث `onChange` وإرسال أمر إعادة تعيين لحقل "المحافظة" عند تشغيل المستمع.

الأحداث التي يمكن "الاستماع" إليها هي:

- `onChange`
- `onBlur`
- `onMount`
- `onSubmit`

```vue
<script setup>
import { useForm } from '@tanstack/vue-form'

const form = useForm({
  defaultValues: {
    country: '',
    province: '',
  },
  // ...
})
</script>

<template>
  <div>
    <form.Field
      name="country"
      :listeners="{
        onChange: ({ value }) => {
          console.log(`Country changed to: ${value}, resetting province`)
          form.setFieldValue('province', '')
        },
      }"
    >
      <template v-slot="{ field }">
        <input
          :value="field.state.value"
          @input="(e) => field.handleChange(e.target.value)"
        />
      </template>
    </form.Field>

    <form.Field name="province">
      <template v-slot="{ field }">
        <input
          :value="field.state.value"
          @input="(e) => field.handleChange(e.target.value)"
        />
      </template>
    </form.Field>
  </div>
</template>
```
