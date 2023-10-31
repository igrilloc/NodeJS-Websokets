# Chat! **NodeJS-Web Sockets** 

NOTA: si bien en este proyecto se práctica:
- Manejo de eventos y protocolos.
- Emitir un Broadcast.
- HTML y CSS.
- Vanilla JavaScript.
- ECMAScript module.
- SQL.
- Persistencia de datos.
El objetivo principal de este proyecto es aprender y prácticar la conexion de NodeJS y WebSockets en un chat.

## **📌 HTTP vs Web Sockets ➡️ https://ably.com/topic/websockets-vs-http**

Comparativa: 01-comparativa.webp

HTTP es un protocolo de comunicación que se basa en el intercambio de mensajes entre un cliente y un servidor. Usa TCP como protocolo de transporte.

El cliente realiza una petición y el servidor responde con una respuesta.

Explicación: 02-http.webp

Web Sockets es un protocolo de comunicación que se basa en el intercambio de mensajes entre un cliente y un servidor. Usa TCP como protocolo de transporte.

La diferencia es que el cliente y el servidor pueden intercambiar mensajes en cualquier momento.

Explicación: 03-websockets.webp

Cuando HTTP es mejor:
- Para pedir recursos
- Recursos cacheables
- Para REST API y uso de metodos HTTP
- Sincronizar eventos

Cuando Web Sockets es mejor:
- Para comunicación bidireccional
- Para comunicación en tiempo real
- Para comunicación de alta frecuencia
- Actualización con poca latencia

# - Instalación de Express
npm install express
```JS
import express from 'express'

const port = process.env.PORT ?? 3000;

const app = express();

app.get('/', (req, res) => {
  res.send('<h1>Front Page</h1>');
});

// no usar app, si no server
server.listen(port, () => {
  console.log(`Started at ${port}`);
});
```

# - Instalando para mejorar el logging de Express
```JS
import express from 'express'
import logger from 'morgan'

const port = process.env.PORT ?? 3000;

const app = express();
app.use(logger('dev'));

app.get('/', (req, res) => {
  res.send('<h1>Front Page</h1>');
});

app.listen(port, () => {
  console.log(`Started at ${port}`);
});
```

# - Servir nuestro HTML
```JS
app.get('/', (req, res) => {
  res.sendFile(process.cwd() + '/client/index.html');
});
```

# - Crear el HTML
```HTML
<!DOCTYPE html>
<html>

<head>
  <meta name="viewport" content="width=device-width,initial-scale=1.0">
  <title>Socket.IO chat</title>
  <style>
    *,
    *:before,
    *:after {
      box-sizing: border-box;
    }

    body {
      margin: 0;
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
      display: grid;
      place-content: center;
      height: 100vh;
      padding: 36px;
      grid-template-rows: 1fr;
    }

    #chat {
      border: 1px solid #ccc;
      border-radius: 4px;
      overflow: hidden;
      width: 350px;
      height: 100%;
      position: relative;
    }
  </style>
</head>

<body>
  <section id="chat">
    <form id="form" action="">
      <input id="input" autocomplete="off" /><button>Enviar</button>
    </form>
  </section>
</body>

</html>
```

# - Instalando socket.io
```bash
npm install socket.io
```

```JS
import { createServer } from 'node:http'
import { Server } from 'socket.io'

const server = createServer(app)
const io = new Server(server)

io.on('connection', (socket) => {
  console.log('a user connected')
})
```

# - Carga de socket en el client
```HTML
<script type="module">
  import { io } from "https://cdn.socket.io/4.3.2/socket.io.esm.min.js";

  const socket = io()
</script>
```
Vemos que ahora cuando se conecta un usuario, se muestra en el servidor "a user connected". Vamos a ver también cuando se desconecta.

```JS
io.on('connection', (socket) => {
  console.log('a user connected')

  socket.on('disconnect', () => {
    console.log('user disconnected')
  })
})
```

# - Mejorando la apariencia del form:
```HTML
<style>
#form {
  bottom: 0;
  display: flex;
  height: 48px;
  left: 0;
  padding: 4px;
  position: absolute;
  right: 0;
}

#input {
  border-radius: 9999px;
  border: 1px solid #eee;
  flex: 1;
  margin: 4px;
  padding: 0 8px;
}

#input:focus {
  outline: none;
}

#form>button {
  background: #09f;
  color: #fff;
  border: none;
  margin: 4px;
  border-radius: 4px;
}

#form>button:hover {
  background: #0af;
}
</style>
</script>
```

# - Enviar eventos
```HTML
<script>
const form = document.getElementById('form');
const input = document.getElementById('input');

form.addEventListener('submit', (event) => {
  event.preventDefault();

  if (input.value) {
    socket.emit('chat message', input.value);
    input.value = '';
  }
});
</script>
```

# - Escuchar evento en el servidor
```JS
io.on('connection', (socket) => {
  console.log('a user connected')

  socket.on('chat message', (msg) => {
    console.log('message: ' + msg)
  })

  socket.on('disconnect', () => {
    console.log('user disconnected')
  })
})
```

# - Enviar broadcast desde el servidor
Emitir el evento a todos los clientes que tenemos conectados:
```JS
io.on('connection', (socket) => {
  console.log('a user connected')

  socket.on('chat message', (msg) => {
    io.emit('chat message', msg) // <-----
  })

  socket.on('disconnect', () => {
    console.log('user disconnected')
  })
})
```

# - Escuchar broadcasting en cliente
```HTML
<script>
  const messages = document.getElementById('messages');

  socket.on('chat message', msg => {
    const item = `<li>${msg}</li>`;
    messages.insertAdjacentHTML('beforeend', item);
  })
</script>
```

# - Mejorar estilos de los mensajes
```HTML
<style>
#messages {
  list-style-type: none;
  margin: 0;
  padding: 0;
}

#messages>li {
  padding: 0.5rem 1rem;
}

#messages>li:nth-child(odd) {
  background: #efefef;
}
</style>
```

# - Recuperar usuario desconectado
¿Qué pasa si el usuario se desconecta? Los mensajes se pierden.
Cómo evitarlo:
```JS
const io = new Server(server, {
  connectionStateRecovery: {}
})
```

# - Mostrar al usuario cuando pierde conexión
```HTML
<script>
  socket.on('disconnect', (socket) => {
    console.log(socket)
    logMessage('disconnected')
  })

  function logMessage(msg) {
    log.insertAdjacentHTML('beforeend', `<p>${msg}</p>`);
  }
</script>

<style>
#log {
  position: fixed;
  bottom: 16px;
  right: 16px;
  height: 50px;
  width: 150px;
  font-size: 10px;
  background: #eee;
}
</style>
```

# - Sincronizar con base de datos
https://turso.tech/

```bash
$ brew install tursodatabase/tap/turso
$ turso auth login
$ turso db create
$ turso db show humble-spectrum
```

```bash
npm install @libsql/client dotenv
```

Crear la conexión:
```JS
import dotenv from 'dotenv'
import { createClient } from "@libsql/client"

dotenv.config()

const db = createClient({
  url: "libsql://humble-spectrum-midudev.turso.io",
  authToken: process.env.DB_TOKEN
});
```

```bash
$ turso db show humble-spectrum
$ turso db tokens create humble-spectrum | pbcopy
```

Crear la tabla si no existe:
```JS
await db.execute(`
  CREATE TABLE IF NOT EXISTS messages (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    content TEXT
  );
`);
```

Guarda el mensaje cada vez que se envía:
```JS
socket.on('chat message', async (msg) => {
  let result;
  try {
    console.log(msg)
  
    result = await db.execute({
      sql: `INSERT INTO messages (content) VALUES (:content)`,
      args: { content: msg }
    });
  } catch (e) {
    console.error(e)
    return
  }

  io.emit('chat message', msg, result.lastInsertRowid.toString())
})
```

# - Saber en qué mensaje se quedó el cliente
```JS
const socket = io({
  auth: {
    serverOffset: 0
  }
})

socket.on('chat message', (msg, serverOffset) => {
  const item = `<li>${msg}</li>`;
  messages.insertAdjacentHTML('beforeend', item);
  socket.auth.serverOffset = serverOffset // <----
})
```

# - Recuperar mensajes anteriores
```JS
io.on('connection', async (socket) => {
  console.log('a user connected')

  socket.on('chat message', async (msg) => {
    // ...
  })

  if (!socket.recovered) {
    try {
      const results = await db.execute({
        sql: `SELECT id, content FROM messages WHERE id > ?`,
        args: [socket.handshake.auth.serverOffset ?? 0]
      })

      results.rows.forEach(row => {
        socket.emit('chat message', row.content, row.id)
      })
    } catch (e) {
      console.error(e)
    }
  }
});
```

# - Tener usuario random
```bash
$ turso db shell humble-spectrum
```

```JS
await db.execute(`
  CREATE TABLE IF NOT EXISTS messages (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    content TEXT,
    user TEXT
  );
`);
```

# - Crear usuario random en el cliente:
```JS
const getUsername = async () => {
  const username = localStorage.getItem('username')
  if (username) {
    logMessage(`generated username ${username}`)
    return username
  }

  const res = await fetch('https://random-data-api.com/api/v2/users')
  const { username: randomUsername } = await res.json()

  localStorage.setItem('username', randomUsername)
  logMessage(`generated username ${randomUsername}`)

  return randomUsername
}

const socket = io({
  auth: {
    username: await getUsername(),
    serverOffset: 0
  }
})
```

# - Mostrar usuario visualmente en el chat
```JS
socket.on('chat message', (msg, serverOffset, username) => {
  const item = `<li>
    <p>${msg}</p><small>enviado por ${username}
  </li>`;
});
```

# - Guardar usuario en la base de datos
```JS
socket.on('chat message', async (msg) => {
let result;
// ⬇️
const user = socket.handshake.auth.username ?? 'anonymous'

try {
  result = await db.execute({ // ⬇️
    sql: `INSERT INTO messages (content, user) VALUES (:content, :user)`,
    args: { content: msg, user }
  });
catch {}

io.emit('chat message', msg, result.lastInsertRowid.toString(), user) // <------
```

```JS
if (!socket.recovered) {
  try {
    const results = await db.execute({ // ⬇️
      sql: `SELECT id, content, user FROM messages WHERE id > ?`,
      args: [socket.handshake.auth.serverOffset ?? 0]
    })

    results.rows.forEach(row => { // ⬇️
      socket.emit('chat message', row.content, row.id, row.user)
    })
  } catch (e) {
    console.error(e)
  }
}
```# NodeJS-Websokets
