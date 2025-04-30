---
source-updated-at: '2025-03-28T15:34:36.000Z'
translation-updated-at: '2025-04-30T21:35:55.481Z'
id: ui-libraries
title: Bibliothèques d'interface
---

## Utilisation de TanStack Form avec des bibliothèques d'interface utilisateur (UI Libraries)

TanStack Form est une bibliothèque headless, offrant une flexibilité totale pour le styliser selon vos besoins. Elle est compatible avec un large éventail de bibliothèques d'interface utilisateur, notamment `Tailwind`, `Material UI`, `Mantine`, ou même du CSS simple.

Ce guide se concentre sur `Material UI` et `Mantine`, mais les concepts sont applicables à toute bibliothèque d'interface utilisateur de votre choix.

### Prérequis

Avant d'intégrer TanStack Form avec une bibliothèque d'interface utilisateur, assurez-vous que les dépendances nécessaires sont installées dans votre projet :

- Pour `Material UI`, suivez les instructions d'installation sur leur [site officiel](https://mui.com/material-ui/getting-started/).
- Pour `Mantine`, consultez leur [documentation](https://mantine.dev/).

Remarque : Bien qu'il soit possible de mélanger les bibliothèques, il est généralement conseillé de n'en utiliser qu'une seule pour maintenir la cohérence et minimiser le surpoids.

### Exemple avec Mantine

Voici un exemple démontrant l'intégration de TanStack Form avec les composants Mantine :

```tsx
import { TextInput, Checkbox } from '@mantine/core'
import { useForm } from '@tanstack/react-form'

export default function App() {
  const { Field, handleSubmit, state } = useForm({
    defaultValues: {
      firstName: '',
      lastName: '',
      isChecked: false,
    },
    onSubmit: async ({ value }) => {
      // Handle form submission
      console.log(value)
    },
  })

  return (
    <>
      <form
        onSubmit={(e) => {
          e.preventDefault()
          handleSubmit()
        }}
      >
        <Field
          name="firstName"
          children={({ state, handleChange, handleBlur }) => (
            <TextInput
              defaultValue={state.value}
              onChange={(e) => handleChange(e.target.value)}
              onBlur={handleBlur}
              placeholder="Enter your name"
            />
          )}
        />
        <Field
          name="isChecked"
          children={({ state, handleChange, handleBlur }) => (
            <Checkbox
              onChange={(e) => handleChange(e.target.checked)}
              onBlur={handleBlur}
              checked={state.value}
            />
          )}
        />
      </form>
      <div>
        <pre>{JSON.stringify(state.values, null, 2)}</pre>
      </div>
    </>
  )
}
```

- Initialement, nous utilisons le hook `useForm` de TanStack et déstructurons les propriétés nécessaires. Cette étape est facultative ; vous pourriez aussi utiliser `const form = useForm()` si vous préférez. L'inférence de types de TypeScript garantit une expérience fluide quelle que soit l'approche.
- Le composant `Field`, dérivé de `useForm`, accepte plusieurs propriétés, telles que `validators`. Pour cette démonstration, nous nous concentrons sur deux propriétés principales : `name` et `children`.
  - La propriété `name` identifie chaque `Field`, par exemple, `firstName` dans notre exemple.
  - La propriété `children` exploite le concept de render props, nous permettant d'intégrer des composants sans abstractions inutiles.
- La conception de TanStack repose fortement sur les render props, fournissant un accès à `children` au sein du composant `Field`. Cette approche est entièrement type-safe. Lors de l'intégration avec des composants Mantine, comme `TextInput`, nous déstructurons sélectivement des propriétés comme `state.value`, `handleChange` et `handleBlur`. Cette approche sélective est due aux légères différences de types entre `TextInput` et le `field` que nous obtenons dans les enfants.
- En suivant ces étapes, vous pouvez intégrer de manière transparente les composants Mantine avec TanStack Form.
- Cette méthodologie est également applicable à d'autres composants, comme `Checkbox`, garantissant une intégration cohérente entre différents éléments d'interface utilisateur.

### Utilisation avec Material UI

Le processus d'intégration des composants Material UI est similaire. Voici un exemple utilisant TextField et Checkbox de Material UI :

```tsx
        <Field
            name="lastName"
            children={({ state, handleChange, handleBlur }) => {
              return (
                <TextField
                  id="filled-basic"
                  label="Filled"
                  variant="filled"
                  defaultValue={state.value}
                  onChange={(e) => handleChange(e.target.value)}
                  onBlur={handleBlur}
                  placeholder="Enter your last name"
                />
              );
            }}
          />

           <Field
            name="isMuiCheckBox"
            children={({ state, handleChange, handleBlur }) => {
              return (
                <MuiCheckbox
                  onChange={(e) => handleChange(e.target.checked)}
                  onBlur={handleBlur}
                  checked={state.value}
                />
              );
            }}
          />

```

- L'approche d'intégration est la même qu'avec Mantine.
- La principale différence réside dans les propriétés spécifiques des composants Material UI et les options de style.
