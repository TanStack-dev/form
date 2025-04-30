---
source-updated-at: '2025-03-28T15:34:36.000Z'
translation-updated-at: '2025-04-30T22:05:40.535Z'
id: ui-libraries
title: مكتبات واجهة المستخدم
---

## استخدام TanStack Form مع مكتبات واجهة المستخدم (UI Libraries)

تعتبر TanStack Form مكتبة بدون واجهة جاهزة (headless)، مما يمنحك مرونة كاملة في تنسيقها كما ترغب. وهي متوافقة مع مجموعة واسعة من مكتبات واجهة المستخدم، بما في ذلك `Tailwind` و`Material UI` و`Mantine` أو حتى CSS عادي.

يركز هذا الدليل على `Material UI` و`Mantine`، لكن المفاهيم قابلة للتطبيق مع أي مكتبة واجهة مستخدم تختارها.

### المتطلبات الأساسية

قبل دمج TanStack Form مع مكتبة واجهة مستخدم، تأكد من تثبيت التبعيات اللازمة في مشروعك:

- بالنسبة لـ `Material UI`، اتبع تعليمات التثبيت على [الموقع الرسمي](https://mui.com/material-ui/getting-started/).
- بالنسبة لـ `Mantine`، راجع [الوثائق](https://mantine.dev/).

ملاحظة: بينما يمكنك خلط المكتبات، يُنصح عمومًا بالالتزام بواحدة للحفاظ على الاتساق وتقليل التضخم.

### مثال مع Mantine

إليك مثالًا يوضح تكامل TanStack Form مع مكونات Mantine:

```tsx
import { TextInput, Checkbox } from '@mantine/core'
import { useForm } from '@tanstack/react-form'

export default function App() {
  const { Field, handleSubmit, state } = useForm({
    defaultValues: {
      firstName: '',
      lastName: '',
      isChecked: false,
    },
    onSubmit: async ({ value }) => {
      // Handle form submission
      console.log(value)
    },
  })

  return (
    <>
      <form
        onSubmit={(e) => {
          e.preventDefault()
          handleSubmit()
        }}
      >
        <Field
          name="firstName"
          children={({ state, handleChange, handleBlur }) => (
            <TextInput
              defaultValue={state.value}
              onChange={(e) => handleChange(e.target.value)}
              onBlur={handleBlur}
              placeholder="Enter your name"
            />
          )}
        />
        <Field
          name="isChecked"
          children={({ state, handleChange, handleBlur }) => (
            <Checkbox
              onChange={(e) => handleChange(e.target.checked)}
              onBlur={handleBlur}
              checked={state.value}
            />
          )}
        />
      </form>
      <div>
        <pre>{JSON.stringify(state.values, null, 2)}</pre>
      </div>
    </>
  )
}
```

- في البداية، نستخدم الخطاف `useForm` من TanStack ونقوم بفك الخصائص الضرورية. هذه الخطوة اختيارية؛ يمكنك أيضًا استخدام `const form = useForm()` إذا كنت تفضل ذلك. يضمن استنتاج الأنواع في TypeScript تجربة سلسة بغض النظر عن النهج.
- مكون `Field`، المستمد من `useForm`، يقبل عدة خصائص مثل `validators`. في هذا العرض، نركز على خاصيتين أساسيتين: `name` و`children`.
  - تحدد خاصية `name` كل `Field`، على سبيل المثال، `firstName` في مثالنا.
  - تستفيد خاصية `children` من مفهوم خاصية التصيير (render props)، مما يسمح لنا بدمج المكونات دون تجريدات غير ضرورية.
- يعتمد تصميم TanStack بشكل كبير على خاصية التصيير، مما يوفر الوصول إلى `children` داخل مكون `Field`. هذا النهج آمن تمامًا من حيث الأنواع. عند التكامل مع مكونات Mantine مثل `TextInput`، نقوم بفك الخصائص بشكل انتقائي مثل `state.value` و`handleChange` و`handleBlur`. هذا النهج الانتقائي يرجع إلى الاختلافات الطفيفة في الأنواع بين `TextInput` و`field` الذي نحصل عليه في `children`.
- باتباع هذه الخطوات، يمكنك دمج مكونات Mantine مع TanStack Form بسلاسة.
- هذه المنهجية قابلة للتطبيق بنفس القدر على مكونات أخرى مثل `Checkbox`، مما يضمن تكاملًا متسقًا عبر عناصر واجهة المستخدم المختلفة.

### الاستخدام مع Material UI

عملية تكامل مكونات Material UI متشابهة. إليك مثالًا باستخدام TextField وCheckbox من Material UI:

```tsx
        <Field
            name="lastName"
            children={({ state, handleChange, handleBlur }) => {
              return (
                <TextField
                  id="filled-basic"
                  label="Filled"
                  variant="filled"
                  defaultValue={state.value}
                  onChange={(e) => handleChange(e.target.value)}
                  onBlur={handleBlur}
                  placeholder="Enter your last name"
                />
              );
            }}
          />

           <Field
            name="isMuiCheckBox"
            children={({ state, handleChange, handleBlur }) => {
              return (
                <MuiCheckbox
                  onChange={(e) => handleChange(e.target.checked)}
                  onBlur={handleBlur}
                  checked={state.value}
                />
              );
            }}
          />

```

- نهج التكامل هو نفسه كما في حالة Mantine.
- يكمن الاختلاف الرئيسي في خصائص مكونات Material UI المحددة وخيارات التنسيق.
