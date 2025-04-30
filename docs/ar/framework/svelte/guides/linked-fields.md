---
source-updated-at: '2025-04-24T18:50:42.000Z'
translation-updated-at: '2025-04-30T22:08:01.307Z'
id: linked-fields
title: الحقول المرتبطة
---

قد تجد نفسك بحاجة إلى ربط حقلين معًا؛ بحيث يتم التحقق من صحة أحدهما عند تغيير قيمة الحقل الآخر.  
إحدى حالات الاستخدام الشائعة هي عندما يكون لديك حقل `password` وحقل `confirm_password`،  
حيث تريد أن يظهر خطأ في حقل `confirm_password` عندما لا تتطابق قيمته مع حقل `password`،  
بغض النظر عن أي حقل تسبب في تغيير القيمة.

تخيل سير العمل التالي للمستخدم:

- يقوم المستخدم بتحديث حقل تأكيد كلمة المرور.
- يقوم المستخدم بتحديث حقل كلمة المرور الأصلية.

في هذا المثال، سيظل النموذج يحتوي على أخطاء،  
لأن التحقق من صحة حقل "تأكيد كلمة المرور" لم يتم إعادة تشغيله لتمييزه كمقبول.

لحل هذه المشكلة، نحتاج إلى التأكد من إعادة تشغيل التحقق من صحة حقل "تأكيد كلمة المرور" عند تحديث حقل كلمة المرور.  
للقيام بذلك، يمكنك إضافة خاصية `onChangeListenTo` إلى حقل `confirm_password`.

```svelte
<script>
  import { createForm } from '@tanstack/svelte-form'

  const form = createForm(() => ({
    defaultValues: {
      password: '',
      confirm_password: '',
    },
    // ...
  }))
</script>

<div>
  <form.Field name="password">
    {#snippet children(field)}
      <label>
        <div>Password</div>
        <input
          value={field.state.value}
          onChange={(e) => field.handleChange(e.target.value)}
        />
      </label>
    {/snippet}
  </form.Field>
  <form.Field
    name="confirm_password"
    validators={{
      onChangeListenTo: ['password'],
      onChange: ({ value, fieldApi }) => {
        if (value !== fieldApi.form.getFieldValue('password')) {
          return 'Passwords do not match'
        }
        return undefined
      },
    }}
  >
    {#snippet children(field)}
      <div>
        <label>
          <div>Confirm Password</div>
          <input
            value={field.state.value}
            onchange={(e) => field.handleChange(e.target.value)}
          />
        </label>
        {#each field.state.meta.errors as err}
          <div>{err}</div>
        {/each}
      </div>
    {/snippet}
  </form.Field>
</div>
```

هذا ينطبق أيضًا على خاصية `onBlurListenTo`، والتي ستقوم بإعادة تشغيل التحقق من الصحة عند فقدان التركيز (blur) على الحقل.
