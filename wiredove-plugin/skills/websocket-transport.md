---
description: Use this skill when developers need to work with WebSocket connections in Wiredove - including pub server communication, connection management, reconnection logic, and message exchange. WebSocket provides reliable server-backed sync.
---

# WebSocket Transport in Wiredove

WebSocket connections provide reliable synchronization with pub servers, complementing the P2P Trystero layer.

## Overview

- **Purpose**: Persistent connections to pub servers for reliable sync
- **Default Server**: `wss://pub.wiredove.net/`
- **Role**: Primary sync mechanism when P2P peers unavailable

## Connection Setup

### Basic Connection

```javascript
import { apds } from 'apds'
import { render } from './render.js'
import { noteReceived, registerNetworkSenders } from './network_queue.js'

const pubs = new Set()  // Active WebSocket connections

export const makeWs = async (pub) => {
  const ws = new WebSocket(pub)

  ws.onopen = async () => {
    console.log('OPEN')
    pubs.add(ws)

    // Send our known pubkeys and latest messages
    const pubkeys = await apds.getPubkeys() || []
    for (const pubkey of pubkeys) {
      ws.send(pubkey)
      const latest = await apds.getLatest(pubkey)
      if (latest) {
        if (latest.text) {
          ws.send(latest.text)
        } else {
          const blob = await apds.get(latest.opened.substring(13))
          if (blob) ws.send(blob)
        }
        ws.send(latest.sig)
      }
    }
  }

  ws.onmessage = async (m) => {
    noteReceived(m.data)

    if (m.data.length === 44) {
      // It's a hash - respond with content
      const blob = await apds.get(m.data)
      if (blob) ws.send(blob)
    } else {
      // It's content - process it
      await render.shouldWe(m.data)
      await apds.make(m.data)
      await apds.add(m.data)
      await render.blob(m.data)
    }
  }

  ws.onclose = async () => {
    console.log('CLOSED')
    pubs.delete(ws)
    scheduleReconnect()
  }

  ws.onerror = () => {
    scheduleReconnect()
  }
}
```

## Reconnection with Exponential Backoff

```javascript
const wsBackoff = new Map()

const getBackoff = (pub) => {
  let state = wsBackoff.get(pub)
  if (!state) {
    state = { delayMs: 1000, timer: null }
    wsBackoff.set(pub, state)
  }
  return state
}

const scheduleReconnect = (pub, connectFn) => {
  const state = getBackoff(pub)
  if (state.timer) return  // Already scheduled

  state.timer = setTimeout(() => {
    state.timer = null
    connectFn()
    state.delayMs *= 2  // Double delay for next time
  }, state.delayMs)
}

const resetBackoff = (pub) => {
  const state = getBackoff(pub)
  if (state.timer) {
    clearTimeout(state.timer)
    state.timer = null
  }
  state.delayMs = 1000  // Reset to 1 second
}
```

## Integration with Network Queue

```javascript
// Send to all active WebSocket connections
const deliverWs = (msg) => {
  pubs.forEach(pub => {
    pub.send(msg)
  })
}

export const sendWs = async (msg) => {
  if (pubs.size) deliverWs(msg)
}

export const hasWs = () => pubs.size > 0

// Register with network queue
registerNetworkSenders({
  sendWs,
  hasWs
})
```

## Ready Promise Pattern

```javascript
let wsReadyResolver
export let wsReady = new Promise(resolve => {
  wsReadyResolver = resolve
})

// In onopen handler
ws.onopen = async () => {
  pubs.add(ws)
  resetBackoff(pub)
  wsReadyResolver?.()  // Resolve ready promise
  // ...
}

// In onclose handler
ws.onclose = async () => {
  pubs.delete(ws)
  if (!pubs.size) {
    // Recreate promise for next connection
    wsReady = new Promise(resolve => {
      wsReadyResolver = resolve
    })
  }
  scheduleReconnect()
}

// Usage
export const makeWs = async (pub) => {
  connectWs()
  return wsReady
}
```

## Message Protocol

### Message Types

| Length | Type | Action |
|--------|------|--------|
| 44 chars | Hash (pubkey or content) | Request content |
| > 44 chars | Blob (signed msg or content) | Process/store |

### Request Flow

```
Client                    Server
  |                         |
  |-- pubkey (44 chars) --> |  Request latest from author
  |                         |
  |<-- signed message ----  |  Server sends latest
  |<-- content blob ------  |  Server sends content
  |                         |
```

### Response Flow

```
Client                    Server
  |                         |
  |<-- hash (44 chars) ---  |  Server requests content
  |                         |
  |-- blob ------------->   |  Client sends if available
  |                         |
```

## Full WebSocket Module (websocket.js)

```javascript
import { apds } from 'apds'
import { render } from './render.js'
import { noteReceived, registerNetworkSenders } from './network_queue.js'

const pubs = new Set()
const wsBackoff = new Map()

let wsReadyResolver
const createWsReadyPromise = () => new Promise(resolve => {
  wsReadyResolver = resolve
})
export let wsReady = createWsReadyPromise()

const deliverWs = (msg) => {
  pubs.forEach(pub => pub.send(msg))
}

export const sendWs = async (msg) => {
  if (pubs.size) deliverWs(msg)
}

export const hasWs = () => pubs.size > 0

registerNetworkSenders({ sendWs, hasWs })

export const makeWs = async (pub) => {
  const getBackoff = () => {
    let state = wsBackoff.get(pub)
    if (!state) {
      state = { delayMs: 1000, timer: null }
      wsBackoff.set(pub, state)
    }
    return state
  }

  const scheduleReconnect = () => {
    const state = getBackoff()
    if (state.timer) return
    state.timer = setTimeout(() => {
      state.timer = null
      connectWs()
      state.delayMs *= 2
    }, state.delayMs)
  }

  const resetBackoff = () => {
    const state = getBackoff()
    if (state.timer) {
      clearTimeout(state.timer)
      state.timer = null
    }
    state.delayMs = 1000
  }

  const connectWs = () => {
    const ws = new WebSocket(pub)

    ws.onopen = async () => {
      console.log('OPEN')
      pubs.add(ws)
      resetBackoff()
      wsReadyResolver?.()

      const p = await apds.getPubkeys() || []
      for (const pub of p) {
        ws.send(pub)
        const latest = await apds.getLatest(pub)
        if (latest.text) {
          ws.send(latest.text)
        } else {
          const blob = await apds.get(latest.opened.substring(13))
          if (blob) ws.send(blob)
        }
        ws.send(latest.sig)
      }
    }

    ws.onmessage = async (m) => {
      noteReceived(m.data)
      if (m.data.length === 44) {
        const blob = await apds.get(m.data)
        if (blob) ws.send(blob)
      } else {
        await render.shouldWe(m.data)
        await apds.make(m.data)
        await apds.add(m.data)
        await render.blob(m.data)
      }
    }

    ws.onerror = () => scheduleReconnect()

    ws.onclose = async () => {
      console.log('CLOSED')
      pubs.delete(ws)
      if (!pubs.size) {
        wsReady = createWsReadyPromise()
      }
      scheduleReconnect()
    }
  }

  connectWs()
  return wsReady
}
```

## Pub Server

The pub server at `pub.wiredove.net` provides:
- Message storage and retrieval
- Directory services (username to pubkey mapping)
- Push everything/pull everything endpoints

Server code: https://github.com/evbogue/dovepub

## Key Files

- `websocket.js` - WebSocket transport implementation
- `connect.js:10` - WebSocket initialization
- `network_queue.js:26-29` - Transport registration
- `settings.js:74-112` - Push/pull everything via WebSocket
