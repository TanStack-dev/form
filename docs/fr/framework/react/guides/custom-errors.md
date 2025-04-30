---
source-updated-at: '2025-03-04T14:23:57.000Z'
translation-updated-at: '2025-04-30T21:36:16.713Z'
id: custom-errors
title: Erreurs personnalisées
---

# Erreurs personnalisées

TanStack Form offre une flexibilité totale concernant les types de valeurs d'erreur que vous pouvez retourner depuis les validateurs. Les erreurs sous forme de chaînes de caractères sont les plus courantes et les plus simples à manipuler, mais la bibliothèque vous permet de retourner n'importe quel type de valeur depuis vos validateurs.

En règle générale, toute valeur truthy est considérée comme une erreur et marquera le formulaire ou le champ comme invalide, tandis que les valeurs falsy (`false`, `undefined`, `null`, etc.) indiquent qu'il n'y a pas d'erreur, le formulaire ou le champ est valide.

## Retourner des valeurs de type chaîne depuis les formulaires

```tsx
<form.Field
  name="username"
  validators={{
    onChange: ({ value }) =>
      value.length < 3 ? 'Username must be at least 3 characters' : undefined,
  }}
/>
```

Pour une validation au niveau du formulaire affectant plusieurs champs :

```tsx
const form = useForm({
  defaultValues: {
    username: '',
    email: '',
  },
  validators: {
    onChange: ({ value }) => {
      return {
        fields: {
          username:
            value.username.length < 3 ? 'Username too short' : undefined,
          email: !value.email.includes('@') ? 'Invalid email' : undefined,
        },
      }
    },
  },
})
```

Les erreurs sous forme de chaînes sont le type le plus courant et s'affichent facilement dans votre interface :

```tsx
{
  field.state.meta.errors.map((error, i) => (
    <div key={i} className="error">
      {error}
    </div>
  ))
}
```

### Nombres

Utiles pour représenter des quantités, seuils ou magnitudes :

```tsx
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) => (value < 18 ? 18 - value : undefined),
  }}
/>
```

Affichage dans l'interface :

```tsx
{
  /* TypeScript sait que l'erreur est un nombre grâce à votre validateur */
}
;<div className="error">
  Il vous faut {field.state.meta.errors[0]} années supplémentaires pour être
  éligible
</div>
```

### Booléens

Indicateurs simples d'état d'erreur :

```tsx
<form.Field
  name="accepted"
  validators={{
    onChange: ({ value }) => (!value ? true : undefined),
  }}
/>
```

Affichage dans l'interface :

```tsx
{
  field.state.meta.errors[0] === true && (
    <div className="error">Vous devez accepter les conditions</div>
  )
}
```

### Objets

Objets d'erreur riches avec plusieurs propriétés :

```tsx
<form.Field
  name="email"
  validators={{
    onChange: ({ value }) => {
      if (!value.includes('@')) {
        return {
          message: "Format d'email invalide",
          severity: 'error',
          code: 1001,
        }
      }
      return undefined
    },
  }}
/>
```

Affichage dans l'interface :

```tsx
{
  typeof field.state.meta.errors[0] === 'object' && (
    <div className={`error ${field.state.meta.errors[0].severity}`}>
      {field.state.meta.errors[0].message}
      <small> (Code : {field.state.meta.errors[0].code})</small>
    </div>
  )
}
```

Dans l'exemple ci-dessus, cela dépend de l'erreur que vous souhaitez afficher.

### Tableaux

Messages d'erreur multiples pour un seul champ :

```tsx
<form.Field
  name="password"
  validators={{
    onChange: ({ value }) => {
      const errors = []
      if (value.length < 8) errors.push('Mot de passe trop court')
      if (!/[A-Z]/.test(value)) errors.push('Lettre majuscule manquante')
      if (!/[0-9]/.test(value)) errors.push('Chiffre manquant')

      return errors.length ? errors : undefined
    },
  }}
/>
```

Affichage dans l'interface :

```tsx
{
  Array.isArray(field.state.meta.errors) && (
    <ul className="error-list">
      {field.state.meta.errors.map((err, i) => (
        <li key={i}>{err}</li>
      ))}
    </ul>
  )
}
```

## La prop `disableErrorFlat` sur les champs

Par défaut, TanStack Form aplatit les erreurs de toutes les sources de validation (onChange, onBlur, onSubmit) dans un seul tableau `errors`. La prop `disableErrorFlat` préserve les sources des erreurs :

```tsx
<form.Field
  name="email"
  disableErrorFlat
  validators={{
    onChange: ({ value }) =>
      !value.includes('@') ? "Format d'email invalide" : undefined,
    onBlur: ({ value }) =>
      !value.endsWith('.com')
        ? 'Seuls les domaines .com sont autorisés'
        : undefined,
    onSubmit: ({ value }) =>
      value.length < 5 ? 'Email trop court' : undefined,
  }}
/>
```

Sans `disableErrorFlat`, toutes les erreurs seraient combinées dans `field.state.meta.errors`. Avec cette prop, vous pouvez accéder aux erreurs par leur source :

```tsx
{
  field.state.meta.errorMap.onChange && (
    <div className="real-time-error">{field.state.meta.errorMap.onChange}</div>
  )
}

{
  field.state.meta.errorMap.onBlur && (
    <div className="blur-feedback">{field.state.meta.errorMap.onBlur}</div>
  )
}

{
  field.state.meta.errorMap.onSubmit && (
    <div className="submit-error">{field.state.meta.errorMap.onSubmit}</div>
  )
}
```

Ceci est utile pour :

- Afficher différents types d'erreurs avec des traitements d'interface différents
- Prioriser les erreurs (par exemple, afficher les erreurs de soumission plus visiblement)
- Implémenter une divulgation progressive des erreurs

## Sécurité de type des `errors` et `errorMap`

TanStack Form offre une forte sécurité de type pour la gestion des erreurs. Chaque clé dans `errorMap` a exactement le type retourné par son validateur correspondant, tandis que le tableau `errors` contient un type union de toutes les valeurs d'erreur possibles de tous les validateurs :

```tsx
<form.Field
  name="password"
  validators={{
    onChange: ({ value }) => {
      // Retourne une chaîne ou undefined
      return value.length < 8 ? 'Trop court' : undefined
    },
    onBlur: ({ value }) => {
      // Retourne un objet ou undefined
      if (!/[A-Z]/.test(value)) {
        return { message: 'Majuscule manquante', level: 'warning' }
      }
      return undefined
    },
  }}
  children={(field) => {
    // TypeScript sait que errors[0] peut être string | { message: string, level: string } | undefined
    const error = field.state.meta.errors[0]

    // Gestion d'erreur type-safe
    if (typeof error === 'string') {
      return <div className="string-error">{error}</div>
    } else if (error && typeof error === 'object') {
      return <div className={error.level}>{error.message}</div>
    }

    return null
  }}
/>
```

La propriété `errorMap` est également entièrement typée, correspondant aux types de retour de vos fonctions de validation :

```tsx
// Avec disableErrorFlat
<form.Field
  name="email"
  disableErrorFlat
  validators={{
    onChange: ({ value }): string | undefined =>
      !value.includes("@") ? "Email invalide" : undefined,
    onBlur: ({ value }): { code: number, message: string } | undefined =>
      !value.endsWith(".com") ? { code: 100, message: "Domaine incorrect" } : undefined
  }}
  children={(field) => {
    // TypeScript connaît le type exact de chaque source d'erreur
    const onChangeError: string | undefined = field.state.meta.errorMap.onChange;
    const onBlurError: { code: number, message: string } | undefined = field.state.meta.errorMap.onBlur;

    return (/* ... */);
  }}
/>
```

Cette sécurité de type aide à détecter les erreurs à la compilation plutôt qu'à l'exécution, rendant votre code plus fiable et maintenable.
