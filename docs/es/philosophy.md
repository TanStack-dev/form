---
source-updated-at: '2025-04-16T08:45:06.000Z'
translation-updated-at: '2025-04-30T21:47:27.949Z'
id: philosophy
title: Filosofía
---

# Filosofía

Todo proyecto bien establecido debe tener una filosofía que guíe su desarrollo. Sin una filosofía central, el desarrollo puede estancarse en una toma de decisiones interminable y, como resultado, ofrecer APIs más débiles.

Este documento describe los principios fundamentales que impulsan el desarrollo y las características de TanStack Form.

## Actualización de APIs unificadas

Las APIs implican compensaciones. Como resultado, puede ser tentador ofrecer cada conjunto de compensaciones al usuario a través de diferentes APIs. Sin embargo, esto puede llevar a una API fragmentada que es más difícil de aprender y usar.

Aunque esto pueda significar una curva de aprendizaje más pronunciada, significa que no tendrás que cuestionar qué API usar internamente ni tendrás una sobrecarga cognitiva mayor al cambiar entre APIs.

## Los formularios necesitan flexibilidad

TanStack Form está diseñado para ser flexible y personalizable. Si bien muchos formularios pueden seguir patrones similares, siempre hay excepciones; especialmente cuando los formularios son un componente central de tu aplicación.

Como resultado, TanStack Form admite múltiples métodos para la validación:

- **Personalización de tiempos**: Puedes validar al perder el foco (blur), al cambiar (change), al enviar (submit) o incluso al montar (mount).
- **Estrategias de validación**: Puedes validar campos individuales, todo el formulario o un subconjunto de campos.
- **Lógica de validación personalizada**: Puedes escribir tu propia lógica de validación o usar una biblioteca como [Zod](https://zod.dev/) o [Valibot](https://valibot.dev/).
- **Mensajes de error personalizados**: Puedes personalizar los mensajes de error para cada campo devolviendo cualquier objeto desde un validador.
- **Validación asíncrona**: Puedes validar campos de forma asíncrona y tener utilidades comunes como debouncing y cancelación manejadas por ti.

## Lo controlado es genial

En un mundo donde los inputs controlados vs no controlados son un tema candente, TanStack Form está firmemente en el campo de los controlados.

Esto conlleva varias ventajas:

- **Predecible**: Puedes predecir el estado de tu formulario en cualquier momento.
- **Pruebas más fáciles**: Puedes probar fácilmente tus formularios pasando valores y verificando la salida.
- **Soporte fuera del DOM**: Puedes usar TanStack Form con React Native, adaptadores de frameworks como Three.js o cualquier otro renderizador de framework.
- **Lógica condicional mejorada**: Puedes mostrar/ocultar campos condicionalmente según el estado del formulario.
- **Depuración**: Puedes registrar fácilmente el estado del formulario en la consola para depurar problemas.

## Los genéricos son sombríos

Nunca deberías necesitar pasar un genérico o usar un tipo interno al utilizar TanStack Form. Esto se debe a que hemos diseñado la biblioteca para inferir todo a partir de valores predeterminados en tiempo de ejecución.

Al escribir código suficientemente correcto en TanStack Form, no deberías poder distinguir entre el uso en JavaScript y en TypeScript, con la excepción de cualquier conversión de tipos que puedas hacer de valores en tiempo de ejecución.

En lugar de:

```typescript
useForm<MyForm>()
```

Deberías hacer:

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

## Las bibliotecas son liberadoras

Uno de los principales objetivos de TanStack Form es que puedas integrarlo en tu propio sistema de componentes o sistema de diseño.

Para apoyar esto, tenemos varias utilidades que facilitan la creación de tus propios componentes y hooks personalizados:

```typescript
// Exportado desde tu propia biblioteca con componentes pre-configurados para tus formularios.
export const { useAppForm, withForm } = createFormHook(/* opciones */)
```

Sin hacer esto, estás añadiendo mucho más código repetitivo a tus aplicaciones y haciendo que tus formularios sean menos consistentes y amigables para el usuario.
