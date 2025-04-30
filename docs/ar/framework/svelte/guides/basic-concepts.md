---
source-updated-at: '2025-04-24T18:50:42.000Z'
translation-updated-at: '2025-04-30T22:08:37.075Z'
id: basic-concepts
title: المفاهيم الأساسية
---

هذه الصفحة تقدم المفاهيم الأساسية والمصطلحات المستخدمة في مكتبة `@tanstack/svelte-form`. التعرف على هذه المفاهيم سيساعدك على فهم المكتبة والعمل معها بشكل أفضل.

## خيارات النموذج (Form Options)

يمكنك إنشاء خيارات لنموذجك بحيث يمكن مشاركتها بين عدة نماذج باستخدام الدالة `formOptions`.

مثال:

```ts
const formOpts = formOptions({
  defaultValues: {
    firstName: '',
    lastName: '',
    hobbies: [],
  } as Person,
})
```

## نسخة النموذج (Form Instance)

نسخة النموذج (Form Instance) هي كائن يمثل نموذجًا فرديًا ويوفر طرقًا وخصائص للعمل مع النموذج. يمكنك إنشاء نسخة نموذج باستخدام الدالة `createForm`. تقبل الدالة كائنًا يحتوي على دالة `onSubmit`، والتي يتم استدعاؤها عند إرسال النموذج.

```ts
const form = createForm(() => ({
  ...formOpts,
  onSubmit: async ({ value }) => {
    // القيام بشيء ما مع بيانات النموذج
    console.log(value)
  },
}))
```

يمكنك أيضًا إنشاء نسخة نموذج دون استخدام `formOptions`:

```ts
const form = createForm<Person>(() => ({
  onSubmit: async ({ value }) => {
    // القيام بشيء ما مع بيانات النموذج
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

الحقل (Field) يمثل عنصر إدخال فردي في النموذج، مثل حقل نصي أو خانة اختيار. يتم إنشاء الحقول باستخدام مكون `form.Field` المقدم من نسخة النموذج. يقبل المكون خاصية `name`، والتي يجب أن تطابق مفتاحًا في القيم الافتراضية للنموذج. كما يقبل خاصية `children`، وهي دالة عرض تأخذ كائن الحقل كوسيطة.

مثال:

```svelte
<form.Field name="firstName">
  {#snippet children(field)}
    <input
      name={field.name}
      value={field.state.value}
      onblur={field.handleBlur}
      oninput={(e) => field.handleChange(e.target.value)}
    />
  {/snippet}
</form.Field>
```

## حالة الحقل (Field State)

كل حقل له حالته الخاصة، والتي تتضمن قيمته الحالية، حالة التحقق من الصحة، رسائل الخطأ، وبيانات وصفية أخرى. يمكنك الوصول إلى حالة الحقل باستخدام خاصية `field.state`.

مثال:

```ts
const {
  value,
  meta: { errors, isValidating },
} = field.state
```

هناك ثلاث حالات للحقل يمكن أن تكون مفيدة جدًا لرؤية كيفية تفاعل المستخدم مع الحقل. الحقل يكون "ملموسًا" (touched) عندما ينقر/ينتقل إليه المستخدم، "نقيًا" (pristine) حتى يقوم المستخدم بتغيير القيمة فيه، و"متسخًا" (dirty) بعد تغيير القيمة. يمكنك التحقق من هذه الحالات عبر العلامات `isTouched`, `isPristine` و `isDirty`، كما هو موضح أدناه.

```ts
const { isTouched, isPristine, isDirty } = field.state.meta
```

![حالات الحقل](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)

## واجهة برمجة تطبيقات الحقل (Field API)

واجهة برمجة تطبيقات الحقل (Field API) هي كائن يتم تمريره إلى دالة العرض عند إنشاء حقل. توفر طرقًا للعمل مع حالة الحقل.

مثال:

```svelte
<input
  name={field.name}
  value={field.state.value}
  onblur={field.handleBlur}
  oninput={(e) => field.handleChange(e.target.value)}
/>
```

## التحقق من الصحة (Validation)

توفر `@tanstack/svelte-form` تحققًا متزامنًا وغير متزامن من الصحة جاهزًا للاستخدام. يمكن تمرير دوال التحقق من الصحة إلى مكون `form.Field` باستخدام خاصية `validators`.

مثال:

```svelte
<form.Field
  name="firstName"
  validators={{
    onChange: ({ value }) =>
      !value
        ? 'يجب إدخال الاسم الأول'
        : value.length < 3
          ? 'يجب أن يكون الاسم الأول على الأقل 3 أحرف'
          : undefined,
    onChangeAsync: async ({ value }) => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return value.includes('error') && 'غير مسموح بكلمة "error" في الاسم الأول'
    },
  }}
>
  {#snippet children(field)}
    <input
      name={field.name}
      value={field.state.value}
      onBlur={field.handleBlur}
      onInput={(e) => field.handleChange(e.target.value)}
    />
    <p>{field.state.meta.errors[0]}</p>
  {/snippet}
</form.Field>
```

## التحقق من الصحة باستخدام مكتبات المخططات القياسية (Standard Schema Libraries)

بالإضافة إلى خيارات التحقق من الصحة المخصصة، ندعم أيضًا مواصفة [المخطط القياسي](https://github.com/standard-schema/standard-schema).

يمكنك تعريف مخطط باستخدام أي من المكتبات التي تنفذ المواصفة وتمريره إلى مدقق نموذج أو حقل.

المكتبات المدعومة تشمل:

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

```svelte
<script>
  import { z } from 'zod'

  // ...
</script>

<form.Field
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
>
  {#snippet children(field)}
    <input
      name={field.name}
      value={field.state.value}
      onBlur={field.handleBlur}
      onInput={(e) => field.handleChange(e.target.value)}
    />
    <p>{field.state.meta.errors[0]}</p>
  {/snippet}
</form.Field>
```

## التفاعلية (Reactivity)

توفر `@tanstack/svelte-form` طرقًا مختلفة للاشتراك في تغييرات حالة النموذج والحقل، أبرزها خطاف `form.useStore` ومكون `form.Subscribe`. تتيح لك هذه الطرق تحسين أداء عرض النموذج عن طريق تحديث المكونات فقط عند الضرورة.

مثال:

```svelte
<script>
  //...
  const firstName = form.useStore((state) => state.values.firstName)
</script>

<form.Subscribe
  selector={(state) => ({
    canSubmit: state.canSubmit,
    isSubmitting: state.isSubmitting,
  })}
>
  {#snippet children(state)}
    <button type="submit" disabled={!state.canSubmit}>
      {state.isSubmitting ? '...' : 'إرسال'}
    </button>
  {/snippet}
</form.Subscribe>
```

## حقول المصفوفة (Array Fields)

تتيح لك حقول المصفوفة إدارة قائمة من القيم داخل النموذج، مثل قائمة الهوايات. يمكنك إنشاء حقل مصفوفة باستخدام مكون `form.Field` مع الخاصية `mode="array"`.

عند العمل مع حقول المصفوفة، يمكنك استخدام طرق الحقول `pushValue`, `removeValue`, `swapValues` و `moveValue` لإضافة، إزالة، وتبديل القيم في المصفوفة.

مثال:

```svelte
<form.Field name="hobbies" mode="array">
  {#snippet children(hobbiesField)}
    <div>
      الهوايات
      <div>
        {#each hobbiesField.state.value as _, i}
            <div>
              <form.Field name={`hobbies[${i}].name`}>
                {#snippet children(field)}
                  <div>
                    <label for={field.name}>الاسم:</label>
                    <input
                      id={field.name}
                      name={field.name}
                      value={field.state.value}
                      onblur={field.handleBlur}
                      onchange={(e) => field.handleChange(e.target.value)}
                    />
                    <button
                      type="button"
                      onclick={() => hobbiesField.removeValue(i)}
                    >
                      X
                    </button>
                  </div>
                {/snippet}
              </form.Field>
              <form.Field name={`hobbies[${i}].description`}>
                {#snippet children(field)}
                    <div>
                      <label for={field.name}>الوصف:</label>
                      <input
                        id={field.name}
                        name={field.name}
                        value={field.state.value}
                        onblur={field.handleBlur}
                        onchange={(e) => field.handleChange(e.target.value)}
                      />
                    </div>
                {/snippet}
              </form.Field>
            </div>
          {:else}
            لا توجد هوايات.
          {/each}
      </div>
      <button
        type="button"
        onclick={() =>
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
  {/snippet}
</form.Field>
```

هذه هي المفاهيم الأساسية والمصطلحات المستخدمة في مكتبة `@tanstack/svelte-form`. فهم هذه المفاهيم سيساعدك على العمل بشكل أكثر فعالية مع المكتبة وإنشاء نماذج معقدة بسهولة.
