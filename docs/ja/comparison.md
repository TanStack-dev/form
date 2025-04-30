---
source-updated-at: '2025-03-05T09:46:22.000Z'
translation-updated-at: '2025-04-30T21:24:57.734Z'
id: comparison
title: 比較
---

## 比較

> ⚠️ この比較表は作成中であり、まだ完全に正確ではありません。これらのライブラリを使用していて、情報を改善できると思われる場合は、ページ下部の「Edit this page on Github」リンクから変更を提案してください（主張の根拠となるメモや証拠とともに）。

機能/能力の凡例:

- ✅ 追加の設定やコードなしで、最初から組み込まれて利用可能
- 🟡 サポートされているが、非公式のサードパーティまたはコミュニティライブラリ/貢献によるもの
- 🔶 サポートされておりドキュメント化されているが、実装には追加のユーザーコードが必要
- 🛑 公式にはサポートまたはドキュメント化されていない

| 機能                                         | TanStack Form                                | Formik                         | Redux Form                             | React Hook Form                                  | Final Form                             |
| -------------------------------------------- | -------------------------------------------- | ------------------------------ | -------------------------------------- | ------------------------------------------------ | -------------------------------------- |
| GitHubリポジトリ / スター数                  | [![][stars-tanstack-form]][gh-tanstack-form] | [![][stars-formik]][gh-formik] | [![][stars-redux-form]][gh-redux-form] | [![][stars-react-hook-form]][gh-react-hook-form] | [![][stars-final-form]][gh-final-form] |
| サポートフレームワーク                       | React, Vue, Angular, Solid, Lit              | React                          | React                                  | React                                            | React, Vue, Angular, Solid, Vanilla JS |
| バンドルサイズ                               | [![][bp-tanstack-form]][bpl-tanstack-form]   | [![][bp-formik]][bpl-formik]   | [![][bp-redux-form]][bpl-redux-form]   | [![][bp-react-hook-form]][bpl-react-hook-form]   | [![][bp-final-form]][bpl-final-form]   |
| ファーストクラスのTypeScriptサポート         | ✅                                           | ❓                             | ❓                                     | ✅                                               | ✅                                     |
| 完全なTypeScript推論（深いフィールドを含む） | ✅                                           | ❓                             | ❓                                     | ✅                                               | 🛑                                     |
| ヘッドレスUIコンポーネント                   | ✅                                           | ❓                             | ❓                                     | ✅                                               | ❓                                     |
| フレームワーク非依存                         | ✅                                           | ❓                             | ❓                                     | 🛑                                               | ✅                                     |
| 細かいリアクティビティ                       | ✅                                           | ❓                             | ❓                                     | ❓                                               | ✅                                     |
| ネストされたオブジェクト/配列フィールド      | ✅                                           | ✅                             | ❓                                     | ✅\*(1)                                          | ✅                                     |
| 非同期バリデーション                         | ✅                                           | ✅                             | ❓                                     | ✅                                               | ✅                                     |
| 組み込みの非同期バリデーションデバウンス     | ✅                                           | ❓                             | ❓                                     | ❓                                               | ❓                                     |
| スキーマベースのバリデーション               | ✅                                           | ✅                             | ❓                                     | ✅                                               | ❓                                     |
| 公式開発ツール                               | 🛑\*(2)                                      | 🛑                             | ✅\*(3)                                | ✅                                               | ❓                                     |
| SSR (サーバーサイドレンダリング) 連携        | ✅                                           | 🛑                             | 🛑                                     | 🛑                                               | 🛑                                     |
| React Compilerサポート                       | ✅                                           | ❓                             | ❓                                     | 🛑                                               | ❓                                     |

\*(1) ネストされた配列の場合、TypeScriptを使用している場合、react-hook-formでは[フィールド配列を名前でキャストする必要があります](https://react-hook-form.com/docs/usefieldarray)

\*(2) 計画中

\*(3) Redux Devtools経由

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
