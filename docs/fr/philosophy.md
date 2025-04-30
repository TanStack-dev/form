---
source-updated-at: '2025-04-16T08:45:06.000Z'
translation-updated-at: '2025-04-30T21:35:03.589Z'
id: philosophy
title: Philosophie
---

# Philosophie

Tout projet bien établi devrait avoir une philosophie qui guide son développement. Sans une philosophie centrale, le développement peut stagner dans une prise de décision sans fin et aboutir à des API moins solides.

Ce document présente les principes fondamentaux qui orientent le développement et les fonctionnalités de TanStack Form.

## Mise à niveau des API unifiées

Les API impliquent des compromis. Par conséquent, il peut être tentant de rendre chaque ensemble de compromis disponible pour l'utilisateur via différentes API. Cependant, cela peut conduire à une API fragmentée, plus difficile à apprendre et à utiliser.

Bien que cela puisse signifier une courbe d'apprentissage plus élevée, cela signifie que vous n'avez pas à vous demander quelle API utiliser en interne ou à supporter une charge cognitive supplémentaire lors du passage d'une API à une autre.

## Les formulaires nécessitent de la flexibilité

TanStack Form est conçu pour être flexible et personnalisable. Bien que de nombreux formulaires puissent suivre des modèles similaires, il existe toujours des exceptions, en particulier lorsque les formulaires sont un composant central de votre application.

Par conséquent, TanStack Form prend en charge plusieurs méthodes de validation :

- **Personnalisation du timing** : Vous pouvez valider lors du blur, du changement, de la soumission ou même au montage.
- **Stratégies de validation** : Vous pouvez valider des champs individuels, l'ensemble du formulaire ou un sous-ensemble de champs.
- **Logique de validation personnalisée** : Vous pouvez écrire votre propre logique de validation ou utiliser une bibliothèque comme [Zod](https://zod.dev/) ou [Valibot](https://valibot.dev/).
- **Messages d'erreur personnalisés** : Vous pouvez personnaliser les messages d'erreur pour chaque champ en retournant n'importe quel objet depuis un validateur.
- **Validation asynchrone** : Vous pouvez valider des champs de manière asynchrone et bénéficier d'utilitaires courants comme le debouncing et l'annulation gérés pour vous.

## Le mode contrôlé est cool

Dans un monde où les entrées contrôlées vs non contrôlées sont un sujet brûlant, TanStack Form se positionne fermement dans le camp contrôlé.

Cela présente plusieurs avantages :

- **Prévisible** : Vous pouvez prédire l'état de votre formulaire à tout moment.
- **Tests plus faciles** : Vous pouvez facilement tester vos formulaires en passant des valeurs et en vérifiant le résultat.
- **Prise en charge hors DOM** : Vous pouvez utiliser TanStack Form avec React Native, des adaptateurs de framework comme Three.js ou tout autre moteur de rendu.
- **Logique conditionnelle améliorée** : Vous pouvez facilement afficher/masquer des champs conditionnellement en fonction de l'état du formulaire.
- **Débogage** : Vous pouvez facilement enregistrer l'état du formulaire dans la console pour déboguer les problèmes.

## Les génériques sont sinistres

Vous ne devriez jamais avoir besoin de passer un générique ou d'utiliser un type interne lorsque vous utilisez TanStack Form. En effet, nous avons conçu la bibliothèque pour inférer tout à partir des valeurs par défaut à l'exécution.

Lorsque vous écrivez du code TanStack Form suffisamment correct, vous ne devriez pas pouvoir distinguer entre une utilisation en JavaScript et une utilisation en TypeScript, à l'exception des éventuels casts de type que vous pourriez effectuer sur des valeurs d'exécution.

Au lieu de :

```typescript
useForm<MyForm>()
```

Vous devriez faire :

```typescript
interface Person {
  name: string
  age: number
}

const defaultPerson: Person = { name: 'Bill Luo', age: 24 }

useForm({
  defaultValues: defaultPerson,
})
```

## Les bibliothèques sont libératrices

L'un des principaux objectifs de TanStack Form est que vous puissiez l'intégrer dans votre propre système de composants ou système de design.

Pour soutenir cela, nous avons plusieurs utilitaires qui facilitent la création de vos propres composants et hooks personnalisés :

```typescript
// Exporté depuis votre propre bibliothèque avec des composants pré-liés pour vos formulaires.
export const { useAppForm, withForm } = createFormHook(/* options */)
```

Sans cela, vous ajoutez considérablement plus de boilerplate à vos applications et rendez vos formulaires moins cohérents et conviviaux.
