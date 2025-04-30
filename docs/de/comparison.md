---
source-updated-at: '2025-03-05T09:46:22.000Z'
translation-updated-at: '2025-04-30T21:53:20.410Z'
id: comparison
title: Vergleich
---

## Vergleich | TanStack Form

> ⚠️ Diese Vergleichstabelle befindet sich noch im Aufbau und ist noch nicht vollständig korrekt. Falls Sie eine dieser Bibliotheken nutzen und der Meinung sind, dass die Informationen verbessert werden könnten, können Sie gerne Änderungen vorschlagen (mit Anmerkungen oder Belegen für Behauptungen) über den Link "Edit this page on Github" am Ende dieser Seite.

**Legende für Funktionen/Fähigkeiten:**

- ✅ Erstklassig, integriert und sofort nutzbar ohne zusätzliche Konfiguration oder Code
- 🟡 Unterstützt, aber als inoffizielle Drittanbieter- oder Community-Bibliothek/-Beitrag
- 🔶 Unterstützt und dokumentiert, erfordert jedoch zusätzlichen Benutzercode zur Implementierung
- 🛑 Nicht offiziell unterstützt oder dokumentiert.

| Funktion                                                       | TanStack Form                                | Formik                         | Redux Form                             | React Hook Form                                  | Final Form                             |
| -------------------------------------------------------------- | -------------------------------------------- | ------------------------------ | -------------------------------------- | ------------------------------------------------ | -------------------------------------- |
| Github Repo / Sterne                                           | [![][stars-tanstack-form]][gh-tanstack-form] | [![][stars-formik]][gh-formik] | [![][stars-redux-form]][gh-redux-form] | [![][stars-react-hook-form]][gh-react-hook-form] | [![][stars-final-form]][gh-final-form] |
| Unterstützte Frameworks                                        | React, Vue, Angular, Solid, Lit              | React                          | React                                  | React                                            | React, Vue, Angular, Solid, Vanilla JS |
| Bundle-Größe                                                   | [![][bp-tanstack-form]][bpl-tanstack-form]   | [![][bp-formik]][bpl-formik]   | [![][bp-redux-form]][bpl-redux-form]   | [![][bp-react-hook-form]][bpl-react-hook-form]   | [![][bp-final-form]][bpl-final-form]   |
| Erstklassige TypeScript-Unterstützung                          | ✅                                           | ❓                             | ❓                                     | ✅                                               | ✅                                     |
| Vollständig abgeleitete TypeScript-Typen (inkl. tiefer Felder) | ✅                                           | ❓                             | ❓                                     | ✅                                               | 🛑                                     |
| Headless UI-Komponenten                                        | ✅                                           | ❓                             | ❓                                     | ✅                                               | ❓                                     |
| Framework-unabhängig                                           | ✅                                           | ❓                             | ❓                                     | 🛑                                               | ✅                                     |
| Granulare Reaktivität                                          | ✅                                           | ❓                             | ❓                                     | ❓                                               | ✅                                     |
| Verschachtelte Objekt-/Array-Felder                            | ✅                                           | ✅                             | ❓                                     | ✅\*(1)                                          | ✅                                     |
| Asynchrone Validierung                                         | ✅                                           | ✅                             | ❓                                     | ✅                                               | ✅                                     |
| Integrierte Debounce-Funktion für asynchrone Validierung       | ✅                                           | ❓                             | ❓                                     | ❓                                               | ❓                                     |
| Schema-basierte Validierung                                    | ✅                                           | ✅                             | ❓                                     | ✅                                               | ❓                                     |
| Offizielle Entwicklerwerkzeuge                                 | 🛑\*(2)                                      | 🛑                             | ✅\*(3)                                | ✅                                               | ❓                                     |
| SSR-Integration                                                | ✅                                           | 🛑                             | 🛑                                     | 🛑                                               | 🛑                                     |
| React-Compiler-Unterstützung                                   | ✅                                           | ❓                             | ❓                                     | 🛑                                               | ❓                                     |

\*(1) Für verschachtelte Arrays erfordert react-hook-form [eine Typumwandlung des Feldarrays anhand seines Namens](https://react-hook-form.com/docs/usefieldarray), wenn TypeScript verwendet wird.

\*(2) Geplant

\*(3) Über Redux Devtools

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
