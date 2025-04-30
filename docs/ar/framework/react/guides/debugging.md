---
source-updated-at: '2025-03-28T15:34:36.000Z'
translation-updated-at: '2025-04-30T22:05:28.951Z'
id: debugging
title: تصحيح الأخطاء
---

هذه قائمة بالأخطاء الشائعة التي قد تظهر في وحدة التحكم وكيفية إصلاحها.

## تغيير إدخال غير خاضع للتحكم إلى خاضع للتحكم

إذا ظهر هذا الخطأ في وحدة التحكم:

```
Warning: A component is changing an uncontrolled input to be controlled. This is likely caused by the value changing from undefined to a defined value, which should not happen. Decide between using a controlled or uncontrolled input element for the lifetime of the component. More info: https://reactjs.org/link/controlled-components
```

فمن المحتمل أنك نسيت إضافة `defaultValues` في خطاف `useForm` أو استخدام مكون `form.Field`. يحدث هذا لأن حقل الإدخال يتم عرضه قبل تهيئة قيمة النموذج، وبالتالي يتغير من `undefined` إلى `""` عند إدخال نص.

## قيمة الحقل من النوع `unknown`

إذا كنت تستخدم `form.Field` وعند فحص قيمة `field.state.value` وجدت أن قيمة الحقل من النوع `unknown`، فمن المحتمل أن نوع النموذج كان كبيرًا جدًا بحيث لا يمكن تقييمه بأمان.

عادةً ما يكون هذا مؤشرًا على أنك يجب أن تقسم النموذج إلى نماذج أصغر أو تستخدم نوعًا أكثر تحديدًا للنموذج.

حل مؤقت لهذه المشكلة هو تحويل `field.state.value` باستخدام الكلمة المفتاحية `as` في TypeScript:

```tsx
const value = field.state.value as string
```

## `Type instantiation is excessively deep and possibly infinite`

إذا ظهر هذا الخطأ في وحدة التحكم عند تشغيل `tsc`:

```
Type instantiation is excessively deep and possibly infinite
```

فقد واجهت خطأً لم نتمكن من اكتشافه في تعريفات الأنواع الخاصة بنا. بينما بذلنا قصارى جهدنا لضمان دقة الأنواع، هناك بعض الحالات النادرة حيث واجه TypeScript صعوبة في تعقيد الأنواع لدينا.

يرجى [إبلاغنا بهذه المشكلة على GitHub](https://github.com/TanStack/form/issues) حتى نتمكن من إصلاحها. فقط تأكد من تضمين مثال بسيط لإعادة إنتاج المشكلة حتى نتمكن من مساعدتك في تصحيحها.

> ضع في اعتبارك أن هذا الخطأ هو خطأ TypeScript وليس خطأ وقت التشغيل. هذا يعني أن الكود الخاص بك سيظل يعمل على جهاز المستخدم كما هو متوقع.
