---
source-updated-at: '2025-04-27T10:57:16.000Z'
translation-updated-at: '2025-05-06T20:25:21.816Z'
id: submission-handling
title: Submission handling
---

## Transmission de données supplémentaires à la gestion de soumission

Vous pouvez avoir plusieurs types de comportement lors de la soumission, par exemple, retourner à une autre page ou rester sur le formulaire.  
Vous pouvez y parvenir en spécifiant la propriété `onSubmitMeta`. Ces métadonnées seront transmises à la fonction `onSubmit`.

> Remarque : si `form.handleSubmit()` est appelé sans métadonnées, il utilisera les valeurs par défaut fournies.

```angular-ts
import { Component } from '@angular/core';
import { injectForm } from '@tanstack/angular-form';


type FormMeta = {
  submitAction: 'continue' | 'backToMenu' | null;
};

// Les métadonnées ne sont pas obligatoires pour appeler form.handleSubmit().
// Spécifiez les valeurs par défaut à utiliser si aucune métadonnée n'est transmise
const defaultMeta: FormMeta = {
  submitAction: null,
};

@Component({
  selector: 'app-root',
  template: `
    <form (submit)="handleSubmit($event)">
      <button type="submit" (click)="
        handleClick({ submitAction: 'continue' })
      ">Soumettre et continuer</button>
      <button type="submit" (click)="
        handleClick({ submitAction: 'backToMenu' })
      ">Soumettre et retourner au menu</button>
    </form>
  `,
})
export class AppComponent {
  form = injectForm({
    defaultValues: {
      data: '',
    },
    // Définissez les métadonnées attendues lors de la soumission
    onSubmitMeta: defaultMeta,
    onSubmit: async ({ value, meta }) => {
      // Faites quelque chose avec les valeurs transmises via handleSubmit
      console.log(`Action sélectionnée - ${meta.submitAction}`, value);
    },
  });

  handleSubmit(event: SubmitEvent) {
    event.preventDefault();
    event.stopPropagation();
  }

  handleClick(meta: FormMeta) {
    // Remplace les valeurs par défaut spécifiées dans onSubmitMeta
    this.form.handleSubmit(meta);
  }
}
```

## Transformation des données avec les schémas standard

Bien que Tanstack Form propose une [prise en charge des schémas standard](./validation.md) pour la validation, il ne conserve pas les données de sortie du schéma.

La valeur transmise à la fonction `onSubmit` sera toujours les données d'entrée. Pour recevoir les données de sortie d'un schéma standard, analysez-les dans la fonction `onSubmit` :

```tsx
import { z } from 'zod'
// ...

const schema = z.object({
  age: z.string().transform((age) => Number(age)),
})

// Tanstack Form utilise le type d'entrée des schémas standard
const defaultValues: z.input<typeof schema> = {
  age: '13',
}

// ...

@Component({
  // ...
})
export class AppComponent {
  form = injectForm({
    defaultValues,
    validators: {
      onChange: schema,
    },
    onSubmit: ({ value }) => {
      const inputAge: string = value.age
      // Passez les données dans le schéma pour obtenir la valeur transformée
      const result = schema.parse(value)
      const outputAge: number = result.age
    },
  })
}
```
