---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T22:07:13.983Z'
id: linked-fields
title: الحقول المرتبطة
---

قد تجد نفسك بحاجة إلى ربط حقلين معًا؛ بحيث يتم التحقق من صحة أحدهما عند تغيير قيمة الحقل الآخر.  
إحدى حالات الاستخدام الشائعة هي عندما يكون لديك حقل `password` وحقل `confirm_password`،  
حيث تريد أن يظهر خطأ في `confirm_password` عندما لا تتطابق قيمته مع قيمة `password`،  
بغض النظر عن الحقل الذي تسبب في تغيير القيمة.

تخيل سير العمل التالي للمستخدم:

- يقوم المستخدم بتحديث حقل تأكيد كلمة المرور.
- يقوم المستخدم بتحديث حقل كلمة المرور الأساسية.

في هذا المثال، سيظل النموذج يحتوي على أخطاء،  
لأن التحقق من صحة حقل "تأكيد كلمة المرور" لم يتم إعادة تشغيله لتمييزه على أنه مقبول.

لحل هذه المشكلة، نحتاج إلى التأكد من إعادة تشغيل التحقق من صحة "تأكيد كلمة المرور" عند تحديث حقل كلمة المرور.  
للقيام بذلك، يمكنك إضافة خاصية `onChangeListenTo` إلى حقل `confirm_password`.

```tsx
export default function App() {
  const form = createForm(() => ({
    defaultValues: {
      password: '',
      confirm_password: '',
    },
    // ...
  }))

  return (
    <div>
      <form.Field name="password">
        {(field) => (
          <label>
            <div>Password</div>
            <input
              value={field().state.value}
              onChange={(e) => field().handleChange(e.target.value)}
            />
          </label>
        )}
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
        {(field) => (
          <div>
            <label>
              <div>Confirm Password</div>
              <input
                value={field().state.value}
                onChange={(e) => field().handleChange(e.target.value)}
              />
            </label>
            <Index each={field().state.meta.errors}>
              {(err) => <div>{err()}</div>}
            </Index>
          </div>
        )}
      </form.Field>
    </div>
  )
}
```

هذا ينطبق أيضًا على خاصية `onBlurListenTo`، والتي ستقوم بإعادة تشغيل التحقق من الصحة عند فقدان التركيز من الحقل.
