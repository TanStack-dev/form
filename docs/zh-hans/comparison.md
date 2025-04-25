---
source-updated-at: '2025-03-05T09:46:22.000Z'
translation-updated-at: '2025-04-12T04:09:03.149Z'
id: comparison
title: 对比
---
> ⚠️ 本对比表格正在完善中，部分信息可能不够准确。如果您使用过其中任一库并认为信息可以改进，欢迎通过页面底部的 "Edit this page on Github" 链接提交修改建议（需附说明或证据）。

功能/能力说明：

- ✅ 开箱即用，无需额外配置或代码
- 🟡 支持，但需通过第三方或社区库实现
- 🔶 支持且有文档，但需要用户自行编写额外代码
- 🛑 官方不支持或无文档

| 功能                                               | TanStack Form                                | Formik                         | Redux Form                             | React Hook Form                                  | Final Form                             |
| -------------------------------------------------- | -------------------------------------------- | ------------------------------ | -------------------------------------- | ------------------------------------------------ | -------------------------------------- |
| GitHub 仓库/星标数                                 | [![][stars-tanstack-form]][gh-tanstack-form] | [![][stars-formik]][gh-formik] | [![][stars-redux-form]][gh-redux-form] | [![][stars-react-hook-form]][gh-react-hook-form] | [![][stars-final-form]][gh-final-form] |
| 支持框架                                           | React, Vue, Angular, Solid, Lit              | React                          | React                                  | React                                            | React, Vue, Angular, Solid, Vanilla JS |
| 打包体积                                           | [![][bp-tanstack-form]][bpl-tanstack-form]   | [![][bp-formik]][bpl-formik]   | [![][bp-redux-form]][bpl-redux-form]   | [![][bp-react-hook-form]][bpl-react-hook-form]   | [![][bp-final-form]][bpl-final-form]   |
| 一流的 TypeScript 支持                             | ✅                                           | ❓                             | ❓                                     | ✅                                               | ✅                                     |
| 完整的 TypeScript 类型推断（包含深层字段）         | ✅                                           | ❓                             | ❓                                     | ✅                                               | 🛑                                     |
| 无头 UI 组件 (Headless UI)                         | ✅                                           | ❓                             | ❓                                     | ✅                                               | ❓                                     |
| 框架无关性                                         | ✅                                           | ❓                             | ❓                                     | 🛑                                               | ✅                                     |
| 细粒度响应式                                       | ✅                                           | ❓                             | ❓                                     | ❓                                               | ✅                                     |
| 嵌套对象/数组字段                                  | ✅                                           | ✅                             | ❓                                     | ✅\*(1)                                          | ✅                                     |
| 异步验证                                           | ✅                                           | ✅                             | ❓                                     | ✅                                               | ✅                                     |
| 内置异步验证防抖                                   | ✅                                           | ❓                             | ❓                                     | ❓                                               | ❓                                     |
| 基于模式的验证 (Schema-based Validation)           | ✅                                           | ✅                             | ❓                                     | ✅                                               | ❓                                     |
| 官方开发者工具                                     | 🛑\*(2)                                      | 🛑                             | ✅\*(3)                                | ✅                                               | ❓                                     |
| 服务端渲染集成 (SSR integrations)                  | ✅                                           | 🛑                             | 🛑                                     | 🛑                                               | 🛑                                     |
| React 编译器支持 (React Compiler support)          | ✅                                           | ❓                             | ❓                                     | 🛑                                               | ❓                                     |

\*(1) 对于嵌套数组，使用 TypeScript 时 react-hook-form 要求[通过字段名进行类型转换](https://react-hook-form.com/docs/usefieldarray)

\*(2) 计划中

\*(3) 通过 Redux Devtools 实现

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
