---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T21:37:33.894Z'
id: basic-concepts
title: Concepts de base
---

Cette page présente les concepts de base et la terminologie utilisés dans la bibliothèque `@tanstack/angular-form`. Se familiariser avec ces concepts vous aidera à mieux comprendre et travailler avec la bibliothèque.

## Instance de formulaire (Form Instance)

Une instance de formulaire (Form Instance) est un objet qui représente un formulaire individuel et fournit des méthodes et des propriétés pour travailler avec le formulaire. Vous créez une instance de formulaire en utilisant la fonction `injectForm`. Cette fonction accepte un objet avec une fonction `onSubmit`, qui est appelée lorsque le formulaire est soumis.

```typescript
const form = injectForm({
  onSubmit: async ({ value }) => {
    // Faire quelque chose avec les données du formulaire
    console.log(value)
  },
})
```

## Champ (Field)

Un champ (Field) représente un seul élément de saisie de formulaire, comme un champ de texte ou une case à cocher. Les champs sont créés en utilisant la directive `tanstackField`. La directive accepte une propriété `name`, qui doit correspondre à une clé dans les valeurs par défaut du formulaire. Elle expose également une instance nommée `field` des internes de la directive, qui doit être utilisée via une variable de modèle pour accéder aux internes du champ.

Exemple :

```html
<ng-container [tanstackField]="form" name="firstName" #firstName="field">
  <input
    [value]="firstName.api.state.value"
    (blur)="firstName.api.handleBlur()"
    (input)="firstName.api.handleChange($any($event).target.value)"
  />
</ng-container>
```

## État du champ (Field State)

Chaque champ a son propre état (Field State), qui inclut sa valeur actuelle, son statut de validation, ses messages d'erreur et d'autres métadonnées. Vous pouvez accéder à l'état d'un champ en utilisant la propriété `fieldApi.state`.

Exemple :

```tsx
const {
  value,
  meta: { errors, isValidating },
} = field.state
```

Il existe trois états de champ qui peuvent être très utiles pour voir comment l'utilisateur interagit avec un champ. Un champ est _"touché" (touched)_ lorsque l'utilisateur clique ou se déplace dessus, _"vierge" (pristine)_ jusqu'à ce que l'utilisateur modifie sa valeur, et _"modifié" (dirty)_ après que la valeur a été changée. Vous pouvez vérifier ces états via les indicateurs `isTouched`, `isPristine` et `isDirty`, comme vu ci-dessous.

```tsx
const { isTouched, isPristine, isDirty } = field.state.meta
```

![États du champ](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)

## API de champ (Field API)

L'API de champ (Field API) est un objet accessible dans la propriété `tanstackField.api` lors de la création d'un champ. Elle fournit des méthodes pour travailler avec l'état du champ.

Exemple :

```angular-html
<input
  [value]="fieldName.api.state.value"
  (blur)="fieldName.api.handleBlur()"
  (input)="fieldName.api.handleChange($any($event).target.value)"
/>
```

## Validation

`@tanstack/angular-form` fournit à la fois une validation synchrone et asynchrone prête à l'emploi. Les fonctions de validation peuvent être passées à la directive `tanstackField` en utilisant la propriété `validators`.

Exemple :

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container [tanstackField]="form" name="firstName" #firstName="field">
      <input
        [value]="firstName.api.state.value"
        (blur)="firstName.api.handleBlur()"
        (input)="firstName.api.handleChange($any($event).target.value)"
      />
    </ng-container>
  `,
})
export class AppComponent {
  firstNameValidator: FieldValidateFn<any, any, string, any> = ({
                                                                       value,
                                                                     }) =>
    !value
      ? 'Un prénom est requis'
      : value.length < 3
        ? 'Le prénom doit comporter au moins 3 caractères'
        : undefined

  firstNameAsyncValidator: FieldValidateAsyncFn<any, string, any> =
    async ({ value }) => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return value.includes('error') && 'Le prénom ne peut pas contenir "error"'
    }

  form = injectForm({
    defaultValues: {
      firstName: '',
    },
    onSubmit({ value }) {
      console.log(value)
    },
  })
}
```

## Validation avec des bibliothèques de schéma standard (Standard Schema Libraries)

En plus des options de validation personnalisées, nous prenons également en charge la spécification [Standard Schema](https://github.com/standard-schema/standard-schema).

Vous pouvez définir un schéma en utilisant n'importe quelle bibliothèque implémentant la spécification et le passer à un validateur de formulaire ou de champ.

Les bibliothèques prises en charge incluent :

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

Exemple :

```angular-ts
import { z } from 'zod'

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container
      [tanstackField]="form"
      name="firstName"
      [validators]="{
        onChange: z.string().min(3, 'Le prénom doit comporter au moins 3 caractères'),
        onChangeAsyncDebounceMs: 500,
        onChangeAsync: firstNameAsyncValidator
      }"
      #firstName="field"
    >
      <!-- ... -->
    </ng-container>
  `,
})
export class AppComponent {
  firstNameAsyncValidator = z.string().refine(
    async (value) => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return !value.includes('error')
    },
    {
      message: "Le prénom ne peut pas contenir 'error'",
    },
  )

  form = injectForm({
    defaultValues: {
      firstName: '',
    },
    onSubmit({ value }) {
      // Faire quelque chose avec les données du formulaire
      console.log(value)
    },
  })

  z = z
}
```

## Réactivité (Reactivity)

`@tanstack/angular-form` offre un moyen de s'abonner aux changements d'état du formulaire et des champs via `injectStore(this.form, selector)`.

Exemple :

```typescript
import { injectForm, injectStore } from '@tanstack/angular-form'

@Component(/*...*/)
class AppComponent {
  form = injectForm(/*...*/)
  canSubmit = injectStore(this.form, (state) => state.canSubmit)
  isSubmitting = injectStore(this.form, (state) => state.isSubmitting)
}
```

## Écouteurs (Listeners)

`@tanstack/angular-form` vous permet de réagir à des déclencheurs spécifiques et de les "écouter" pour déclencher des effets secondaires.

Exemple :

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container
      [tanstackField]="form"
      name="country"
      [listeners]="{
        onChange: onCountryChange
      }"
      #country="field"
    ></ng-container>
  `,
})

...

onCountryChange: FieldListenerFn<any, any, any, any, string> = ({
    value,
  }) => {
    console.log(`Pays changé en : ${value}, réinitialisation de la province`)
    this.form.setFieldValue('province', '')
  }
```

Plus d'informations sont disponibles sur [Écouteurs](./listeners.md)

## Champs de tableau (Array Fields)

Les champs de tableau (Array Fields) vous permettent de gérer une liste de valeurs dans un formulaire, comme une liste de hobbies. Vous pouvez créer un champ de tableau en utilisant la directive `tanstackField`.

Lorsque vous travaillez avec des champs de tableau, vous pouvez utiliser les méthodes `pushValue`, `removeValue` et `swapValues` des champs pour ajouter, supprimer et échanger des valeurs dans le tableau.

Exemple :

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container [tanstackField]="form" name="hobbies" #hobbies="field">
      <div>
        Hobbies
        <div>
          @if (!hobbies.api.state.value.length) {
            Aucun hobby trouvé
          }
          @for (_ of hobbies.api.state.value; track $index) {
            <div>
              <ng-container
                [tanstackField]="form"
                [name]="getHobbyName($index)"
                #hobbyName="field"
              >
                <div>
                  <label [for]="hobbyName.api.name">Nom :</label>
                  <input
                    [id]="hobbyName.api.name"
                    [name]="hobbyName.api.name"
                    [value]="hobbyName.api.state.value"
                    (blur)="hobbyName.api.handleBlur()"
                    (input)="
                      hobbyName.api.handleChange($any($event).target.value)
                    "
                  />
                  <button
                    type="button"
                    (click)="hobbies.api.removeValue($index)"
                  >
                    X
                  </button>
                </div>
              </ng-container>
              <ng-container
                [tanstackField]="form"
                [name]="getHobbyDesc($index)"
                #hobbyDesc="field"
              >
                <div>
                  <label [for]="hobbyDesc.api.name">Description :</label>
                  <input
                    [id]="hobbyDesc.api.name"
                    [name]="hobbyDesc.api.name"
                    [value]="hobbyDesc.api.state.value"
                    (blur)="hobbyDesc.api.handleBlur()"
                    (input)="
                      hobbyDesc.api.handleChange($any($event).target.value)
                    "
                  />
                </div>
              </ng-container>
            </div>
          }
        </div>
        <button type="button" (click)="hobbies.api.pushValue(defaultHobby)">
          Ajouter un hobby
        </button>
      </div>
    </ng-container>
  `,
})
export class AppComponent {
  defaultHobby = {
    name: '',
    description: '',
    yearsOfExperience: 0,
  }

  getHobbyName = (idx: number) => `hobbies[${idx}].name` as const;
  getHobbyDesc = (idx: number) => `hobbies[${idx}].description` as const;

  form = injectForm({
    defaultValues: {
      hobbies: [] as Array<{
        name: string
        description: string
        yearsOfExperience: number
      }>,
    },
    onSubmit({ value }) {
      alert(JSON.stringify(value))
    },
  })
}
```

Voici les concepts de base et la terminologie utilisés dans la bibliothèque `@tanstack/angular-form`. Comprendre ces concepts vous aidera à travailler plus efficacement avec la bibliothèque et à créer des formulaires complexes avec facilité.
