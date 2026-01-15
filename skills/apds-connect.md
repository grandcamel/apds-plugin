---
description: Help manage APDS peer connections - WebSocket setup, peer discovery, connection handling, and message sync
user_invocable: true
---

# APDS Peer Connection Management

You are helping developers manage peer connections in APDS. APDS uses WebSockets for real-time P2P message synchronization.

## Connection Architecture

```
┌─────────────┐     WebSocket      ┌─────────────┐
│   Client    │◄──────────────────►│   Server    │
│  (cli.js)   │                    │ (serve.js)  │
└─────────────┘                    └─────────────┘
                                          │
                                          │ Gossip
                                          ▼
                                   ┌─────────────┐
                                   │ Other Peers │
                                   └─────────────┘
```

## Known APDS Endpoints

| Endpoint | Type | Description |
|----------|------|-------------|
| `wss://apds.anproto.com` | Server | Public ANProto server |
| `ws://localhost:9000` | Server | Local development server |
| `ws://localhost:8080` | Relay | Local relay server |

## Connecting to Peers

### CLI Connection
```bash
# In cli.js
/connect wss://apds.anproto.com
```

### Programmatic Connection
```javascript
const sockets = new Set()

function connectToPeer(url) {
  const ws = new WebSocket(url)

  ws.onopen = () => {
    console.log('Connected to:', url)
    sockets.add(ws)
  }

  ws.onmessage = async (event) => {
    const data = event.data

    // Hash request (44 chars) - send back the data
    if (data.length === 44) {
      const stored = await apds.get(data)
      if (stored) {
        ws.send(stored)
      }
    }
    // Message data - store it
    else {
      await apds.make(data)
      await apds.add(data)
    }
  }

  ws.onclose = () => {
    console.log('Disconnected from:', url)
    sockets.delete(ws)
  }

  ws.onerror = (err) => {
    console.error('Connection error:', err)
  }

  return ws
}
```

## Message Sync Protocol

### How Sync Works

1. **Hash Exchange**: Peers exchange 44-character hashes
2. **Data Request**: If hash unknown, peer requests full data
3. **Data Response**: Peer sends the full message/blob
4. **Gossip Queue**: Missing hashes are tracked and re-requested

### Sync Flow
```
Peer A                              Peer B
  │                                    │
  │──── send hash (44 chars) ─────────►│
  │                                    │ Check: do I have this?
  │◄─── send full data if missing ─────│
  │                                    │
  │ Store & verify signature           │
  │                                    │
```

### Request Specific Data
```javascript
// Request a specific hash from peers
function requestFromPeers(hash) {
  sockets.forEach(ws => {
    if (ws.readyState === WebSocket.OPEN) {
      ws.send(hash)  // 44-char hash triggers data response
    }
  })
}
```

## Gossip Protocol

The gossip system handles missing data automatically:

```javascript
import { createGossip } from './gossip.js'

const gossipQueue = createGossip({
  // Function to get current peers
  getPeers: () => sockets,

  // Check if we have a hash locally
  has: async (hash) => !!(await apds.get(hash)),

  // Request hash from a peer
  request: (peer, hash) => {
    if (peer.readyState === WebSocket.OPEN) {
      peer.send(hash)
    }
  },

  // How often to retry missing hashes
  intervalMs: 10000
})

// Start the gossip loop
gossipQueue.start()

// When data arrives, mark hash as resolved
ws.onmessage = async (event) => {
  const hash = await apds.make(event.data)
  gossipQueue.resolve(hash)
}

// When we discover missing data, enqueue it
gossipQueue.enqueue(missingHash)
```

## Connection Management

### Track Active Connections
```javascript
const connections = new Map()

function addConnection(url) {
  if (connections.has(url)) {
    console.log('Already connected to:', url)
    return connections.get(url)
  }

  const ws = new WebSocket(url)

  ws.onopen = () => {
    connections.set(url, ws)
    console.log('Connected:', url)
    console.log('Active connections:', connections.size)
  }

  ws.onclose = () => {
    connections.delete(url)
    console.log('Disconnected:', url)
  }

  return ws
}

function removeConnection(url) {
  const ws = connections.get(url)
  if (ws) {
    ws.close()
    connections.delete(url)
  }
}

function listConnections() {
  return Array.from(connections.keys())
}
```

### Auto-Reconnect
```javascript
function connectWithRetry(url, maxRetries = 5) {
  let retries = 0

  function attempt() {
    const ws = new WebSocket(url)

    ws.onopen = () => {
      retries = 0  // Reset on success
      sockets.add(ws)
    }

    ws.onclose = () => {
      sockets.delete(ws)
      if (retries < maxRetries) {
        retries++
        const delay = Math.min(1000 * Math.pow(2, retries), 30000)
        console.log(`Reconnecting in ${delay}ms (attempt ${retries})`)
        setTimeout(attempt, delay)
      }
    }
  }

  attempt()
}
```

## Broadcasting Messages

### Send to All Peers
```javascript
function broadcast(data) {
  sockets.forEach(ws => {
    if (ws.readyState === WebSocket.OPEN) {
      ws.send(data)
    }
  })
}

// After composing a message, broadcast it
const hash = await apds.compose('Hello peers!')
const msg = await apds.get(hash)
const opened = await apds.open(msg)
const blob = await apds.get(opened.substring(13))

// Send all parts
broadcast(blob)  // Content
broadcast(msg)   // Signature
broadcast(hash)  // Hash
```

### Sync Full History
```javascript
async function syncAllMessages() {
  const log = await apds.getHashLog()

  for (const hash of log) {
    broadcast(hash)  // Peers will request data if needed
  }
}
```

## Server-Side Connections (serve.js)

The server accepts incoming WebSocket connections:

```javascript
Deno.serve({ port: 9000 }, async (request) => {
  // Upgrade HTTP to WebSocket
  const { socket, response } = Deno.upgradeWebSocket(request)

  socket.onopen = () => {
    sockets.add(socket)
    console.log('Client connected')
  }

  socket.onmessage = async (event) => {
    // Handle incoming data...
  }

  socket.onclose = () => {
    sockets.delete(socket)
    console.log('Client disconnected')
  }

  return response
})
```

## Troubleshooting Connections

### Connection Refused
```bash
# Check if server is running
curl http://localhost:9000/all

# Check port availability
lsof -i :9000
```

### WebSocket Errors
```javascript
ws.onerror = (error) => {
  console.error('WebSocket error:', error)
  // Common causes:
  // - Server not running
  // - Wrong URL/port
  // - SSL certificate issues (wss://)
  // - CORS restrictions (browser)
}
```

### Messages Not Syncing
1. Check connection state: `ws.readyState === WebSocket.OPEN`
2. Verify gossip queue is running: `gossipQueue.start()`
3. Check if hash is in missing set
4. Verify message signatures are valid

### SSL/TLS Issues
```javascript
// For local development without SSL
const ws = new WebSocket('ws://localhost:9000')

// For production with SSL
const ws = new WebSocket('wss://apds.anproto.com')
```

## Best Practices

1. **Track connection state** - Always check `readyState` before sending
2. **Handle disconnections** - Implement reconnection logic
3. **Use gossip** - Let the gossip queue handle missing data
4. **Limit connections** - Don't open too many simultaneous connections
5. **Clean up** - Remove closed connections from sets/maps
6. **Timeout requests** - Don't wait forever for missing data
