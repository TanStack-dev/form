---
source-updated-at: '2024-07-19T07:45:40.000Z'
translation-updated-at: '2025-04-30T21:47:07.630Z'
id: quick-start
title: Inicio rápido
---

# Inicio Rápido

Lo mínimo indispensable para comenzar con TanStack Form es crear un formulario y agregar un campo. Tenga en cuenta que este ejemplo no incluye validación ni manejo de errores... por ahora.

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
          <label [for]="fullName.api.name">Nombre completo:</label>
          <input
            [name]="fullName.api.name"
            [value]="fullName.api.state.value"
            (blur)="fullName.api.handleBlur()"
            (input)="fullName.api.handleChange($any($event).target.value)"
          />
        </ng-container>
      </div>
      <button type="submit">Enviar</button>
    </form>
  `,
})
export class AppComponent {
  form = injectForm({
    defaultValues: {
      fullName: '',
    },
    onSubmit({ value }) {
      // Hacer algo con los datos del formulario
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

A partir de aquí, estará listo para explorar todas las demás características de TanStack Form.
