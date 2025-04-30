---
source-updated-at: '2025-03-20T03:01:05.000Z'
translation-updated-at: '2025-04-30T21:53:32.641Z'
id: reactivity
title: Reaktivität
---

# Reaktivität

TanStack Form verursacht keine erneuten Renderings (re-renders), wenn mit dem Formular interagiert wird. Daher kann es vorkommen, dass Sie versuchen, einen Formular- oder Feldstatuswert zu verwenden, ohne Erfolg zu haben.

Wenn Sie auf reaktive Werte zugreifen möchten, müssen Sie diese über eine von zwei Methoden abonnieren: `useStore` oder die `form.Subscribe`-Komponente.

Einige Anwendungsfälle für diese Abonnements sind das Rendern aktueller Feldwerte, die Entscheidung, was basierend auf einer Bedingung gerendert werden soll, oder die Verwendung von Feldwerten innerhalb der Logik Ihrer Komponente.

> Für Situationen, in denen Sie auf Trigger "reagieren" möchten, sehen Sie sich die [Listener](./listeners.md)-API an.

## useStore

Der `useStore`-Hook ist ideal, wenn Sie innerhalb der Logik Ihrer Komponente auf Formularwerte zugreifen müssen. `useStore` nimmt zwei Parameter entgegen. Erstens den Formular-Store (form store) und zweitens einen Selektor, um den gewünschten Teil des Formulars, den Sie abonnieren möchten, genau anzupassen.

```tsx
const firstName = useStore(form.store, (state) => state.values.firstName)
const errors = useStore(form.store, (state) => state.errorMap)
```

Sie können jeden Teil des Formularstatus im Selektor abrufen.

> Beachten Sie, dass `useStore` ein komplettes Komponenten-Rendering (re-render) verursacht, sobald sich der abonnierte Wert ändert.

Während es MÖGLICH ist, den Selektor wegzulassen, widerstehen Sie dem Drang, da dies zu vielen unnötigen erneuten Renderings führen würde, sobald sich irgendein Teil des Formularstatus ändert.

## form.Subscribe

Die `form.Subscribe`-Komponente eignet sich am besten, wenn Sie innerhalb der Benutzeroberfläche (UI) Ihrer Komponente auf etwas reagieren müssen. Zum Beispiel das Anzeigen oder Ausblenden von UI-Elementen basierend auf dem Wert eines Formularfelds.

```tsx
<form.Subscribe
  selector={(state) => state.values.firstName}
  children={(firstName) => (
    <form.Field>
      {(field) => (
        <input
          name="lastName"
          value={field.state.lastName}
          onChange={field.handleChange}
        />
      )}
    </form.Field>
  )}
/>
```

> Die `form.Subscribe`-Komponente löst keine Renderings auf Komponentenebene aus. Jedes Mal, wenn sich der abonnierte Wert ändert, wird nur die `form.Subscribe`-Komponente neu gerendert.

Die Entscheidung zwischen `useStore` und `form.Subscribe` hängt im Wesentlichen davon ab: Wenn es in der Benutzeroberfläche (UI) gerendert wird, greifen Sie zu `form.Subscribe` für seine Optimierungsvorteile. Wenn Sie die Reaktivität innerhalb der Logik benötigen, dann ist `useStore` die richtige Wahl.
