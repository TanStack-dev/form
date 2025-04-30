---
source-updated-at: '2025-04-16T08:45:06.000Z'
translation-updated-at: '2025-04-30T21:53:09.348Z'
id: philosophy
title: Philosophie
---

Jedes etablierte Projekt sollte eine Philosophie haben, die seine Entwicklung leitet. Ohne eine Kernphilosophie kann die Entwicklung in endlosen Entscheidungsprozessen stagnieren und als Folge schwächere APIs hervorbringen.

Dieses Dokument skizziert die Grundprinzipien, die die Entwicklung und den Funktionsumfang von TanStack Form antreiben.

## Einheitliche APIs verbessern

APIs bringen Kompromisse mit sich. Dadurch kann es verlockend sein, jeden Satz von Kompromissen dem Nutzer über verschiedene APIs zugänglich zu machen. Dies kann jedoch zu einer fragmentierten API führen, die schwerer zu erlernen und zu nutzen ist.

Auch wenn dies eine steilere Lernkurve bedeuten mag, heißt das, dass Sie nicht darüber nachdenken müssen, welche API intern zu verwenden ist, oder einen höheren kognitiven Aufwand haben, wenn Sie zwischen APIs wechseln.

## Formulare brauchen Flexibilität

TanStack Form ist darauf ausgelegt, flexibel und anpassbar zu sein. Während viele Formulare ähnlichen Mustern folgen mögen, gibt es immer Ausnahmen – insbesondere, wenn Formulare eine Kernkomponente Ihrer Anwendung sind.

Daher unterstützt TanStack Form mehrere Methoden für die Validierung:

- **Zeitliche Anpassungen**: Sie können bei Verlassen des Felds (blur), bei Änderung (change), beim Absenden (submit) oder sogar beim Laden (mount) validieren.
- **Validierungsstrategien**: Sie können einzelne Felder, das gesamte Formular oder eine Teilmenge von Feldern validieren.
- **Benutzerdefinierte Validierungslogik**: Sie können Ihre eigene Validierungslogik schreiben oder eine Bibliothek wie [Zod](https://zod.dev/) oder [Valibot](https://valibot.dev/) verwenden.
- **Benutzerdefinierte Fehlermeldungen**: Sie können die Fehlermeldungen für jedes Feld anpassen, indem Sie ein beliebiges Objekt aus einem Validator zurückgeben.
- **Asynchrone Validierung**: Sie können Felder asynchron validieren und haben gängige Hilfsmittel wie Debouncing und Abbruch für sich erledigt.

## Controlled ist Cool

In einer Welt, in der kontrollierte (controlled) vs. unkontrollierte (uncontrolled) Eingabefelder ein heiß diskutiertes Thema sind, steht TanStack Form fest auf der Seite der kontrollierten Eingaben.

Dies bringt eine Reihe von Vorteilen mit sich:

- **Vorhersehbar**: Sie können den Zustand Ihres Formulars zu jedem Zeitpunkt vorhersagen.
- **Einfachere Tests**: Sie können Ihre Formulare leicht testen, indem Sie Werte übergeben und die Ausgabe überprüfen.
- **Nicht-DOM-Unterstützung**: Sie können TanStack Form mit React Native, Three.js-Framework-Adaptern oder jedem anderen Framework-Renderer verwenden.
- **Erweiterte bedingte Logik**: Sie können Felder basierend auf dem Formularzustand leicht ein- oder ausblenden.
- **Debugging**: Sie können den Formularzustand leicht in der Konsole protokollieren, um Probleme zu debuggen.

## Generics sind grimmig

Sie sollten niemals ein Generic übergeben oder einen internen Typ verwenden müssen, wenn Sie TanStack Form nutzen. Das liegt daran, dass wir die Bibliothek so gestaltet haben, dass alles von Runtime-Standardwerten abgeleitet wird.

Wenn Sie ausreichend korrekten TanStack Form-Code schreiben, sollten Sie keinen Unterschied zwischen JavaScript- und TypeScript-Nutzung feststellen können – mit Ausnahme etwaiger Typumwandlungen, die Sie möglicherweise mit Runtime-Werten vornehmen.

Anstatt:

```typescript
useForm<MyForm>()
```

Sollten Sie Folgendes tun:

```typescript
interface Person {
  name: string
  age: number
}

const defaultPerson: Person = { name: 'Bill Luo', age: 24 }

useForm({
  defaultValues: defaultPerson,
})
```

## Bibliotheken sind befreiend

Eines der Hauptziele von TanStack Form ist, dass Sie es in Ihr eigenes Komponentensystem oder Designsystem einbinden sollten.

Um dies zu unterstützen, bieten wir eine Reihe von Hilfsmitteln, die das Erstellen eigener Komponenten und angepasster Hooks erleichtern:

```typescript
// Exportiert aus Ihrer eigenen Bibliothek mit vorgebundenen Komponenten für Ihre Formulare.
export const { useAppForm, withForm } = createFormHook(/* options */)
```

Wenn Sie dies nicht tun, fügen Sie Ihren Anwendungen erheblich mehr Boilerplate-Code hinzu und machen Ihre Formulare weniger konsistent und benutzerfreundlich.
