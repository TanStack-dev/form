---
source-updated-at: '2024-07-19T07:45:40.000Z'
translation-updated-at: '2025-04-30T21:34:40.199Z'
id: quick-start
title: Démarrage rapide
---

# Démarrage rapide

Le strict minimum pour commencer avec TanStack Form est de créer un formulaire et d'y ajouter un champ. Notez que cet exemple n'inclut aucune validation ni gestion d'erreurs... pour l'instant.

```angular-ts
import { Component } from '@angular/core'
import { bootstrapApplication } from '@angular/platform-browser'
import { TanStackField, injectForm } from '@tanstack/angular-form'

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <form (submit)="handleSubmit($event)">
      <div>
        <ng-container
          [tanstackField]="form"
          name="fullName"
          #fullName="field"
        >
          <label [for]="fullName.api.name">First Name:</label>
          <input
            [name]="fullName.api.name"
            [value]="fullName.api.state.value"
            (blur)="fullName.api.handleBlur()"
            (input)="fullName.api.handleChange($any($event).target.value)"
          />
        </ng-container>
      </div>
      <button type="submit">Submit</button>
    </form>
  `,
})
export class AppComponent {
  form = injectForm({
    defaultValues: {
      fullName: '',
    },
    onSubmit({ value }) {
      // Do something with form data
      console.log(value)
    },
  })

  handleSubmit(event: SubmitEvent) {
    event.preventDefault()
    event.stopPropagation()
    this.form.handleSubmit()
  }
}

bootstrapApplication(AppComponent).catch((err) => console.error(err))
```

À partir de là, vous serez prêt à explorer toutes les autres fonctionnalités de TanStack Form !
