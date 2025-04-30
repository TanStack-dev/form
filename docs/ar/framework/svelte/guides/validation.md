---
source-updated-at: '2025-04-24T18:50:42.000Z'
translation-updated-at: '2025-04-30T22:09:28.230Z'
id: form-validation
title: التحقق من صحة النموذج
---

في صميم وظائف TanStack Form يوجد مفهوم التحقق (validation). يجعل TanStack Form التحقق قابلاً للتخصيص بدرجة كبيرة:

- يمكنك التحكم في وقت إجراء التحقق (عند التغيير، عند الإدخال، عند فقدان التركيز، عند الإرسال...)
- يمكن تعريف قواعد التحقق على مستوى الحقل أو على مستوى النموذج
- يمكن أن يكون التحقق متزامنًا أو غير متزامن (على سبيل المثال، نتيجة لاستدعاء API)

## متى يتم إجراء التحقق؟

يعود الأمر لك! المكون `<form.Field />` يقبل بعض دوال الرد (callbacks) كخصائص مثل `onChange` أو `onBlur`. يتم تمرير القيمة الحالية للحقل وكائن fieldAPI إلى هذه الدوال، مما يتيح لك إجراء التحقق. إذا وجدت خطأ في التحقق، ما عليك سوى إرجاع رسالة الخطأ كسلسلة نصية وسوف تكون متاحة في `field.state.meta.errors`.

إليك مثالًا:

```svelte
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {#snippet children(field)}
    <label for={field.name}>Age:</label>
    <input
      id={field.name}
      name={field.name}
      value={field.state.value}
      type="number"
      onchange={(e) => field.handleChange(e.target.valueAsNumber)}
    />
    {#if field.state.meta.errors}
      <em role="alert">{field.state.meta.errors.join(', ')}</em>
    {/if}
  {/snippet}
</form.Field>
```

في المثال أعلاه، يتم التحقق عند كل ضغطة مفتاح (`onchange`). إذا أردنا بدلاً من ذلك أن يتم التحقق عند فقدان التركيز على الحقل، سنغير الكود كما يلي:

```svelte
<form.Field
  name="age"
  validators={{
    onBlur: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {#snippet children(field)}
    <label for={field.name}>Age:</label>
    <input
      id={field.name}
      name={field.name}
      value={field.state.value}
      type="number"
      onblur={field.handleBlur}
      onchange={(e) => field.handleChange(e.target.valueAsNumber)}
    />
    {#if field.state.meta.errors}
      <em role="alert">{field.state.meta.errors.join(', ')}</em>
    {/if}
  {/snippet}
</form.Field>
```

لذا يمكنك التحكم في وقت إجراء التحقق من خلال تنفيذ دالة الرد المطلوبة. يمكنك حتى تنفيذ أجزاء مختلفة من التحقق في أوقات مختلفة:

```svelte
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
    onBlur: ({ value }) => (value < 0 ? 'Invalid value' : undefined),
  }}
>
  {#snippet children(field)}
    <label for={field.name}>Age:</label>
    <input
      id={field.name}
      name={field.name}
      value={field.state.value}
      type="number"
      onblur={field.handleBlur}
      onchange={(e) => field.handleChange(e.target.valueAsNumber)}
    />
    {#if field.state.meta.errors}
      <em role="alert">{field.state.meta.errors.join(', ')}</em>
    {/if}
  {/snippet}
</form.Field>
```

في المثال أعلاه، نقوم بالتحقق من أشياء مختلفة لنفس الحقل في أوقات مختلفة (عند كل ضغطة مفتاح وعند فقدان التركيز على الحقل). بما أن `field.state.meta.errors` هو مصفوفة، يتم عرض جميع الأخطاء ذات الصلة في وقت معين. يمكنك أيضًا استخدام `field.state.meta.errorMap` للحصول على الأخطاء بناءً على وقت إجراء التحقق (onChange، onBlur إلخ...). المزيد من المعلومات حول عرض الأخطاء أدناه.

## عرض الأخطاء

بمجرد وضع التحقق الخاص بك، يمكنك تعيين الأخطاء من مصفوفة ليتم عرضها في واجهة المستخدم الخاصة بك:

```svelte
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {#snippet children(field)}
    <!-- ... -->
    {#if field.state.meta.errors}
      <em role="alert">{field.state.meta.errors.join(', ')}</em>
    {/if}
  {/snippet}
</form.Field>
```

أو استخدم خاصية `errorMap` للوصول إلى الخطأ المحدد الذي تبحث عنه:

```svelte
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {#snippet children(field)}
    <!-- ... -->
    {#if field.state.meta.errorMap['onChange']}
      <em role="alert">{field.state.meta.errorMap['onChange']}</em>
    {/if}
  {/snippet}
</form.Field>
```

من الجدير بالذكر أن مصفوفة `errors` و `errorMap` تتطابق مع الأنواع التي يتم إرجاعها بواسطة أدوات التحقق. هذا يعني أن:

```svelte
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) => (value < 13 ? { isOldEnough: false } : undefined),
  }}
>
  {#snippet children(field)}
    <!-- ... -->
    <!-- errorMap.onChange is type `{isOldEnough: false} | undefined` -->
    <!-- meta.errors is type `Array<{isOldEnough: false} | undefined>` -->
    {#if field.state.meta.errorMap['onChange']?.isOldEnough}
        <em>The user is not old enough</em>
    {/if}
  {/snippet}
</form.Field>
```

## التحقق على مستوى الحقل مقابل مستوى النموذج

كما هو موضح أعلاه، كل `<form.Field>` يقبل قواعد التحقق الخاصة به عبر دوال الرد `onChange`، `onBlur` إلخ... من الممكن أيضًا تعريف قواعد التحقق على مستوى النموذج (بدلاً من حقل بحقل) عن طريق تمرير دوال رد مماثلة إلى الخطاف `createForm()`.

مثال:

```svelte
<script>
  import { createForm } from '@tanstack/svelte-form'

  const form = createForm(() => ({
    defaultValues: {
      age: 0,
    },
    onSubmit: async ({ value }) => {
      console.log(value)
    },
    validators: {
      // أضف أدوات التحقق إلى النموذج بنفس الطريقة التي تضيفها إلى حقل
      onChange({ value }) {
        if (value.age < 13) {
          return 'Must be 13 or older to sign'
        }
        return undefined
      },
    },
  }))

  // اشترك في خريطة أخطاء النموذج حتى يتم عرض التحديثات عليها
  // بدلاً من ذلك، يمكنك استخدام `form.Subscribe`
  const formErrorMap = form.useStore((state) => state.errorMap)
</script>

<div>
  <!-- ... -->
  {#if formErrorMap.current.onChange}
    <div>
      <em>There was an error on the form: {formErrorMap.current.onChange}</em>
    </div>
  {/if}
  <!-- ... -->
</div>
```

## التحقق الوظيفي غير المتزامن

بينما نتوقع أن معظم عمليات التحقق ستكون متزامنة، هناك العديد من الحالات التي يكون فيها استدعاء شبكة أو عملية غير متزامنة أخرى مفيدة للتحقق منها.

للقيام بذلك، لدينا طرق مخصصة مثل `onChangeAsync`، `onBlurAsync`، وغيرها التي يمكن استخدامها للتحقق:

```svelte
<form.Field
  name="age"
  validators={{
    onChangeAsync: async ({ value }) => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return value < 13 ? 'You must be 13 to make an account' : undefined
    },
  }}
>
  {#snippet children(field)}
    <label for={field.name}>Age:</label>
    <input
      id={field.name}
      name={field.name}
      value={field.state.value}
      type="number"
      onchange={(e) => field.handleChange(e.target.valueAsNumber)}
    />
    {#if field.state.meta.errors}
      <em role="alert">{field.state.meta.errors.join(', ')}</em>
    {/if}
  {/snippet}
</form.Field>
```

يمكن أن يتعايش التحقق المتزامن وغير المتزامن. على سبيل المثال، من الممكن تعريف كل من `onBlur` و `onBlurAsync` لنفس الحقل:

```svelte
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
  {#snippet children(field)}
    <label for={field.name}>Age:</label>
    <input
      id={field.name}
      name={field.name}
      value={field.state.value}
      type="number"
      onblur={field.handleBlur}
      onchange={(e) => field.handleChange(e.target.valueAsNumber)}
    />
    {#if field.state.meta.errors}
      <em role="alert">{field.state.meta.errors.join(', ')}</em>
    {/if}
  {/snippet}
</form.Field>
```

يتم تشغيل طريقة التحقق المتزامنة (`onBlur`) أولاً وتتم تشغيل الطريقة غير المتزامنة (`onBlurAsync`) فقط إذا نجحت الطريقة المتزامنة (`onBlur`). لتغيير هذا السلوك، قم بتعيين خيار `asyncAlways` على `true`، وسيتم تشغيل الطريقة غير المتزامنة بغض النظر عن نتيجة الطريقة المتزامنة.

### تضبيط مضمن (Built-in Debouncing)

بينما تعتبر الاستدعاءات غير المتزامنة هي الطريقة المثلى للتحقق من قاعدة البيانات، فإن تشغيل طلب شبكة عند كل ضغطة مفتاح هو طريقة جيدة لتنفيذ هجوم حجب الخدمة (DDOS) على قاعدة البيانات الخاصة بك.

بدلاً من ذلك، نمكن طريقة سهلة لتضبيط استدعاءاتك غير المتزامنة عن طريق إضافة خاصية واحدة:

```svelte
<form.Field
  name="age"
  asyncDebounceMs={500}
  validators={{
    onChangeAsync: async ({ value }) => {
      // ...
    },
  }}
>
  <!-- ... -->
</form.Field>
```

سيؤدي هذا إلى تضبيط كل استدعاء غير متزامن بتأخير 500 مللي ثانية. يمكنك حتى تجاوز هذه الخاصية لكل خاصية تحقق:

```svelte
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
>
  <!-- ... -->
</form.Field>
```

> سيؤدي هذا إلى تشغيل `onChangeAsync` كل 1500 مللي ثانية بينما سيتم تشغيل `onBlurAsync` كل 500 مللي ثانية.

## التحقق من خلال مكتبات المخططات (Schema Libraries)

بينما توفر الدوال مرونة وتخصيصًا أكبر للتحقق الخاص بك، يمكن أن تكون مطولة بعض الشيء. للمساعدة في حل هذا، هناك مكتبات توفر تحققًا قائمًا على المخططات لتسهيل التحقق المختصر والصارم النوع بشكل كبير. يمكنك أيضًا تعريف مخطط واحد لنموذجك بالكامل وتمريره إلى مستوى النموذج، وسيتم نشر الأخطاء تلقائيًا إلى الحقول.

### مكتبات المخططات القياسية (Standard Schema Libraries)

يدعم TanStack Form بشكل أصلي جميع المكتبات التي تتبع [مواصفات المخطط القياسي](https://github.com/standard-schema/standard-schema)، وأبرزها:

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

_ملاحظة:_ تأكد من استخدام أحدث إصدار من مكتبات المخططات حيث أن الإصدارات الأقدم قد لا تدعم المخطط القياسي بعد.

لاستخدام المخططات من هذه المكتبات، يمكنك تمريرها إلى خاصية `validators` كما تفعل مع دالة مخصصة:

```svelte
<script>
  import { z } from 'zod'

  // ...

  const form = createForm(() => ({
    // ...
  }))
</script>

<form.Field
  name="age"
  validators={{
    onChange: z.number().gte(13, 'You must be 13 to make an account'),
  }}
>
  <!-- ... -->
</form.Field>
```

كما يتم دعم التحقق غير المتزامن على مستوى النموذج والحقل:

```svelte
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
        message: 'You can only increase the age',
      },
    ),
  }}
>
  <!-- ... -->
</form.Field>
```

## منع إرسال النماذج غير الصالحة

يتم أيضًا تشغيل دوال الرد `onChange`، `onBlur` إلخ... عند إرسال النموذج ويتم حظر الإرسال إذا كان النموذج غير صالح.

يحتوي كائن حالة النموذج على علامة `canSubmit` تكون خاطئة عندما يكون أي حقل غير صالح وقد تم لمس النموذج (`canSubmit` تكون صحيحة حتى يتم لمس النموذج، حتى إذا كانت بعض الحقول "غير صالحة" تقنيًا بناءً على خصائص `onChange`/`onBlur` الخاصة بها).

يمكنك الاشتراك فيها عبر `form.Subscribe` واستخدام القيمة من أجل، على سبيل المثال، تعطيل زر الإرسال عندما يكون النموذج غير صالح (عمليًا، الأزرار المعطلة ليست قابلة للوصول، استخدم `aria-disabled` بدلاً من ذلك).

```svelte
<script>
  import { createForm } from '@tanstack/svelte-form'

  const form = createForm(() => ({
    /* ... */
  }))
</script>

<!-- ... -->

<!-- زر إرسال ديناميكي -->
<form.Subscribe
  selector={(state) => ({
    canSubmit: state.canSubmit,
    isSubmitting: state.isSubmitting,
  })}
  children={(state) => (
    <button type="submit" disabled={!state().canSubmit}>
      {state().isSubmitting ? '...' : 'Submit'}
    </button>
  )}
>
  {#snippet children(state)}
    <button type="submit" disabled={!state.canSubmit}>
      {state.isSubmitting ? '...' : 'Submit'}
    </button>
  {/snippet}
</form.Subscribe>
```
