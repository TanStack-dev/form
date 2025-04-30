---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T22:07:19.359Z'
id: basic-concepts
title: المفاهيم الأساسية
---

````markdown
# المفاهيم والمصطلحات الأساسية

تقدم هذه الصفحة المفاهيم والمصطلحات الأساسية المستخدمة في مكتبة `@tanstack/angular-form`. التعرف على هذه المفاهيم سيساعدك على فهم المكتبة والعمل معها بشكل أفضل.

## نسخة النموذج (Form Instance)

نسخة النموذج هي كائن يمثل نموذجًا فرديًا ويوفر طرقًا وخصائص للعمل مع النموذج. يتم إنشاء نسخة النموذج باستخدام دالة `injectForm`. تقبل الدالة كائنًا يحتوي على دالة `onSubmit`، والتي يتم استدعاؤها عند إرسال النموذج.

```typescript
const form = injectForm({
  onSubmit: async ({ value }) => {
    // القيام بشيء ما مع بيانات النموذج
    console.log(value)
  },
})
```
````

## الحقل (Field)

الحقل يمثل عنصر إدخال فردي في النموذج، مثل حقل نصي أو خانة اختيار. يتم إنشاء الحقول باستخدام التوجيه `tanstackField`. يقبل التوجيه خاصية `name`، والتي يجب أن تطابق مفتاحًا في القيم الافتراضية للنموذج. كما يعرض نسخة مسماة `field` من الأجزاء الداخلية للتوجيه والتي يجب استخدامها عبر متغير قالب للوصول إلى الأجزاء الداخلية للحقل.

مثال:

```html
<ng-container [tanstackField]="form" name="firstName" #firstName="field">
  <input
    [value]="firstName.api.state.value"
    (blur)="firstName.api.handleBlur()"
    (input)="firstName.api.handleChange($any($event).target.value)"
  />
</ng-container>
```

## حالة الحقل (Field State)

كل حقل له حالته الخاصة، والتي تتضمن قيمته الحالية، وحالة التحقق، ورسائل الخطأ، وبيانات وصفية أخرى. يمكنك الوصول إلى حالة الحقل باستخدام خاصية `fieldApi.state`.

مثال:

```tsx
const {
  value,
  meta: { errors, isValidating },
} = field.state
```

هناك ثلاث حالات للحقل يمكن أن تكون مفيدة جدًا لمعرفة كيفية تفاعل المستخدم مع الحقل. الحقل يكون "ملموسًا" (touched) عندما ينقر/ينتقل إليه المستخدم، "نقيًا" (pristine) حتى يغير المستخدم قيمته، و "متسخًا" (dirty) بعد تغيير القيمة. يمكنك التحقق من هذه الحالات عبر العلامات `isTouched`، `isPristine` و `isDirty` كما هو موضح أدناه.

```tsx
const { isTouched, isPristine, isDirty } = field.state.meta
```

![حالات الحقل](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)

## واجهة برمجة تطبيقات الحقل (Field API)

واجهة برمجة تطبيقات الحقل هي كائن يتم الوصول إليه في خاصية `tanstackField.api` عند إنشاء حقل. توفر طرقًا للعمل مع حالة الحقل.

مثال:

```angular-html
<input
  [value]="fieldName.api.state.value"
  (blur)="fieldName.api.handleBlur()"
  (input)="fieldName.api.handleChange($any($event).target.value)"
/>
```

## التحقق (Validation)

توفر `@tanstack/angular-form` تحققًا متزامنًا وغير متزامن جاهزًا للاستخدام. يمكن تمرير دوال التحقق إلى توجيه `tanstackField` باستخدام خاصية `validators`.

مثال:

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container [tanstackField]="form" name="firstName" #firstName="field">
      <input
        [value]="firstName.api.state.value"
        (blur)="firstName.api.handleBlur()"
        (input)="firstName.api.handleChange($any($event).target.value)"
      />
    </ng-container>
  `,
})
export class AppComponent {
  firstNameValidator: FieldValidateFn<any, any, string, any> = ({
                                                                       value,
                                                                     }) =>
    !value
      ? 'يجب إدخال الاسم الأول'
      : value.length < 3
        ? 'يجب أن يكون الاسم الأول مكون من 3 أحرف على الأقل'
        : undefined

  firstNameAsyncValidator: FieldValidateAsyncFn<any, string, any> =
    async ({ value }) => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return value.includes('error') && 'غير مسموح بكلمة "error" في الاسم الأول'
    }

  form = injectForm({
    defaultValues: {
      firstName: '',
    },
    onSubmit({ value }) {
      console.log(value)
    },
  })
}
```

## التحقق باستخدام مكتبات المخططات القياسية (Standard Schema Libraries)

بالإضافة إلى خيارات التحقق المخصصة، نحن ندعم أيضًا مواصفة [المخطط القياسي](https://github.com/standard-schema/standard-schema).

يمكنك تحديد مخطط باستخدام أي من المكتبات التي تنفذ المواصفة وتمريره إلى محقق النموذج أو الحقل.

المكتبات المدعومة تشمل:

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

مثال:

```angular-ts
import { z } from 'zod'

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container
      [tanstackField]="form"
      name="firstName"
      [validators]="{
        onChange: z.string().min(3, 'يجب أن يكون الاسم الأول مكون من 3 أحرف على الأقل'),
        onChangeAsyncDebounceMs: 500,
        onChangeAsync: firstNameAsyncValidator
      }"
      #firstName="field"
    >
      <!-- ... -->
    </ng-container>
  `,
})
export class AppComponent {
  firstNameAsyncValidator = z.string().refine(
    async (value) => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return !value.includes('error')
    },
    {
      message: "غير مسموح بكلمة 'error' في الاسم الأول",
    },
  )

  form = injectForm({
    defaultValues: {
      firstName: '',
    },
    onSubmit({ value }) {
      // القيام بشيء ما مع بيانات النموذج
      console.log(value)
    },
  })

  z = z
}
```

## التفاعلية (Reactivity)

توفر `@tanstack/angular-form` طريقة للاشتراك في تغييرات حالة النموذج والحقل عبر `injectStore(this.form, selector)`.

مثال:

```typescript
import { injectForm, injectStore } from '@tanstack/angular-form'

@Component(/*...*/)
class AppComponent {
  form = injectForm(/*...*/)
  canSubmit = injectStore(this.form, (state) => state.canSubmit)
  isSubmitting = injectStore(this.form, (state) => state.isSubmitting)
}
```

## المستمعون (Listeners)

تسمح لك `@tanstack/angular-form` بالتفاعل مع محفزات محدقة و"الاستماع" إليها لتنفيذ تأثيرات جانبية.

مثال:

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
  `,
})

...

onCountryChange: FieldListenerFn<any, any, any, any, string> = ({
    value,
  }) => {
    console.log(`تم تغيير الدولة إلى: ${value}, إعادة تعيين المحافظة`)
    this.form.setFieldValue('province', '')
  }
```

يمكن العثور على المزيد من المعلومات في [المستمعون](./listeners.md)

## حقول المصفوفات (Array Fields)

تسمح لك حقول المصفوفات بإدارة قائمة من القيم داخل النموذج، مثل قائمة الهوايات. يمكنك إنشاء حقل مصفوفة باستخدام توجيه `tanstackField`.

عند العمل مع حقول المصفوفات، يمكنك استخدام طرق `pushValue`، `removeValue`، و `swapValues` لإضافة، إزالة، وتبديل القيم في المصفوفة.

مثال:

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container [tanstackField]="form" name="hobbies" #hobbies="field">
      <div>
        الهوايات
        <div>
          @if (!hobbies.api.state.value.length) {
            لا توجد هوايات
          }
          @for (_ of hobbies.api.state.value; track $index) {
            <div>
              <ng-container
                [tanstackField]="form"
                [name]="getHobbyName($index)"
                #hobbyName="field"
              >
                <div>
                  <label [for]="hobbyName.api.name">الاسم:</label>
                  <input
                    [id]="hobbyName.api.name"
                    [name]="hobbyName.api.name"
                    [value]="hobbyName.api.state.value"
                    (blur)="hobbyName.api.handleBlur()"
                    (input)="
                      hobbyName.api.handleChange($any($event).target.value)
                    "
                  />
                  <button
                    type="button"
                    (click)="hobbies.api.removeValue($index)"
                  >
                    X
                  </button>
                </div>
              </ng-container>
              <ng-container
                [tanstackField]="form"
                [name]="getHobbyDesc($index)"
                #hobbyDesc="field"
              >
                <div>
                  <label [for]="hobbyDesc.api.name">الوصف:</label>
                  <input
                    [id]="hobbyDesc.api.name"
                    [name]="hobbyDesc.api.name"
                    [value]="hobbyDesc.api.state.value"
                    (blur)="hobbyDesc.api.handleBlur()"
                    (input)="
                      hobbyDesc.api.handleChange($any($event).target.value)
                    "
                  />
                </div>
              </ng-container>
            </div>
          }
        </div>
        <button type="button" (click)="hobbies.api.pushValue(defaultHobby)">
          إضافة هواية
        </button>
      </div>
    </ng-container>
  `,
})
export class AppComponent {
  defaultHobby = {
    name: '',
    description: '',
    yearsOfExperience: 0,
  }

  getHobbyName = (idx: number) => `hobbies[${idx}].name` as const;
  getHobbyDesc = (idx: number) => `hobbies[${idx}].description` as const;

  form = injectForm({
    defaultValues: {
      hobbies: [] as Array<{
        name: string
        description: string
        yearsOfExperience: number
      }>,
    },
    onSubmit({ value }) {
      alert(JSON.stringify(value))
    },
  })
}
```

هذه هي المفاهيم والمصطلحات الأساسية المستخدمة في مكتبة `@tanstack/angular-form`. فهم هذه المفاهيم سيساعدك على العمل بشكل أكثر فعالية مع المكتبة وإنشاء نماذج معقدة بسهولة.

```

```
