---
source-updated-at: '2025-03-05T09:46:22.000Z'
translation-updated-at: '2025-04-30T22:04:58.271Z'
id: comparison
title: المقارنة
---

> ⚠️ جدول المقارنة هذا قيد الإنشاء وهو ليس دقيقًا بالكامل بعد. إذا كنت تستخدم أيًا من هذه المكتبات وتشعر أن المعلومات يمكن تحسينها، فلا تتردد في اقتراح التغييرات (مع ملاحظات أو أدلة على الادعاءات) باستخدام رابط "Edit this page on Github" الموجود في أسفل هذه الصفحة.

مفتاح الميزات/الإمكانيات:

- ✅ مدعوم بشكل أساسي وجاهز للاستخدام دون الحاجة إلى إعدادات أو أكواد إضافية
- 🟡 مدعوم، ولكن عبر مكتبة أو مساهمة مجتمعية غير رسمية
- 🔶 مدعوم وموثق، لكنه يتطلب كتابة أكواد إضافية من المستخدم لتنفيذه
- 🛑 غير مدعوم رسميًا أو موثق

| الميزة                                                  | TanStack Form                                | Formik                         | Redux Form                             | React Hook Form                                  | Final Form                             |
| ------------------------------------------------------- | -------------------------------------------- | ------------------------------ | -------------------------------------- | ------------------------------------------------ | -------------------------------------- |
| مستودع جيتهاب / النجوم                                  | [![][stars-tanstack-form]][gh-tanstack-form] | [![][stars-formik]][gh-formik] | [![][stars-redux-form]][gh-redux-form] | [![][stars-react-hook-form]][gh-react-hook-form] | [![][stars-final-form]][gh-final-form] |
| أطر العمل المدعومة                                      | React, Vue, Angular, Solid, Lit              | React                          | React                                  | React                                            | React, Vue, Angular, Solid, Vanilla JS |
| حجم الحزمة                                              | [![][bp-tanstack-form]][bpl-tanstack-form]   | [![][bp-formik]][bpl-formik]   | [![][bp-redux-form]][bpl-redux-form]   | [![][bp-react-hook-form]][bpl-react-hook-form]   | [![][bp-final-form]][bpl-final-form]   |
| دعم TypeScript بشكل أساسي                               | ✅                                           | ❓                             | ❓                                     | ✅                                               | ✅                                     |
| استنتاج TypeScript الكامل (بما في ذلك الحقول المتداخلة) | ✅                                           | ❓                             | ❓                                     | ✅                                               | 🛑                                     |
| مكونات واجهة مستخدم Headless                            | ✅                                           | ❓                             | ❓                                     | ✅                                               | ❓                                     |
| مستقل عن أطر العمل                                      | ✅                                           | ❓                             | ❓                                     | 🛑                                               | ✅                                     |
| تفاعلية دقيقة                                           | ✅                                           | ❓                             | ❓                                     | ❓                                               | ✅                                     |
| حقول كائنات/مصفوفات متداخلة                             | ✅                                           | ✅                             | ❓                                     | ✅\*(1)                                          | ✅                                     |
| تحقق غير متزامن                                         | ✅                                           | ✅                             | ❓                                     | ✅                                               | ✅                                     |
| تضبيط التحقق غير المتزامن مدمج                          | ✅                                           | ❓                             | ❓                                     | ❓                                               | ❓                                     |
| تحقق قائم على Schema                                    | ✅                                           | ✅                             | ❓                                     | ✅                                               | ❓                                     |
| أدوات مطورين من الدرجة الأولى                           | 🛑\*(2)                                      | 🛑                             | ✅\*(3)                                | ✅                                               | ❓                                     |
| تكاملات SSR                                             | ✅                                           | 🛑                             | 🛑                                     | 🛑                                               | 🛑                                     |
| دعم React Compiler                                      | ✅                                           | ❓                             | ❓                                     | 🛑                                               | ❓                                     |

\*(1) بالنسبة للمصفوفات المتداخلة، يتطلب react-hook-form [إجراء تحويل لحقل المصفوفة باسمه](https://react-hook-form.com/docs/usefieldarray) إذا كنت تستخدم TypeScript

\*(2) مخطط له

\*(3) عبر أدوات Redux للمطورين

[bpl-tanstack-form]: https://bundlephobia.com/result?p=@tanstack/react-form
[bp-tanstack-form]: https://badgen.net/bundlephobia/minzip/@tanstack/react-form?label=💾
[gh-tanstack-form]: https://github.com/TanStack/form
[stars-tanstack-form]: https://img.shields.io/github/stars/TanStack/form?label=%F0%9F%8C%9F
[bpl-formik]: https://bundlephobia.com/result?p=formik
[bp-formik]: https://badgen.net/bundlephobia/minzip/formik?label=💾
[gh-formik]: https://github.com/jaredpalmer/formik
[stars-formik]: https://img.shields.io/github/stars/jaredpalmer/formik?label=%F0%9F%8C%9F
[bpl-redux-form]: https://bundlephobia.com/result?p=redux-form
[bp-redux-form]: https://badgen.net/bundlephobia/minzip/redux-form?label=💾
[gh-redux-form]: https://github.com/redux-form/redux-form
[stars-redux-form]: https://img.shields.io/github/stars/redux-form/redux-form?label=%F0%9F%8C%9F
[bpl-react-hook-form]: https://bundlephobia.com/result?p=react-hook-form
[bp-react-hook-form]: https://badgen.net/bundlephobia/minzip/react-hook-form?label=💾
[gh-react-hook-form]: https://github.com/react-hook-form/react-hook-form
[stars-react-hook-form]: https://img.shields.io/github/stars/react-hook-form/react-hook-form?label=%F0%9F%8C%9F
[bpl-final-form]: https://bundlephobia.com/result?p=final-form
[bp-final-form]: https://badgen.net/bundlephobia/minzip/final-form?label=💾
[gh-final-form]: https://github.com/final-form/final-form
[stars-final-form]: https://img.shields.io/github/stars/final-form/final-form?label=%F0%9F%8C%9F
