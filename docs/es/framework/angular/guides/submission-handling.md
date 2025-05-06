---
source-updated-at: '2025-04-27T10:57:16.000Z'
translation-updated-at: '2025-05-06T20:20:46.816Z'
id: submission-handling
title: Submission handling
---

## Pasar datos adicionales al manejo de envíos

Puede tener múltiples tipos de comportamiento de envío, por ejemplo, volver a otra página o permanecer en el formulario.  
Puede lograr esto especificando la propiedad `onSubmitMeta`. Estos metadatos se pasarán a la función `onSubmit`.

> Nota: si `form.handleSubmit()` se llama sin metadatos, utilizará los valores predeterminados proporcionados.

```angular-ts
import { Component } from '@angular/core';
import { injectForm } from '@tanstack/angular-form';


type FormMeta = {
  submitAction: 'continue' | 'backToMenu' | null;
};

// Los metadatos no son obligatorios para llamar a form.handleSubmit().
// Especifique qué valores usar como predeterminados si no se pasan metadatos
const defaultMeta: FormMeta = {
  submitAction: null,
};

@Component({
  selector: 'app-root',
  template: `
    <form (submit)="handleSubmit($event)">
      <button type="submit" (click)="
        handleClick({ submitAction: 'continue' })
      ">Enviar y continuar</button>
      <button type="submit" (click)="
        handleClick({ submitAction: 'backToMenu' })
      ">Enviar y volver al menú</button>
    </form>
  `,
})
export class AppComponent {
  form = injectForm({
    defaultValues: {
      data: '',
    },
    // Defina qué valores de metadatos esperar en el envío
    onSubmitMeta: defaultMeta,
    onSubmit: async ({ value, meta }) => {
      // Hacer algo con los valores pasados mediante handleSubmit
      console.log(`Acción seleccionada - ${meta.submitAction}`, value);
    },
  });

  handleSubmit(event: SubmitEvent) {
    event.preventDefault();
    event.stopPropagation();
  }

  handleClick(meta: FormMeta) {
    // Sobrescribe el valor predeterminado especificado en onSubmitMeta
    this.form.handleSubmit(meta);
  }
}
```

## Transformar datos con Esquemas Estándar (Standard Schemas)

Si bien Tanstack Form proporciona [soporte para Esquemas Estándar (Standard Schemas)](./validation.md) para validación, no conserva los datos de salida del Esquema.

El valor pasado a la función `onSubmit` siempre serán los datos de entrada. Para recibir los datos de salida de un Esquema Estándar, analícelos en la función `onSubmit`:

```tsx
import { z } from 'zod'
// ...

const schema = z.object({
  age: z.string().transform((age) => Number(age)),
})

// Tanstack Form usa el tipo de entrada de los Esquemas Estándar
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
      // Pásalo por el esquema para obtener el valor transformado
      const result = schema.parse(value)
      const outputAge: number = result.age
    },
  })
}
```
