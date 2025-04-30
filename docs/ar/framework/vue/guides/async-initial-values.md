---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T22:05:44.080Z'
id: async-initial-values
title: القيم الأولية غير المتزامنة
---

لنفترض أنك تريد جلب بعض البيانات من واجهة برمجة التطبيقات (API) واستخدامها كقيم أولية لنموذج.

رغم أن هذه المشكلة تبدو بسيطة ظاهريًا، إلا أن هناك تعقيدات خفية ربما لم تفكر فيها حتى الآن.

على سبيل المثال، قد ترغب في عرض مؤشر تحميل أثناء جلب البيانات، أو التعامل مع الأخطاء بطريقة أنيقة.

بالمثل، قد تجد نفسك تبحث عن طريقة لتخزين البيانات مؤقتًا (cache) حتى لا تضطر إلى جلبها في كل مرة يتم فيها عرض النموذج.

بينما يمكننا تنفيذ العديد من هذه الميزات من الصفر، إلا أن النتيجة ستشبه إلى حد كبير مشروعًا آخر نطوره: [TanStack Query](https://tanstack.com/query).

لذلك، يوضح لك هذا الدليل كيف يمكنك دمج TanStack Form مع TanStack Query لتحقيق السلوك المطلوب.

## الاستخدام الأساسي

```vue
<script setup lang="ts">
import { useForm } from '@tanstack/vue-form'
import { useQuery } from '@tanstack/vue-query'
import { watchEffect, reactive } from 'vue'

const { data, isLoading } = useQuery({
  queryKey: ['data'],
  queryFn: async () => {
    await new Promise((resolve) => setTimeout(resolve, 1000))
    return { firstName: 'FirstName', lastName: 'LastName' }
  },
})

const firstName = computed(() => data.value?.firstName || '')
const lastName = computed(() => data.value?.lastName || '')

const defaultValues = reactive({
  firstName,
  lastName,
})

const form = useForm({
  defaultValues,
  onSubmit: async ({ value }) => {
    // Do something with form data
    console.log(value)
  },
})
</script>

<template>
  <p v-if="isLoading">Loading..</p>
  <form v-else @submit.prevent.stop="form.handleSubmit">
    <!-- ... -->
  </form>
</template>
```

سيؤدي هذا إلى عرض نص تحميل حتى يتم جلب البيانات، ثم سيتم عرض النموذج مع البيانات التي تم جلبها كقيم أولية.
