---
source-updated-at: '2025-04-24T18:50:42.000Z'
translation-updated-at: '2025-04-30T22:07:40.062Z'
id: async-initial-values
title: القيم الأولية غير المتزامنة
---

لنفترض أنك تريد جلب بعض البيانات من واجهة برمجة التطبيقات (API) واستخدامها كقيمة أولية لنموذج (form).

رغم أن هذه المشكلة تبدو بسيطة على السطح، إلا أن هناك تعقيدات خفية ربما لم تفكر فيها حتى الآن.

على سبيل المثال، قد ترغب في عرض مؤشر تحميل (loading spinner) أثناء جلب البيانات، أو ربما تريد معالجة الأخطاء بطريقة أنيقة. بالمثل، قد تجد نفسك تبحث عن طريقة لتخزين البيانات مؤقتًا (cache) حتى لا تضطر إلى جلبها في كل مرة يتم فيها عرض النموذج.

رغم أنه يمكننا تنفيذ العديد من هذه الميزات من الصفر، إلا أن النتيجة ستشبه إلى حد كبير مشروعًا آخر نطوره: [TanStack Query](https://tanstack.com/query).

لذلك، يوضح لك هذا الدليل كيفية دمج TanStack Form مع TanStack Query لتحقيق السلوك المطلوب.

## الاستخدام الأساسي

```svelte
<script>
  import { createForm } from '@tanstack/svelte-form'
  import { createQuery } from '@tanstack/svelte-query'

    const { data, isLoading } = createQuery(() => ({
      queryKey: ['data'],
      queryFn: async () => {
        await new Promise((resolve) => setTimeout(resolve, 1000))
        return { firstName: 'FirstName', lastName: 'LastName' }
      },
    }))

    const form = createForm(() => ({
      defaultValues: {
        firstName: $data?.firstName ?? '',
        lastName: $data?.lastName ?? '',
      },
      onSubmit: async ({ value }) => {
        // Do something with form data
        console.log(value)
      },
    }))
</script>

{#if $isLoading}
  <p>Loading...</p>
{:else}
  <!-- form... -->
{/if}
```

سيؤدي هذا إلى عرض مؤشر تحميل حتى يتم جلب البيانات، ثم سيتم عرض النموذج مع البيانات التي تم جلبها كقيم أولية.
