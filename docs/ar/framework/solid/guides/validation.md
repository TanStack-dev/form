---
source-updated-at: '2025-04-29T10:21:58.000Z'
translation-updated-at: '2025-04-30T22:08:37.654Z'
id: form-validation
title: التحقق من صحة النموذج
---

في صميم وظائف TanStack Form تكمن فكرة التحقق (validation). يجعل TanStack Form التحقق قابلاً للتخصيص بدرجة كبيرة:

- يمكنك التحكم في وقت إجراء التحقق (عند التغيير، عند الإدخال، عند فقدان التركيز، عند الإرسال...)
- يمكن تعريف قواعد التحقق على مستوى الحقل أو على مستوى النموذج
- يمكن أن يكون التحقق متزامنًا أو غير متزامن (على سبيل المثال، نتيجة لاستدعاء API)

## متى يتم إجراء التحقق؟

يعود الأمر لك! المكون `<Field />` يقبل بعض وظائف الاستدعاء (callbacks) كخصائص (props) مثل `onChange` أو `onBlur`. يتم تمرير القيمة الحالية للحقل بالإضافة إلى كائن fieldAPI إلى هذه الوظائف، مما يتيح لك إجراء التحقق. إذا وجدت خطأ في التحقق، ما عليك سوى إرجاع رسالة الخطأ كسلسلة نصية وسيتم تخزينها في `field().state.meta.errors`.

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
      <label for={field().name}>Age:</label>
      <input
        id={field().name}
        name={field().name}
        value={field().state.value}
        type="number"
        onInput={(e) => field().handleChange(e.target.valueAsNumber)}
      />
      {!field().state.meta.isValid ? (
        <em role="alert">{field().state.meta.errors.join(', ')}</em>
      ) : null}
    </>
  )}
</form.Field>
```

في المثال أعلاه، يتم التحقق عند كل ضغطة مفتاح (`onChange`). إذا أردنا بدلاً من ذلك أن يتم التحقق عند فقدان التركيز على الحقل، سنغير الكود كالتالي:

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
      <label for={field().name}>Age:</label>
      <input
        id={field().name}
        name={field().name}
        value={field().state.value}
        type="number"
        // الاستماع لحدث onBlur على الحقل
        onBlur={field().handleBlur}
        // نحتاج دائمًا إلى تنفيذ onInput حتى يتلقى TanStack Form التغييرات
        onInput={(e) => field().handleChange(e.target.valueAsNumber)}
      />
      {!field().state.meta.isValid ? (
        <em role="alert">{field().state.meta.errors.join(', ')}</em>
      ) : null}
    </>
  )}
</form.Field>
```

لذا يمكنك التحكم في وقت إجراء التحقق من خلال تنفيذ وظيفة الاستدعاء المطلوبة. يمكنك حتى تنفيذ أجزاء مختلفة من التحقق في أوقات مختلفة:

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
      <label for={field().name}>Age:</label>
      <input
        id={field().name}
        name={field().name}
        value={field().state.value}
        type="number"
        // الاستماع لحدث onBlur على الحقل
        onBlur={field().handleBlur}
        // نحتاج دائمًا إلى تنفيذ onInput حتى يتلقى TanStack Form التغييرات
        onInput={(e) => field().handleChange(e.target.valueAsNumber)}
      />
      {!field().state.meta.isValid ? (
        <em role="alert">{field().state.meta.errors.join(', ')}</em>
      ) : null}
    </>
  )}
</form.Field>
```

في المثال أعلاه، نقوم بالتحقق من أشياء مختلفة على نفس الحقل في أوقات مختلفة (عند كل ضغطة مفتاح وعند فقدان التركيز على الحقل). بما أن `field().state.meta.errors` هي مصفوفة، يتم عرض جميع الأخطاء ذات الصلة في وقت معين. يمكنك أيضًا استخدام `field().state.meta.errorMap` للحصول على الأخطاء بناءً على _وقت_ إجراء التحقق (onChange، onBlur إلخ...). المزيد من المعلومات حول عرض الأخطاء أدناه.

## عرض الأخطاء

بمجرد وضع التحقق الخاص بك، يمكنك تعيين الأخطاء من مصفوفة ليتم عرضها في واجهة المستخدم:

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
        {!field().state.meta.isValid ? (
          <em>{field().state.meta.errors.join(',')}</em>
        ) : null}
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
      {field().state.meta.errorMap['onChange'] ? (
        <em>{field().state.meta.errorMap['onChange']}</em>
      ) : null}
    </>
  )}
</form.Field>
```

من الجدير بالذكر أن مصفوفة `errors` و `errorMap` تتطابق مع الأنواع التي تُرجعها دوال التحقق. هذا يعني أن:

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
      {!field().state.meta.errorMap['onChange']?.isOldEnough ? (
        <em>The user is not old enough</em>
      ) : null}
    </>
  )}
</form.Field>
```

## التحقق على مستوى الحقل مقابل مستوى النموذج

كما هو موضح أعلاه، يقبل كل `<Field>` قواعد التحقق الخاصة به عبر دوال الاستدعاء `onChange`، `onBlur` إلخ... من الممكن أيضًا تعريف قواعد التحقق على مستوى النموذج (بدلاً من حقل بحقل) عن طريق تمرير دوال استدعاء مماثلة إلى الخطاف `createForm()`.

مثال:

```tsx
export default function App() {
  const form = createForm(() => ({
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
  }))

  // الاشتراك في خريطة أخطاء النموذج حتى يتم عرض التحديثات
  // بدلاً من ذلك، يمكنك استخدام `form.Subscribe`
  const formErrorMap = form.useStore((state) => state.errorMap)

  return (
    <div>
      {/* ... */}
      {formErrorMap().onChange ? (
        <div>
          <em>There was an error on the form: {formErrorMap().onChange}</em>
        </div>
      ) : null}
      {/* ... */}
    </div>
  )
}
```

### تعيين أخطاء على مستوى الحقل من دوال تحقق النموذج

يمكنك تعيين أخطاء على الحقول من دوال تحقق النموذج. أحد حالات الاستخدام الشائعة لهذا هو التحقق من جميع الحقول عند الإرسال عن طريق استدعاء نقطة نهاية API واحدة في دالة التحقق `onSubmitAsync` للنموذج.

```tsx
import { Show } from 'solid-js'
import { createForm } from '@tanstack/solid-form'

export default function App() {
  const form = createForm(() => ({
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
        const hasErrors = await validateDataOnServer(value)
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
  }))

  return (
    <div>
      <form
        onSubmit={(e) => {
          e.preventDefault()
          e.stopPropagation()
          void form.handleSubmit()
        }}
      >
        <form.Field
          name="age"
          children={(field) => (
            <>
              <label for={field().name}>Age:</label>
              <input
                id={field().name}
                name={field().name}
                value={field().state.value}
                type="number"
                onChange={(e) => field().handleChange(e.target.valueAsNumber)}
              />
              <Show when={field().state.meta.errors.length > 0}>
                <em role="alert">{field().state.meta.errors.join(', ')}</em>
              </Show>
            </>
          )}
        />
        <form.Subscribe
          selector={(state) => ({ errors: state.errors })}
          children={(state) => (
            <Show when={state().errors.length > 0}>
              <div>
                <em>
                  There was an error on the form: {state().errors.join(', ')}
                </em>
              </div>
            </Show>
          )}
        />

        <button type="submit">Submit</button>
        {/*...*/}
      </form>
    </div>
  )
}
```

> شيء جدير بالذكر هو أنه إذا كان لديك دالة تحقق للنموذج تُرجع خطأً، فقد يتم استبدال هذا الخطأ بتحقق الحقل المحدد.
>
> هذا يعني أن:
>
> ```tsx
>  const form = createForm(() => ({
>    defaultValues: {
>      age: 0,
>    },
>    validators: {
>      onChange: ({ value }) => {
>        return {
>          fields: {
>            age: value.age < 12 ? 'Too young!' : undefined,
>          },
>        };
>      },
>    },
>  }));
>
>  return (
>    <form.Field
>      name="age"
>      validators={{
>        onChange: ({ value }) => (value % 2 === 0 ? 'Must be odd!' : undefined),
>      }}
>      children={() => <>{/* ... */}</>}
>    />
>  );
> }
> ```
>
> سيعرض فقط `'Must be odd!'` حتى إذا تم إرجاع خطأ 'Too young!' من تحقق مستوى النموذج.

## التحقق الوظيفي غير المتزامن

بينما نتوقع أن معظم عمليات التحقق ستكون متزامنة، هناك العديد من الحالات التي يكون فيها استدعاء شبكة أو عملية غير متزامنة أخرى مفيدة للتحقق منها.

للقيام بذلك، لدينا طرق مخصصة `onChangeAsync`، `onBlurAsync`، وطرق أخرى يمكن استخدامها للتحقق:

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
      <label for={field().name}>Age:</label>
      <input
        id={field().name}
        name={field().name}
        value={field().state.value}
        type="number"
        onInput={(e) => field().handleChange(e.target.valueAsNumber)}
      />
      {!field().state.meta.isValid ? (
        <em role="alert">{field().state.meta.errors.join(', ')}</em>
      ) : null}
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
      <label for={field().name}>Age:</label>
      <input
        id={field().name}
        name={field().name}
        value={field().state.value}
        type="number"
        onBlur={field().handleBlur}
        onInput={(e) => field().handleChange(e.target.valueAsNumber)}
      />
      {field().state.meta.errors ? (
        <em role="alert">{field().state.meta.errors.join(', ')}</em>
      ) : null}
    </>
  )}
</form.Field>
```

يتم تشغيل طريقة التحقق المتزامنة (`onBlur`) أولاً وتتم تشغيل الطريقة غير المتزامنة (`onBlurAsync`) فقط إذا نجحت الطريقة المتزامنة (`onBlur`). لتغيير هذا السلوك، اضبط الخيار `asyncAlways` على `true`، وسيتم تشغيل الطريقة غير المتزامنة بغض النظر عن نتيجة الطريقة المتزامنة.

### تضمين إزالة الارتداد (Debouncing) المدمج

بينما تعتبر الاستدعاءات غير المتزامنة هي الطريقة المثلى عند التحقق من قاعدة البيانات، فإن تشغيل طلب شبكة عند كل ضغطة مفتاح هو طريقة جيدة لتنفيذ هجوم حجب الخدمة (DDOS) على قاعدة البيانات الخاصة بك.

بدلاً من ذلك، نمكن طريقة سهلة لإزالة الارتداد لاستدعاءاتك غير المتزامنة عن طريق إضافة خاصية واحدة:

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

سيؤدي هذا إلى إزالة الارتداد لكل استدعاء غير متزامن بتأخير 500 مللي ثانية. يمكنك حتى تجاوز هذه الخاصية لكل خاصية تحقق:

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

## التحقق من خلال مكتبات المخططات (Schema Libraries)

بينما توفر الدوال مرونة وتخصيصًا أكبر لتحققك، يمكن أن تكون مطولة قليلاً. للمساعدة في حل هذا، هناك مكتبات توفر تحققًا قائمًا على المخططات لجعل التحقق المختصر والصارم نوعيًا أسهل بكثير. يمكنك أيضًا تعريف مخطط واحد لنموذجك بالكامل وتمريره إلى مستوى النموذج، وسيتم نشر الأخطاء تلقائيًا إلى الحقول.

### مكتبات المخططات القياسية

يدعم TanStack Form بشكل أصلي جميع المكتبات التي تتبع [مواصفات Standard Schema](https://github.com/standard-schema/standard-schema)، وأبرزها:

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

_ملاحظة:_ تأكد من استخدام أحدث إصدار من مكتبات المخططات حيث قد لا تدعم الإصدارات الأقدم مواصفات Standard Schema بعد.

> لن يوفر لك التحقق قيمًا محولة. راجع [معالجة الإرسال](./submission-handling.md) لمزيد من المعلومات.

لاستخدام المخططات من هذه المكتبات، يمكنك تمريرها إلى خاصية `validators` كما تفعل مع دالة مخصصة:

```tsx
import { z } from 'zod'

// ...

const form = createForm(() => ({
  // ...
}))

;<form.Field
  name="age"
  validators={{
    onChange: z.number().gte(13, 'You must be 13 to make an account'),
  }}
  children={(field) => {
    return <>{/* ... */}</>
```
