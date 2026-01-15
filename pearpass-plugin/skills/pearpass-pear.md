---
description: "Use this skill when working with Pear Runtime in PearPass Desktop, including worklet communication, pear-run, pear-electron, staging, deployment, or P2P networking."
invocable: true
---

# PearPass Pear Runtime Guide

You are helping a developer understand and work with Pear Runtime in PearPass Desktop.

## What is Pear Runtime?

Pear Runtime is a distributed P2P application platform that provides:
- **pear-electron** - Desktop GUI framework (Electron-based)
- **pear-run** - Isolated worklet execution
- **hyperswarm** - P2P networking
- **hyperdht** - Distributed hash table
- **corestore** - Distributed data storage

## Pear Configuration

From `package.json`:

```json
{
  "pear": {
    "name": "pearpass-app-desktop",
    "pre": "pear-electron/pre",
    "routes": ".",
    "unrouted": [
      "/node_modules/pearpass-lib-vault-core/src/worklet/app.js"
    ],
    "gui": {
      "backgroundColor": "#1F2430",
      "height": "1024",
      "width": "1440"
    },
    "links": [
      "https://hooks.slack.com/..."
    ],
    "stage": {
      "entrypoints": [
        "node_modules/pearpass-lib-vault-core/src/worklet/app.js"
      ],
      "ignore": [
        ".github", "appling", ".git", ".gitignore",
        "packages", "src"
      ]
    }
  }
}
```

### Key Configuration

| Field | Purpose |
|-------|---------|
| `pre` | Pre-bundle for Electron |
| `routes` | URL routing root |
| `unrouted` | Files excluded from URL routing |
| `gui.backgroundColor` | Window background color |
| `gui.height/width` | Window dimensions |
| `stage.entrypoints` | Files to bundle for staging |
| `stage.ignore` | Directories excluded from staging |

## Pear Global Object

Available in both main app and worklet:

```javascript
/** @typedef {import('pear-interface')} */
/* global Pear */

// Check if in production
const isProduction = !!Pear.config.key

// App link for production paths
const applink = Pear.config.applink

// Storage path
const storage = Pear.config.storage

// Tier (dev/prod)
const tier = Pear.config.tier

// Register teardown callback
Pear.teardown(() => {
  cleanup()
})
```

## Worklet Pattern

### Creating the Pipe

From `src/services/createOrGetPipe.js`:

```javascript
import pearRun from 'pear-run'

let pipe

const WORKLET_PATH_DEV = './node_modules/pearpass-lib-vault-core/src/worklet/app.js'
const WORKLET_PATH_PROD = Pear.config.applink + '/node_modules/pearpass-lib-vault-core/src/worklet/app.js'

export const createOrGetPipe = () => {
  if (pipe) return pipe

  // Start worklet in isolated process
  pipe = pearRun(Pear.config.key ? WORKLET_PATH_PROD : WORKLET_PATH_DEV)

  // Clean up on app teardown
  Pear.teardown(() => {
    if (pipe) {
      pipe.end()
      pipe = null
    }
  })

  return pipe
}
```

### Creating the Client

From `src/services/createOrGetPearpassClient.js`:

```javascript
import { PearpassVaultClient } from 'pearpass-lib-vault-core'

let client

export const createOrGetPearpassClient = (pipe, storage, options) => {
  if (client) return client

  client = new PearpassVaultClient(pipe, storage, options)
  return client
}
```

### Usage in App

From `app.tsx`:

```javascript
const storage = Pear.config.storage

const pipe = createOrGetPipe()
const client = createOrGetPearpassClient(pipe, storage, {
  debugMode: false
})

setPearpassVaultClient(client)
```

## Worklet Structure

The worklet runs in `packages/pearpass-lib-vault-core/src/worklet/`:

```
worklet/
├── app.js              # Worklet entry point
├── appCore.js          # Core application logic
├── appDeps.js          # Dependencies injection
├── hashPassword.js     # Password hashing
├── encryptVaultKeyWithHashedPassword.js
├── decryptVaultKey.js
├── getDecryptionKey.js
├── pearpassPairer.js   # Device pairing
├── rateLimiter.js      # Rate limiting
├── validateAndSanitizePath.js
├── getForbiddenRoots.js
└── utils/
    ├── swarm.js        # Hyperswarm utilities
    ├── dht.js          # HyperDHT utilities
    ├── isPearWorker.js
    ├── workletLogger.js
    └── parseRequestData.js
```

## Development Workflow

### Running in Development

```bash
# Build TypeScript and run with Pear
npm run dev

# This runs:
# tsc && concurrently "tsc --watch" "pear run -d ."
```

### Direct Pear Commands

```bash
# Run in development mode
pear run --dev .

# Run with debug output
pear run -d .
```

## Staging and Deployment

### Build for Staging

```bash
# Full build (i18n + TypeScript)
npm run build:pear

# Stage the app
pear stage dev
```

### Run Staged Version

```bash
# After staging, you get a pear:// URL
pear run pear://GENERATED_KEY
```

### Important Notes

- Production serves from `dist/` directory
- Source `src/` is in `stage.ignore`
- Worklet path changes between dev and prod
- Use `Pear.config.key` to detect environment

## P2P Networking

### Hyperswarm (from vault-core)

```javascript
import Hyperswarm from 'hyperswarm'

const swarm = new Hyperswarm()

// Join a topic
const topic = Buffer.from('my-topic-hash', 'hex')
const discovery = swarm.join(topic)

// Handle connections
swarm.on('connection', (socket, info) => {
  // socket is a duplex stream
  socket.pipe(processor).pipe(socket)
})
```

### HyperDHT

```javascript
import DHT from 'hyperdht'

const dht = new DHT()

// Create server
const server = dht.createServer()
server.listen(keyPair)

// Connect to peer
const socket = dht.connect(publicKey)
```

## Storage (Corestore)

From vault-core, uses corestore for distributed storage:

```javascript
import Corestore from 'corestore'

const store = new Corestore(storagePath)

// Get a hypercore
const core = store.get({ name: 'vault' })
await core.ready()

// Append data
await core.append(encryptedData)

// Read data
const data = await core.get(index)
```

## Common Issues

### Worklet Not Starting

```bash
# Check if worklet path exists
ls -la node_modules/pearpass-lib-vault-core/src/worklet/app.js

# Ensure submodules initialized
npm run update-submodules
```

### Production Path Issues

```javascript
// Always use Pear.config.key to detect environment
const workletPath = Pear.config.key
  ? Pear.config.applink + '/path/to/worklet.js'
  : './path/to/worklet.js'
```

### Teardown Not Called

```javascript
// Always register teardown for cleanup
Pear.teardown(() => {
  pipe?.end()
  swarm?.destroy()
  store?.close()
})
```

## Debugging

### Enable Debug Mode

```javascript
const client = createOrGetPearpassClient(pipe, storage, {
  debugMode: true  // Enables verbose logging
})
```

### Worklet Logging

From `workletLogger.js`:
```javascript
import { logger } from './utils/workletLogger'

logger.info('COMPONENT', 'Message', data)
logger.error('COMPONENT', 'Error', error)
```

## Key Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| pear-electron | 1.7.25-rc.0 | Desktop GUI |
| pear-run | 1.0.5 | Worklet runner |
| pear-bridge | 1.2.4 | Bridge utilities |
| pear-ipc | 6.4.0 | IPC communication |
| hyperswarm | 4.16.0 | P2P networking |
| hyperdht | 6.27.0 | DHT networking |
| corestore | 7.6.1 | Distributed storage |
| rocksdb-native | 3.11.2 | Local storage backend |

## Resources

- [Pear Runtime Docs](https://docs.pears.com/guides)
- [Hyperswarm GitHub](https://github.com/hyperswarm/hyperswarm)
- [Corestore GitHub](https://github.com/holepunchto/corestore)
