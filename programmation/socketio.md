# Socket.io — Fiche de référence complète

---

## C'est quoi Socket.io ?

Socket.io est une bibliothèque qui permet la **communication bidirectionnelle en temps réel** entre un serveur Node.js et des clients (navigateur, mobile, autre serveur). Elle repose sur les WebSockets avec un fallback automatique (long polling) si les WebSockets ne sont pas disponibles.

---

## Installation

```bash
# Serveur
npm install socket.io

# Client (navigateur via npm)
npm install socket.io-client

# Client CDN (navigateur directement)
# <script src="/socket.io/socket.io.js"></script>
# (servi automatiquement par le serveur Socket.io)
```

---

## Mise en place de base

### Serveur (Node.js + Express)

```js
const express = require('express');
const { createServer } = require('http');
const { Server } = require('socket.io');

const app = express();
const httpServer = createServer(app);
const io = new Server(httpServer, {
  cors: {
    origin: 'http://localhost:3000', // ou '*' en dev
    methods: ['GET', 'POST']
  }
});

io.on('connection', (socket) => {
  console.log('Client connecté :', socket.id);

  socket.on('disconnect', (raison) => {
    console.log('Client déconnecté :', raison);
  });
});

httpServer.listen(3000, () => {
  console.log('Serveur démarré sur http://localhost:3000');
});
```

### Client (navigateur)

```html
<!-- index.html -->
<script src="/socket.io/socket.io.js"></script>
<script>
  const socket = io('http://localhost:3000');

  socket.on('connect', () => {
    console.log('Connecté, ID :', socket.id);
  });

  socket.on('disconnect', () => {
    console.log('Déconnecté');
  });
</script>
```

### Client (Node.js / ES module)

```js
import { io } from 'socket.io-client';

const socket = io('http://localhost:3000', {
  auth: { token: 'montoken' },
  reconnection: true,
  reconnectionAttempts: 5,
  reconnectionDelay: 1000
});

socket.on('connect', () => console.log('Connecté :', socket.id));
```

---

## Émettre et recevoir des événements

### Schéma général

```
Émetteur                     Récepteur
socket.emit('event', data) → socket.on('event', (data) => {})
```

### Côté serveur

```js
io.on('connection', (socket) => {

  // Recevoir un événement du client
  socket.on('message', (data) => {
    console.log('Reçu :', data);
  });

  // Recevoir avec acquittement (acknowledgement)
  socket.on('creer-user', (data, callback) => {
    const user = creerUser(data);
    callback({ success: true, user }); // réponse au client
  });

  // Émettre vers ce client uniquement
  socket.emit('bienvenue', { message: 'Bonjour !' });

  // Émettre vers tous sauf ce client
  socket.broadcast.emit('nouveau-client', { id: socket.id });

  // Émettre vers tous (ce client inclus)
  io.emit('alerte', { texte: 'Serveur en maintenance' });
});
```

### Côté client

```js
// Envoyer un événement
socket.emit('message', { texte: 'Bonjour tout le monde' });

// Envoyer avec acquittement
socket.emit('creer-user', { nom: 'Alice' }, (reponse) => {
  if (reponse.success) console.log('User créé :', reponse.user);
});

// Recevoir un événement
socket.on('bienvenue', (data) => {
  console.log(data.message);
});

// Écouter une seule fois
socket.once('init', (data) => {
  console.log('Init reçu :', data);
});

// Supprimer un écouteur
const handler = (data) => console.log(data);
socket.on('event', handler);
socket.off('event', handler);
```

---

## Rooms (salons)

Les rooms permettent de regrouper des sockets et d'émettre vers un groupe précis.

```js
io.on('connection', (socket) => {

  // Rejoindre une room
  socket.join('salle-1');
  socket.join(['salle-1', 'salle-2']); // plusieurs rooms

  // Quitter une room
  socket.leave('salle-1');

  // Émettre vers une room (ce socket inclus)
  io.to('salle-1').emit('message', { texte: 'Bonjour la salle 1' });

  // Émettre vers une room (ce socket exclu)
  socket.to('salle-1').emit('message', { texte: 'Quelqu'un a rejoint' });

  // Émettre vers plusieurs rooms
  io.to('salle-1').to('salle-2').emit('alerte', { ... });

  // Connaître les rooms d'un socket
  console.log(socket.rooms); // Set { socket.id, 'salle-1' }

  // Connaître les sockets d'une room
  const sockets = await io.in('salle-1').fetchSockets();

  // Cas d'usage : chat de groupe
  socket.on('rejoindre-salon', (salon) => {
    socket.join(salon);
    socket.to(salon).emit('notification', { texte: `${socket.id} a rejoint` });
  });

  socket.on('message-salon', ({ salon, texte }) => {
    io.to(salon).emit('message', { texte, auteur: socket.id });
  });
});
```

---

## Namespaces (espaces de noms)

Les namespaces permettent de créer des canaux distincts sur le même serveur/port.

```js
// Serveur

// Namespace par défaut : /
io.on('connection', socket => { ... });

// Namespace personnalisé
const chat = io.of('/chat');
const admin = io.of('/admin');

chat.on('connection', (socket) => {
  console.log('Connexion chat :', socket.id);
  chat.emit('bienvenue', 'Bienvenue dans le chat');
});

admin.use((socket, next) => {
  // Middleware de namespace : authentification
  if (socket.handshake.auth.role === 'admin') next();
  else next(new Error('Accès refusé'));
});

admin.on('connection', (socket) => {
  console.log('Admin connecté :', socket.id);
});
```

```js
// Client
const chatSocket = io('http://localhost:3000/chat');
const adminSocket = io('http://localhost:3000/admin', {
  auth: { role: 'admin' }
});
```

---

## Middlewares

```js
// Middleware global (tous les namespaces)
io.use((socket, next) => {
  const token = socket.handshake.auth.token;
  if (!token) return next(new Error('Non authentifié'));

  try {
    socket.user = jwt.verify(token, process.env.SECRET);
    next();
  } catch {
    next(new Error('Token invalide'));
  }
});

// Accéder aux données de handshake
socket.handshake.auth;      // { token: '...' }
socket.handshake.query;     // ?page=2
socket.handshake.headers;
socket.handshake.address;   // IP du client
```

---

## Données persistantes sur le socket

```js
// Stocker des données sur un socket
socket.data.userId = '123';
socket.data.room = 'salle-1';

// Accéder depuis n'importe où
io.on('connection', (socket) => {
  console.log(socket.data.userId);
});
```

---

## Adapter — Plusieurs serveurs (scaling)

Pour faire fonctionner Socket.io sur plusieurs instances Node.js :

```bash
npm install @socket.io/redis-adapter ioredis
```

```js
const { createClient } = require('ioredis');
const { createAdapter } = require('@socket.io/redis-adapter');

const pubClient = createClient({ host: 'localhost', port: 6379 });
const subClient = pubClient.duplicate();

io.adapter(createAdapter(pubClient, subClient));
```

---

## Exemple complet — Chat en temps réel

### Serveur

```js
const express = require('express');
const { createServer } = require('http');
const { Server } = require('socket.io');

const app = express();
const httpServer = createServer(app);
const io = new Server(httpServer, { cors: { origin: '*' } });

const utilisateurs = new Map(); // socketId → { nom, salon }

io.on('connection', (socket) => {

  socket.on('rejoindre', ({ nom, salon }) => {
    socket.data.nom = nom;
    socket.data.salon = salon;
    utilisateurs.set(socket.id, { nom, salon });

    socket.join(salon);
    socket.to(salon).emit('notification', `${nom} a rejoint le salon`);
    socket.emit('bienvenue', { message: `Bienvenue dans ${salon} !` });

    // Envoyer la liste des membres
    const membres = [...utilisateurs.values()]
      .filter(u => u.salon === salon)
      .map(u => u.nom);
    io.to(salon).emit('membres', membres);
  });

  socket.on('message', ({ texte }) => {
    const { nom, salon } = socket.data;
    io.to(salon).emit('message', {
      auteur: nom,
      texte,
      horodatage: new Date().toISOString()
    });
  });

  socket.on('disconnect', () => {
    const user = utilisateurs.get(socket.id);
    if (user) {
      utilisateurs.delete(socket.id);
      socket.to(user.salon).emit('notification', `${user.nom} a quitté le salon`);
    }
  });
});

httpServer.listen(3000);
```

### Client

```html
<script src="/socket.io/socket.io.js"></script>
<script>
  const socket = io();

  // Rejoindre
  socket.emit('rejoindre', { nom: 'Alice', salon: 'général' });

  // Envoyer un message
  document.querySelector('#form').addEventListener('submit', (e) => {
    e.preventDefault();
    const texte = document.querySelector('#input').value;
    socket.emit('message', { texte });
    document.querySelector('#input').value = '';
  });

  // Recevoir les messages
  socket.on('message', ({ auteur, texte, horodatage }) => {
    const li = document.createElement('li');
    li.textContent = `[${auteur}] ${texte}`;
    document.querySelector('#messages').appendChild(li);
  });

  socket.on('notification', (msg) => {
    console.log('ℹ', msg);
  });

  socket.on('membres', (liste) => {
    document.querySelector('#membres').textContent = liste.join(', ');
  });
</script>
```

---

## Options de configuration

```js
const io = new Server(httpServer, {
  // CORS
  cors: {
    origin: ['http://localhost:3000', 'https://monapp.com'],
    methods: ['GET', 'POST'],
    credentials: true
  },

  // Transports
  transports: ['websocket', 'polling'], // websocket en priorité

  // Ping / pong (détection déconnexion)
  pingInterval: 25000,   // toutes les 25s
  pingTimeout: 5000,     // délai avant de considérer déconnecté

  // Taille max des messages
  maxHttpBufferSize: 1e6, // 1 MB

  // Path personnalisé
  path: '/mon-socket/',   // défaut : /socket.io/
});
```

---

## Options de connexion client

```js
const socket = io('http://localhost:3000', {
  auth: { token: 'montoken' },       // envoyé dans le handshake
  query: { version: '1.0' },         // paramètres de query
  transports: ['websocket'],         // forcer WebSocket
  reconnection: true,
  reconnectionAttempts: Infinity,
  reconnectionDelay: 1000,           // délai initial (ms)
  reconnectionDelayMax: 5000,        // délai max
  timeout: 20000,                    // délai de connexion
  autoConnect: false,                // connexion manuelle
});

// Connexion manuelle
socket.connect();
socket.disconnect();
```

---

## Gestion des événements système

```js
// Client
socket.on('connect', () => { ... });
socket.on('disconnect', (raison) => { ... });
socket.on('connect_error', (err) => {
  console.log('Erreur de connexion :', err.message);
});
socket.on('reconnect', (tentative) => { ... });
socket.on('reconnect_attempt', (tentative) => { ... });
socket.on('reconnect_failed', () => { ... });

// Serveur
io.on('connection', (socket) => {
  socket.on('disconnect', (raison) => {
    // Raisons : 'transport close', 'transport error',
    //           'server namespace disconnect', 'client namespace disconnect'
  });

  socket.on('error', (err) => {
    console.error('Erreur socket :', err);
  });
});
```

---

## Méthodes utiles côté serveur

```js
// Obtenir tous les sockets connectés
const sockets = await io.fetchSockets();

// Émettre vers un socket spécifique par son ID
io.to(socketId).emit('event', data);

// Connaître le nombre de connexions
const count = io.engine.clientsCount;

// Déconnecter un socket
io.in(socketId).disconnectSockets(true);

// Faire rejoindre une room à tous les sockets
io.socketsJoin('nouvelle-room');

// Faire quitter une room à tous les sockets
io.socketsLeave('ancienne-room');
```

---

## Résumé des méthodes d'émission

| Méthode | Destinataires |
|---|---|
| `socket.emit('ev', data)` | Ce socket uniquement |
| `socket.broadcast.emit('ev', data)` | Tous sauf ce socket |
| `io.emit('ev', data)` | Tous les sockets |
| `io.to('room').emit('ev', data)` | Room entière (ce socket inclus) |
| `socket.to('room').emit('ev', data)` | Room entière (ce socket exclu) |
| `io.except('room').emit('ev', data)` | Tous sauf cette room |
| `io.of('/ns').emit('ev', data)` | Tous dans ce namespace |

---

## Bonnes pratiques

| Pratique | Explication |
|---|---|
| Authentifier dans le middleware | Vérifier le token avant `connection` |
| Nommer les événements clairement | `user:joined`, `message:sent` (préfixe:action) |
| Valider les données reçues | Ne jamais faire confiance au client |
| Utiliser les rooms plutôt que les IDs | Plus maintenable pour les groupes |
| Gérer les déconnexions proprement | Nettoyer les états, notifier les autres |
| Limiter la fréquence d'émission | Éviter le flooding côté client |
| Utiliser des acquittements | Pour les opérations critiques (paiement, etc.) |
| Logger les événements en dev | Pour déboguer facilement |