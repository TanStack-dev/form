---
source-updated-at: '2025-03-28T15:34:36.000Z'
translation-updated-at: '2025-04-30T21:48:10.586Z'
id: debugging
title: Depuración
---

Aquí hay una lista de errores comunes que podrías ver en la consola y cómo solucionarlos.

## Cambiar un input no controlado a controlado

Si ves este error en la consola:

```
Warning: A component is changing an uncontrolled input to be controlled. This is likely caused by the value changing from undefined to a defined value, which should not happen. Decide between using a controlled or uncontrolled input element for the lifetime of the component. More info: https://reactjs.org/link/controlled-components
```

Es probable que hayas olvidado los `defaultValues` en tu Hook `useForm` o en el uso del componente `form.Field`. Esto ocurre porque el input se está renderizando antes de que se inicialice el valor del formulario y, por lo tanto, está cambiando de `undefined` a `""` cuando se ingresa texto en un input.

## El valor del campo es de tipo `unknown`

Si estás usando `form.Field` y, al inspeccionar el valor de `field.state.value`, ves que el valor de un campo es de tipo `unknown`, es probable que el tipo de tu formulario sea demasiado amplio para que podamos evaluarlo de manera segura.

Esto suele ser una señal de que deberías dividir tu formulario en formularios más pequeños o usar un tipo más específico para tu formulario.

Una solución temporal para este problema es hacer un casting de `field.state.value` usando la palabra clave `as` de TypeScript:

```tsx
const value = field.state.value as string
```

## `La instanciación del tipo es excesivamente profunda y posiblemente infinita`

Si ves este error en la consola al ejecutar `tsc`:

```
Type instantiation is excessively deep and possibly infinite
```

Has encontrado un error que no detectamos en nuestras definiciones de tipos. Aunque hemos hecho nuestro mejor esfuerzo para garantizar que nuestros tipos sean lo más precisos posible, hay algunos casos extremos en los que TypeScript tiene dificultades con la complejidad de nuestros tipos.

Por favor, [reporta este problema en GitHub](https://github.com/TanStack/form/issues) para que podamos solucionarlo. Asegúrate de incluir una reproducción mínima para que podamos ayudarte a depurarlo.

> Ten en cuenta que este error es un error de TypeScript y no un error en tiempo de ejecución. Esto significa que tu código seguirá ejecutándose en la máquina del usuario como se espera.
