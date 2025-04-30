---
source-updated-at: '2025-03-01T06:06:46.000Z'
translation-updated-at: '2025-04-30T21:34:35.028Z'
id: typescript
title: TypeScript
---

# TypeScript

TanStack Form est écrit à 100% en **TypeScript** avec des génériques, des contraintes et des interfaces de la plus haute qualité pour garantir que la bibliothèque et vos projets soient aussi type-safe que possible !

Points à garder à l'esprit :

- `strict: true` est requis dans votre `tsconfig.json` pour tirer le meilleur parti des types de TanStack Form
- Les types nécessitent actuellement TypeScript v5.4 ou supérieur
- Les modifications apportées aux types dans ce dépôt sont considérées comme **non breaking** et sont généralement publiées sous forme de modifications **patch** semver (sinon chaque amélioration de type serait une version majeure !).
- Il est **fortement recommandé de verrouiller la version de votre package react-form sur un numéro de patch spécifique et de mettre à niveau en s'attendant à ce que les types puissent être corrigés ou améliorés entre les versions**
- L'API publique de TanStack Form non liée aux types suit toujours semver très strictement.
