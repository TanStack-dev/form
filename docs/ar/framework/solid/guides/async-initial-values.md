---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T22:07:08.009Z'
id: async-initial-values
title: القيم الأولية غير المتزامنة
---

لنفترض أنك تريد جلب بعض البيانات من واجهة برمجة التطبيقات (API) واستخدامها كقيمة أولية لنموذج.

على الرغم من أن هذه المشكلة تبدو بسيطة على السطح، إلا أن هناك تعقيدات خفية ربما لم تفكر فيها حتى الآن.

على سبيل المثال، قد ترغب في عرض مؤشر تحميل أثناء جلب البيانات، أو قد ترغب في معالجة الأخطاء بطريقة أنيقة. وبالمثل، قد تجد نفسك تبحث عن طريقة لتخزين البيانات مؤقتًا حتى لا تضطر إلى جلبها في كل مرة يتم فيها عرض النموذج.

بينما يمكننا تنفيذ العديد من هذه الميزات من الصفر، إلا أن النتيجة النهائية ستشبه إلى حد كبير مشروعًا آخر نعمل عليه: [TanStack Query](https://tanstack.com/query).

لذلك، يوضح لك هذا الدليل كيف يمكنك دمج TanStack Form مع TanStack Query لتحقيق السلوك المطلوب.

## الاستخدام الأساسي

```tsx
import { createForm } from '@tanstack/solid-form'
import { createQuery } from '@tanstack/solid-query'

export default function App() {
  const { data, isLoading } = createQuery(() => ({
    queryKey: ['data'],
    queryFn: async () => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return { firstName: 'FirstName', lastName: 'LastName' }
    },
  }))

  const form = createForm(() => ({
    defaultValues: {
      firstName: data?.firstName ?? '',
      lastName: data?.lastName ?? '',
    },
    onSubmit: async ({ value }) => {
      // Do something with form data
      console.log(value)
    },
  }))

  if (isLoading) return <p>Loading..</p>

  return null
}
```

سيؤدي هذا إلى عرض مؤشر تحميل حتى يتم جلب البيانات، ثم سيتم عرض النموذج مع البيانات التي تم جلبها كقيم أولية.
