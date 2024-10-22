# Suspense en Vue 3

***Suspense*** es una potente característica introducida en Vue 3, permite gestionar componentes asincronos y mejorar la experiencia de ususario proporcionando estado de carga y contenido alternativo.

Proporciona una manera de suspender el renderizado hasta que los datos (en el caso de hacer un fetch()) o los componentes necesarios esten totalmente disponibles. Veamos su funcionamiento y como lo podemos utilizar para crear mejores aplicaciones.

### Entendiendo lo que hace "Suspense"

Es un concepto que se ha recogido de React y se ha implementado en Vue 3 par abordar el problema de la gestión de componentes asíncronos. Los componentes asíncronos son componentes que necesitan obtener datos o realizar alguna operación antes de que se representen en la vista.

Vue 3, simplifica el proceso de la gestión de componentes asíncronos proporcionando una manera declarativa de especificar el estado de carga y el contenido. Permite definir como se ha de comportar la aplicación mientras espera que se completen todas las operaciones asíncronas.

### Usando Suspense en Vue 3

Para aprender a usar Suspense, necesitamos enternder los siguientes conceptos y técnicas:

1. **Componentes asíncronos**: Lo primero es que Suspense gestiona los componentes asíncronos. Un componente asíncrono es un componente que se carga de manera asícrona, como cuando se se obtienen datos de una API. Los componentes asícronos se definen mediante la función defineAsyncComponent.
2. **SuspenseFallBack**: Permite definir un componente o contenido que se mostrará mientras se cargan los datos o componente asíncrono.
3. **Gestión de errores**: Suspense también ofrece una manera de gestionar los errores que se puedan producir mientras se efectua la carga. Es posible utilizar un hook y mostrar los mensajes de error adecuados o contenido alternativo.


### Ejemplo de Suspense en Vue 3

Veamos un ejemplo que muestra como usar Suspense en Vue 3.

```jsx
<script setup>
import { defineAsyncComponent } from 'vue';
import { OtroComponenteNoAsincrono } from './components'

const AsyncComponent = defineAsyncComponent(() =>
  import('./AsyncComponent.vue')
);
</script>

<template>
  <div>
    <h1>Este es un componente asíncrono</h1>
    <Suspense>
      <template #default>
        <ComponenteAsincrono />
      </template>
      <template #fallback>
        <OtroComponenteNoAsincrono />
      </template>
    </Suspense>
  </div>
</template>
```

Dentro de Suspense creamos dos templates, el template que esperará hasta cargar el componente asíncrono y el componente que se mostrará mientras se efectua la carga.

De esta manera se gestion el estado de carga de forma automática.