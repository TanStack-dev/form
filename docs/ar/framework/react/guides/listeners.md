---
source-updated-at: '2025-04-27T15:54:55.000Z'
translation-updated-at: '2025-04-30T22:05:23.537Z'
id: listeners
title: المستمعون
---

في الحالات التي تريد فيها "التأثير" أو "الرد" على المحفزات، هناك واجهة برمجة التطبيقات الخاصة بالمستمعين (listener API). على سبيل المثال، إذا كنت كمطور تريد إعادة تعيين حقل في النموذج نتيجة لتغيير حقل آخر، فستستخدم واجهة برمجة التطبيقات الخاصة بالمستمعين.

تخيل سير العمل التالي للمستخدم:

- يختار المستخدم دولة من القائمة المنسدلة.
- ثم يختار محافظة من قائمة منسدلة أخرى.
- يقوم المستخدم بتغيير الدولة المختارة إلى دولة مختلفة.

في هذا المثال، عندما يقوم المستخدم بتغيير الدولة، يجب إعادة تعيين المحافظة المختارة لأنها لم تعد صالحة. باستخدام واجهة برمجة التطبيقات الخاصة بالمستمعين، يمكننا الاشتراك في حدث `onChange` وإرسال أمر إعادة تعيين إلى حقل "المحافظة" عند تشغيل المستمع.

الأحداث التي يمكن "الاستماع" إليها هي:

- `onChange`
- `onBlur`
- `onMount`
- `onSubmit`

```tsx
function App() {
  const form = useForm({
    defaultValues: {
      country: '',
      province: '',
    },
    // ...
  })

  return (
    <div>
      <form.Field
        name="country"
        listeners={{
          onChange: ({ value }) => {
            console.log(`Country changed to: ${value}, resetting province`)
            form.setFieldValue('province', '')
          },
        }}
      >
        {(field) => (
          <label>
            <div>Country</div>
            <input
              value={field.state.value}
              onChange={(e) => field.handleChange(e.target.value)}
            />
          </label>
        )}
      </form.Field>

      <form.Field name="province">
        {(field) => (
          <label>
            <div>Province</div>
            <input
              value={field.state.value}
              onChange={(e) => field.handleChange(e.target.value)}
            />
          </label>
        )}
      </form.Field>
    </div>
  )
}
```

### إزالة الاهتزاز المدمج (Built-in Debouncing)

إذا كنت تقوم بطلب API داخل مستمع، فقد ترغب في إزالة الاهتزاز (debounce) للمكالمات لأن ذلك قد يؤدي إلى مشاكل في الأداء.
نحن نتيح طريقة سهلة لإزالة الاهتزاز للمستمعين عن طريق إضافة `onChangeDebounceMs` أو `onBlurDebounceMs`.

```tsx
<form.Field
  name="country"
  listeners={{
    onChangeDebounceMs: 500, // إزالة اهتزاز لمدة 500 مللي ثانية
    onChange: ({ value }) => {
      console.log(`Country changed to: ${value} without a change within 500ms, resetting province`)
      form.setFieldValue('province', '')
    },
  }}
>
  {(field) => (
    /* ... */
  )}
</form.Field>
```

### مستمعو النموذج (Form listeners)

على مستوى أعلى، تتوفر المستمعون أيضًا على مستوى النموذج، مما يمنحك الوصول إلى أحداث `onMount` و `onSubmit`، ويسمح بنشر `onChange` و `onBlur` إلى جميع الحقول الفرعية للنموذج. يمكن أيضًا إزالة الاهتزاز لمستمعي النموذج بنفس الطريقة التي تمت مناقشتها سابقًا.

مستمعو `onMount` و `onSubmit` لديهم الخصائص التالية:

- `formApi`

مستمعو `onChange` و `onBlur` لديهم الوصول إلى:

- `fieldApi`
- `formApi`

```tsx
const form = useForm({
  listeners: {
    onMount: ({ formApi }) => {
      // خدمة تسجيل مخصصة
      loggingService('mount', formApi.state.values)
    },

    onChange: ({ formApi, fieldApi }) => {
      // منطق الحفظ التلقائي
      if (formApi.state.isValid) {
        formApi.handleSubmit()
      }

      // يمثل fieldApi الحقل الذي أطلق الحدث.
      console.log(fieldApi.name, fieldApi.state.value)
    },
    onChangeDebounceMs: 500,
  },
})
```
