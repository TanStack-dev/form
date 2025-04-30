---
source-updated-at: '2025-03-01T06:06:46.000Z'
translation-updated-at: '2025-04-30T21:52:40.833Z'
id: typescript
title: TypeScript
---

TanStack Form ist zu 100 % in **TypeScript** geschrieben und verwendet hochwertige Generics, Constraints und Interfaces, um sicherzustellen, dass die Bibliothek und Ihre Projekte so typsicher wie möglich sind!

Wichtige Hinweise:

- `strict: true` ist in Ihrer `tsconfig.json` erforderlich, um das Beste aus den Typen von TanStack Form herauszuholen
- Die Typen erfordern derzeit TypeScript v5.4 oder höher
- Änderungen an den Typen in diesem Repository gelten als **non-breaking** und werden in der Regel als **Patch**-SemVer-Änderungen veröffentlicht (ansonsten wäre jede Typverbesserung eine Major-Version!).
- Es wird **dringend empfohlen, die Version Ihres react-form-Pakets auf einen bestimmten Patch-Release festzulegen und ein Upgrade mit der Erwartung durchzuführen, dass Typen zwischen Releases korrigiert oder verbessert werden können**.
- Die nicht typbezogene öffentliche API von TanStack Form folgt weiterhin strikt den SemVer-Regeln.
