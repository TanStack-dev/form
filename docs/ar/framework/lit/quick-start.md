---
source-updated-at: '2025-03-01T08:30:20.000Z'
translation-updated-at: '2025-04-30T22:04:29.511Z'
id: quick-start
title: بدء سريع
---

# بدء سريع

الحد الأدنى للبدء مع TanStack Form هو إنشاء `TanstackFormController` كما هو موضح أدناه باستخدام واجهة `Employee` لنموذج الاختبار الخاص بنا:

```ts
interface Employee {
  firstName: string
  lastName: string
  employed: boolean
  jobTitle: string
}

#form = new TanStackFormController()<Employee>(this, {
  defaultValues: {
    firstName: '',
    lastName: '',
    employed: false,
    jobTitle: '',
  },
})
```

في هذا المثال، يشير `this` إلى نسخة `LitElement` التي تريد استخدام TanStack Form داخلها.

لربط عنصر نموذج في القالب الخاص بك مع TanStack Form، استخدم دالة `field` الخاصة بـ `TanstackFormController`.

المعامل الأول لـ `field` هو `FieldOptions` والثاني هو رد نداء (callback) لعنصر التصيير الخاص بك.

```ts
field(FieldOptions, callback)
```

يجب أن يبدو نموذج الاختبار المكتمل الخاص بك مشابهًا لما يلي. يجمع النموذج الاسم الأول من حقل إدخال المستخدم:

```ts
export class TestForm extends LitElement {
  #form = new TanStackFormController<Employee>(this, {
    defaultValues: {
      firstName: '',
      lastName: '',
      employed: false,
      jobTitle: '',
    },
  })
  render() {
    return html` <p>Please enter your first name></p>
      ${this.#form.field(
        {
          name: `firstName`,
          validators: {
            onChange: ({ value }) =>
              value.length < 3 ? 'Not long enough' : undefined,
          },
        },
        (field: FieldApi<Employee, 'firstName'>) => {
          return html` <div>
            <label class="first-name-label">First Name</label>
            <input
              id="firstName"
              type="text"
              placeholder="First Name"
              @blur="${() => field.handleBlur()}"
              .value="${field.getValue()}"
              @input="${(event: InputEvent) => {
                if (event.currentTarget) {
                  const newValue = (event.currentTarget as HTMLInputElement)
                    .value
                  field.handleChange(newValue)
                }
              }}"
            />
          </div>`
        },
      )}`
  }
}
```

كن على علم بأنك بحاجة إلى التعامل مع تحديث العنصر والنموذج بنفسك كما هو موضح في المثال أعلاه.
