---
source-updated-at: '2025-03-05T09:46:22.000Z'
translation-updated-at: '2025-04-25T20:31:55.484Z'
id: comparison
title: 比較
---

> ⚠️ 此比較表格正在建構中，內容尚未完全準確。如果您使用過其中任何函式庫並認為資訊可以改進，歡迎透過頁面底部的「Edit this page on Github」連結提出修改建議（需附上說明或佐證資料）。

功能/能力對照說明：

- ✅ 原生支援、內建功能，無需額外設定或程式碼即可使用
- 🟡 支援，但需透過非官方的第三方或社群函式庫/貢獻
- 🔶 支援且有文件說明，但需要使用者自行撰寫額外程式碼來實作
- 🛑 官方不支援或無相關文件

| 功能                                         | TanStack Form                                | Formik                         | Redux Form                             | React Hook Form                                  | Final Form                             |
| -------------------------------------------- | -------------------------------------------- | ------------------------------ | -------------------------------------- | ------------------------------------------------ | -------------------------------------- |
| GitHub 儲存庫/星數                           | [![][stars-tanstack-form]][gh-tanstack-form] | [![][stars-formik]][gh-formik] | [![][stars-redux-form]][gh-redux-form] | [![][stars-react-hook-form]][gh-react-hook-form] | [![][stars-final-form]][gh-final-form] |
| 支援的框架                                   | React, Vue, Angular, Solid, Lit              | React                          | React                                  | React                                            | React, Vue, Angular, Solid, Vanilla JS |
| 套件大小                                     | [![][bp-tanstack-form]][bpl-tanstack-form]   | [![][bp-formik]][bpl-formik]   | [![][bp-redux-form]][bpl-redux-form]   | [![][bp-react-hook-form]][bpl-react-hook-form]   | [![][bp-final-form]][bpl-final-form]   |
| 完善的 TypeScript 支援                       | ✅                                           | ❓                             | ❓                                     | ✅                                               | ✅                                     |
| 完整的 TypeScript 型別推論（包含深層欄位）   | ✅                                           | ❓                             | ❓                                     | ✅                                               | 🛑                                     |
| 無頭 UI 元件 (Headless UI)                   | ✅                                           | ❓                             | ❓                                     | ✅                                               | ❓                                     |
| 框架無關性                                   | ✅                                           | ❓                             | ❓                                     | 🛑                                               | ✅                                     |
| 細粒度響應式更新                             | ✅                                           | ❓                             | ❓                                     | ❓                                               | ✅                                     |
| 支援巢狀物件/陣列欄位                        | ✅                                           | ✅                             | ❓                                     | ✅\*(1)                                          | ✅                                     |
| 非同步驗證                                   | ✅                                           | ✅                             | ❓                                     | ✅                                               | ✅                                     |
| 內建非同步驗證防抖動                         | ✅                                           | ❓                             | ❓                                     | ❓                                               | ❓                                     |
| 基於結構描述的驗證 (Schema-based Validation) | ✅                                           | ✅                             | ❓                                     | ✅                                               | ❓                                     |
| 官方開發者工具                               | 🛑\*(2)                                      | 🛑                             | ✅\*(3)                                | ✅                                               | ❓                                     |
| 伺服器渲染整合 (SSR)                         | ✅                                           | 🛑                             | 🛑                                     | 🛑                                               | 🛑                                     |
| React 編譯器支援                             | ✅                                           | ❓                             | ❓                                     | 🛑                                               | ❓                                     |

\*(1) 對於巢狀陣列，若使用 TypeScript，react-hook-form 要求您[需透過欄位名稱轉換欄位陣列](https://react-hook-form.com/docs/usefieldarray)

\*(2) 規劃中

\*(3) 透過 Redux Devtools 實現

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
