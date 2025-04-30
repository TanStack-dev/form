---
source-updated-at: '2025-03-28T15:34:36.000Z'
translation-updated-at: '2025-04-30T21:35:42.958Z'
id: debugging
title: Débogage
---

Voici une liste d'erreurs courantes que vous pourriez rencontrer dans la console et comment les résoudre.

## Changement d'un champ non contrôlé en champ contrôlé

Si vous voyez cette erreur dans la console :

```
Warning: A component is changing an uncontrolled input to be controlled. This is likely caused by the value changing from undefined to a defined value, which should not happen. Decide between using a controlled or uncontrolled input element for the lifetime of the component. More info: https://reactjs.org/link/controlled-components
```

Il est probable que vous ayez oublié les `defaultValues` dans votre Hook `useForm` ou l'utilisation du composant `form.Field`. Cela se produit parce que le champ est rendu avant que la valeur du formulaire ne soit initialisée, passant ainsi de `undefined` à `""` lorsqu'une saisie de texte est effectuée.

## La valeur du champ est de type `unknown`

Si vous utilisez `form.Field` et que, en inspectant la valeur de `field.state.value`, vous constatez qu'elle est de type `unknown`, il est probable que le type de votre formulaire soit trop large pour être évalué de manière sûre.

Cela indique généralement que vous devriez diviser votre formulaire en formulaires plus petits ou utiliser un type plus spécifique pour votre formulaire.

Une solution temporaire à ce problème consiste à effectuer un cast de `field.state.value` en utilisant le mot-clé `as` de TypeScript :

```tsx
const value = field.state.value as string
```

## `Type instantiation is excessively deep and possibly infinite`

Si vous voyez cette erreur dans la console lors de l'exécution de `tsc` :

```
Type instantiation is excessively deep and possibly infinite
```

Vous avez rencontré un bug que nous n'avons pas détecté dans nos définitions de types. Bien que nous ayons fait de notre mieux pour garantir que nos types soient aussi précis que possible, certains cas limites posent problème à TypeScript en raison de la complexité de nos types.

Merci de [nous signaler ce problème sur GitHub](https://github.com/TanStack/form/issues) afin que nous puissions le corriger. Assurez-vous d'inclure une reproduction minimale pour que nous puissions vous aider à déboguer.

> Notez que cette erreur est une erreur TypeScript et non une erreur d'exécution. Cela signifie que votre code s'exécutera toujours comme prévu sur la machine de l'utilisateur.
