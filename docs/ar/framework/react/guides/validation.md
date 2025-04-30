---
source-updated-at: '2025-04-29T10:21:58.000Z'
translation-updated-at: '2025-04-30T22:06:49.768Z'
id: form-validation
title: التحقق من صحة النموذج
---

في صميم وظائف TanStack Form يكمن مفهوم التحقق (validation). يجعل TanStack Form التحقق قابلاً للتخصيص بدرجة كبيرة:

- يمكنك التحكم في وقت إجراء التحقق (عند التغيير، عند الإدخال، عند فقدان التركيز، عند الإرسال...)
- يمكن تعريف قواعد التحقق على مستوى الحقل أو على مستوى النموذج
- يمكن أن يكون التحقق متزامنًا أو غير متزامن (على سبيل المثال، نتيجة لاستدعاء API)

## متى يتم إجراء التحقق؟

الأمر يعود إليك! مكون `<Field />` يقبل بعض دوال الرد (callbacks) كخصائص (props) مثل `onChange` أو `onBlur`. يتم تمرير القيمة الحالية للحقل بالإضافة إلى كائن fieldAPI إلى هذه الدوال، مما يتيح لك إجراء التحقق. إذا وجدت خطأ في التحقق، ما عليك سوى إرجاع رسالة الخطأ كسلسلة نصية وسوف تكون متاحة في `field.state.meta.errors`.

إليك مثال:

```tsx
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {(field) => (
    <>
      <label htmlFor={field.name}>Age:</label>
      <input
        id={field.name}
        name={field.name}
        value={field.state.value}
        type="number"
        onChange={(e) => field.handleChange(e.target.valueAsNumber)}
      />
      {!field.state.meta.isValid && (
        <em role="alert">{field.state.meta.errors.join(', ')}</em>
      )}
    </>
  )}
</form.Field>
```

في المثال أعلاه، يتم التحقق عند كل ضغطة مفتاح (`onChange`). إذا أردنا بدلاً من ذلك أن يتم التحقق عند فقدان التركيز على الحقل، سنغير الكود كما يلي:

```tsx
<form.Field
  name="age"
  validators={{
    onBlur: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {(field) => (
    <>
      <label htmlFor={field.name}>Age:</label>
      <input
        id={field.name}
        name={field.name}
        value={field.state.value}
        type="number"
        // الاستماع لحدث onBlur على الحقل
        onBlur={field.handleBlur}
        // نحتاج دائمًا إلى تنفيذ onChange حتى يتلقى TanStack Form التغييرات
        onChange={(e) => field.handleChange(e.target.valueAsNumber)}
      />
      {!field.state.meta.isValid && (
        <em role="alert">{field.state.meta.errors.join(', ')}</em>
      )}
    </>
  )}
</form.Field>
```

لذا يمكنك التحكم في وقت إجراء التحقق من خلال تنفيذ دالة الرد المطلوبة. يمكنك حتى تنفيذ أجزاء مختلفة من التحقق في أوقات مختلفة:

```tsx
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
    onBlur: ({ value }) => (value < 0 ? 'Invalid value' : undefined),
  }}
>
  {(field) => (
    <>
      <label htmlFor={field.name}>Age:</label>
      <input
        id={field.name}
        name={field.name}
        value={field.state.value}
        type="number"
        // الاستماع لحدث onBlur على الحقل
        onBlur={field.handleBlur}
        // نحتاج دائمًا إلى تنفيذ onChange حتى يتلقى TanStack Form التغييرات
        onChange={(e) => field.handleChange(e.target.valueAsNumber)}
      />
      {!field.state.meta.isValid && (
        <em role="alert">{field.state.meta.errors.join(', ')}</em>
      )}
    </>
  )}
</form.Field>
```

في المثال أعلاه، نقوم بالتحقق من أشياء مختلفة على نفس الحقل في أوقات مختلفة (عند كل ضغطة مفتاح وعند فقدان التركيز على الحقل). بما أن `field.state.meta.errors` هي مصفوفة، يتم عرض جميع الأخطاء ذات الصلة في وقت معين. يمكنك أيضًا استخدام `field.state.meta.errorMap` للحصول على الأخطاء بناءً على _وقت_ إجراء التحقق (onChange، onBlur إلخ...). المزيد من المعلومات حول عرض الأخطاء أدناه.

## عرض الأخطاء

بمجرد إعداد التحقق الخاص بك، يمكنك تعيين الأخطاء من مصفوفة ليتم عرضها في واجهة المستخدم:

```tsx
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {(field) => {
    return (
      <>
        {/* ... */}
        {!field.state.meta.isValid && (
          <em>{field.state.meta.errors.join(',')}</em>
        )}
      </>
    )
  }}
</form.Field>
```

أو استخدام خاصية `errorMap` للوصول إلى الخطأ المحدد الذي تبحث عنه:

```tsx
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {(field) => (
    <>
      {/* ... */}
      {field.state.meta.errorMap['onChange'] ? (
        <em>{field.state.meta.errorMap['onChange']}</em>
      ) : null}
    </>
  )}
</form.Field>
```

من الجدير بالذكر أن مصفوفة `errors` و `errorMap` تتطابق مع الأنواع التي يتم إرجاعها بواسطة دوال التحقق. هذا يعني أن:

```tsx
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) => (value < 13 ? { isOldEnough: false } : undefined),
  }}
>
  {(field) => (
    <>
      {/* ... */}
      {/* errorMap.onChange هو من النوع `{isOldEnough: false} | undefined` */}
      {/* meta.errors هو من النوع `Array<{isOldEnough: false} | undefined>` */}
      {!field.state.meta.errorMap['onChange']?.isOldEnough ? (
        <em>The user is not old enough</em>
      ) : null}
    </>
  )}
</form.Field>
```

## التحقق على مستوى الحقل مقابل مستوى النموذج

كما هو موضح أعلاه، كل `<Field>` تقبل قواعد التحقق الخاصة بها عبر دوال الرد `onChange`، `onBlur` إلخ... من الممكن أيضًا تعريف قواعد التحقق على مستوى النموذج (بدلاً من حقل بحقل) عن طريق تمرير دوال رد مماثلة إلى خطاف `useForm()`.

مثال:

```tsx
export default function App() {
  const form = useForm({
    defaultValues: {
      age: 0,
    },
    onSubmit: async ({ value }) => {
      console.log(value)
    },
    validators: {
      // إضافة دوال التحقق إلى النموذج بنفس الطريقة التي تضيفها إلى حقل
      onChange({ value }) {
        if (value.age < 13) {
          return 'Must be 13 or older to sign'
        }
        return undefined
      },
    },
  })

  // الاشتراك في خريطة الأخطاء للنموذج حتى يتم عرض التحديثات
  // بدلاً من ذلك، يمكنك استخدام `form.Subscribe`
  const formErrorMap = useStore(form.store, (state) => state.errorMap)

  return (
    <div>
      {/* ... */}
      {formErrorMap.onChange ? (
        <div>
          <em>There was an error on the form: {formErrorMap.onChange}</em>
        </div>
      ) : null}
      {/* ... */}
    </div>
  )
}
```

### تعيين أخطاء على مستوى الحقل من دوال التحقق الخاصة بالنموذج

يمكنك تعيين أخطاء على الحقول من دوال التحقق الخاصة بالنموذج. أحد حالات الاستخدام الشائعة لهذا هو التحقق من جميع الحقول عند الإرسال عن طريق استدعاء نقطة نهاية API واحدة في دالة التحقق `onSubmitAsync` للنموذج.

```tsx
export default function App() {
  const form = useForm({
    defaultValues: {
      age: 0,
      socials: [],
      details: {
        email: '',
      },
    },
    validators: {
      onSubmitAsync: async ({ value }) => {
        // التحقق من القيمة على الخادم
        const hasErrors = await verifyDataOnServer(value)
        if (hasErrors) {
          return {
            form: 'Invalid data', // المفتاح `form` اختياري
            fields: {
              age: 'Must be 13 or older to sign',
              // تعيين أخطاء على الحقول المتداخلة مع اسم الحقل
              'socials[0].url': 'The provided URL does not exist',
              'details.email': 'An email is required',
            },
          }
        }

        return null
      },
    },
  })

  return (
    <div>
      <form
        onSubmit={(e) => {
          e.preventDefault()
          e.stopPropagation()
          void form.handleSubmit()
        }}
      >
        <form.Field name="age">
          {(field) => (
            <>
              <label htmlFor={field.name}>Age:</label>
              <input
                id={field.name}
                name={field.name}
                value={field.state.value}
                type="number"
                onChange={(e) => field.handleChange(e.target.valueAsNumber)}
              />
              {!field.state.meta.isValid && (
                <em role="alert">{field.state.meta.errors.join(', ')}</em>
              )}
            </>
          )}
        </form.Field>
        <form.Subscribe
          selector={(state) => [state.errorMap]}
          children={([errorMap]) =>
            errorMap.onSubmit ? (
              <div>
                <em>There was an error on the form: {errorMap.onSubmit}</em>
              </div>
            ) : null
          }
        />
        {/*...*/}
      </form>
    </div>
  )
}
```

> شيء جدير بالذكر هو أنه إذا كان لديك دالة تحقق للنموذج تُرجع خطأً، فقد يتم استبدال هذا الخطأ بخطأ التحقق المحدد للحقل.
>
> هذا يعني أن:
>
> ```jsx
> const form = useForm({
>   defaultValues: {
>     age: 0,
>   },
>   validators: {
>     onChange: ({ value }) => {
>       return {
>         fields: {
>           age: value.age < 12 ? 'Too young!' : undefined,
>         },
>       }
>     },
>   },
> })
>
> // ...
>
> return (
>   <form.Field
>     name="age"
>     validators={{
>       onChange: ({ value }) => (value % 2 === 0 ? 'Must be odd!' : undefined),
>     }}
>     children={() => <>{/* ... */}</>}
>   />
> )
> ```
>
> سيعرض فقط `'Must be odd!'` حتى إذا تم إرجاع خطأ 'Too young!' بواسطة تحقق مستوى النموذج.

## التحقق الوظيفي غير المتزامن

بينما نتوقع أن معظم عمليات التحقق ستكون متزامنة، هناك العديد من الحالات التي يكون فيها استدعاء شبكة أو عملية غير متزامنة أخرى مفيدة للتحقق منها.

للقيام بذلك، لدينا طرق مخصصة `onChangeAsync`، `onBlurAsync`، وغيرها التي يمكن استخدامها للتحقق منها:

```tsx
<form.Field
  name="age"
  validators={{
    onChangeAsync: async ({ value }) => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return value < 13 ? 'You must be 13 to make an account' : undefined
    },
  }}
>
  {(field) => (
    <>
      <label htmlFor={field.name}>Age:</label>
      <input
        id={field.name}
        name={field.name}
        value={field.state.value}
        type="number"
        onChange={(e) => field.handleChange(e.target.valueAsNumber)}
      />
      {!field.state.meta.isValid && (
        <em role="alert">{field.state.meta.errors.join(', ')}</em>
      )}
    </>
  )}
</form.Field>
```

يمكن أن يتعايش التحقق المتزامن وغير المتزامن. على سبيل المثال، من الممكن تعريف كل من `onBlur` و `onBlurAsync` على نفس الحقل:

```tsx
<form.Field
  name="age"
  validators={{
    onBlur: ({ value }) => (value < 13 ? 'You must be at least 13' : undefined),
    onBlurAsync: async ({ value }) => {
      const currentAge = await fetchCurrentAgeOnProfile()
      return value < currentAge ? 'You can only increase the age' : undefined
    },
  }}
>
  {(field) => (
    <>
      <label htmlFor={field.name}>Age:</label>
      <input
        id={field.name}
        name={field.name}
        value={field.state.value}
        type="number"
        onBlur={field.handleBlur}
        onChange={(e) => field.handleChange(e.target.valueAsNumber)}
      />
      {!field.state.meta.isValid && (
        <em role="alert">{field.state.meta.errors.join(', ')}</em>
      )}
    </>
  )}
</form.Field>
```

يتم تشغيل طريقة التحقق المتزامنة (`onBlur`) أولاً وتتم تشغيل الطريقة غير المتزامنة (`onBlurAsync`) فقط إذا نجحت الطريقة المتزامنة (`onBlur`). لتغيير هذا السلوك، اضبط خيار `asyncAlways` على `true`، وسيتم تشغيل الطريقة غير المتزامنة بغض النظر عن نتيجة الطريقة المتزامنة.

### تضمين Debouncing المدمج

بينما تعد الاستدعاءات غير المتزامنة هي الطريقة المثلى عند التحقق من قاعدة البيانات، تشغيل طلب شبكة عند كل ضغطة مفتاح هو طريقة جيدة لتنفيذ هجوم DDOS على قاعدة البيانات الخاصة بك.

بدلاً من ذلك، نمكن طريقة سهلة لـ debouncing استدعاءاتك `async` عن طريق إضافة خاصية واحدة:

```tsx
<form.Field
  name="age"
  asyncDebounceMs={500}
  validators={{
    onChangeAsync: async ({ value }) => {
      // ...
    },
  }}
  children={(field) => {
    return <>{/* ... */}</>
  }}
/>
```

سيؤدي هذا إلى debouncing كل استدعاء غير متزامن بتأخير 500 مللي ثانية. يمكنك حتى تجاوز هذه الخاصية لكل خاصية تحقق:

```tsx
<form.Field
  name="age"
  asyncDebounceMs={500}
  validators={{
    onChangeAsyncDebounceMs: 1500,
    onChangeAsync: async ({ value }) => {
      // ...
    },
    onBlurAsync: async ({ value }) => {
      // ...
    },
  }}
  children={(field) => {
    return <>{/* ... */}</>
  }}
/>
```

سيؤدي هذا إلى تشغيل `onChangeAsync` كل 1500 مللي ثانية بينما سيتم تشغيل `onBlurAsync` كل 500 مللي ثانية.

## التحقق من خلال مكتبات Schema

بينما توفر الدوال مرونة وتخصيصًا أكبر لتحققك، يمكن أن تكون مطولة قليلاً. للمساعدة في حل ذلك، هناك مكتبات توفر تحققًا قائمًا على Schema لجعل التحقق المختصر والصارم من النوع أسهل بكثير. يمكنك أيضًا تعريف Schema واحد لنموذجك بالكامل وتمريره إلى مستوى النموذج، وسيتم نشر الأخطاء تلقائيًا إلى الحقول.

### مكتبات Schema القياسية

يدعم TanStack Form بشكل أصلي جميع المكتبات التي تتبع [مواصفات Standard Schema](https://github.com/standard-schema/standard-schema)، وأبرزها:

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)
- [Effect/Schema](https://effect.website/docs/schema/standard-schema/)

_ملاحظة:_ تأكد من استخدام أحدث إصدار من مكتبات Schema حيث قد لا تدعم الإصدارات الأقدم مواصفات Standard Schema بعد.

> لن يوفر لك التحقق قيمًا محولة. راجع [معالجة الإرسال](./submission-handling.md) لمزيد من المعلومات.

لاستخدام Schemas من هذه المكتبات، يمكنك تمريرها إلى خاصية `validators` كما تفعل مع دالة مخصصة:

```tsx
const userSchema = z.object({
  age: z.number().gte(13, 'You must be 13 to make an account'),
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

يتم أيضًا دعم عمليات التحقق غير المتزامنة على مستوى النموذج والحقل:

```tsx
<form.Field
  name="age"
  validators={{
    onChange: z.number().gte(13, 'You must be 13 to make an account'),
    onChangeAsyncDebounceMs: 500,
    onChangeAsync: z.number().refine(
      async (value) => {
        const currentAge = await fetchCurrentAgeOnProfile()
        return value >= currentAge
      },
      {
        message
```
