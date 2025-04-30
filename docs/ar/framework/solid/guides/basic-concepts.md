---
source-updated-at: '2025-03-14T21:54:05.000Z'
translation-updated-at: '2025-04-30T22:07:35.679Z'
id: basic-concepts
title: المفاهيم الأساسية
---

هذه الصفحة تقدم المفاهيم الأساسية والمصطلحات المستخدمة في مكتبة `@tanstack/solid-form`. التعرف على هذه المفاهيم سيساعدك على فهم المكتبة والعمل معها بشكل أفضل.

## خيارات النموذج (Form Options)

يمكنك إنشاء خيارات لنموذجك بحيث يمكن مشاركتها بين عدة نماذج باستخدام الدالة `formOptions`.

مثال:

```tsx
const formOpts = formOptions({
  defaultValues: {
    firstName: '',
    lastName: '',
    hobbies: [],
  } as Person,
})
```

## نسخة النموذج (Form Instance)

نسخة النموذج (Form Instance) هي كائن يمثل نموذجًا فرديًا ويوفر طرقًا وخصائص للعمل مع النموذج. يمكنك إنشاء نسخة نموذج باستخدام الخطاف (hook) `createForm` المقدم من خيارات النموذج. يقبل الخطاف كائنًا يحتوي على دالة `onSubmit`، والتي يتم استدعاؤها عند إرسال النموذج.

```tsx
const form = createForm(() => ({
  ...formOpts,
  onSubmit: async ({ value }) => {
    // افعل شيئًا ببيانات النموذج
    console.log(value)
  },
}))
```

يمكنك أيضًا إنشاء نسخة نموذج دون استخدام `formOptions` باستخدام واجهة برمجة التطبيقات المستقلة `createForm`:

```tsx
const form = createForm<Person>(() => ({
  onSubmit: async ({ value }) => {
    // افعل شيئًا ببيانات النموذج
    console.log(value)
  },
  defaultValues: {
    firstName: '',
    lastName: '',
    hobbies: [],
  },
}))
```

## الحقل (Field)

الحقل (Field) يمثل عنصر إدخال فردي في النموذج، مثل حقل نصي أو خانة اختيار. يتم إنشاء الحقول باستخدام مكون `form.Field` المقدم من نسخة النموذج. يقبل المكون خاصية `name` التي يجب أن تطابق مفتاحًا في القيم الافتراضية للنموذج. كما يقبل خاصية `children`، وهي دالة عرض (render prop) تأخذ كائن حقل كوسيط.

مثال:

```tsx
<form.Field
  name="firstName"
  children={(field) => (
    <input
      name={field().name}
      value={field().state.value}
      onBlur={field().handleBlur}
      onInput={(e) => field().handleChange(e.target.value)}
    />
  )}
/>
```

## حالة الحقل (Field State)

كل حقل له حالته الخاصة، والتي تتضمن قيمته الحالية، حالة التحقق من الصحة، رسائل الخطأ، وبيانات وصفية أخرى. يمكنك الوصول إلى حالة الحقل باستخدام الخاصية `field().state`.

مثال:

```tsx
const {
  value,
  meta: { errors, isValidating },
} = field().state
```

هناك ثلاث حالات للحقل يمكن أن تكون مفيدة جدًا لمعرفة كيفية تفاعل المستخدم مع الحقل. الحقل يكون "ملموسًا" (touched) عندما ينقر/ينتقل إليه المستخدم، "نقيًا" (pristine) حتى يغير المستخدم قيمته، و"متسخًا" (dirty) بعد تغيير القيمة. يمكنك التحقق من هذه الحالات عبر الأعلام `isTouched`، `isPristine` و `isDirty` كما يلي.

```tsx
const { isTouched, isPristine, isDirty } = field().state.meta
```

![حالات الحقل](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)

## واجهة برمجة تطبيقات الحقل (Field API)

واجهة برمجة تطبيقات الحقل (Field API) هي كائن يتم تمريره إلى دالة العرض (render prop) عند إنشاء حقل. توفر طرقًا للعمل مع حالة الحقل.

مثال:

```tsx
<input
  name={field().name}
  value={field().state.value}
  onBlur={field().handleBlur}
  onInput={(e) => field().handleChange(e.target.value)}
/>
```

## التحقق من الصحة (Validation)

توفر `@tanstack/solid-form` تحققًا متزامنًا وغير متزامن من الصحة جاهزًا للاستخدام. يمكن تمرير دوال التحقق من الصحة إلى مكون `form.Field` باستخدام خاصية `validators`.

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
        name={field().name}
        value={field().state.value}
        onBlur={field().handleBlur}
        onInput={(e) => field().handleChange(e.target.value)}
      />
      <p>{field().state.meta.errors[0]}</p>
    </>
  )}
/>
```

## التحقق من الصحة باستخدام مكتبات المخططات القياسية (Standard Schema Libraries)

بالإضافة إلى خيارات التحقق من الصحة المخصصة، نحن ندعم أيضًا مواصفات [المخطط القياسي (Standard Schema)](https://github.com/standard-schema/standard-schema).

يمكنك تعريف مخطط باستخدام أي من المكتبات التي تنفذ المواصفات وتمريره إلى مدقق النموذج أو الحقل.

المكتبات المدعومة تشمل:

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

```tsx
import { z } from 'zod'

// ...
;<form.Field
  name="firstName"
  validators={{
    onChange: z.string().min(3, 'يجب أن يكون الاسم الأول على الأقل 3 أحرف'),
    onChangeAsyncDebounceMs: 500,
    onChangeAsync: z.string().refine(
      async (value) => {
        await new Promise((resolve) => setTimeout(resolve, 1000))
        return !value.includes('error')
      },
      {
        message: 'غير مسموح بكلمة "error" في الاسم الأول',
      },
    ),
  }}
  children={(field) => (
    <>
      <input
        name={field().name}
        value={field().state.value}
        onBlur={field().handleBlur}
        onInput={(e) => field().handleChange(e.target.value)}
      />
      <p>{field().state.meta.errors[0]}</p>
    </>
  )}
/>
```

## التفاعلية (Reactivity)

توفر `@tanstack/solid-form` طرقًا مختلفة للاشتراك في تغييرات حالة النموذج والحقل، أبرزها الخطاف `form.useStore` ومكون `form.Subscribe`. تتيح لك هذه الطرق تحسين أداء عرض النموذج عن طريق تحديث المكونات فقط عند الضرورة.

مثال:

```tsx
const firstName = form.useStore((state) => state.values.firstName)
//...
<form.Subscribe
  selector={(state) => ({
    canSubmit: state.canSubmit,
    isSubmitting: state.isSubmitting,
  })}
  children={(state) => (
    <button type="submit" disabled={!state().canSubmit}>
      {state().isSubmitting ? '...' : 'إرسال'}
    </button>
  )}
/>
```

## حقول المصفوفة (Array Fields)

تتيح لك حقول المصفوفة (Array Fields) إدارة قائمة من القيم داخل النموذج، مثل قائمة الهوايات. يمكنك إنشاء حقل مصفوفة باستخدام مكون `form.Field` مع الخاصية `mode="array"`.

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
        <Show
          when={hobbiesField().state.value.length > 0}
          fallback={'لا توجد هوايات.'}
        >
          <Index each={hobbiesField().state.value}>
            {(_, i) => (
              <div>
                <form.Field
                  name={`hobbies[${i}].name`}
                  children={(field) => (
                    <div>
                      <label for={field().name}>الاسم:</label>
                      <input
                        id={field().name}
                        name={field().name}
                        value={field().state.value}
                        onBlur={field().handleBlur}
                        onInput={(e) => field().handleChange(e.target.value)}
                      />
                      <button
                        type="button"
                        onClick={() => hobbiesField().removeValue(i)}
                      >
                        X
                      </button>
                    </div>
                  )}
                />
                <form.Field
                  name={`hobbies[${i}].description`}
                  children={(field) => {
                    return (
                      <div>
                        <label for={field().name}>الوصف:</label>
                        <input
                          id={field().name}
                          name={field().name}
                          value={field().state.value}
                          onBlur={field().handleBlur}
                          onInput={(e) => field().handleChange(e.target.value)}
                        />
                      </div>
                    )
                  }}
                />
              </div>
            )}
          </Index>
        </Show>
      </div>
      <button
        type="button"
        onClick={() =>
          hobbiesField().pushValue({
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

هذه هي المفاهيم الأساسية والمصطلحات المستخدمة في مكتبة `@tanstack/solid-form`. فهم هذه المفاهيم سيساعدك على العمل بشكل أكثر فعالية مع المكتبة وإنشاء نماذج معقدة بسهولة.
