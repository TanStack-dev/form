---
source-updated-at: '2025-05-08T07:42:29.000Z'
translation-updated-at: '2025-05-08T23:54:45.818Z'
id: basic-concepts
title: المفاهيم الأساسية
---

هذه الصفحة تقدم المفاهيم الأساسية والمصطلحات المستخدمة في مكتبة `@tanstack/react-form`. التعرف على هذه المفاهيم سيساعدك على فهم المكتبة والعمل معها بشكل أفضل.

## خيارات النموذج (Form Options)

يمكنك إنشاء خيارات لنموذجك حتى يمكن مشاركتها بين عدة نماذج باستخدام دالة `formOptions`.

مثال:

```tsx
interface User {
  firstName: string
  lastName: string
  hobbies: Array<string>
}
const defaultUser: User = { firstName: '', lastName: '', hobbies: [] }

const formOpts = formOptions({
  defaultValues: defaultUser,
})
```

## نسخة النموذج (Form Instance)

نسخة النموذج (Form Instance) هي كائن يمثل نموذجًا فرديًا ويوفر طرقًا وخصائص للعمل مع النموذج. يمكنك إنشاء نسخة نموذج باستخدام الخطاف (hook) `useForm` المقدم من خيارات النموذج. يقبل الخطاف كائنًا يحتوي على دالة `onSubmit`، والتي يتم استدعاؤها عند تقديم النموذج.

```tsx
const form = useForm({
  ...formOpts,
  onSubmit: async ({ value }) => {
    // القيام بشيء ما ببيانات النموذج
    console.log(value)
  },
})
```

يمكنك أيضًا إنشاء نسخة نموذج دون استخدام `formOptions` باستخدام واجهة برمجة التطبيقات (API) المستقلة `useForm`:

```tsx
interface User {
  firstName: string
  lastName: string
  hobbies: Array<string>
}
const defaultUser: User = { firstName: '', lastName: '', hobbies: [] }

const form = useForm({
  defaultValues: defaultUser,
  onSubmit: async ({ value }) => {
    // القيام بشيء ما ببيانات النموذج
    console.log(value)
  },
})
```

## الحقل (Field)

الحقل (Field) يمثل عنصر إدخال فردي في النموذج، مثل حقل نصي أو خانة اختيار. يتم إنشاء الحقول باستخدام مكون `form.Field` المقدم من نسخة النموذج. يقبل المكون خاصية `name`، والتي يجب أن تطابق مفتاحًا في القيم الافتراضية للنموذج. كما يقبل خاصية `children`، وهي دالة عرض (render prop) تأخذ كائن حقل كوسيط لها.

مثال:

```tsx
<form.Field
  name="firstName"
  children={(field) => (
    <>
      <input
        value={field.state.value}
        onBlur={field.handleBlur}
        onChange={(e) => field.handleChange(e.target.value)}
      />
      <FieldInfo field={field} />
    </>
  )}
/>
```

## حالة الحقل (Field State)

كل حقل له حالته الخاصة، والتي تتضمن قيمته الحالية، حالة التحقق من الصحة، رسائل الخطأ، وبيانات وصفية أخرى. يمكنك الوصول إلى حالة الحقل باستخدام خاصية `field.state`.

مثال:

```tsx
const {
  value,
  meta: { errors, isValidating },
} = field.state
```

هناك ثلاث حالات في البيانات الوصفية يمكن أن تكون مفيدة لمعرفة كيفية تفاعل المستخدم مع الحقل:

- _"isTouched"_: بعد أن ينقر المستخدم/ينتقل إلى الحقل
- _"isPristine"_: حتى يغير المستخدم قيمة الحقل
- _"isDirty"_: بعد تغيير قيمة الحقل

```tsx
const { isTouched, isPristine, isDirty } = field.state.meta
```

![حالات الحقل](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)

## فهم 'isDirty' في المكتبات المختلفة

حالة `dirty` غير الدائمة (Non-Persistent)

- **المكتبات**: React Hook Form (RHF), Formik, Final Form.
- **السلوك**: الحقل يكون 'dirty' إذا اختلفت قيمته عن القيمة الافتراضية. العودة إلى القيمة الافتراضية تجعله 'نظيفًا' مرة أخرى.

حالة `dirty` الدائمة (Persistent)

- **المكتبات**: Angular Form, Vue FormKit.
- **السلوك**: يبقى الحقل 'dirty' بمجرد تغييره، حتى إذا عاد إلى القيمة الافتراضية.

لقد اخترنا نموذج حالة 'dirty' الدائمة. لدعم حالة 'dirty' غير الدائمة أيضًا، نقدم العلامة `isDefault`. تعمل هذه العلامة كعكس لحالة 'dirty' غير الدائمة.

```tsx
const { isTouched, isPristine, isDirty, isDefaultValue } = field.state.meta

// السطر التالي يعيد إنشاء وظيفة حالة `dirty` غير الدائمة.
const nonPersistentIsDirty = !isDefaultValue
```

![حالات الحقل الممتدة](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states-extended.png)

## واجهة برمجة تطبيقات الحقل (Field API)

واجهة برمجة تطبيقات الحقل (Field API) هي كائن يتم تمريره إلى دالة العرض (render prop) عند إنشاء حقل. توفر طرقًا للعمل مع حالة الحقل.

مثال:

```tsx
<input
  value={field.state.value}
  onBlur={field.handleBlur}
  onChange={(e) => field.handleChange(e.target.value)}
/>
```

## التحقق من الصحة (Validation)

توفر `@tanstack/react-form` التحقق من الصحة المتزامن وغير المتزامن بشكل جاهز. يمكن تمرير دوال التحقق من الصحة إلى مكون `form.Field` باستخدام خاصية `validators`.

مثال:

```tsx
<form.Field
  name="firstName"
  validators={{
    onChange: ({ value }) =>
      !value
        ? 'الاسم الأول مطلوب'
        : value.length < 3
          ? 'يجب أن يكون الاسم الأول على الأقل 3 أحرف'
          : undefined,
    onChangeAsync: async ({ value }) => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return value.includes('error') && 'غير مسموح بكلمة "error" في الاسم الأول'
    },
  }}
  children={(field) => (
    <>
      <input
        value={field.state.value}
        onBlur={field.handleBlur}
        onChange={(e) => field.handleChange(e.target.value)}
      />
      <FieldInfo field={field} />
    </>
  )}
/>
```

## التحقق من الصحة مع مكتبات المخططات القياسية (Standard Schema Libraries)

بالإضافة إلى خيارات التحقق من الصحة المخصصة، نحن ندعم أيضًا مواصفة [المخطط القياسي (Standard Schema)](https://github.com/standard-schema/standard-schema).

يمكنك تعريف مخطط باستخدام أي من المكتبات التي تنفذ المواصفة وتمريره إلى مدقق النموذج أو الحقل.

المكتبات المدعومة تشمل:

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

```tsx
import { z } from 'zod'

const userSchema = z.object({
  age: z.number().gte(13, 'يجب أن يكون عمرك 13 لإنشاء حساب'),
})

function App() {
  const form = useForm({
    defaultValues: {
      age: 0,
    },
    validators: {
      onChange: userSchema,
    },
  })
  return (
    <div>
      <form.Field
        name="age"
        children={(field) => {
          return <>{/* ... */}</>
        }}
      />
    </div>
  )
}
```

## التفاعلية (Reactivity)

توفر `@tanstack/react-form` طرقًا مختلفة للاشتراك في تغييرات حالة النموذج والحقل، أبرزها الخطاف `useStore(form.store)` ومكون `form.Subscribe`. تتيح لك هذه الطرق تحسين أداء عرض النموذج عن طريق تحديث المكونات فقط عند الضرورة.

مثال:

```tsx
const firstName = useStore(form.store, (state) => state.values.firstName)
//...
<form.Subscribe
  selector={(state) => [state.canSubmit, state.isSubmitting]}
  children={([canSubmit, isSubmitting]) => (
    <button type="submit" disabled={!canSubmit}>
      {isSubmitting ? '...' : 'إرسال'}
    </button>
  )}
/>
```

من المهم تذكر أنه بينما خاصية `selector` في الخطاف `useStore` اختيارية، يوصى بشدة بتوفير واحدة، لأن حذفها سيؤدي إلى إعادة عرض غير ضرورية.

```tsx
// الاستخدام الصحيح
const firstName = useStore(form.store, (state) => state.values.firstName)
const errors = useStore(form.store, (state) => state.errorMap)
// الاستخدام غير الصحيح
const store = useStore(form.store)
```

ملاحظة: يُنصح بعدم استخدام الخطاف `useField` لتحقيق التفاعلية لأنه مصمم لاستخدام مدروس داخل مكون `form.Field`. قد ترغب في استخدام `useStore(form.store)` بدلاً من ذلك.

## المستمعون (Listeners)

تتيح لك `@tanstack/react-form` الاستجابة لمحفزات محددة و"الاستماع" إليها لتنفيذ تأثيرات جانبية.

مثال:

```tsx
<form.Field
  name="country"
  listeners={{
    onChange: ({ value }) => {
      console.log(`تم تغيير الدولة إلى: ${value}, إعادة تعيين المحافظة`)
      form.setFieldValue('province', '')
    },
  }}
/>
```

يمكن العثور على المزيد من المعلومات في [المستمعون](./listeners.md)

## حقول المصفوفة (Array Fields)

تتيح لك حقول المصفوفة (Array Fields) إدارة قائمة من القيم داخل النموذج، مثل قائمة الهوايات. يمكنك إنشاء حقل مصفوفة باستخدام مكون `form.Field` مع خاصية `mode="array"`.

عند العمل مع حقول المصفوفة، يمكنك استخدام طرق الحقول `pushValue`، `removeValue`، `swapValues` و `moveValue` لإضافة، إزالة، وتبديل القيم في المصفوفة.

مثال:

```tsx
<form.Field
  name="hobbies"
  mode="array"
  children={(hobbiesField) => (
    <div>
      الهوايات
      <div>
        {!hobbiesField.state.value.length
          ? 'لا توجد هوايات.'
          : hobbiesField.state.value.map((_, i) => (
              <div key={i}>
                <form.Field
                  name={`hobbies[${i}].name`}
                  children={(field) => {
                    return (
                      <div>
                        <label htmlFor={field.name}>الاسم:</label>
                        <input
                          id={field.name}
                          name={field.name}
                          value={field.state.value}
                          onBlur={field.handleBlur}
                          onChange={(e) => field.handleChange(e.target.value)}
                        />
                        <button
                          type="button"
                          onClick={() => hobbiesField.removeValue(i)}
                        >
                          X
                        </button>
                        <FieldInfo field={field} />
                      </div>
                    )
                  }}
                />
                <form.Field
                  name={`hobbies[${i}].description`}
                  children={(field) => {
                    return (
                      <div>
                        <label htmlFor={field.name}>الوصف:</label>
                        <input
                          id={field.name}
                          name={field.name}
                          value={field.state.value}
                          onBlur={field.handleBlur}
                          onChange={(e) => field.handleChange(e.target.value)}
                        />
                        <FieldInfo field={field} />
                      </div>
                    )
                  }}
                />
              </div>
            ))}
      </div>
      <button
        type="button"
        onClick={() =>
          hobbiesField.pushValue({
            name: '',
            description: '',
            yearsOfExperience: 0,
          })
        }
      >
        إضافة هواية
      </button>
    </div>
  )}
/>
```

## أزرار إعادة التعيين (Reset Buttons)

عند استخدام `<button type="reset">` مع دالة `form.reset()` من TanStack Form، تحتاج إلى منع سلوك إعادة تعيين HTML الافتراضي لتجنب إعادة تعيين غير متوقعة لعناصر النموذج (خاصة عناصر `<select>`) إلى قيم HTML الأولية.
استخدم `event.preventDefault()` داخل معالج النقر (`onClick`) للزر لمنع إعادة التعيين الأصلية.

مثال:

```tsx
<button
  type="reset"
  onClick={(event) => {
    event.preventDefault()
    form.reset()
  }}
>
  إعادة التعيين
</button>
```

بدلاً من ذلك، يمكنك استخدام `<button type="button">` لمنع إعادة تعيين HTML الأصلية.

```tsx
<button
  type="button"
  onClick={() => {
    form.reset()
  }}
>
  إعادة التعيين
</button>
```

هذه هي المفاهيم الأساسية والمصطلحات المستخدمة في مكتبة `@tanstack/react-form`. فهم هذه المفاهيم سيساعدك على العمل بشكل أكثر فعالية مع المكتبة وإنشاء نماذج معقدة بسهولة.
