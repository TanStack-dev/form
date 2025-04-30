---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T21:37:47.413Z'
id: basic-concepts
title: Concepts de base
---

# Concepts de base et terminologie

Cette page présente les concepts de base et la terminologie utilisés dans la bibliothèque `@tanstack/lit-form`. Se familiariser avec ces concepts vous aidera à mieux comprendre et travailler avec la bibliothèque et son utilisation avec Lit.

## Options de formulaire (Form Options)

Vous pouvez créer des options pour votre formulaire afin qu'elles puissent être partagées entre plusieurs formulaires en utilisant la fonction `formOptions`.

Par exemple :

```tsx
const formOpts = formOptions({
  defaultValues: {
    firstName: '',
    lastName: '',
    employed: false,
    jobTitle: '',
  } as Employee,
})
```

## Instance de formulaire (Form Instance)

Une Instance de formulaire (Form Instance) est un objet qui représente un formulaire individuel et fournit des méthodes et propriétés pour travailler avec le formulaire. Vous créez une instance de formulaire en utilisant l'interface `TanStackFormController` fournie par `@tanstack/lit-form`. Le `TanStackFormController` est instancié avec la classe du formulaire courant (`this`) et quelques options de formulaire par défaut. Il initialise l'état du formulaire, gère la soumission du formulaire et fournit des méthodes pour gérer les champs du formulaire et leur validation.

```tsx
#form = new TanStackFormController(this, {
  defaultValues: {
    firstName: '',
    lastName: '',
    employed: false,
    jobTitle: '',
  } as Employee,
})
```

Vous pouvez également créer une instance de formulaire sans utiliser `formOptions` en utilisant l'API autonome de `TanStackFormController` :

```tsx
#form = new TanStackFormController(this, {
  ...formOpts,
})
```

## Champ (Field)

Un Champ (Field) représente un seul élément de saisie de formulaire, comme une zone de texte ou une case à cocher. Les champs sont créés en utilisant la méthode `field(FieldOptions, callback)` fournie par l'instance de formulaire. Le composant accepte un objet `FieldOptions` et une fonction de rappel qui reçoit un objet `FieldApi`. Cet objet fournit des méthodes pour obtenir la valeur actuelle du champ, gérer les changements de saisie et gérer les événements de perte de focus (blur).

Par exemple :

```ts
 ${this.#form.field(
    {
      name: `firstName`,
      validators: {
        onChange: ({ value }) =>
          value.length < 3 ? "Not long enough" : undefined,
        },
      },
      (field: FieldApi<Employee, "firstName">) => {
        return html` <div>
          <label class="first-name-label">First Name</label>
          <input
           id="firstName"
           type="text"
           class="first-name-input"
           placeholder="First Name"
           @blur="${() => field.handleBlur()}"
           .value="${field.state.value}"
           @input="${(event: InputEvent) => {
           if (event.currentTarget) {
            const newValue = (event.currentTarget as HTMLInputElement).value;
            field.handleChange(newValue);
           }
          }}"
        />
      </div>`;
    },
)}
```

## État du champ (Field State)

Chaque champ possède son propre état (Field State), qui inclut sa valeur actuelle, son statut de validation, les messages d'erreur et d'autres métadonnées. Vous pouvez accéder à l'état d'un champ via sa propriété `field.state`.

```tsx
const {
  value,
  meta: { errors, isValidating },
} = field.state
```

Il existe trois états de champ qui peuvent être très utiles pour comprendre comment l'utilisateur interagit avec un champ. Un champ est _"touché" (touched)_ lorsque l'utilisateur clique ou se positionne dessus, _"vierge" (pristine)_ tant que l'utilisateur n'a pas modifié sa valeur, et _"modifié" (dirty)_ après que la valeur a été changée. Vous pouvez vérifier ces états via les indicateurs `isTouched`, `isPristine` et `isDirty`, comme illustré ci-dessous.

```tsx
const { isTouched, isPristine, isDirty } = field.state.meta
```

![États des champs (Field states)](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)
