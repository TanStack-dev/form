---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T22:06:48.577Z'
id: basic-concepts
title: المفاهيم الأساسية
---

هذه الصفحة تقدم المفاهيم الأساسية والمصطلحات المستخدمة في مكتبة `@tanstack/vue-form`. التعرف على هذه المفاهيم سيساعدك على فهم المكتبة والعمل معها بشكل أفضل.

## خيارات النموذج (Form Options)

يمكنك إنشاء خيارات للنموذج بحيث يمكن مشاركتها بين عدة نماذج باستخدام الدالة `formOptions`.

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

نسخة النموذج (Form Instance) هي كائن يمثل نموذجًا فرديًا ويوفر طرقًا وخصائص للعمل مع النموذج. يمكنك إنشاء نسخة نموذج باستخدام الدالة `useForm`. تقبل الدالة كائنًا يحتوي على دالة `onSubmit`، والتي يتم استدعاؤها عند إرسال النموذج.

```js
const form = useForm({
  ...formOpts,
  onSubmit: async ({ value }) => {
    // القيام بشيء ما مع بيانات النموذج
    console.log(value)
  },
})
```

يمكنك أيضًا إنشاء نسخة نموذج دون استخدام `formOptions` باستخدام واجهة `useForm` المستقلة:

```ts
const form = useForm({
  onSubmit: async ({ value }) => {
    // القيام بشيء ما مع بيانات النموذج
    console.log(value)
  },
  defaultValues: {
    firstName: '',
    lastName: '',
    hobbies: [],
  } as Person,
})
```

## الحقل (Field)

الحقل (Field) يمثل عنصر إدخال فردي في النموذج، مثل حقل نصي أو خانة اختيار. يتم إنشاء الحقول باستخدام مكون `form.Field` المقدم من نسخة النموذج. يقبل المكون خاصية `name`، والتي يجب أن تطابق مفتاحًا في القيم الافتراضية للنموذج. كما يقبل فتحة محددة النطاق (scoped slot) باستخدام التوجيه `v-slot` والتي تأخذ كائن `field` كوسيط لها.

مثال:

```vue
<template>
  <!-- ... -->
  <form.Field name="fullName">
    <template v-slot="{ field }">
      <input
        :name="field.name"
        :value="field.state.value"
        @blur="field.handleBlur"
        @input="(e) => field.handleChange(e.target.value)"
      />
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

## حالة الحقل (Field State)

كل حقل له حالته الخاصة، والتي تتضمن قيمته الحالية، وحالة التحقق، ورسائل الخطأ، وبيانات وصفية أخرى. يمكنك الوصول إلى حالة الحقل باستخدام خاصية `field.state`.

مثال:

```js
const {
  value,
  meta: { errors, isValidating },
} = field.state
```

هناك ثلاث حالات للحقل يمكن أن تكون مفيدة جدًا لمعرفة كيفية تفاعل المستخدم مع الحقل. الحقل يكون "ملموسًا" (touched) عندما ينقر/ينتقل إليه المستخدم، "نقيًا" (pristine) حتى يقوم المستخدم بتغيير القيمة فيه، و"متسخًا" (dirty) بعد تغيير القيمة. يمكنك التحقق من هذه الحالات عبر العلامات `isTouched`، `isPristine` و `isDirty` كما هو موضح أدناه.

```js
const { isTouched, isPristine, isDirty } = field.state.meta
```

![حالات الحقل](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)

## واجهة الحقل (Field API)

واجهة الحقل (Field API) هي كائن يتم توفيره عبر فتحة محددة النطاق (scoped slot) باستخدام التوجيه `v-slot`. تستقبل هذه الفتحة وسيطًا اسمه `field` يوفر طرقًا وخصائص للعمل مع حالة الحقل.

مثال:

```vue
<template v-slot="{ field }">
  <input
    :name="field.name"
    :value="field.state.value"
    @blur="field.handleBlur"
    @input="(e) => field.handleChange(e.target.value)"
  />
</template>
```

## التحقق (Validation)

توفر `@tanstack/vue-form` تحققًا متزامنًا وغير متزامن جاهزًا للاستخدام. يمكن تمرير دوال التحقق إلى مكون `form.Field` باستخدام خاصية `validators`.

مثال:

```vue
<template>
    <!-- ... -->
    <form.Field
        name="firstName"
        :validators="{
            onChange: ({ value }) =>
                !value
                    ? `يجب إدخال الاسم الأول`
                    : value.length < 3
                        ? `يجب أن يكون الاسم الأول مكونًا من 3 أحرف على الأقل`
                        : undefined,
            onChangeAsync: async ({ value }) => {
                await new Promise((resolve) => setTimeout(resolve, 1000))
                return value.includes('error') && 'غير مسموح بكلمة "error" في الاسم الأول'
        },
    }"
    >
        <template v-slot="{ field }">
            <input
                :value="field.state.value"
                @input="(e) => field.handleChange(e.target.value)"
                @blur="field.handleBlur"
            />
            <FieldInfo :field="field" />
        </template>
    </form.Field>
    <!-- ... -->
</template>
```

## التحقق باستخدام مكتبات المخططات القياسية (Validation with Standard Schema Libraries)

بالإضافة إلى خيارات التحقق المخصصة، نحن نقدم أيضًا دعمًا لمواصفة [Standard Schema](https://github.com/standard-schema/standard-schema).

يمكنك تعريف مخطط باستخدام أي من المكتبات التي تنفذ المواصفة وتمريره إلى محقق النموذج أو الحقل.

المكتبات المدعومة تشمل:

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

```vue
<script setup lang="ts">
import { useForm } from '@tanstack/vue-form'
import { z } from 'zod'

const form = useForm({
  // ...
})

const onChangeFirstName = z.string().refine(
  async (value) => {
    await new Promise((resolve) => setTimeout(resolve, 1000))
    return !value.includes('error')
  },
  {
    message: "غير مسموح بكلمة 'error' في الاسم الأول",
  },
)
</script>

<template>
  <!-- ... -->
  <form.Field
    name="firstName"
    :validators="{
      onChange: z
        .string()
        .min(3, 'يجب أن يكون الاسم الأول مكونًا من 3 أحرف على الأقل'),
      onChangeAsyncDebounceMs: 500,
      onChangeAsync: onChangeFirstName,
    }"
  >
    <template v-slot="{ field, state }">
      <label :htmlFor="field.name">الاسم الأول:</label>
      <input
        :id="field.name"
        :name="field.name"
        :value="field.state.value"
        @input="(e) => field.handleChange((e.target as HTMLInputElement).value)"
        @blur="field.handleBlur"
      />
      <FieldInfo :state="state" />
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

## التفاعلية (Reactivity)

توفر `@tanstack/vue-form` طرقًا مختلفة للاشتراك في تغييرات حالة النموذج والحقل، أبرزها طريقة `form.useStore` ومكون `form.Subscribe`. تتيح لك هذه الطرق تحسين أداء عرض النموذج عن طريق تحديث المكونات فقط عند الضرورة.

مثال:

```vue
<script setup lang="ts">
// ...
const firstName = form.useStore((state) => state.values.firstName)
</script>

<template>
  <!-- ... -->
  <form.Subscribe>
    <template v-slot="{ canSubmit, isSubmitting }">
      <button type="submit" :disabled="!canSubmit">
        {{ isSubmitting ? '...' : 'إرسال' }}
      </button>
    </template>
  </form.Subscribe>
  <!-- ... -->
</template>
```

## المستمعون (Listeners)

تسمح لك `@tanstack/vue-form` بالاستجابة لمحفزات محددة و"الاستماع" إليها لتنفيذ تأثيرات جانبية.

مثال:

```vue
<template>
  <form.Field
    name="country"
    :listeners="{
      onChange: ({ value }) => {
        console.log(`تم تغيير الدولة إلى: ${value}, جاري إعادة تعيين المحافظة`)
        form.setFieldValue('province', '')
      },
    }"
  >
    <template v-slot="{ field }">
      <input
        :value="field.state.value"
        @input="(e) => field.handleChange(e.target.value)"
      />
    </template>
  </form.Field>
</template>
```

يمكن العثور على المزيد من المعلومات في [المستمعون](./listeners.md)

ملاحظة: يُنصح بعدم استخدام طريقة `form.useField` لتحقيق التفاعلية لأنها مصممة لاستخدام مدروس داخل مكون `form.Field`. قد ترغب في استخدام `form.useStore` بدلاً من ذلك.

## حقول المصفوفة (Array Fields)

تتيح لك حقول المصفوفة (Array Fields) إدارة قائمة من القيم داخل النموذج، مثل قائمة الهوايات. يمكنك إنشاء حقل مصفوفة باستخدام مكون `form.Field` مع الخاصية `mode="array"`.

عند العمل مع حقول المصفوفة، يمكنك استخدام الطرق `pushValue`، `removeValue`، `swapValues` و `moveValue` لإضافة، إزالة، وتبديل القيم في المصفوفة.

مثال:

```vue
<template>
  <form @submit.prevent.stop="form.handleSubmit">
    <form.Field name="hobbies" mode="array">
      <template v-slot="{ field: hobbiesField }">
        <div>
          الهوايات
          <div>
            <div
              v-if="
                Array.isArray(hobbiesField.state.value) &&
                !hobbiesField.state.value.length
              "
            >
              لا توجد هوايات.
            </div>
            <div v-else>
              <div v-for="(_, i) in hobbiesField.state.value" :key="i">
                <form.Field :name="`hobbies[${i}].name`">
                  <template v-slot="{ field }">
                    <div>
                      <label :for="field.name">الاسم:</label>
                      <input
                        :id="field.name"
                        :name="field.name"
                        :value="field.state.value"
                        @blur="field.handleBlur"
                        @input="(e) => field.handleChange(e.target.value)"
                      />
                      <button
                        type="button"
                        @click="hobbiesField.removeValue(i)"
                      >
                        X
                      </button>
                      <FieldInfo :field="field" />
                    </div>
                  </template>
                </form.Field>
                <form.Field :name="`hobbies[${i}].description`">
                  <template v-slot="{ field }">
                    <div>
                      <label :for="field.name">الوصف:</label>
                      <input
                        :id="field.name"
                        :name="field.name"
                        :value="field.state.value"
                        @blur="field.handleBlur"
                        @input="(e) => field.handleChange(e.target.value)"
                      />
                      <FieldInfo :field="field" />
                    </div>
                  </template>
                </form.Field>
              </div>
            </div>
            <button
              type="button"
              @click="
                hobbiesField.pushValue({
                  name: '',
                  description: '',
                  yearsOfExperience: 0,
                })
              "
            >
              إضافة هواية
            </button>
          </div>
        </div>
      </template>
    </form.Field>
  </form>
</template>
```

هذه هي المفاهيم الأساسية والمصطلحات المستخدمة في مكتبة `@tanstack/vue-form`. فهم هذه المفاهيم سيساعدك على العمل بشكل أكثر فعالية مع المكتبة وإنشاء نماذج معقدة بسهولة.
