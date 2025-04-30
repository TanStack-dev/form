---
source-updated-at: '2025-03-01T06:06:46.000Z'
translation-updated-at: '2025-04-30T21:47:02.543Z'
id: typescript
title: TypeScript
---

# TypeScript

TanStack Form está escrito 100% en **TypeScript** con genéricos, restricciones e interfaces de la más alta calidad para garantizar que la biblioteca y tus proyectos sean lo más tipados posible.

Aspectos a tener en cuenta:

- `strict: true` es necesario en tu `tsconfig.json` para aprovechar al máximo los tipos de TanStack Form
- Actualmente, los tipos requieren usar TypeScript v5.4 o superior
- Los cambios en los tipos en este repositorio se consideran **no rompedores** y generalmente se lanzan como cambios **patch** en semver (¡de lo contrario, cada mejora de tipos sería una versión mayor!).
- **Se recomienda encarecidamente que fijes la versión de tu paquete react-form a un parche específico y actualices con la expectativa de que los tipos pueden corregirse o mejorarse entre cualquier lanzamiento**
- La API pública de TanStack Form no relacionada con tipos sigue cumpliendo semver de manera muy estricta.
