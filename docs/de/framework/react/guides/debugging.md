---
source-updated-at: '2025-03-28T15:34:36.000Z'
translation-updated-at: '2025-04-30T21:53:56.817Z'
id: debugging
title: Fehlerbehebung
---

Hier ist eine Liste häufiger Fehler, die in der Konsole auftreten können, sowie deren Lösungen.

## Änderung eines unkontrollierten Inputs zu einem kontrollierten Input

Wenn Sie diesen Fehler in der Konsole sehen:

```
Warning: A component is changing an uncontrolled input to be controlled. This is likely caused by the value changing from undefined to a defined value, which should not happen. Decide between using a controlled or uncontrolled input element for the lifetime of the component. More info: https://reactjs.org/link/controlled-components
```

Haben Sie wahrscheinlich die `defaultValues` in Ihrem `useForm`-Hook oder der `form.Field`-Komponente vergessen. Dieser Fehler tritt auf, weil der Input gerendert wird, bevor der Formularwert initialisiert ist, und sich daher von `undefined` zu `""` ändert, sobald eine Texteingabe erfolgt.

## Feldwert hat den Typ `unknown`

Wenn Sie `form.Field` verwenden und bei der Überprüfung des Werts von `field.state.value` feststellen, dass der Wert eines Felds den Typ `unknown` hat, ist es wahrscheinlich, dass Ihr Formulartyp zu umfangreich war, um ihn sicher auszuwerten.

Dies ist typischerweise ein Zeichen dafür, dass Sie Ihr Formular in kleinere Formulare aufteilen oder einen spezifischeren Typ für Ihr Formular verwenden sollten.

Eine mögliche Lösung für dieses Problem ist das Casten von `field.state.value` mit dem `as`-Schlüsselwort von TypeScript:

```tsx
const value = field.state.value as string
```

## `Type instantiation is excessively deep and possibly infinite`

Wenn Sie diesen Fehler in der Konsole sehen, während Sie `tsc` ausführen:

```
Type instantiation is excessively deep and possibly infinite
```

Sind Sie auf einen Bug in unseren Typdefinitionen gestoßen, den wir nicht erfasst haben. Obwohl wir unser Bestes getan haben, um die Typen so genau wie möglich zu gestalten, gibt es einige Randfälle, in denen TypeScript mit der Komplexität unserer Typen überfordert ist.

Bitte [melden Sie dieses Problem auf GitHub](https://github.com/TanStack/form/issues), damit wir es beheben können. Stellen Sie sicher, dass Sie eine minimale Reproduktion beifügen, damit wir Ihnen bei der Fehlersuche helfen können.

> Beachten Sie, dass dieser Fehler ein TypeScript-Fehler und kein Laufzeitfehler ist. Das bedeutet, dass Ihr Code auf dem Rechner des Nutzers wie erwartet ausgeführt wird.
