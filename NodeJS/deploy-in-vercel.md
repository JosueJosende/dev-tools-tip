# Desplegar proyecto NodeJs, Express en Vercel

### Preparar el proyecto

Antes de nada si no tienes el archivo package.json, desde la términal corremos el siguiente comando:
```sh
$ npm init -y
```

El archivo principal, (esto es muy importante) se ha de llamar ***index.js*** y no de otra forma, como app.js o server.js.  
En el archivo ***package.json*** también ha de hacer referencaia a ***index.js***.

### Configuración

Creamos un archivo con nombre ***vercel.json***, dentro de este archivo introducimos las configuraciónes básicas que necesia vercel.
```json
{
    "builds": [
        {
            "src": "./index.js",
            "use": "@vercel/node"
        }
    ],
    "routes": [
        {
            "src": "/(.*)",
            "dest": "/"
        }
    ]
}
```
Builds: Definen como se construye la aplicación, en este punto usamos nuestro archivo index.js empleado dentro del entorno de ejecución @vercel/node.

Routes: Define como se manejan las rutas de la aplicación. Todas las URl representadas como /(.*) Se van a redirigir a la raiz del sitio (/).

Nuestra estructura de proyecto deberá tener estos archivos y carpetas para que el desplegue a vercel sea satisfactorio.

```
📁 node_modules  
📄 index.js  
📄 package.json  
📄 package-lock.json  
📄 vercel.json  
```
### Desplegando el proyecto

Hay dos manera de hacer el despliegue a vercel, un es vinculando el repositorio y la otra es utilizando la CLI de vercel y desplegandolo por medio de comandos de consola.

Por medio de la CLI de vercel, necesitamos tener instalado el paquete de vercel.
```
$ npm install -g vercel
```

Una vez instalado es tan fácil como desde la términal correr el comando 

```
$ vercel
```

Ahora  la CLI de vercel nos hara una serie de preguntas:

***Set up and deploy*** : [ y ]  

***Wich scope do you want to deploy to?*** : cuenta_vercel  
(debería aparencer tu cuenta con la que te logeaste, si no aparece deberás iniciar sesión en vercel)  

***Link to existing project?*** : [ n ]  
(pregunta si queremos vincular un proyecto de vercel existente que ya tengamos creado en vercel, introducimos un ***n*** a menos que sea lo que queramos)  

***What's is your project's name?*** : my-best-project-with-node  
(si en la anterior pregunta fué si, debermos pasar el nombre del proyecto, si no introducimos un nombre al que queramos darle a nuestro proyecto)

***In wich directory is your code located?*** : ./  
(nos sugiere por defecto ***./*** y es correcto, ya que nuestro archivo index.js ha de estar en la raiz del proyecto)  

### Finalizando

Ahora generará los links para desplegar el proyecto y veremos las URL's como se muestran en consola.

Para subirlo a producción deberas correr el siguiene comando desde la términal (en el paso anterior ya lo suguiere que lo hagas)

```
$ vercel --prod
```
