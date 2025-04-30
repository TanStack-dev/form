---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T22:06:08.523Z'
id: linked-fields
title: الحقول المرتبطة
---

قد تجد نفسك بحاجة إلى ربط حقلين معًا؛ بحيث يتم التحقق من صحة أحدهما عند تغيير قيمة الحقل الآخر.  
إحدى حالات الاستخدام الشائعة هي عندما يكون لديك حقل `password` وحقل `confirm_password`،  
حيث تريد أن يظهر خطأ في `confirm_password` عندما لا تتطابق قيمته مع `password`،  
بغض النظر عن أي حقل تسبب في تغيير القيمة.

تخيل سير العمل التالي:

- يقوم المستخدم بتحديث حقل تأكيد كلمة المرور.
- يقوم المستخدم بتحديث حقل كلمة المرور الأساسي.

في هذا المثال، سيظل النموذج يحتوي على أخطاء،  
لأن التحقق من صحة حقل "تأكيد كلمة المرور" لم يتم إعادة تشغيله لتمييزه بأنه مقبول.

لحل هذه المشكلة، نحتاج إلى التأكد من إعادة تشغيل التحقق من صحة "تأكيد كلمة المرور" عند تحديث حقل كلمة المرور.  
للقيام بذلك، يمكنك إضافة خاصية `onChangeListenTo` إلى حقل `confirm_password`.

```vue
<script setup lang="ts">
import { useForm } from '@tanstack/vue-form'

const form = useForm({
  defaultValues: {
    password: '',
    confirm_password: '',
  },
  // ...
})
</script>

<template>
  <div>
    <form @submit.prevent.stop="form.handleSubmit">
      <div>
        <form.Field name="password">
          <template v-slot="{ field }">
            <div>Password:</div>
            <input
              :value="field.state.value"
              @input="
                (e) => field.handleChange((e.target as HTMLInputElement).value)
              "
            />
          </template>
        </form.Field>
        <form.Field
          name="confirm_password"
          :validators="{
            onChangeListenTo: ['password'],
            onChange: ({ value, fieldApi }) => {
              if (value !== fieldApi.form.getFieldValue('password')) {
                return 'Passwords do not match'
              }
              return undefined
            },
          }"
        >
          <template v-slot="{ field }">
            <div>Confirm Password:</div>
            <input
              :value="field.state.value"
              @input="
                (e) => field.handleChange((e.target as HTMLInputElement).value)
              "
            />
            <div v-for="(err, index) in field.state.meta.errors" :key="index">
              {{ err }}
            </div>
          </template>
        </form.Field>
      </div>
      <button type="submit">Submit</button>
    </form>
  </div>
</template>
```

هذا ينطبق أيضًا على خاصية `onBlurListenTo`، والتي ستقوم بإعادة تشغيل التحقق من الصحة عند فقدان التركيز (blur) على الحقل.
