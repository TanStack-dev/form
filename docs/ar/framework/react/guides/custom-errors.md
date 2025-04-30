---
source-updated-at: '2025-03-04T14:23:57.000Z'
translation-updated-at: '2025-04-30T22:05:57.425Z'
id: custom-errors
title: الأخطاء المخصصة
---

يوفر TanStack Form مرونة كاملة في أنواع قيم الأخطاء التي يمكنك إرجاعها من المدققات (validators). تعد أخطاء النصوص (string errors) الأكثر شيوعًا وسهولة في التعامل، لكن المكتبة تسمح لك بإرجاع أي نوع من القيم من المدققات الخاصة بك.

كقاعدة عامة، أي قيمة حقيقية (truthy) تعتبر خطأً وستحدد النموذج أو الحقل على أنه غير صالح، بينما القيم غير الحقيقية (falsy) مثل `false`، `undefined`، `null`، إلخ...) تعني عدم وجود خطأ، وأن النموذج أو الحقل صالح.

## إرجاع قيم نصية من النماذج

```tsx
<form.Field
  name="username"
  validators={{
    onChange: ({ value }) =>
      value.length < 3 ? 'Username must be at least 3 characters' : undefined,
  }}
/>
```

للتحقق على مستوى النموذج الذي يؤثر على عدة حقول:

```tsx
const form = useForm({
  defaultValues: {
    username: '',
    email: '',
  },
  validators: {
    onChange: ({ value }) => {
      return {
        fields: {
          username:
            value.username.length < 3 ? 'Username too short' : undefined,
          email: !value.email.includes('@') ? 'Invalid email' : undefined,
        },
      }
    },
  },
})
```

أخطاء النصوص هي النوع الأكثر شيوعًا ويمكن عرضها بسهولة في واجهة المستخدم:

```tsx
{
  field.state.meta.errors.map((error, i) => (
    <div key={i} className="error">
      {error}
    </div>
  ))
}
```

### الأرقام

مفيدة لتمثيل الكميات أو العتبات أو المقادير:

```tsx
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) => (value < 18 ? 18 - value : undefined),
  }}
/>
```

عرض في واجهة المستخدم:

```tsx
{
  /* TypeScript يعرف أن الخطأ رقم بناءً على المدقق الخاص بك */
}
;<div className="error">
  You need {field.state.meta.errors[0]} more years to be eligible
</div>
```

### القيم المنطقية

علامات بسيطة للإشارة إلى حالة الخطأ:

```tsx
<form.Field
  name="accepted"
  validators={{
    onChange: ({ value }) => (!value ? true : undefined),
  }}
/>
```

عرض في واجهة المستخدم:

```tsx
{
  field.state.meta.errors[0] === true && (
    <div className="error">You must accept the terms</div>
  )
}
```

### الكائنات

كائنات أخطاء غنية بخصائص متعددة:

```tsx
<form.Field
  name="email"
  validators={{
    onChange: ({ value }) => {
      if (!value.includes('@')) {
        return {
          message: 'Invalid email format',
          severity: 'error',
          code: 1001,
        }
      }
      return undefined
    },
  }}
/>
```

عرض في واجهة المستخدم:

```tsx
{
  typeof field.state.meta.errors[0] === 'object' && (
    <div className={`error ${field.state.meta.errors[0].severity}`}>
      {field.state.meta.errors[0].message}
      <small> (Code: {field.state.meta.errors[0].code})</small>
    </div>
  )
}
```

في المثال أعلاه يعتمد على نوع الخطأ الذي تريد عرضه.

### المصفوفات

رسائل أخطاء متعددة لحقل واحد:

```tsx
<form.Field
  name="password"
  validators={{
    onChange: ({ value }) => {
      const errors = []
      if (value.length < 8) errors.push('Password too short')
      if (!/[A-Z]/.test(value)) errors.push('Missing uppercase letter')
      if (!/[0-9]/.test(value)) errors.push('Missing number')

      return errors.length ? errors : undefined
    },
  }}
/>
```

عرض في واجهة المستخدم:

```tsx
{
  Array.isArray(field.state.meta.errors) && (
    <ul className="error-list">
      {field.state.meta.errors.map((err, i) => (
        <li key={i}>{err}</li>
      ))}
    </ul>
  )
}
```

## خاصية `disableErrorFlat` على الحقول

بشكل افتراضي، يقوم TanStack Form بتسطيح الأخطاء من جميع مصادر التحقق (onChange، onBlur، onSubmit) في مصفوفة `errors` واحدة. تحافظ خاصية `disableErrorFlat` على مصادر الأخطاء:

```tsx
<form.Field
  name="email"
  disableErrorFlat
  validators={{
    onChange: ({ value }) =>
      !value.includes('@') ? 'Invalid email format' : undefined,
    onBlur: ({ value }) =>
      !value.endsWith('.com') ? 'Only .com domains allowed' : undefined,
    onSubmit: ({ value }) => (value.length < 5 ? 'Email too short' : undefined),
  }}
/>
```

بدون `disableErrorFlat`، سيتم دمج جميع الأخطاء في `field.state.meta.errors`. مع تفعيلها، يمكنك الوصول إلى الأخطاء حسب مصدرها:

```tsx
{
  field.state.meta.errorMap.onChange && (
    <div className="real-time-error">{field.state.meta.errorMap.onChange}</div>
  )
}

{
  field.state.meta.errorMap.onBlur && (
    <div className="blur-feedback">{field.state.meta.errorMap.onBlur}</div>
  )
}

{
  field.state.meta.errorMap.onSubmit && (
    <div className="submit-error">{field.state.meta.errorMap.onSubmit}</div>
  )
}
```

هذا مفيد لـ:

- عرض أنواع مختلفة من الأخطاء بمعالجات واجهة مستخدم مختلفة
- تحديد أولويات الأخطاء (مثل عرض أخطاء الإرسال بشكل أكثر بروزًا)
- تنفيذ الكشف التدريجي عن الأخطاء

## السلامة النوعية لـ `errors` و `errorMap`

يوفر TanStack Form سلامة نوعية قوية لمعالجة الأخطاء. كل مفتاح في `errorMap` له نفس النوع الذي تم إرجاعه من المدقق المقابل، بينما تحتوي مصفوفة `errors` على نوع اتحاد (union type) لجميع قيم الأخطاء المحتملة من جميع المدققات:

```tsx
<form.Field
  name="password"
  validators={{
    onChange: ({ value }) => {
      // هذا يُرجع نصًا أو undefined
      return value.length < 8 ? 'Too short' : undefined
    },
    onBlur: ({ value }) => {
      // هذا يُرجع كائنًا أو undefined
      if (!/[A-Z]/.test(value)) {
        return { message: 'Missing uppercase', level: 'warning' }
      }
      return undefined
    },
  }}
  children={(field) => {
    // TypeScript يعرف أن errors[0] يمكن أن يكون string | { message: string, level: string } | undefined
    const error = field.state.meta.errors[0]

    // معالجة الأخطاء بطريقة آمنة للنوع
    if (typeof error === 'string') {
      return <div className="string-error">{error}</div>
    } else if (error && typeof error === 'object') {
      return <div className={error.level}>{error.message}</div>
    }

    return null
  }}
/>
```

خاصية `errorMap` أيضًا مكتوبة بشكل كامل، لتتوافق مع أنواع الإرجاع لوظائف التحقق الخاصة بك:

```tsx
// مع تفعيل disableErrorFlat
<form.Field
  name="email"
  disableErrorFlat
  validators={{
    onChange: ({ value }): string | undefined =>
      !value.includes("@") ? "Invalid email" : undefined,
    onBlur: ({ value }): { code: number, message: string } | undefined =>
      !value.endsWith(".com") ? { code: 100, message: "Wrong domain" } : undefined
  }}
  children={(field) => {
    // TypeScript يعرف النوع الدقيق لكل مصدر خطأ
    const onChangeError: string | undefined = field.state.meta.errorMap.onChange;
    const onBlurError: { code: number, message: string } | undefined = field.state.meta.errorMap.onBlur;

    return (/* ... */);
  }}
/>
```

تساعد هذه السلامة النوعية في اكتشاف الأخطاء في وقت الترجمة بدلاً من وقت التشغيل، مما يجعل الكود أكثر موثوقية وسهولة في الصيانة.
