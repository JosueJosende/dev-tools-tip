# Construir servidor seguro con Node y Express

1. Controlar y limitar la cantidad de solicitudes entrantes (Rate limit).
2. Habilitar cors (Cross-Origin Resource Sharing).
3. Ajustar tiempo de conexión (Keep Alive).
4. Agregar tiempos de espera en solicitudes HTTP (Timeouts).
5. Limitar el tamaño del cuerpo HTTP (Payload size).
6. Validar y desinfectar los datos de entrada de nuestra aplicación.
7. Realizar consultas parametrizadas SQL.
8. Agregar políticas de seguridad de contenido (CSP).
9. Implementar algún método de autenticación como JWT.
10. Auditar y reparar dependencias con NPM audit.

### Controlar y limitar la cantidad de solicitudes entrantes (rate limit)

Partiendo por la primera buena práctica de seguridad, debemos controlar y limitar la cantidad de solicitudes. Para esto utilizaremos una técnica llamada “rate limit” (limite de tasa). El rate limit se aplica principalmente para evitar la sobrecarga de servidores, mejorar la seguridad y prevenir ataques de denegación de servicio (DoS) o ataques de fuerza bruta. A su vez, al imponer un límite en la tasa de solicitudes, se asegura que los usuarios o las aplicaciones no realicen un uso excesivo o abusivo de los recursos.

El rate limit se suele expresar en términos de solicitudes permitidas por unidad de tiempo, como por ejemplo “100 solicitudes por minuto”. Esto significa que un usuario o una aplicación puede realizar hasta 100 solicitudes dentro de un período de un minuto. Si se supera ese límite, las solicitudes adicionales pueden ser rechazadas, limitadas o retrasadas. Los rate limits pueden aplicarse a nivel de IP, usuario, clave de API u otro identificador único.

A continuación, un ejemplo básico con Nodejs y express, utilizando el paquete “express-rate-limit” :

```js
const express = require('express');
const rateLimit = require('express-rate-limit');
const app = express();

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // Limit each IP to 100 requests per `window` (here, per 15 minutes)
  standardHeaders: true, // Return rate limit info in the `RateLimit-*` headers
  legacyHeaders: false, // Disable the `X-RateLimit-*` headers
});

app.use(limiter);
```

### Habilitar cors (Cross-Origin Resource Sharing)

Cross-Origin Resource Sharing (CORS) es un mecanismo de seguridad utilizado por los navegadores web para permitir o restringir las solicitudes de recursos (como archivos JavaScript, imágenes o datos) entre diferentes dominios u orígenes. De forma predeterminada, los navegadores aplican la “Política del mismo origen” (Same-Origin Policy), que impide que un script en un sitio web acceda a recursos en otro dominio. Sin embargo, CORS proporciona un medio para que los servidores especifiquen de manera controlada qué dominios pueden acceder a sus recursos y cómo.

La implementación de CORS se realiza en el lado del servidor. El servidor debe estar configurado para enviar los encabezados CORS adecuados en las respuestas a las solicitudes de origen cruzado. Los navegadores, a su vez, aplican estas políticas de CORS para determinar si permiten o bloquean las solicitudes de origen cruzado.

A continuación, un ejemplo básico con Nodejs y express, utilizando el paquete “cors” :

```js
const express = require('express');
const cors = require('cors');
const app = express();

// Enable CORS with specific options
app.use(cors({
  origin: 'http://localhost:3000',
  methods: ['GET', 'POST'],
  allowedHeaders: ['Content-Type', 'Authorization']
}));
```

### Ajustar tiempo de conexión (Keep Alive)

Keep Alive se refiere a una técnica o configuración que permite mantener una conexión abierta entre el cliente y el servidor por un período de tiempo más largo que el típico, incluso cuando no se están transmitiendo datos. En lugar de abrir y cerrar conexiones repetidamente para cada transmisión de datos, Keep-Alive permite reutilizar la misma conexión para varias transmisiones.

Gracias al keep alive podemos mitigar algunos ataques como la denegación de servicios, ya que estamos limitando la capacidad de un atacante de agotar los recursos mediante la apertura repetitiva de conexiones.

A su vez, al mantener una conexión activa durante un período más largo, se pueden establecer mecanismos de protección adicionales para mitigar los ataques CSRF. Por ejemplo, se puede utilizar un token CSRF que se renueva periódicamente mientras la conexión permanece abierta, lo que dificulta la realización de ataques CSRF.

Es importante tener en cuenta que la configuración de “keep alive” debe ser cuidadosamente ajustada para equilibrar el tiempo de espera de las conexiones abiertas y el uso eficiente de los recursos del servidor.

A continuación, un ejemplo básico con Nodejs y express:

```js
const express = require('express');
const app = express();

const server = app.listen(3000, () => {
  console.log('server on port 3000');
});

server.keepAliveTimeout = 30 * 1000; // 30 seconds
server.headersTimeout = 35 * 1000; // 35 seconds
```

### Agregar tiempos de espera en solicitudes HTTP (Timeouts)

Agregar timeouts en las solicitudes HTTP es una buena práctica para mejorar la estabilidad y el control sobre nuestra aplicación. Los timeouts evitan que tu aplicación se bloquee indefinidamente esperando una respuesta. Con un timeout configurado, puedes detectar cuando una solicitud está tardando demasiado y liberar los recursos asociados a esa conexión.

A continuación, un ejemplo básico con Nodejs y express:

```js
const express = require('express');
const app = express();

app.use((req, res, next) => {
  req.setTimeout(5000); // Set request timeout to 5 seconds
  res.setTimeout(5000); // Set response timeout to 5 seconds
  next();
});
```

### Limitar el tamaño del cuerpo HTTP (Payload size).

Establecer límites en el tamaño del cuerpo de una solicitud o respuesta puede ayudar a mitigar riesgos y proteger tu aplicación contra el ataque de desbordamiento de búfer. Si un atacante intenta manipular la petición para incluir código malicioso o datos excesivamente grandes, un limitar el tamaño puede interrumpir la solicitud y evitar posibles vulnerabilidades.

A continuación, un ejemplo básico con Nodejs y express:

```js
const express = require('express');
const app = express();
app.use(express.json());

const limitPayloadSize = (req, res, next) => {
  const MAX_PAYLOAD_SIZE = 1024 * 1024; // 1MB
  if (req.headers['content-length'] && parseInt(req.headers['content-length']) > MAX_PAYLOAD_SIZE) {
    return res.status(413).json({ error: 'Payload size exceeds the limit' });
  }
  next();
}

app.use(limitPayloadSize);

app.listen(3000, () => {
  console.log('listen on port 3000')
});
```

### Validar y desinfectar los datos de entrada de nuestra aplicación

Validar los datos de entrada de nuestra aplicación es fundamental para garantizar la integridad, seguridad y eficiencia de las operaciones.

La validación de datos implica verificar que los datos de entrada cumplan con ciertos criterios, como el formato correcto, la longitud adecuada o la ausencia de caracteres no permitidos. Al validar los datos, se evita que información incorrecta o maliciosa ingrese al sistema, lo que puede prevenir vulnerabilidades como la inyección de código o la manipulación indebida de los datos.

Por otro lado, la desinfección de datos implica limpiar y filtrar los datos de entrada para eliminar cualquier contenido potencialmente peligroso, como código malicioso o caracteres especiales que puedan causar problemas en el sistema. Esto es particularmente importante para prevenir ataques de inyección de código, como SQL injection o Cross-Site Scripting (XSS), donde los atacantes intentan aprovechar las vulnerabilidades del sistema mediante la manipulación de los datos ingresados.

A continuación, un ejemplo básico con Nodejs y express, utilizando el paquete ***Joi***.

```js
const express = require('express');
const joi = require('joi');

const app = express();
app.use(express.json());

const schema = joi.object({
  username: joi.string().alphanum().min(3).max(30).required(),
  email: joi.string().email().required(),
  password: joi.string().pattern(new RegExp('^[a-zA-Z0-9]{3,30}$')).required(),
});

app.post('/register', (req, res) => {
  const { error } = schema.validate(req.body);
  if (error) {
    return res.status(400).json({ error: error.details[0].message });
  }
  res.status(200).json({ message: 'Success' });
});

app.listen(3000, () => {
  console.log('Server listen on port 3000');
});
```

### Realizar consultas parametrizadas SQL

Las consultas parametrizadas en SQL proporcionan una capa adicional de seguridad al prevenir ataques de inyección de SQL. Además, facilitan la reutilización de consultas y ayudan a evitar errores de sintaxis y codificación.

Las consultas parametrizadas son una buena práctica recomendada siempre que se interactúe con bases de datos SQL para garantizar la seguridad, integridad y eficiencia en el acceso a los datos.

A continuación, un ejemplo básico con Nodejs y express, utilizando el paquete mysql2.

```js
const mysql = require('mysql2');
const express = require('express');
const app = express();

const connection = mysql.createConnection({
  host: 'localhost',
  user: 'user',
  password: 'pass',
  database: 'database',
});

app.get('/users/:id', (req, res) => {
  const userId = req.params.id;
  const query = 'SELECT * FROM users WHERE id = ?';
  
  connection.query(query, [userId], (error, results) => {
    if (error) {
      console.error('Error:', error);
      return res.status(500).json({ error: 'Error query' });
    }
    res.json(results);
  });
});

app.listen(3000, () => {
  console.log('Server on port 3000');
});
```

### Agregar políticas de seguridad de contenido (CSP)

Las políticas de seguridad de contenido (Content Security Policies, CSP) son un conjunto de directivas y reglas que se pueden implementar en una aplicación web para especificar qué tipos de contenido están permitidos y restringidos en la página. Estas políticas ayudan a proteger la aplicación contra diversos ataques, como Cross-Site Scripting (XSS) y inyección de contenido no deseado.

La CSP permite al propietario del sitio web definir y comunicar al navegador del usuario qué orígenes de contenido (dominios, protocolos, etc.) son considerados seguros y confiables para ser cargados y ejecutados en la página. Esto se logra mediante la inclusión de una directiva de política de seguridad de contenido en la respuesta HTTP.

Las políticas de seguridad de contenido pueden especificar restricciones sobre diferentes tipos de contenido, como scripts JavaScript, estilos CSS, imágenes, iframes, fuentes web, entre otros. Al definir estas restricciones, se reduce la superficie de ataque y se evita que el contenido no autorizado se ejecute en la página, mitigando así los riesgos asociados con los ataques de XSS, inyección de contenido no deseado y otras técnicas de explotación.

A continuación, un ejemplo básico con Nodejs y express.

```js
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.send('Hello!');
});

app.use((req, res, next) => {
  res.setHeader('Content-Security-Policy', "default-src 'self'; script-src 'self'; style-src 'self'; img-src 'self'; font-src 'self'");
  next();
});

app.listen(3000, () => {
  console.log(`Server listen on port 3000`);
});
```

#### Implementar algún método de autenticación como JWT.

Un método de autenticación es un proceso o mecanismo utilizado para verificar la identidad de un usuario o entidad que intenta acceder a un sistema o aplicación. Su propósito es asegurar que el usuario es quien dice ser y permitir o denegar el acceso a los recursos protegidos según los privilegios y derechos asignados.

Los métodos de autenticación pueden variar en su implementación y nivel de seguridad, pero siempre es recomendado implementar uno.

A continuación, un ejemplo básico con Nodejs y express, implementando una autenticación basada en token utilizando el paquete jsonwebtoken.

```js
const express = require('express');
const jwt = require('jsonwebtoken');
const app = express();

const secretKey = 'secret key';

app.post('/login', (req, res) => {
  const username = req.body.username;
  const password = req.body.password;

  const token = jwt.sign({ username, password }, secretKey, { expiresIn: '1h' });
  res.json({ token });
});


app.get('/protected', verifyToken, (req, res) => {
  res.send('Protected route!');
});


function verifyToken(req, res, next) {
  const token = req.headers.authorization;

  if (!token) {
    return res.status(401).json({ message: 'Token not provider' });
  }

  jwt.verify(token, secretKey, (err, decoded) => {
    if (err) {
      return res.status(403).json({ message: 'Invalid Token' });
    }
    req.user = decoded;
    next();
  });
}

app.listen(3000, () => {
  console.log(`Server on port 3000`);
});
```

### Auditar y reparar dependencias con NPM audit

npm audit es una herramienta integrada en el administrador de paquetes npm (Node Package Manager) que permite identificar y reportar posibles vulnerabilidades en las dependencias utilizadas en un proyecto de Node.js.

Cuando se ejecuta el comando npm audit, se analiza el archivo package-lock.json (o npm-shrinkwrap.json) del proyecto, el cual contiene información detallada sobre las versiones de las dependencias instaladas. La herramienta npm audit compara esta información con una base de datos de vulnerabilidades conocidas y proporciona un informe detallado sobre cualquier problema de seguridad que pueda existir.

El informe de npm audit muestra las vulnerabilidades encontradas, clasificadas en diferentes niveles de severidad (bajo, moderado, alto y crítico). Para cada vulnerabilidad, se proporciona información sobre la vulnerabilidad específica, las dependencias afectadas y las posibles soluciones o actualizaciones disponibles.

```sh
sudo npm audit fix
```

Posteriormente, podemos ejecutar el comando npm audit fix para solucionar automáticamente las vulnerabilidades detectadas en un proyecto de Node.js.

Cuando se ejecuta el comando npm audit fix, npm analiza el archivo package-lock.json (o npm-shrinkwrap.json) del proyecto en busca de vulnerabilidades conocidas en las dependencias. Luego, intenta solucionar estas vulnerabilidades aplicando las actualizaciones de paquetes disponibles que resuelven los problemas de seguridad.

⚠️ Por último, es importante tener en cuenta que npm audit fix intentará aplicar las actualizaciones que no rompan la compatibilidad del proyecto. Sin embargo, en algunos casos, las actualizaciones pueden introducir cambios que afecten el funcionamiento del código existente. Por lo tanto, es recomendable revisar cuidadosamente los cambios propuestos por npm audit fix antes de aceptarlos y ejecutar pruebas exhaustivas para asegurarse de que no haya impactos no deseados en el proyecto.