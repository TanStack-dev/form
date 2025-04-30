---
source-updated-at: '2025-03-28T15:34:36.000Z'
translation-updated-at: '2025-04-30T22:04:42.199Z'
id: async-initial-values
title: القيم الأولية غير المتزامنة
---

لنفترض أنك تريد جلب بعض البيانات من واجهة برمجة التطبيقات (API) واستخدامها كقيمة أولية لنموذج.

رغم أن هذه المشكلة تبدو بسيطة على السطح، إلا أن هناك تعقيدات خفية ربما لم تفكر فيها حتى الآن.

على سبيل المثال، قد ترغب في عرض مؤشر تحميل أثناء جلب البيانات، أو التعامل مع الأخطاء بطريقة أنيقة. وبالمثل، قد تجد نفسك تبحث عن طريقة لتخزين البيانات مؤقتًا (cache) حتى لا تضطر إلى جلبها في كل مرة يتم فيها عرض النموذج.

بينما يمكننا تنفيذ العديد من هذه الميزات من الصفر، إلا أن النتيجة ستشبه إلى حد كبير مشروعًا آخر نشارك في تطويره: [TanStack Query](https://tanstack.com/query).

لذلك، يوضح لك هذا الدليل كيف يمكنك دمج TanStack Form مع TanStack Query لتحقيق السلوك المطلوب.

## الاستخدام الأساسي

```tsx
import { useForm } from '@tanstack/react-form'
import { useQuery } from '@tanstack/react-query'

export default function App() {
  const {data, isLoading} = useQuery({
    queryKey: ['data'],
    queryFn: async () => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return {firstName: 'FirstName', lastName: "LastName"}
    }
  })

  const form = useForm({
    defaultValues: {
      firstName: data?.firstName ?? '',
      lastName: data?.lastName ?? '',
    },
    onSubmit: async ({ value }) => {
      // Do something with form data
      console.log(value)
    },
  })

  if (isLoading) return <p>Loading..</p>

  return (
    // ...
  )
}
```

سيؤدي هذا إلى عرض مؤشر تحميل حتى يتم جلب البيانات، ثم عرض النموذج مع البيانات التي تم جلبها كقيم أولية.
