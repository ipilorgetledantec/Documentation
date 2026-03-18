# Node.js — Fiche de référence complète

---

## C'est quoi Node.js ?

Node.js est un **runtime JavaScript** côté serveur, basé sur le moteur V8 de Chrome. Il utilise un modèle **non-bloquant et événementiel** (event loop), idéal pour les applications I/O intensives (API, serveurs web, temps réel).

---

## Installation et version

```bash
node -v         # version Node
npm -v          # version npm
npx -v          # version npx

# Gestionnaire de versions recommandé
nvm install 20
nvm use 20
```

---

## Initialiser un projet

```bash
npm init -y                  # crée package.json
npm install express          # installer un paquet
npm install -D nodemon       # dépendance de dev
npm install -g typescript    # installation globale
npm uninstall express
npm update
npm list                     # paquets installés
npm audit                    # vérifier les vulnérabilités
```

### package.json — clés essentielles

```json
{
  "name": "mon-projet",
  "version": "1.0.0",
  "main": "index.js",
  "type": "module",
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js",
    "test": "jest"
  },
  "dependencies": {
    "express": "^4.18.0"
  },
  "devDependencies": {
    "nodemon": "^3.0.0"
  }
}
```

> `"type": "module"` pour utiliser les imports ES6 (`import/export`). Sans cette clé, Node utilise CommonJS (`require`).

---

## Modules

### CommonJS (par défaut)

```js
// Exporter
module.exports = { saluer, PI };
module.exports = function() { ... }; // export unique

// Importer
const { saluer } = require('./utils');
const express = require('express');
```

### ES Modules (avec `"type": "module"`)

```js
import express from 'express';
import { readFile } from 'fs/promises';
export const PI = 3.14;
export default function handler() { ... }
```

---

## Modules natifs essentiels

### `fs` — Système de fichiers

```js
const fs = require('fs');
const fsp = require('fs/promises'); // version async

// Lire
const contenu = fs.readFileSync('fichier.txt', 'utf8');
const contenu = await fsp.readFile('fichier.txt', 'utf8');

// Écrire
fs.writeFileSync('out.txt', 'données');
await fsp.writeFile('out.txt', 'données');

// Ajouter
await fsp.appendFile('log.txt', 'nouvelle ligne\n');

// Exister / supprimer / renommer
await fsp.access('fichier.txt');                // jette si absent
await fsp.unlink('fichier.txt');
await fsp.rename('old.txt', 'new.txt');

// Dossiers
await fsp.mkdir('dossier', { recursive: true });
await fsp.rmdir('dossier', { recursive: true });
const fichiers = await fsp.readdir('dossier');
const stats = await fsp.stat('fichier.txt');
```

### `path` — Chemins de fichiers

```js
const path = require('path');

path.join('dossier', 'sous', 'fichier.txt');  // dossier/sous/fichier.txt
path.resolve('relatif/path');                 // chemin absolu
path.dirname('/a/b/c.txt');                   // /a/b
path.basename('/a/b/c.txt');                  // c.txt
path.extname('fichier.txt');                  // .txt
path.parse('/a/b/c.txt');                     // { dir, base, name, ext }

// Variable globale utile
__dirname;   // répertoire du fichier courant (CommonJS)
__filename;  // chemin complet du fichier courant (CommonJS)

// Équivalent ESM
import { fileURLToPath } from 'url';
const __dirname = path.dirname(fileURLToPath(import.meta.url));
```

### `os` — Système d'exploitation

```js
const os = require('os');

os.platform();      // 'linux', 'darwin', 'win32'
os.arch();          // 'x64'
os.cpus();          // infos CPU
os.totalmem();      // mémoire totale en octets
os.freemem();       // mémoire libre
os.homedir();       // /home/user
os.tmpdir();        // /tmp
os.hostname();
```

### `http` / `https` — Serveur HTTP natif

```js
const http = require('http');

const server = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({ message: 'OK' }));
});

server.listen(3000, () => {
  console.log('Serveur sur http://localhost:3000');
});
```

### `url` — Manipulation d'URLs

```js
const { URL } = require('url');

const u = new URL('https://example.com/api?page=2&limit=10');
u.hostname;           // 'example.com'
u.pathname;           // '/api'
u.searchParams.get('page');  // '2'
u.searchParams.set('page', 3);
```

### `crypto` — Cryptographie

```js
const crypto = require('crypto');

// Hash
crypto.createHash('sha256').update('texte').digest('hex');

// UUID
crypto.randomUUID();

// Octets aléatoires
crypto.randomBytes(16).toString('hex');

// HMAC
crypto.createHmac('sha256', 'secret').update('données').digest('hex');
```

### `events` — Émetteur d'événements

```js
const EventEmitter = require('events');

class MonEmetteur extends EventEmitter {}
const emetteur = new MonEmetteur();

emetteur.on('data', (payload) => console.log(payload));
emetteur.once('connexion', () => console.log('connecté')); // une seule fois
emetteur.emit('data', { id: 1 });
emetteur.off('data', handler);    // supprimer un écouteur
emetteur.removeAllListeners('data');
```

### `stream` — Flux de données

```js
const { Readable, Writable, Transform, pipeline } = require('stream');
const { promisify } = require('util');
const pipe = promisify(pipeline);

// Lire un gros fichier en streaming
const fs = require('fs');
const lecture = fs.createReadStream('gros-fichier.csv');
const ecriture = fs.createWriteStream('sortie.txt');

await pipe(lecture, ecriture);

// Transform stream
const majuscule = new Transform({
  transform(chunk, encoding, callback) {
    callback(null, chunk.toString().toUpperCase());
  }
});
```

### `child_process` — Processus enfants

```js
const { exec, execSync, spawn } = require('child_process');

// Synchrone
const resultat = execSync('ls -la').toString();

// Asynchrone
exec('ls -la', (err, stdout, stderr) => {
  if (err) throw err;
  console.log(stdout);
});

// spawn (streaming, pour gros outputs)
const ls = spawn('ls', ['-la', '/usr']);
ls.stdout.on('data', (data) => console.log(data.toString()));
ls.on('close', (code) => console.log(`code: ${code}`));
```

### `util` — Utilitaires

```js
const util = require('util');

util.promisify(fs.readFile);   // convertit callback → promise
util.format('%s a %d ans', 'Alice', 30);
util.inspect(obj, { depth: null, colors: true });
util.isDeepStrictEqual(a, b);
```

---

## Variables d'environnement

```js
// Lire
process.env.NODE_ENV;    // 'development', 'production', ...
process.env.PORT;

// Fichier .env (avec dotenv)
// npm install dotenv
require('dotenv').config();
// ou avec import
import 'dotenv/config';

// .env
PORT=3000
DB_URL=mongodb://localhost:27017/madb
SECRET=monsecretsupersecret
```

---

## process — Infos sur le processus

```js
process.env;          // variables d'environnement
process.argv;         // arguments CLI : ['node', 'script.js', '--port', '3000']
process.cwd();        // répertoire courant
process.exit(0);      // quitter (0 = succès, 1 = erreur)
process.pid;          // PID du processus
process.version;      // version de Node
process.platform;     // 'linux', 'darwin', 'win32'
process.memoryUsage();
process.uptime();

// Écouter les signaux
process.on('SIGINT', () => {
  console.log('Arrêt propre...');
  process.exit(0);
});

process.on('uncaughtException', (err) => {
  console.error('Erreur non gérée:', err);
  process.exit(1);
});

process.on('unhandledRejection', (reason) => {
  console.error('Promise non gérée:', reason);
});
```

---

## Express — Framework web

```bash
npm install express
```

```js
const express = require('express');
const app = express();

// Middlewares essentiels
app.use(express.json());                        // parser JSON
app.use(express.urlencoded({ extended: true })); // parser form
app.use(express.static('public'));              // fichiers statiques

// Routes
app.get('/api/users', (req, res) => {
  res.json({ users: [] });
});

app.post('/api/users', (req, res) => {
  const { nom } = req.body;
  res.status(201).json({ nom });
});

app.put('/api/users/:id', (req, res) => {
  const { id } = req.params;
  res.json({ id, ...req.body });
});

app.delete('/api/users/:id', (req, res) => {
  res.status(204).send();
});

// Paramètres et query
req.params.id;           // /users/:id → { id: '42' }
req.query.page;          // /users?page=2
req.body;                // body JSON ou form
req.headers;
req.method;
req.url;

// Réponses
res.json({ data });
res.status(404).json({ erreur: 'Non trouvé' });
res.send('texte');
res.sendFile(path.join(__dirname, 'index.html'));
res.redirect('/autre-route');
res.set('Content-Type', 'text/plain');

// Router (organisation)
const router = express.Router();
router.get('/', handler);
router.post('/', handler);
app.use('/api/users', router);

// Middleware personnalisé
app.use((req, res, next) => {
  console.log(`${req.method} ${req.url}`);
  next(); // passer au suivant
});

// Gestion d'erreurs (4 paramètres obligatoires)
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ erreur: 'Erreur serveur' });
});

app.listen(3000, () => console.log('Serveur démarré sur le port 3000'));
```

---

## Connexion à une base de données

### MongoDB avec Mongoose

```bash
npm install mongoose
```

```js
const mongoose = require('mongoose');

await mongoose.connect(process.env.DB_URL);

const schemaUser = new mongoose.Schema({
  nom: { type: String, required: true },
  email: { type: String, unique: true },
  age: Number,
  createdAt: { type: Date, default: Date.now }
});

const User = mongoose.model('User', schemaUser);

// CRUD
const user = await User.create({ nom: 'Alice', email: 'alice@mail.com' });
const users = await User.find({ age: { $gte: 18 } });
const un = await User.findById(id);
await User.findByIdAndUpdate(id, { nom: 'Bob' }, { new: true });
await User.findByIdAndDelete(id);
await User.countDocuments({ age: { $gte: 18 } });
```

### PostgreSQL avec pg

```bash
npm install pg
```

```js
const { Pool } = require('pg');

const pool = new Pool({
  connectionString: process.env.DB_URL
});

const { rows } = await pool.query(
  'SELECT * FROM users WHERE id = $1',
  [userId]
);
```

---

## Authentification JWT

```bash
npm install jsonwebtoken bcrypt
```

```js
const jwt = require('jsonwebtoken');
const bcrypt = require('bcrypt');

// Hacher un mot de passe
const hash = await bcrypt.hash(motDePasse, 10);
const valide = await bcrypt.compare(motDePasse, hash);

// Créer un token
const token = jwt.sign(
  { userId: user._id, role: user.role },
  process.env.SECRET,
  { expiresIn: '7d' }
);

// Vérifier un token
const payload = jwt.verify(token, process.env.SECRET);

// Middleware d'authentification
function authMiddleware(req, res, next) {
  const token = req.headers.authorization?.split(' ')[1];
  if (!token) return res.status(401).json({ erreur: 'Non authentifié' });
  try {
    req.user = jwt.verify(token, process.env.SECRET);
    next();
  } catch {
    res.status(401).json({ erreur: 'Token invalide' });
  }
}
```

---

## Structure de projet recommandée

```
mon-projet/
├── src/
│   ├── config/
│   │   └── db.js
│   ├── controllers/
│   │   └── userController.js
│   ├── middleware/
│   │   └── auth.js
│   ├── models/
│   │   └── User.js
│   ├── routes/
│   │   └── users.js
│   └── app.js
├── .env
├── .gitignore
├── package.json
└── index.js
```

---

## Debugging et logs

```bash
# Mode debug natif
node --inspect index.js        # Chrome DevTools
node --inspect-brk index.js   # avec point d'arrêt au démarrage

# Variables utiles
NODE_ENV=development node app.js
DEBUG=express:* node app.js
```

```js
// Logger avec winston
const winston = require('winston');
const logger = winston.createLogger({
  level: 'info',
  transports: [
    new winston.transports.Console(),
    new winston.transports.File({ filename: 'app.log' })
  ]
});
```

---

## Conseils et bonnes pratiques

| Bonne pratique | Explication |
|---|---|
| Toujours utiliser `async/await` | Évite le callback hell |
| Centraliser la gestion d'erreurs | Middleware d'erreur Express |
| Ne jamais exposer les `.env` | Ajouter au `.gitignore` |
| Valider les entrées | `joi`, `zod` ou `express-validator` |
| Limiter les requêtes | `express-rate-limit` |
| Utiliser `helmet` | En-têtes de sécurité HTTP |
| Logger les erreurs | `winston` ou `pino` |
| Utiliser des variables d'env | Jamais de secrets en dur |