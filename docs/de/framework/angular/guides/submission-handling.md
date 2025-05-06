---
source-updated-at: '2025-04-27T10:57:16.000Z'
translation-updated-at: '2025-05-06T20:22:06.784Z'
id: submission-handling
title: Submission handling
---

## Zusätzliche Daten an die Übermittlungsverarbeitung übergeben

Es kann verschiedene Arten von Übermittlungsverhalten geben, zum Beispiel das Zurückkehren zu einer anderen Seite oder das Verbleiben auf dem Formular.  
Dies kann durch die Angabe der Eigenschaft `onSubmitMeta` erreicht werden. Diese Metadaten werden an die `onSubmit`-Funktion übergeben.

> Hinweis: Wenn `form.handleSubmit()` ohne Metadaten aufgerufen wird, werden die angegebenen Standardwerte verwendet.

```angular-ts
import { Component } from '@angular/core';
import { injectForm } from '@tanstack/angular-form';


type FormMeta = {
  submitAction: 'continue' | 'backToMenu' | null;
};

// Metadaten sind nicht erforderlich, um form.handleSubmit() aufzurufen.
// Legen Sie fest, welche Werte als Standard verwendet werden sollen, wenn keine Metadaten übergeben werden
const defaultMeta: FormMeta = {
  submitAction: null,
};

@Component({
  selector: 'app-root',
  template: `
    <form (submit)="handleSubmit($event)">
      <button type="submit" (click)="
        handleClick({ submitAction: 'continue' })
      ">Absenden und fortfahren</button>
      <button type="submit" (click)="
        handleClick({ submitAction: 'backToMenu' })
      ">Absenden und zurück zum Menü</button>
    </form>
  `,
})
export class AppComponent {
  form = injectForm({
    defaultValues: {
      data: '',
    },
    // Definieren Sie, welche Metadaten bei der Übermittlung erwartet werden
    onSubmitMeta: defaultMeta,
    onSubmit: async ({ value, meta }) => {
      // Verarbeiten Sie die über handleSubmit übergebenen Werte
      console.log(`Ausgewählte Aktion - ${meta.submitAction}`, value);
    },
  });

  handleSubmit(event: SubmitEvent) {
    event.preventDefault();
    event.stopPropagation();
  }

  handleClick(meta: FormMeta) {
    // Überschreibt den in onSubmitMeta angegebenen Standardwert
    this.form.handleSubmit(meta);
  }
}
```

## Daten mit Standard-Schemas transformieren

Während Tanstack Form [Standard-Schema-Unterstützung](./validation.md) für die Validierung bietet, werden die Ausgabedaten des Schemas nicht beibehalten.

Der an die `onSubmit`-Funktion übergebene Wert entspricht immer den Eingabedaten. Um die Ausgabedaten eines Standard-Schemas zu erhalten, müssen diese in der `onSubmit`-Funktion geparst werden:

```tsx
import { z } from 'zod'
// ...

const schema = z.object({
  age: z.string().transform((age) => Number(age)),
})

// Tanstack Form verwendet den Eingabetyp von Standard-Schemas
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
      // Daten durch das Schema parsen, um den transformierten Wert zu erhalten
      const result = schema.parse(value)
      const outputAge: number = result.age
    },
  })
}
```
