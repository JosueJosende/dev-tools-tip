# Fetching de datos en Vue 3

La optimización de fetching de datos es crucial para un buena ejecucion y rendimiento de la página web. Vue ofrece diferentes maneras de llevar esto a cabo, pero, encontrar la mejor manera para cada tipo especifico de llamada, no es tan sencillo.

Veamos diferentes escenarios:  

### Diferentes tipos de llamadas API  

No todas las llamadas API son iguales. Podemos separarlas en dos catergorías:  
  1. LLamadas para recuperar datos de aplicación
  2. LLamadas para recuperar datos de la página web

Cada uno se gestiona de manera diferente.

### Datos de Aplicación

Estas son las llamadas que afectan a la aplicación entera y deben enviar la solicitud lo antes posible. Por ejemmplo, cuando se necesita recuperar la configuación de la aplicación, información de autenticación, etc.

La opción más comun y probablemente la más fácil de entender es hacer el fetching de datos desde el archivo ***main.js***

Un ejemplo sencillo que recupere los datos de configuración de nuestra aplicación y los almacene para posteriormente pueda usarse.

```js
  // /src/composables/useAppSettings.js

  import { ref } from 'vue'
  import { fetchAppSettings } from '@/composables/appSettings/api'

  const appSettings = ref()

  export function useAppSettings() {
    async function fetch() {
      const response = await fetchAppSettings()

      appSettings.value = response
    }

    return { appSettings, fetch }
  }
```

Ahora desde nuestro archivo principal de la aplicación ***main.js*** podemos hacer las llamadas API.

```js
  // main.js

  import { createApp } from 'vue'
  import { ./styles.css }
  import App from './App.vue'

  import { useAuth } from '@composables/auth/useAuth'
  import { useAppSettings } from '@composables/appSettings/useAppSettings'
  // import { useOtherComposable } from '@composables/otherComposable/useOtherComposable'
  
  await useAuth().authenticate()
  await useAppSettings().fetch()
  // await useOtherComposable()

  createApp(App).mount('#app')
```

De esta manera cada composable es iniciado y recupera los datos inmediatamente.

Nota que cada llamada usa la palabra reservada "await". Esto indica que cada llamada se hará secuencialmente.


**Authenticate**  ➡️ **Settings** ➡️ **OtherComposable**

Esto afectará a la carga inicial de la página o aplicación, esperará a que se cumplan estas llamadas antes de empezar a renderizar.

Es posible hacer que varias promesas puedan ejecutarse en paralelo (esto aumetará el rendimiento de la aplicación), para conseguirlo, usamos ***promise.all***.

```js
  // main.js

  import { createApp } from 'vue'
  import { ./styles.css }
  import App from './App.vue'

  import { useAuth } from '@composables/auth/useAuth'
  import { useAppSettings } from '@composables/appSettings/useAppSettings'
  // import { useOtherComposable } from '@composables/otherComposable/useOtherComposable'
  
  await useAuth().authenticate()
  
  Promise.all([
    await useAppSettings().fetch()
    // await useOtherComposable()
  ])

  createApp(App).mount('#app')
```

Primero, nos autenticamos y luego recuperamos la configuración y otros datos de la aplicación en paralelo. La idea es que se pueda anular una alternancia para un usuario específico.

Aunque la lógica de la aplicación sea diferente, siempre es posible realizar llamadas en paralelo, esto es esencial para mejorar el rendimiento de la aplicación.

No es una buena idea eliminar los *awaits* para mejorar el rendimiento. Nuestra aplicaición se renderizará con los valores por defecto y luego se renderizará la interfaz de usuario, lo que puede llevar a una mala experiencia y probablemente a errores.

### Datos de Página

Ahora veremos el fetching a nivel de página, usando la combinación de *await* y *Suspense*.

En un componente podemos hacer una llamada API dentro de 