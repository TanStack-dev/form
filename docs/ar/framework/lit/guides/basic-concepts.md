---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T22:07:36.448Z'
id: basic-concepts
title: المفاهيم الأساسية
---

# المفاهيم الأساسية والمصطلحات

تقدم هذه الصفحة المفاهيم الأساسية والمصطلحات المستخدمة في مكتبة `@tanstack/lit-form`. التعرف على هذه المفاهيم سيساعدك على فهم واستخدام المكتبة بشكل أفضل مع إطار العمل Lit.

## خيارات النموذج (Form Options)

يمكنك إنشاء خيارات للنموذج بحيث يمكن مشاركتها بين عدة نماذج باستخدام الدالة `formOptions`.

مثال:

```tsx
const formOpts = formOptions({
  defaultValues: {
    firstName: '',
    lastName: '',
    employed: false,
    jobTitle: '',
  } as Employee,
})
```

## نسخة النموذج (Form Instance)

نسخة النموذج هي كائن يمثل نموذجًا فرديًا ويوفر طرقًا وخصائص للعمل مع النموذج. يمكنك إنشاء نسخة نموذج باستخدام واجهة `TanStackFormController` المقدمة من `@tanstack/lit-form`. يتم إنشاء `TanStackFormController` مع صنف النموذج الحالي (`this`) وبعض خيارات النموذج الافتراضية. يقوم بتهيئة حالة النموذج، والتعامل مع إرسال النموذج، وتوفير طرق لإدارة حقول النموذج والتحقق منها.

```tsx
#form = new TanStackFormController(this, {
  defaultValues: {
    firstName: '',
    lastName: '',
    employed: false,
    jobTitle: '',
  } as Employee,
})
```

يمكنك أيضًا إنشاء نسخة نموذج دون استخدام `formOptions` باستخدام واجهة برمجة التطبيقات المستقلة `TanStackFormController`:

```tsx
#form = new TanStackFormController(this, {
  ...formOpts,
})
```

## الحقل (Field)

الحقل يمثل عنصر إدخال فردي في النموذج، مثل حقل نصي أو خانة اختيار. يتم إنشاء الحقول باستخدام الدالة `field(FieldOptions, callback)` المقدمة من نسخة النموذج. يأخذ المكون كائن `FieldOptions` ودالة رد تستقبل كائن `FieldApi`. يوفر هذا الكائن طرقًا للحصول على القيمة الحالية للحقل، والتعامل مع تغييرات الإدخال، وأحداث فقدان التركيز (blur).

مثال:

```ts
 ${this.#form.field(
    {
      name: `firstName`,
      validators: {
        onChange: ({ value }) =>
          value.length < 3 ? "Not long enough" : undefined,
        },
      },
      (field: FieldApi<Employee, "firstName">) => {
        return html` <div>
          <label class="first-name-label">First Name</label>
          <input
           id="firstName"
           type="text"
           class="first-name-input"
           placeholder="First Name"
           @blur="${() => field.handleBlur()}"
           .value="${field.state.value}"
           @input="${(event: InputEvent) => {
           if (event.currentTarget) {
            const newValue = (event.currentTarget as HTMLInputElement).value;
            field.handleChange(newValue);
           }
          }}"
        />
      </div>`;
    },
)}
```

## حالة الحقل (Field State)

كل حقل له حالته الخاصة، والتي تتضمن قيمته الحالية، وحالة التحقق منه، ورسائل الخطأ، وبيانات وصفية أخرى. يمكنك الوصول إلى حالة الحقل باستخدام خاصية `field.state`.

```tsx
const {
  value,
  meta: { errors, isValidating },
} = field.state
```

هناك ثلاث حالات للحقل يمكن أن تكون مفيدة جدًا لمعرفة كيفية تفاعل المستخدم مع الحقل. يكون الحقل "ملموسًا" (touched) عندما ينقر/ينتقل إليه المستخدم، "بكرًا" (pristine) حتى يقوم المستخدم بتغيير قيمته، و"متسخًا" (dirty) بعد تغيير القيمة. يمكنك التحقق من هذه الحالات عبر العلامات `isTouched`، `isPristine` و `isDirty` كما هو موضح أدناه.

```tsx
const { isTouched, isPristine, isDirty } = field.state.meta
```

![حالات الحقل](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)
