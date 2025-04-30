---
source-updated-at: '2025-04-29T10:21:58.000Z'
translation-updated-at: '2025-04-30T22:07:47.091Z'
id: form-validation
title: التحقق من صحة النموذج
---

في صميم وظائف TanStack Form تكمن فكرة التحقق (validation). يجعل TanStack Form التحقق قابلاً للتخصيص بشكل كبير:

- يمكنك التحكم في وقت إجراء التحقق (عند التغيير، عند الإدخال، عند فقدان التركيز، عند الإرسال...)
- يمكن تعريف قواعد التحقق على مستوى الحقل أو على مستوى النموذج
- يمكن أن يكون التحقق متزامنًا أو غير متزامن (على سبيل المثال، نتيجة لاستدعاء API)

## متى يتم إجراء التحقق؟

يعود الأمر لك! مكون `<Field />` يقبل بعض ردود النداء (callbacks) كخصائص مثل `onChange` أو `onBlur`. يتم تمرير القيمة الحالية للحقل بالإضافة إلى كائن `fieldAPI` إلى هذه الردود، مما يتيح لك إجراء التحقق. إذا وجدت خطأ في التحقق، ما عليك سوى إرجاع رسالة الخطأ كسلسلة نصية وسيتاح الوصول إليها في `field.state.meta.errors`.

إليك مثالًا:

```vue
<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onChange: ({ value }) =>
        value < 13 ? 'You must be 13 to make an account' : undefined,
    }"
  >
    <template v-slot="{ field }">
      <label :for="field.name">Age:</label>
      <input
        :id="field.name"
        :name="field.name"
        :value="field.state.value"
        type="number"
        @input="(e) => field.handleChange((e.target as HTMLInputElement).valueAsNumber)
                "
      />
      <em role="alert" v-if="!field.state.meta.isValid">{{
        field.state.meta.errors.join(', ')
      }}</em>
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

في المثال أعلاه، يتم التحقق عند كل ضغطة مفتاح (`onChange`). إذا أردنا بدلاً من ذلك أن يتم التحقق عند فقدان التركيز على الحقل، سنغير الكود كما يلي:

```vue
<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onBlur: ({ value }) =>
        value < 13 ? 'You must be 13 to make an account' : undefined,
    }"
  >
    <template v-slot="{ field }">
      <label :for="field.name">Age:</label>
      <!-- نحتاج دائمًا إلى تنفيذ onChange حتى يتلقى TanStack Form التغييرات -->
      <!-- الاستماع إلى حدث onBlur على الحقل -->
      <input
        :id="field.name"
        :name="field.name"
        :value="field.state.value"
        type="number"
        @blur="field.handleBlur"
        @input="(e) => field.handleChange((e.target as HTMLInputElement).valueAsNumber)
                "
      />
      <em role="alert" v-if="!field.state.meta.isValid">{{
        field.state.meta.errors.join(', ')
      }}</em>
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

لذا يمكنك التحكم في وقت إجراء التحقق من خلال تنفيذ رد النداء المطلوب. يمكنك حتى تنفيذ أجزاء مختلفة من التحقق في أوقات مختلفة:

```vue
<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onChange: ({ value }) =>
        value < 13 ? 'You must be 13 to make an account' : undefined,
      onBlur: ({ value }) => (value < 0 ? 'Invalid value' : undefined),
    }"
  >
    <template v-slot="{ field }">
      <label :for="field.name">Age:</label>
      <!-- نحتاج دائمًا إلى تنفيذ onChange حتى يتلقى TanStack Form التغييرات -->
      <!-- الاستماع إلى حدث onBlur على الحقل -->
      <input
        :id="field.name"
        :name="field.name"
        :value="field.state.value"
        type="number"
        @blur="field.handleBlur"
        @input="(e) => field.handleChange((e.target as HTMLInputElement).valueAsNumber)
                "
      />
      <em role="alert" v-if="!field.state.meta.isValid">{{
        field.state.meta.errors.join(', ')
      }}</em>
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

في المثال أعلاه، نقوم بالتحقق من أشياء مختلفة على نفس الحقل في أوقات مختلفة (عند كل ضغطة مفتاح وعند فقدان التركيز على الحقل). بما أن `field.state.meta.errors` هي مصفوفة، يتم عرض جميع الأخطاء ذات الصلة في وقت معين. يمكنك أيضًا استخدام `field.state.meta.errorMap` للحصول على الأخطاء بناءً على _وقت_ إجراء التحقق (onChange، onBlur إلخ...). المزيد من المعلومات حول عرض الأخطاء أدناه.

## عرض الأخطاء

بمجرد وضع التحقق الخاص بك، يمكنك تعيين الأخطاء من مصفوفة ليتم عرضها في واجهة المستخدم:

```vue
<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onChange: ({ value }) =>
        value < 13 ? 'You must be 13 to make an account' : undefined,
    }"
  >
    <template v-slot="{ field }">
      <!-- ... -->
      <em role="alert" v-if="!field.state.meta.isValid">{{
        field.state.meta.errors.join(', ')
      }}</em>
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

أو استخدام خاصية `errorMap` للوصول إلى الخطأ المحدد الذي تبحث عنه:

```vue
<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onChange: ({ value }) =>
        value < 13 ? 'You must be 13 to make an account' : undefined,
    }"
  >
    <template v-slot="{ field }">
      <!-- ... -->
      <em role="alert" v-if="field.state.meta.errorMap['onChange']">{{
        field.state.meta.errorMap['onChange']
      }}</em>
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

من الجدير بالذكر أن مصفوفة `errors` و `errorMap` تتطابق مع الأنواع التي يتم إرجاعها بواسطة أدوات التحقق. هذا يعني أن:

```vue
<form.Field
  name="age"
  :validators="{
    onChange: ({ value }) => (value < 13 ? { isOldEnough: false } : undefined),
  }"
>
  <template v-slot="{ field }">
      <!-- ... -->
      <!-- errorMap.onChange هو من النوع `{isOldEnough: false} | undefined` -->
	  <!-- meta.errors هو من النوع `Array<{isOldEnough: false} | undefined>` -->
      <em v-if="!field.state.meta.errorMap['onChange']?.isOldEnough">The user is not old enough</em>
  </template>
</form.Field>
```

## التحقق على مستوى الحقل مقابل مستوى النموذج

كما هو موضح أعلاه، يقبل كل `<Field>` قواعد التحقق الخاصة به عبر ردود النداء `onChange`، `onBlur` إلخ... من الممكن أيضًا تعريف قواعد التحقق على مستوى النموذج (بدلاً من حقل بحقل) عن طريق تمرير ردود نداء مماثلة إلى دالة `useForm()`.

مثال:

```vue
<script setup lang="ts">
import { useForm } from '@tanstack/vue-form'

const form = useForm({
  defaultValues: {
    age: 0,
  },
  onSubmit: async ({ value }) => {
    console.log(value)
  },
  validators: {
    // إضافة أدوات التحقق إلى النموذج بنفس الطريقة التي تضيفها إلى حقل
    onChange({ value }) {
      if (value.age < 13) {
        return 'Must be 13 or older to sign'
      }
      return undefined
    },
  },
})

// الاشتراك في خريطة أخطاء النموذج حتى يتم عرض التحديثات عليها
// بدلاً من ذلك، يمكنك استخدام `form.Subscribe`
const formErrorMap = form.useStore((state) => state.errorMap)
</script>

<template>
  <!-- ... -->
  <div v-if="formErrorMap.onChange">
    <em role="alert">
      There was an error on the form: {{ formErrorMap.onChange }}
    </em>
  </div>
  <!-- ... -->
</template>
```

### تعيين أخطاء على مستوى الحقل من أدوات تحقق النموذج

يمكنك تعيين أخطاء على الحقول من أدوات تحقق النموذج. أحد حالات الاستخدام الشائعة لهذا هو التحقق من جميع الحقول عند الإرسال عن طريق استدعاء نقطة نهاية API واحدة في أداة التحقق `onSubmitAsync` للنموذج.

```vue
<script setup lang="ts">
import { useForm } from '@tanstack/vue-form'

const form = useForm({
  defaultValues: {
    age: 0,
    socials: [],
    details: {
      email: '',
    },
  },
  validators: {
    // إضافة أدوات التحقق إلى النموذج بنفس الطريقة التي تضيفها إلى حقل
    onSubmitAsync({ value }) {
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
</script>

<template>
  <!-- ... -->
</template>
```

> شيء جدير بالذكر هو أنه إذا كان لديك دالة تحقق للنموذج تُرجع خطأً، فقد يتم استبدال هذا الخطأ بتحقق الحقل المحدد.
>
> هذا يعني أن:
>
> ```vue
> <script setup lang="ts">
> import { useForm } from '@tanstack/vue-form'
>
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
> </script>
>
> <template>
>   <!-- ... -->
>   <form.Field
>     name="age"
>     :validators="{
>       onChange: ({ value }) => (value % 2 === 0 ? 'Must be odd!' : undefined),
>     }"
>   >
>     <template v-slot="{ field }">
>       <!-- ... -->
>     </template>
>   </form.Field>
> </template>
> ```
>
> سيعرض فقط `'Must be odd!'` حتى إذا تم إرجاع خطأ 'Too young!' بواسطة تحقق مستوى النموذج.

## التحقق الوظيفي غير المتزامن

بينما نشك في أن معظم عمليات التحقق ستكون متزامنة، هناك العديد من الحالات التي يكون فيها استدعاء شبكة أو عملية غير متزامنة أخرى مفيدًا للتحقق منها.

للقيام بذلك، لدينا طرق مخصصة مثل `onChangeAsync`، `onBlurAsync`، وغيرها التي يمكن استخدامها للتحقق:

```vue
<script setup lang="ts">
// ...

const onChangeAge = async ({ value }) => {
  await new Promise((resolve) => setTimeout(resolve, 1000))
  return value < 13 ? 'You must be 13 to make an account' : undefined
}
</script>

<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onChangeAsync: onChangeAge,
    }"
  >
    <template v-slot="{ field }">
      <label :for="field.name">Age:</label>
      <input
        :id="field.name"
        :name="field.name"
        :value="field.state.value"
        type="number"
        @input="
          (e) =>
            field.handleChange((e.target as HTMLInputElement).valueAsNumber)
        "
      />
      <em role="alert" v-if="!field.state.meta.isValid">{{
        field.state.meta.errors.join(', ')
      }}</em>
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

يمكن أن يتعايش التحقق المتزامن وغير المتزامن. على سبيل المثال، من الممكن تعريف كل من `onBlur` و `onBlurAsync` على نفس الحقل:

```vue
<script setup lang="ts">
// ...

const onBlurAge = ({ value }) => (value < 0 ? 'Invalid value' : undefined)

const onBlurAgeAsync = async ({ value }) => {
  const currentAge = await fetchCurrentAgeOnProfile()
  return value < currentAge ? 'You can only increase the age' : undefined
}
</script>

<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onBlur: onBlurAge,
      onBlurAsync: onBlurAgeAsync,
    }"
  >
    <template v-slot="{ field }">
      <label :for="field.name">Age:</label>
      <input
        :id="field.name"
        :name="field.name"
        :value="field.state.value"
        type="number"
        @blur="field.handleBlur"
        @input="
          (e) =>
            field.handleChange((e.target as HTMLInputElement).valueAsNumber)
        "
      />
      <em role="alert" v-if="!field.state.meta.isValid">{{
        field.state.meta.errors.join(', ')
      }}</em>
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

يتم تشغيل طريقة التحقق المتزامنة (`onBlur`) أولاً وتتم تشغيل الطريقة غير المتزامنة (`onBlurAsync`) فقط إذا نجحت الطريقة المتزامنة (`onBlur`). لتغيير هذا السلوك، قم بتعيين خيار `asyncAlways` على `true`، وسيتم تشغيل الطريقة غير المتزامنة بغض النظر عن نتيجة الطريقة المتزامنة.

### تضمين Debouncing المدمج

بينما تكون الاستدعاءات غير المتزامنة هي الطريقة المثلى للتحقق من قاعدة البيانات، فإن تشغيل طلب شبكة عند كل ضغطة مفتاح هو طريقة جيدة لتنفيذ هجوم DDOS على قاعدة البيانات الخاصة بك.

بدلاً من ذلك، نمكن طريقة سهلة لـ debouncing استدعاءاتك `async` عن طريق إضافة خاصية واحدة:

```vue
<template>
  <!-- ... -->
  <form.Field
    name="age"
    :async-debounce-ms="500"
    :validators="{
      onChangeAsync: async ({ value }) => {
        // ...
      },
    }"
  >
    <template v-slot="{ field }">
      <!-- ... -->
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

سيؤدي هذا إلى debouncing كل استدعاء غير متزامن بتأخير 500 مللي ثانية. يمكنك حتى تجاوز هذه الخاصية لكل خاصية تحقق:

```vue
<template>
  <!-- ... -->
  <form.Field
    name="age"
    :async-debounce-ms="500"
    :validators="{
      onChangeAsyncDebounceMs: 1500,
      onChangeAsync: async ({ value }) => {
        // ...
      },
      onBlurAsync: async ({ value }) => {
        // ...
      },
    }"
  >
    <template v-slot="{ field }">
      <!-- ... -->
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

سيؤدي هذا إلى تشغيل `onChangeAsync` كل 1500 مللي ثانية بينما سيتم تشغيل `onBlurAsync` كل 500 مللي ثانية.

## التحقق من خلال مكتبات Schema

بينما توفر الدوال مرونة وتخصيصًا أكبر لتحققك، يمكن أن تكون مطولة بعض الشيء. للمساعدة في حل هذا، هناك مكتبات توفر تحققًا قائمًا على schema لجعل التحقق المختصر والصارم نوعيًا أسهل بكثير. يمكنك أيضًا تعريف schema واحد لنموذجك بالكامل وتمريره إلى مستوى النموذج، وسيتم نشر الأخطاء تلقائيًا إلى الحقول.

### مكتبات Schema القياسية

يدعم TanStack Form بشكل أصلي جميع المكتبات التي تتبع [مواصفات Standard Schema](https://github.com/standard-schema/standard-schema)، وأبرزها:

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

_ملاحظة:_ تأكد من استخدام أحدث إصدار من مكتبات schema حيث قد لا تدعم الإصدارات القديمة Standard Schema بعد.

> لن يوفر لك التحقق قيمًا محولة. راجع [معالجة الإرسال](./submission-handling.md) لمزيد من المعلومات.

لاستخدام schemas من هذه المكتبات، يمكنك تمريرها إلى خاصية `validators` كما تفعل مع دالة مخصصة:

```vue
<script setup lang="ts">
import { z } from 'zod'
// ...

const form = useForm({
  // ...
})
</script>

<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onChange: z.number().gte(13, '
```
