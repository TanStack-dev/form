---
source-updated-at: '2025-04-29T10:21:58.000Z'
translation-updated-at: '2025-04-30T21:51:07.899Z'
id: form-validation
title: Validación de formularios
---

En el núcleo de las funcionalidades de TanStack Form se encuentra el concepto de validación. TanStack Form hace que la validación sea altamente personalizable:

- Puede controlar cuándo realizar la validación (al cambiar, al ingresar, al perder el foco, al enviar...)
- Las reglas de validación pueden definirse a nivel de campo o a nivel de formulario
- La validación puede ser síncrona o asíncrona (por ejemplo, como resultado de una llamada API)

## ¿Cuándo se realiza la validación?

¡Depende de usted! La directiva `[tanstackField]` acepta algunas funciones de retorno (callbacks) como propiedades, como `onChange` u `onBlur`. Estas funciones reciben el valor actual del campo, así como el objeto `fieldAPI`, para que pueda realizar la validación. Si encuentra un error de validación, simplemente devuelva el mensaje de error como cadena y estará disponible en `field.api.state.meta.errors`.

Aquí hay un ejemplo:

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container
      [tanstackField]="form"
      name="age"
      [validators]="{
        onChange: ageValidator
      }"
      #age="field"
    >
      <label [for]="age.api.name">Edad:</label>
      <input
        [id]="age.api.name"
        [name]="age.api.name"
        [value]="age.api.state.value"
        type="number"
        (input)="age.api.handleChange($any($event).target.valueAsNumber)"
      />
      @if (age.api.state.meta.errors) {
        <em role="alert">{{ age.api.state.meta.errors.join(', ') }}</em>
      }
    </ng-container>
  `,
})
export class AppComponent {
  ageValidator: FieldValidateFn<any, any, any, any, number> = ({ value }) =>
    value < 13 ? 'Debes tener al menos 13 años para crear una cuenta' : undefined

  // ...
}
```

En el ejemplo anterior, la validación se realiza en cada pulsación de tecla (`onChange`). Si, en cambio, quisiéramos que la validación se realice cuando el campo pierde el foco, cambiaríamos el código anterior así:

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container
      [tanstackField]="form"
      name="age"
      [validators]="{
        onBlur: ageValidator
      }"
      #age="field"
    >
      <label [for]="age.api.name">Edad:</label>
      <!-- Siempre necesitamos implementar onChange para que TanStack Form reciba los cambios -->
      <!-- Escuchar el evento onBlur en el campo -->
      <input
        [id]="age.api.name"
        [name]="age.api.name"
        [value]="age.api.state.value"
        type="number"
        (blur)='age.api.handleBlur()'
        (input)="age.api.handleChange($any($event).target.valueAsNumber)"
      />
      @if (age.api.state.meta.errors) {
        <em role="alert">{{ age.api.state.meta.errors.join(', ') }}</em>
      }
    </ng-container>
  `,
})
export class AppComponent {
  ageValidator: FieldValidateFn<any, any, any, any, number> = ({ value }) =>
    value < 13 ? 'Debes tener al menos 13 años para crear una cuenta' : undefined

  // ...
}
```

Así que puede controlar cuándo se realiza la validación implementando la función de retorno deseada. Incluso puede realizar diferentes validaciones en diferentes momentos:

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container
      [tanstackField]="form"
      name="age"
      [validators]="{
        onChange: ageValidator,
        onBlur: minimumAgeValidator
      }"
      #age="field"
    >
      <label [for]="age.api.name">Edad:</label>
      <!-- Siempre necesitamos implementar onChange para que TanStack Form reciba los cambios -->
      <!-- Escuchar el evento onBlur en el campo -->
      <input
        [id]="age.api.name"
        [name]="age.api.name"
        [value]="age.api.state.value"
        type="number"
        (blur)="age.api.handleBlur()"
        (input)="age.api.handleChange($any($event).target.valueAsNumber)"
      />
      @if (!age.api.state.meta.isValid) {
        <em role="alert">{{ age.api.state.meta.errors.join(', ') }}</em>
      }
    </ng-container>
  `,
})
export class AppComponent {
  ageValidator: FieldValidateFn<any, any, any, any, number> = ({ value }) =>
    value < 13 ? 'Debes tener al menos 13 años para crear una cuenta' : undefined

  minimumAgeValidator: FieldValidateFn<any, any, any, any, number> = ({
    value,
  }) => (value < 0 ? 'Valor inválido' : undefined)

  // ...
}
```

En el ejemplo anterior, estamos validando diferentes aspectos del mismo campo en diferentes momentos (en cada pulsación de tecla y al perder el foco del campo). Dado que `field.state.meta.errors` es un arreglo, se muestran todos los errores relevantes en un momento dado. También puede usar `field.state.meta.errorMap` para obtener errores basados en _cuándo_ se realizó la validación (onChange, onBlur, etc.). Más información sobre cómo mostrar errores a continuación.

## Mostrar errores

Una vez que tenga configurada su validación, puede mapear los errores desde un arreglo para mostrarlos en su interfaz de usuario:

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container
      [tanstackField]="form"
      name="age"
      [validators]="{
        onChange: ageValidator
      }"
      #age="field"
    >
      <!-- ... -->
      @if (age.api.state.meta.errors) {
        <em role="alert">{{ age.api.state.meta.errors.join(', ') }}</em>
      }
    </ng-container>
  `,
})
export class AppComponent {
  ageValidator: FieldValidateFn<any, any, any, any, number> = ({ value }) =>
    value < 13 ? 'Debes tener al menos 13 años para crear una cuenta' : undefined

  // ...
}
```

O usar la propiedad `errorMap` para acceder al error específico que está buscando:

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container
      [tanstackField]="form"
      name="age"
      [validators]="{
        onChange: ageValidator
      }"
      #age="field"
    >
      <!-- ... -->
      @if (age.api.state.meta.errorMap['onChange']) {
        <em role="alert">{{ age.api.state.meta.errorMap['onChange'] }}</em>
      }
    </ng-container>
  `,
})
export class AppComponent {
  ageValidator: FieldValidateFn<any, any, any, any, number> = ({ value }) =>
    value < 13 ? 'Debes tener al menos 13 años para crear una cuenta' : undefined

  // ...
}
```

Vale la pena mencionar que nuestro arreglo `errors` y el `errorMap` coinciden con los tipos devueltos por los validadores. Esto significa que:

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container
      [tanstackField]="form"
      name="age"
      [validators]="{
        onChange: ageValidator
      }"
      #age="field"
    >
      <!-- ... -->
      <!-- errorMap.onChange es de tipo `{isOldEnough: false} | undefined` -->
	  <!-- meta.errors es de tipo `Array<{isOldEnough: false} | undefined>` -->
      @if (!age.api.state.meta.errorMap['onChange']?.isOldEnough) {
        <em role="alert">El usuario no tiene la edad suficiente</em>
      }
    </ng-container>
  `,
})
export class AppComponent {
  ageValidator: FieldValidateFn<any, any, any, any, number> = ({ value }) =>
    value < 13 ? 'Debes tener al menos 13 años para crear una cuenta' : undefined

  // ...
}
```

## Validación a nivel de campo vs a nivel de formulario

Como se mostró anteriormente, cada `[tanstackField]` acepta sus propias reglas de validación a través de las funciones de retorno `onChange`, `onBlur`, etc. También es posible definir reglas de validación a nivel de formulario (en lugar de campo por campo) pasando funciones de retorno similares a la función `injectForm()`.

Ejemplo:

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <div>
      <ng-container [tanstackField]="form" name="age" #age="field">
        <!-- ... -->
        @if (formErrorMap().onChange) {
          <div>
            <em
              >Hubo un error en el formulario: {{ formErrorMap().onChange }}</em
            >
          </div>
        }
        <!-- ... -->
      </ng-container>
    </div>
  `,
})
export class AppComponent {
  form = injectForm({
    defaultValues: {
      age: 0,
    },
    onSubmit({ value }) {
      console.log(value)
    },
    validators: {
      // Agregue validadores al formulario de la misma manera que los agregaría a un campo
      onChange({ value }) {
        if (value.age < 13) {
          return 'Debes tener al menos 13 años para registrarte'
        }
        return undefined
      },
    },
  })

  // Suscríbase al mapa de errores del formulario para que las actualizaciones se rendericen
  formErrorMap = injectStore(this.form, (state) => state.errorMap)
}
```

### Establecer errores a nivel de campo desde los validadores del formulario

Puede establecer errores en los campos desde los validadores del formulario. Un caso de uso común para esto es validar todos los campos al enviar llamando a un único endpoint API en el validador `onSubmitAsync` del formulario.

```angular-ts
@Component({
  selector: 'app-root',
  imports: [TanStackField],
  template: `
    <form (submit)="handleSubmit($event)">
      <div>
        <ng-container
          [tanstackField]="form"
          name="age"
          #ageField="field"
        >
          <label [for]="ageField.api.name">Edad:</label>
          <input
            type="number"
            [name]="ageField.api.name"
            [value]="ageField.api.state.value"
            (blur)="ageField.api.handleBlur()"
            (input)="ageField.api.handleChange($any($event).target.valueAsNumber)"
          />
          @if (ageField.api.state.meta.errors.length > 0) {
            <em role="alert">{{ ageField.api.state.meta.errors.join(', ') }}</em>
          }
        </ng-container>
      </div>
      <button type="submit">Enviar</button>
    </form>
  `,
})

export class AppComponent {
  form = injectForm({
    defaultValues: {
      age: 0,
      socials: [],
      details: {
        email: '',
      },
    },
    validators: {
      onSubmitAsync: async ({ value }) => {
        // Validar el valor en el servidor
        const hasErrors = await verifyDataOnServer(value)
        if (hasErrors) {
          return {
            form: 'Datos inválidos', // La clave `form` es opcional
            fields: {
              age: 'Debes tener al menos 13 años para registrarte',
              // Establecer errores en campos anidados con el nombre del campo
              'socials[0].url': 'La URL proporcionada no existe',
              'details.email': 'Se requiere un correo electrónico',
            },
          };
        }

        return null;
      },
    },
  });

  handleSubmit(event: SubmitEvent) {
    event.preventDefault();
    event.stopPropagation();
    this.form.handleSubmit();
  }
}
```

> Algo que vale la pena mencionar es que si tiene una función de validación de formulario que devuelve un error, ese error puede ser sobrescrito por la validación específica del campo.
>
> Esto significa que:
>
> ```angular-ts
> @Component({
>   selector: 'app-root',
>   standalone: true,
>   imports: [TanStackField],
>   template: `
>       <div>
>         <ng-container
>          [tanstackField]="form"
>         name="age"
>        #ageField="field"
>       [validators]="{
>        onChange: fieldValidator
>     }"
>  >
>   <input type="number" [value]="ageField.api.state.value"
>   (input)="ageField.api.handleChange($any($event).target.valueAsNumber)"
>   />
>    @if (ageField.api.state.meta.errors.length > 0) {
>       <em role="alert">{{ ageField.api.state.meta.errors.join(', ') }}</em>
>     }
>   </ng-container>
> </div>
> `,
> })
> export class AppComponent {
>   form = injectForm({
>     defaultValues: {
>       age: 0,
>     },
>     validators: {
>       onChange: ({ value }) => {
>         return {
>           fields: {
>             age: value.age < 12 ? '¡Demasiado joven!' : undefined,
>           },
>         };
>       },
>     },
>   });
>
>   fieldValidator: FieldValidateFn<any, any, number> = ({ value }) =>
>     value % 2 === 0 ? '¡Debe ser impar!' : undefined;
> }
>
> ```
>
> Solo mostrará `'¡Debe ser impar!'` incluso si el error '¡Demasiado joven!' es devuelto por la validación a nivel de formulario.

## Validación Funcional Asíncrona

Si bien sospechamos que la mayoría de las validaciones serán síncronas, hay muchas instancias donde una llamada de red u otra operación asíncrona sería útil para validar.

Para hacer esto, tenemos métodos dedicados `onChangeAsync`, `onBlurAsync` y otros que pueden usarse para validar:

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container
      [tanstackField]="form"
      name="age"
      [validators]="{ onChangeAsync: ageValidator }"
      #age="field"
    >
      <label [for]="age.api.name">Apellido:</label>
      <input
        [id]="age.api.name"
        [name]="age.api.name"
        [value]="age.api.state.value"
        type="number"
        (input)="age.api.handleChange($any($event).target.valueAsNumber)"
      />
      @if (age.api.state.meta.errors) {
        <em role="alert">{{ age.api.state.meta.errors.join(', ') }}</em>
      }
    </ng-container>
  `,
})
export class AppComponent {
  ageValidator: FieldValidateAsyncFn<any, string, number> = async ({
    value,
  }) => {
    await new Promise((resolve) => setTimeout(resolve, 1000))
    return value < 13 ? 'Debes tener al menos 13 años para crear una cuenta' : undefined
  }

  // ...
}
```

Las validaciones síncronas y asíncronas pueden coexistir. Por ejemplo, es posible definir tanto `onBlur` como `onBlurAsync` en el mismo campo:

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container
      [tanstackField]="form"
      name="age"
      [validators]="{ onBlur: ensureAge13, onBlurAsync: ensureOlderAge }"
      #age="field"
    >
      <label [for]="age.api.name">Apellido:</label>
      <input
        [id]="age.api.name"
        [name]="age.api.name"
        [value]="age.api.state.value"
        type='number'
        (blur)="age.api.handleBlur()"
        (input)="age.api.handleChange($any($event).target.value)"
      />
      @if (age.api.state.meta.errors) {
        <em role="alert">{{ age.api.state.meta.errors.join(', ') }}</em>
      }
    </ng-container>
  `,
})
export class AppComponent {
  ensureAge13: FieldValidateFn<any, any, any, any, number> = ({ value }) =>
    value < 13 ? 'Debes tener al menos 13 años' : undefined

  ensureOlderAge: FieldValidateAsyncFn<any, string, number> = async ({
    value,
  }) => {
    const currentAge = await fetchCurrentAgeOnProfile()
    return value < currentAge ? 'Solo puedes aumentar la edad' : undefined
  }

  // ...
}
```

El método de validación síncrono (`onBlur`) se ejecuta primero y el método asíncrono (`onBlurAsync`) solo se ejecuta si el síncrono (`onBlur`) tiene éxito. Para cambiar este comportamiento, establezca la opción `asyncAlways` en `true`, y el método as
