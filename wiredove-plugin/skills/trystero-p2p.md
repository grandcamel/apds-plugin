---
description: Use this skill when developers need to work with Trystero P2P connections in Wiredove - including room management, peer-to-peer messaging, WebRTC setup, and the gossip protocol. Trystero enables serverless communication via BitTorrent DHT.
---

# Trystero P2P in Wiredove

Trystero enables peer-to-peer WebRTC connections bootstrapped via BitTorrent DHT, allowing Wiredove to operate without central servers.

**Library**: trystero-torrent.min.js (bundled)
**External Docs**: https://github.com/dmotz/trystero

## Core Concepts

- **Rooms**: Virtual spaces where peers connect
- **Actions**: Named message channels within a room
- **Peers**: Connected clients identified by unique IDs
- **DHT Bootstrap**: BitTorrent DHT used for peer discovery

## Importing Trystero

```javascript
import { joinRoom } from './trystero-torrent.min.js'
```

## Creating a Room

```javascript
const room = joinRoom(
  {
    appId: 'wiredovetestnet',     // Unique app identifier
    password: 'shared-secret'     // Optional room password
  },
  roomId  // Room name (e.g., pubkey or topic)
)
```

## Creating Action Channels

Actions are bidirectional message channels:

```javascript
// Create a channel for hashes
const [sendHash, onHash] = room.makeAction('hash')

// Create a channel for blobs
const [sendBlob, onBlob] = room.makeAction('blob')

// Attach to room object for easy access
room.sendHash = sendHash
room.sendBlob = sendBlob
```

## Sending Messages

```javascript
// Send to all peers
sendBlob(data)

// Send to specific peer
sendBlob(data, peerId)

// Message types (by convention in Wiredove)
if (message.length === 44) {
  sendHash(message)   // It's a hash - request content
} else {
  sendBlob(message)   // It's content - share it
}
```

## Receiving Messages

```javascript
onHash(async (hash, peerId) => {
  console.log(`Received hash request: ${hash} from ${peerId}`)

  // Respond with content if we have it
  const content = await apds.get(hash)
  if (content) {
    sendBlob(content, peerId)
  }

  // Also send latest from that author
  const latest = await apds.getLatest(hash)
  if (latest) {
    sendBlob(latest.sig)
  }
})

onBlob(async (blob, peerId) => {
  console.log(`Received blob from ${peerId}`)

  // Process the received content
  await apds.make(blob)
  await apds.add(blob)
  await render.blob(blob)
})
```

## Peer Lifecycle Events

```javascript
room.onPeerJoin(async (peerId) => {
  console.log(`${peerId} joined the room`)

  // Share our latest message with new peer
  const latest = await apds.getLatest(await apds.pubkey())
  if (latest) {
    sendBlob(latest.sig)
  }
})

room.onPeerLeave((peerId) => {
  console.log(`${peerId} left the room`)
})
```

## Complete Room Setup (from gossip.js)

```javascript
import { apds } from 'apds'
import { joinRoom } from './trystero-torrent.min.js'
import { render } from './render.js'
import { noteReceived, registerNetworkSenders } from './network_queue.js'

export let chan  // Global room reference

let roomReadyResolver
export let roomReady = new Promise(resolve => {
  roomReadyResolver = resolve
})

export const hasRoom = () => !!chan

export const sendTry = (m) => {
  if (chan) {
    if (m.length === 44) {
      chan.sendHash(m)
    } else {
      chan.sendBlob(m)
    }
  }
}

// Register with network queue
registerNetworkSenders({
  sendGossip: sendTry,
  hasGossip: hasRoom
})

export const makeRoom = async (pubkey) => {
  if (!chan) {
    const room = joinRoom({
      appId: 'wiredovetestnet',
      password: 'iajwoiejfaowiejfoiwajfe'
    }, pubkey)

    const [sendHash, onHash] = room.makeAction('hash')
    const [sendBlob, onBlob] = room.makeAction('blob')

    room.sendHash = sendHash
    room.sendBlob = sendBlob

    onHash(async (hash, id) => {
      noteReceived(hash)
      const get = await apds.get(hash)
      if (get) { sendBlob(get, id) }
      const latest = await apds.getLatest(hash)
      if (latest) { sendBlob(latest.sig) }
    })

    onBlob(async (blob, id) => {
      noteReceived(blob)
      await apds.make(blob)
      await render.shouldWe(blob)
      await apds.add(blob)
      await render.blob(blob)
    })

    room.onPeerJoin(async (id) => {
      console.log(id + ' joined the room ' + pubkey)
      const latest = await apds.getLatest(await apds.pubkey())
      if (latest) { sendBlob(latest.sig) }
    })

    room.onPeerLeave(id => {
      console.log(id + ' left the room ' + pubkey)
    })

    chan = room
    roomReadyResolver?.()
  }

  return roomReady
}
```

## Room Patterns

### Single Global Room
Wiredove uses a single room for all messages:

```javascript
// In connect.js
makeRoom('wiredovev1')  // Global room ID
```

### Per-Author Rooms (Alternative)
For more targeted sync:

```javascript
// Create room per author pubkey
for (const pubkey of pubkeys) {
  await makeRoom(pubkey)
}
```

## Integration with Network Queue

Trystero is one of two transport layers (along with WebSocket):

```javascript
import { registerNetworkSenders } from './network_queue.js'

registerNetworkSenders({
  sendGossip: sendTry,   // Function to send via Trystero
  hasGossip: hasRoom     // Function to check if connected
})
```

Then messages can be sent via both:

```javascript
import { queueSend } from './network_queue.js'

queueSend(message, 'both')    // WebSocket + Trystero
queueSend(message, 'gossip')  // Trystero only
```

## Debugging Tips

1. **Check room connection**: `console.log(hasRoom())`
2. **Monitor peer events**: Add logging to onPeerJoin/Leave
3. **Verify DHT connectivity**: Trystero uses BitTorrent DHT - firewall/NAT issues can prevent connections
4. **Test locally**: Run multiple browser tabs to test P2P

## Key Files in Wiredove

- `gossip.js` - Main Trystero room management
- `connect.js:11` - Room initialization
- `network_queue.js:26-31` - Transport registration
- `trystero-torrent.min.js` - Bundled Trystero library
