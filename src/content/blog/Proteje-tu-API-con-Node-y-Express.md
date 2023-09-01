---
title: Protege tu API con Node y Express
author: Diego Franco Roy
pubDatetime: 2023-08-30T22:00:00.000Z
postSlug: protege-tu-api-con-node-y-express
featured: true
description: Consejos de seguridad para protejer tu API con Node y Express.
tags:
  - Seguridad
  - Express
  - Nodejs
ogImage: /blog-images/og-image.jpeg
---

# Cómo Proteger tu API con Node.js y Express

Las APIs (Interfaces de Programación de Aplicaciones) se han convertido en componentes esenciales para el desarrollo de aplicaciones modernas. Sin embargo, a medida que las APIs se vuelven más cruciales, también se vuelven más susceptibles a amenazas de seguridad. Proteger tu API es crucial para garantizar la integridad de tus datos y la privacidad de tus usuarios. En este artículo, exploraremos cómo puedes proteger tu API utilizando Node.js y Express, dos tecnologías populares en el desarrollo web.

## Las mejores prácticas de seguridad en el servidor

* [Utilizar HTTPS](#utilizar-https)
* [Validar datos de entrada](#validar-datos-de-entrada)
* [Autenticación y Autorización](#autenticación-y-autorización)
* [Limitar las solicitudes](#limitar-las-solicitudes)
* [Implementar CORS](#implementar-cors)
* [Utilizar Helmet](#utilizar-helmet)
* [Monitoriza y Registra Actividades](#monitoriza-y-registra-actividades)
* [Mantener dependencias actualizadas](#mantener-dependencias-actualizadas)

### Utilizar HTTPS

El primer paso para proteger tu API es asegurarte de que las comunicaciones entre el cliente y el servidor estén encriptadas. Utilizar HTTPS (Protocolo Seguro de Transferencia de Hipertexto) es fundamental para evitar que los datos sean interceptados por terceros malintencionados. Puedes obtener un certificado SSL/TLS de una autoridad de certificación confiable para habilitar HTTPS en tu servidor Node.js.

```javascript
const https = require('https');
const fs = require('fs');
const express = require('express');

const app = express();
const options = {
  key: fs.readFileSync('ruta/a/clave-privada.key'),
  cert: fs.readFileSync('ruta/a/certificado.crt')
};

const server = https.createServer(options, app);

server.listen(3000, () => {
  console.log('Listening...');
});
```

### Validar datos de entrada

La validación de datos de entrada es esencial para prevenir ataques como la [inyección de SQL](https://es.wikipedia.org/wiki/Inyecci%C3%B3n_SQL "inyección SQL") y el [cross-site scripting (XSS)](https://es.wikipedia.org/wiki/Cross-site_scripting "cross site scripting"). Utiliza bibliotecas como express-validator para validar y sanitizar los datos de entrada antes de procesarlos en tu API.

```javascript
const { body, validationResult } = require('express-validator');
const express = require('express');
const app = express();

app.post('/singup', [
  body('email').isEmail(),
  body('password').isStrongPassword(),
], (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({ errores: errors.array() });
  }
  // Process data...
});
```

### Autenticación y Autorización

Implementa un sistema de autenticación sólido para asegurarte de que solo los usuarios autorizados puedan acceder a tu API. Puedes utilizar estrategias de autenticación como JWT (Tokens de Acceso JSON) o OAuth.

```javascript
const express = require('express');
const jwt = require('jsonwebtoken');
const app = express();

// Authentication middleware
const verificarToken = (req, res, next) => {
  const token = req.header('Authorization');

  if (!token) {
    return res.status(401).json({ mensaje: 'Access denied.' });
  }

  try {
    const decodificado = jwt.verify(token, 'secret');
    req.usuario = decodificado.usuario;
    next();
  } catch (error) {
    res.status(401).json({ mensaje: 'Invalid token.' });
  }
};

app.get('/protected-data', verificarToken, (req, res) => {
  // Access allowed for authenticated users only
});
```

### Limitar las solicitudes

Para evitar ataques de [denegación de servicio (DoS)](https://es.wikipedia.org/wiki/Ataque_de_denegaci%C3%B3n_de_servicio "ataque DoS"), puedes implementar límites en las solicitudes que tu API puede manejar en un intervalo de tiempo determinado utilizando bibliotecas como express-rate-limit.

#### Limitar por IP

```javascript
const express = require('express');
const rateLimit = require('express-rate-limit');
const app = express();

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // maximum 100 requests per IP in the interval,
  standardHeaders: true,
});

app.use(limiter);
```

#### Limitar por usuario

```javascript
const express = require('express');
const rateLimit = require('express-rate-limit');
const app = express();

// Authentication middleware
// ...

// Rate limiting middleware
const userRateLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // Maximum requests per time window
  keyGenerator: (req) => req.user.id, // Key generator based on the user ID
  handler: (req, res) => {
    res.status(429).json({ error: 'Speed limit reached. Try again later.' });
  },
});

app.use(userRateLimiter);

app.get('/protected', [verificarToken, userRateLimiter], (req, res) => {
  res.json({ message: 'Acceso concedido a la ruta protegida' });
});


app.listen(3000, () => {
  console.log('listen on port 3000');
});
```

#### Limitar por tamaño de cabezera

Esto sirve para proteger tu aplicación contra el [ataque de desbordamiento de búfer](https://es.wikipedia.org/wiki/Desbordamiento_de_b%C3%BAfer "ataque de desbordamiento de búfer").

```javascript
const express = require('express');
const app = express();
app.use(express.json());

const limitPayloadSize = (req, res, next) => {
  const MAX_PAYLOAD_SIZE = 1024 * 1024; // 1MB
  if (req.headers['content-length'] && parseInt(req.headers['content-length']) > MAX_PAYLOAD_SIZE){
    return res.status(413).json({ error: 'Payload size exceeds the limit' });
  }
  next();
}

app.use(limitPayloadSize);

app.listen(3000, () => {
  console.log('listen on port 3000')
});
```

También se puede hacer usando body-parser o un reverse proxy.

```javascript
const express = require('express');
const bodyParser = require('body-parser');
const app = express();

app.use(bodyParser.json({ limit: '1mb' }));

app.listen(3000, () => {
  console.log('listen on port 3000');
});
```

### Implementar [CORS](https://developer.mozilla.org/es/docs/Web/HTTP/CORS "cors")

Por defecto, los navegadores aplican la [“Política del mismo origen” (Same-Origin Policy)](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy "Política del mismo origen"), que impide que un script en un sitio web acceda a recursos en otro dominio. Pero si tu API permite solicitudes desde diferentes dominios (CORS), configura cuidadosamente las opciones de CORS para evitar que sitios no autorizados accedan a tu API.

```javascript
const express = require('express');
const cors = require('cors');
const app = express();

const opcionesCors = {
  origin: ['https://domain1.com', 'https://domain2.com'],
  methods: 'GET,PUT,POST,DELETE',
};

app.use(cors(opcionesCors));
```

### Utilizar [Helmet](https://helmetjs.github.io/ "Helmet")

Helmet es una colección de funciones de middleware que establecen cabeceras HTTP relacionadas con la seguridad.

* csp establece la cabecera ***Content-Security-Policy*** para evitar ataques de scripts entre sitios y otras inyecciones entre sitios.
* hidePoweredBy elimina la cabecera ***X-Powered-By***.
* hsts establece la cabecera ***Strict-Transport-Security*** que fuerza conexiones seguras (HTTP sobre SSL/TLS) con el servidor.
* ieNoOpen establece ***X-Download-Options*** para IE8+.
* noCache establece cabeceras ***Cache-Control*** y ***Pragma*** para inhabilitar el almacenamiento en memoria caché del lado de cliente.
* noSniff establece ***X-Content-Type-Options*** para evitar que los navegadores rastreen mediante MIME una respuesta del tipo de contenido declarado.
* frameguard establece la cabecera ***X-Frame-Options*** para proporcionar protección contra el clickhacking.
* xssFilter establece ***X-XSS-Protection*** para habilitar el filtro de scripts entre sitios (XSS) en los navegadores web más recientes.

```javascript
const express = require('express');
const helmet = require('helmet');

const app = express();

app.use(helmet());

app.listen(3000, () => {
  console.log('Server is running on port 3000');
});
```

#### Como mínimo, inhabilitar la cabecera X-Powered-By

Si no quiere utilizar Helmet por lo menos inhabilite la cabecera X-Powered-By. Los atacantes pueden utilizar esta cabecera (que está habilitada de forma predeterminada) para detectar las aplicaciones que ejecutan Express e iniciar ataques con destinos específicos.

```javascript
app.disable('x-powered-by');
```

### Monitoriza y Registra Actividades

El registro y la monitorización son increíblemente importantes para una seguridad consistente en Node.js. Monitorizar tus registros te da una visión de lo que está pasando en tu aplicación para que puedas investigar cualquier cosa sospechosa. Algunos niveles importantes de registro son info, error, warn y debug.

```javascript
const express = require("express");
const winston = require("winston");
const app = express();

const logger = winston.createLogger({
  level: "debug",
  format: winston.format.json(),
  transports: [new winston.transports.Console()],
});

app.get("/", (req, res, next) => {
  logger.debug("Home '/' route.");
  res.status(200).send("Logging Hello World..");
});

app.get("/data", (req, res, next) => {
  try {
    throw new Error("Not found!");
  } catch (error) {
    logger.error("Data Error: Not found");
    res.status(404).send("Error!");
  }
});

app.listen(3000, () => {
  logger.info("Server Listening On Port 3000");
});
```

### Mantener dependencias actualizadas

Las vulnerabilidades de seguridad pueden surgir de las dependencias desactualizadas. Utiliza herramientas como npm audit para verificar las vulnerabilidades en tus paquetes y asegúrate de mantener tus dependencias actualizadas.

#### Npm audit

![](/blog-images/npm-audit.png)

#### Npm audit fix![](/blog-images/npm-audit-fix.png)
