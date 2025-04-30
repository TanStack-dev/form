---
source-updated-at: '2025-03-28T15:34:36.000Z'
translation-updated-at: '2025-04-30T22:06:54.950Z'
id: form-composition
title: تكوين النموذج
---

# تكوين النماذج (Form Composition)

إحدى الانتقادات الشائعة لـ TanStack Form هي طول الكود المطلوب لاستخدامه بشكل افتراضي. بينما يمكن أن يكون هذا مفيدًا لأغراض تعليمية - للمساعدة في تعميق فهم واجهات برمجة التطبيقات (APIs) - إلا أنه ليس مثاليًا لحالات الاستخدام في الإنتاج.

ونتيجة لذلك، بينما يوفر `form.Field` أقوى وأكثر استخدامات TanStack Form مرونة، فإننا نقدم واجهات برمجة التطبيقات (APIs) التي تغلفه وتجعل كود التطبيق الخاص بك أقل طولًا.

## خطافات النماذج المخصصة (Custom Form Hooks)

أقوى طريقة لتكوين النماذج هي إنشاء خطافات نماذج مخصصة. يتيح لك ذلك إنشاء خطاف نموذج مصمم خصيصًا لاحتياجات تطبيقك، بما في ذلك مكونات واجهة المستخدم المخصصة المربوطة مسبقًا والمزيد.

في أبسط صوره، `createFormHook` هي دالة تأخذ `fieldContext` و `formContext` وتعيد خطاف `useAppForm`.

> خطاف `useAppForm` غير المخصص هذا مطابق لـ `useForm`، لكن هذا سيتغير بسرعة مع إضافة المزيد من الخيارات إلى `createFormHook`.

```tsx
import { createFormHookContexts, createFormHook } from '@tanstack/react-form'

// تصدير useFieldContext لاستخدامه في مكوناتك المخصصة
export const { fieldContext, formContext, useFieldContext } =
  createFormHookContexts()

const { useAppForm } = createFormHook({
  fieldContext,
  formContext,
  // سنتعلم المزيد عن هذه الخيارات لاحقًا
  fieldComponents: {},
  formComponents: {},
})

function App() {
  const form = useAppForm({
    // يدعم جميع خيارات useForm
    defaultValues: {
      firstName: 'John',
      lastName: 'Doe',
    },
  })

  return <form.Field /> // ...
}
```

### مكونات الحقول المربوطة مسبقًا (Pre-bound Field Components)

بمجرد وضع هذا الهيكل الأساسي، يمكنك البدء في إضافة مكونات حقول ونماذج مخصصة إلى خطاف النموذج الخاص بك.

> ملاحظة: يجب أن يكون `useFieldContext` هو نفسه الذي تم تصديره من سياق النموذج المخصص الخاص بك

```tsx
import { useFieldContext } from './form-context.tsx'

export function TextField({ label }: { label: string }) {
  // يستنتج `Field` أنه يجب أن يكون له نوع `value` من `string`
  const field = useFieldContext<string>()
  return (
    <label>
      <div>{label}</div>
      <input
        value={field.state.value}
        onChange={(e) => field.handleChange(e.target.value)}
      />
    </label>
  )
}
```

ثم يمكنك تسجيل هذا المكون مع خطاف النموذج الخاص بك.

```tsx
import { TextField } from './text-field.tsx'

const { useAppForm } = createFormHook({
  fieldContext,
  formContext,
  fieldComponents: {
    TextField,
  },
  formComponents: {},
})
```

واستخدامه في نموذجك:

```tsx
function App() {
  const form = useAppForm({
    defaultValues: {
      firstName: 'John',
      lastName: 'Doe',
    },
  })

  return (
    // لاحظ `AppField` بدلاً من `Field`؛ يوفر `AppField` السياق المطلوب
    <form.AppField
      name="firstName"
      children={(field) => <field.TextField label="First Name" />}
    />
  )
}
```

هذا لا يسمح لك بإعادة استخدام واجهة المستخدم لمكونك المشترك فحسب، بل يحتفظ أيضًا بأمان النوع الذي تتوقعه من TanStack Form: اكتب `name` بشكل خاطئ وستحصل على خطأ في TypeScript.

#### ملاحظة حول الأداء (A note on performance)

بينما يعد السياق أداة قيمة في نظام React البيئي، هناك قلق مشروع من العديد من المستخدمين بأن توفير قيمة تفاعلية عبر سياق سيؤدي إلى إعادة عرض غير ضرورية.

> غير ملم بهذا القلق حول الأداء؟ [منشور مدونة Mark Erikson الذي يشرح سبب حل Redux للعديد من هذه المشكلات](https://blog.isquaredsoftware.com/2021/01/context-redux-differences/) هو مكان رائع للبدء.

بينما هذا قلق جيد للإشارة إليه، إلا أنه ليس مشكلة لـ TanStack Form؛ القيم المقدمة عبر السياق ليست تفاعلية في حد ذاتها، ولكنها بدلاً من ذلك عبارة عن مثيلات فئة ثابتة مع خصائص تفاعلية ([باستخدام TanStack Store كتنفيذ للإشارات لتعزيز الأداء](https://tanstack.com/store)).

### مكونات النماذج المربوطة مسبقًا (Pre-bound Form Components)

بينما يحل `form.AppField` العديد من المشكلات مع القوالب الجاهزة وإعادة استخدام الحقول، إلا أنه لا يحل مشكلة القوالب الجاهزة وإعادة استخدام النماذج.

على وجه الخصوص، القدرة على مشاركة مثيلات `form.Subscribe` لـ، على سبيل المثال، زر إرسال نموذج تفاعلي هو حالة استخدام شائعة.

```tsx
function SubscribeButton({ label }: { label: string }) {
  const form = useFormContext()
  return (
    <form.Subscribe selector={(state) => state.isSubmitting}>
      {(isSubmitting) => <button disabled={isSubmitting}>{label}</button>}
    </form.Subscribe>
  )
}

const { useAppForm, withForm } = createFormHook({
  fieldComponents: {},
  formComponents: {
    SubscribeButton,
  },
  fieldContext,
  formContext,
})

function App() {
  const form = useAppForm({
    defaultValues: {
      firstName: 'John',
      lastName: 'Doe',
    },
  })

  return (
    <form.AppForm>
      // لاحظ مكون `AppForm` المغلف؛ يوفر `AppForm` السياق المطلوب
      <form.SubscribeButton label="Submit" />
    </form.AppForm>
  )
}
```

## تقسيم النماذج الكبيرة إلى أجزاء أصغر (Breaking big forms into smaller pieces)

أحيانًا تصبح النماذج كبيرة جدًا؛ هذا مجرد ما يحدث أحيانًا. بينما يدعم TanStack Form النماذج الكبيرة جيدًا، إلا أنه ليس ممتعًا العمل مع ملفات طويلة تحتوي على مئات أو آلاف الأسطر من التعليمات البرمجية.

لحل هذه المشكلة، ندعم تقسيم النماذج إلى أجزاء أصغر باستخدام مكون الترتيب الأعلى (higher-order component) `withForm`.

```tsx
const { useAppForm, withForm } = createFormHook({
  fieldComponents: {
    TextField,
  },
  formComponents: {
    SubscribeButton,
  },
  fieldContext,
  formContext,
})

const ChildForm = withForm({
  // هذه القيم تستخدم فقط للتحقق من النوع، ولا تستخدم في وقت التشغيل
  // هذا يسمح لك بـ `...formOpts` من `formOptions` دون الحاجة إلى إعادة تعريف الخيارات
  defaultValues: {
    firstName: 'John',
    lastName: 'Doe',
  },
  // اختياري، لكنه يضيف خصائص إلى دالة `render` بالإضافة إلى `form`
  props: {
    // هذه الخصائص أيضًا مضبوطة كقيم افتراضية لدالة `render`
    title: 'Child Form',
  },
  render: function Render({ form, title }) {
    return (
      <div>
        <p>{title}</p>
        <form.AppField
          name="firstName"
          children={(field) => <field.TextField label="First Name" />}
        />
        <form.AppForm>
          <form.SubscribeButton label="Submit" />
        </form.AppForm>
      </div>
    )
  },
})

function App() {
  const form = useAppForm({
    defaultValues: {
      firstName: 'John',
      lastName: 'Doe',
    },
  })

  return <ChildForm form={form} title={'Testing'} />
}
```

### الأسئلة الشائعة حول `withForm` (`withForm` FAQ)

> لماذا مكون ترتيب أعلى (higher-order component) بدلاً من خطاف (hook)؟

بينما تعد الخطافات مستقبل React، إلا أن مكونات الترتيب الأعلى لا تزال أداة قوية للتكوين. على وجه الخصوص، تتيح لنا واجهة برمجة التطبيقات (API) لـ `useForm` الحصول على أمان قوي للنوع دون مطالبة المستخدمين بتمرير أنواع عامة (generics).

> لماذا أحصل على أخطاء ESLint حول الخطافات في `render`؟

يبحث ESLint عن الخطافات في المستوى الأعلى للدالة، وقد لا يتم التعرف على `render` كمكون من المستوى الأعلى، اعتمادًا على كيفية تعريفك لها.

```tsx
// سيؤدي هذا إلى أخطاء ESLint مع استخدام الخطافات
const ChildForm = withForm({
  // ...
  render: ({ form, title }) => {
    // ...
  },
})
```

```tsx
// هذا يعمل بشكل جيد
const ChildForm = withForm({
  // ...
  render: function Render({ form, title }) {
    // ...
  },
})
```

## تقليل حجم حزم مكونات النماذج والحقول (Tree-shaking form and field components)

بينما الأمثلة السابقة رائعة للبدء، إلا أنها ليست مثالية لحالات استخدام معينة حيث قد يكون لديك مئات من مكونات النماذج والحقول.
على وجه الخصوص، قد لا ترغب في تضمين جميع مكونات النماذج والحقول الخاصة بك في حزمة كل ملف يستخدم خطاف النموذج الخاص بك.

لحل هذه المشكلة، يمكنك مزج واجهة برمجة التطبيقات (API) `createFormHook` من TanStack مع مكونات React `lazy` و `Suspense`:

```typescript
// src/hooks/form-context.ts
import { createFormHookContexts } from '@tanstack/react-form'

export const { fieldContext, useFieldContext, formContext, useFormContext } =
  createFormHookContexts()
```

```tsx
// src/components/text-field.tsx
import { useFieldContext } from '../hooks/form-context.tsx'

export default function TextField({ label }: { label: string }) {
  const field = useFieldContext<string>()

  return (
    <label>
      <div>{label}</div>
      <input
        value={field.state.value}
        onChange={(e) => field.handleChange(e.target.value)}
      />
    </label>
  )
}
```

```tsx
// src/hooks/form.ts
import { lazy } from 'react'
import { createFormHook } from '@tanstack/react-form'

const TextField = lazy(() => import('../components/text-fields.tsx'))

const { useAppForm, withForm } = createFormHook({
  fieldContext,
  formContext,
  fieldComponents: {
    TextField,
  },
  formComponents: {},
})
```

```tsx
// src/App.tsx
import { Suspense } from 'react'
import { PeoplePage } from './features/people/page.tsx'

export default function App() {
  return (
    <Suspense fallback={<p>Loading...</p>}>
      <PeopleForm />
    </Suspense>
  )
}
```

سيظهر هذا fallback لـ Suspense أثناء تحميل مكون `TextField`، ثم يعرض النموذج بمجرد تحميله.

## تجميع كل شيء معًا (Putting it all together)

الآن بعد أن غطينا أساسيات إنشاء خطافات نماذج مخصصة، دعونا نجمع كل شيء معًا في مثال واحد.

```tsx
// /src/hooks/form.ts، لاستخدامه عبر التطبيق بأكمله
const { fieldContext, useFieldContext, formContext, useFormContext } =
  createFormHookContexts()

function TextField({ label }: { label: string }) {
  const field = useFieldContext<string>()
  return (
    <label>
      <div>{label}</div>
      <input
        value={field.state.value}
        onChange={(e) => field.handleChange(e.target.value)}
      />
    </label>
  )
}

function SubscribeButton({ label }: { label: string }) {
  const form = useFormContext()
  return (
    <form.Subscribe selector={(state) => state.isSubmitting}>
      {(isSubmitting) => <button disabled={isSubmitting}>{label}</button>}
    </form.Subscribe>
  )
}

const { useAppForm, withForm } = createFormHook({
  fieldComponents: {
    TextField,
  },
  formComponents: {
    SubscribeButton,
  },
  fieldContext,
  formContext,
})

// /src/features/people/shared-form.ts، لاستخدامه عبر ميزات `people`
const formOpts = formOptions({
  defaultValues: {
    firstName: 'John',
    lastName: 'Doe',
  },
})

// /src/features/people/nested-form.ts، لاستخدامه في صفحة `people`
const ChildForm = withForm({
  ...formOpts,
  // اختياري، لكنه يضيف خصائص إلى دالة `render` خارج `form`
  props: {
    title: 'Child Form',
  },
  render: ({ form, title }) => {
    return (
      <div>
        <p>{title}</p>
        <form.AppField
          name="firstName"
          children={(field) => <field.TextField label="First Name" />}
        />
        <form.AppForm>
          <form.SubscribeButton label="Submit" />
        </form.AppForm>
      </div>
    )
  },
})

// /src/features/people/page.ts
const Parent = () => {
  const form = useAppForm({
    ...formOpts,
  })

  return <ChildForm form={form} title={'Testing'} />
}
```

## إرشادات استخدام واجهة برمجة التطبيقات (API Usage Guidance)

إليك مخططًا لمساعدتك في تحديد واجهات برمجة التطبيقات التي يجب استخدامها:

![](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/react_form_composability.svg)
