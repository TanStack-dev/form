---
source-updated-at: '2025-03-20T03:01:05.000Z'
translation-updated-at: '2025-04-30T21:35:19.035Z'
id: reactivity
title: Réactivité
---

# Réactivité (Reactivity)

Tanstack Form ne provoque pas de nouveaux rendus lors de l'interaction avec le formulaire. Vous pourriez donc vous retrouver à essayer d'utiliser une valeur d'état du formulaire ou d'un champ sans succès.

Si vous souhaitez accéder à des valeurs réactives, vous devrez vous y abonner en utilisant l'une des deux méthodes suivantes : `useStore` ou le composant `form.Subscribe`.

Certaines utilisations de ces abonnements incluent l'affichage de valeurs de champs à jour, la détermination de ce qu'il faut afficher en fonction d'une condition, ou l'utilisation de valeurs de champs dans la logique de votre composant.

> Pour les situations où vous souhaitez "réagir" à des déclencheurs, consultez l'API [listener](./listeners.md).

## useStore

Le hook `useStore` est parfait lorsque vous avez besoin d'accéder aux valeurs du formulaire dans la logique de votre composant. `useStore` prend deux paramètres. Premièrement, le store du formulaire. Deuxièmement, un sélecteur pour affiner la partie du formulaire à laquelle vous souhaitez vous abonner.

```tsx
const firstName = useStore(form.store, (state) => state.values.firstName)
const errors = useStore(form.store, (state) => state.errorMap)
```

Vous pouvez accéder à n'importe quelle partie de l'état du formulaire dans le sélecteur.

> Notez que `useStore` provoquera un nouveau rendu complet du composant chaque fois que la valeur à laquelle vous êtes abonné change.

Bien qu'il soit possible d'omettre le sélecteur, résistez à cette envie car cela entraînerait de nombreux rendus inutiles chaque fois que l'état du formulaire change.

## form.Subscribe

Le composant `form.Subscribe` est le mieux adapté lorsque vous devez réagir à quelque chose dans l'interface utilisateur de votre composant. Par exemple, afficher ou masquer des éléments en fonction de la valeur d'un champ de formulaire.

```tsx
<form.Subscribe
  selector={(state) => state.values.firstName}
  children={(firstName) => (
    <form.Field>
      {(field) => (
        <input
          name="lastName"
          value={field.state.lastName}
          onChange={field.handleChange}
        />
      )}
    </form.Field>
  )}
/>
```

> Le composant `form.Subscribe` ne déclenche pas de rendus au niveau du composant. Chaque fois que la valeur à laquelle vous êtes abonné change, seul le composant `form.Subscribe` est re-rendu.

Le choix entre `useStore` et `form.Subscribe` se résume principalement à ceci : si c'est affiché dans l'interface utilisateur, optez pour `form.Subscribe` pour ses optimisations, et si vous avez besoin de réactivité dans la logique, alors `useStore` est le choix à faire.
