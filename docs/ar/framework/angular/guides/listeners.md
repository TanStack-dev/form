---
source-updated-at: '2024-11-25T20:59:59.000Z'
translation-updated-at: '2025-05-06T22:54:11.871Z'
id: listeners
title: Side effects for event triggers
---

في الحالات التي ترغب فيها "بالتأثير" أو "الاستجابة" للمحفزات، يوجد واجهة برمجة التطبيقات الخاصة بالمستمع (listener API). على سبيل المثال، إذا كنت كمطور ترغب في إعادة تعيين حقل نموذج نتيجة لتغيير حقل آخر، فستستخدم واجهة برمجة التطبيقات الخاصة بالمستمع.

تخيل سير العمل التالي للمستخدم:

- يختار المستخدم دولة من القائمة المنسدلة.
- ثم يختار محافظة من قائمة منسدلة أخرى.
- يقوم المستخدم بتغيير الدولة المختارة إلى دولة مختلفة.

في هذا المثال، عندما يقوم المستخدم بتغيير الدولة، يجب إعادة تعيين المحافظة المختارة لأنها لم تعد صالحة. باستخدام واجهة برمجة التطبيقات الخاصة بالمستمع، يمكننا الاشتراك في حدث `onChange` وإرسال أمر إعادة تعيين إلى حقل "المحافظة" عند تشغيل المستمع.

الأحداث التي يمكن "الاستماع" إليها هي:

- `onChange`
- `onBlur`
- `onMount`
- `onSubmit`

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container
      [tanstackField]="form"
      name="country"
      [listeners]="{
        onChange: onCountryChange
      }"
      #country="field"
    ></ng-container>

    <ng-container
      [tanstackField]="form"
      name="province"
      #province="field"
    ></ng-container>
  `,
})

export class AppComponent {
  form = injectForm({
    defaultValues: {
      country: '',
      province: '',
    },
  })

  onCountryChange: FieldListenerFn<any, any, any, any, string> = ({
    value,
  }) => {
    console.log(`Country changed to: ${value}, resetting province`)
    this.form.setFieldValue('province', '')
  }
}
```
