---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T21:56:04.906Z'
id: basic-concepts
title: Grundkonzepte
---

# Grundlegende Konzepte und Terminologie

Diese Seite führt in die grundlegenden Konzepte und Terminologien ein, die in der `@tanstack/lit-form`-Bibliothek verwendet werden. Wenn Sie sich mit diesen Konzepten vertraut machen, können Sie die Bibliothek und ihre Verwendung mit Lit besser verstehen und nutzen.

## Formularoptionen

Sie können Optionen für Ihr Formular erstellen, um sie zwischen mehreren Formularen gemeinsam zu nutzen, indem Sie die Funktion `formOptions` verwenden.

Beispiel:

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

## Formularinstanz

Eine Formularinstanz (Form Instance) ist ein Objekt, das ein einzelnes Formular repräsentiert und Methoden sowie Eigenschaften für die Arbeit mit dem Formular bereitstellt. Sie erstellen eine Formularinstanz mithilfe der `TanStackFormController`-Schnittstelle, die von `@tanstack/lit-form` bereitgestellt wird. Der `TanStackFormController` wird mit der aktuellen Formularklasse (`this`) und einigen Standardformularoptionen instanziiert. Er initialisiert den Formularzustand, behandelt die Formularübermittlung und stellt Methoden zur Verwaltung von Formularfeldern und deren Validierung bereit.

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

Sie können auch eine Formularinstanz erstellen, ohne `formOptions` zu verwenden, indem Sie die eigenständige `TanStackFormController`-API nutzen:

```tsx
#form = new TanStackFormController(this, {
  ...formOpts,
})
```

## Feld

Ein Feld (Field) repräsentiert ein einzelnes Formulareingabeelement, wie z. B. ein Textfeld oder eine Checkbox. Felder werden mithilfe der `field(FieldOptions, callback)`-Methode erstellt, die von der Formularinstanz bereitgestellt wird. Die Komponente akzeptiert ein `FieldOptions`-Objekt und eine Callback-Funktion, die ein `FieldApi`-Objekt empfängt. Dieses Objekt stellt Methoden bereit, um den aktuellen Wert des Felds abzurufen, Eingabeänderungen zu behandeln und Blur-Ereignisse zu verwalten.

Beispiel:

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

## Feldzustand

Jedes Feld hat seinen eigenen Zustand (Field State), der seinen aktuellen Wert, den Validierungsstatus, Fehlermeldungen und weitere Metadaten enthält. Sie können den Zustand eines Felds über seine `field.state`-Eigenschaft abrufen.

```tsx
const {
  value,
  meta: { errors, isValidating },
} = field.state
```

Es gibt drei Feldzustände, die sehr nützlich sein können, um zu sehen, wie ein Benutzer mit einem Feld interagiert. Ein Feld ist _"touched"_ (berührt), wenn der Benutzer hineinklickt oder es per Tab auswählt, _"pristine"_ (unverändert), bis der Benutzer den Wert ändert, und _"dirty"_ (verändert), nachdem der Wert geändert wurde. Sie können diese Zustände über die Flags `isTouched`, `isPristine` und `isDirty` überprüfen, wie unten gezeigt.

```tsx
const { isTouched, isPristine, isDirty } = field.state.meta
```

![Feldzustände](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)
