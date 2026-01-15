---
description: Manage APDS peer connections - list, add, remove, and test connectivity to peers
user_invocable: true
arguments:
  - name: action
    description: "Action: list, add, remove, test, or discover"
    required: false
    default: "list"
  - name: peer_url
    description: "Peer WebSocket URL (for add/remove/test)"
    required: false
---

# APDS Peer Management

Manage WebSocket peer connections for APDS message synchronization.

## Usage

```
/apds-peers [action] [peer_url]
```

## Actions

### list (default)
Show all known and connected peers:

```
Known Peers:
  1. wss://apds.anproto.com     [connected]
  2. ws://localhost:8080         [disconnected]
  3. wss://peer.example.com      [connected]

Active Connections: 2/3
```

### add <url>
Add a new peer to the connection list:

```
> /apds-peers add wss://new-peer.example.com

Adding peer: wss://new-peer.example.com
  ✓ URL valid
  ✓ Connection established
  ✓ Peer added to list
  ✓ Initial sync started

Peer added successfully.
```

### remove <url>
Remove a peer from the connection list:

```
> /apds-peers remove ws://localhost:8080

Removing peer: ws://localhost:8080
  ✓ Connection closed
  ✓ Removed from peer list

Peer removed.
```

### test <url>
Test connectivity to a specific peer:

```
> /apds-peers test wss://apds.anproto.com

Testing: wss://apds.anproto.com

Connection:
  ✓ WebSocket handshake (45ms)
  ✓ Connection established

Protocol:
  ✓ Sent pubkey request
  ✓ Received response (3 messages)
  ✓ Message signatures valid

Latency:
  → Round-trip: 89ms
  → Messages/sec: ~33

Peer is healthy.
```

### discover
Discover peers from connected nodes:

```
> /apds-peers discover

Querying connected peers for their peer lists...

Discovered peers:
  1. wss://apds.anproto.com     (already connected)
  2. wss://relay.anproto.net    (new)
  3. wss://peer3.example.com    (new)

Add new peers? [Y/n]
```

## Implementation

### Peer Storage

Store peer list in a configuration file:

```json
// peers.json
{
  "peers": [
    {
      "url": "wss://apds.anproto.com",
      "added": "2026-01-15T00:00:00Z",
      "lastSeen": "2026-01-15T12:34:56Z",
      "messageCount": 1234,
      "autoConnect": true
    }
  ],
  "defaultPeers": [
    "wss://apds.anproto.com"
  ]
}
```

### Connection Logic

```javascript
// Peer manager
class PeerManager {
  constructor() {
    this.connections = new Map()
    this.peers = this.loadPeers()
  }

  loadPeers() {
    try {
      return JSON.parse(Deno.readTextFileSync('peers.json'))
    } catch {
      return { peers: [], defaultPeers: ['wss://apds.anproto.com'] }
    }
  }

  savePeers() {
    Deno.writeTextFileSync('peers.json', JSON.stringify(this.peers, null, 2))
  }

  async connect(url) {
    if (this.connections.has(url)) {
      return this.connections.get(url)
    }

    const ws = new WebSocket(url)

    return new Promise((resolve, reject) => {
      ws.onopen = () => {
        this.connections.set(url, ws)
        this.updateLastSeen(url)
        resolve(ws)
      }
      ws.onerror = reject
      ws.onclose = () => this.connections.delete(url)
    })
  }

  disconnect(url) {
    const ws = this.connections.get(url)
    if (ws) {
      ws.close()
      this.connections.delete(url)
    }
  }

  list() {
    return this.peers.peers.map(peer => ({
      ...peer,
      connected: this.connections.has(peer.url)
    }))
  }

  async add(url) {
    // Validate URL
    if (!url.startsWith('ws://') && !url.startsWith('wss://')) {
      throw new Error('Invalid WebSocket URL')
    }

    // Test connection
    await this.connect(url)

    // Add to list if not exists
    if (!this.peers.peers.find(p => p.url === url)) {
      this.peers.peers.push({
        url,
        added: new Date().toISOString(),
        lastSeen: new Date().toISOString(),
        messageCount: 0,
        autoConnect: true
      })
      this.savePeers()
    }
  }

  remove(url) {
    this.disconnect(url)
    this.peers.peers = this.peers.peers.filter(p => p.url !== url)
    this.savePeers()
  }

  updateLastSeen(url) {
    const peer = this.peers.peers.find(p => p.url === url)
    if (peer) {
      peer.lastSeen = new Date().toISOString()
      this.savePeers()
    }
  }
}
```

### Testing a Peer

```javascript
async function testPeer(url) {
  const results = {
    connection: false,
    handshake: null,
    protocol: false,
    latency: null
  }

  const start = Date.now()

  try {
    const ws = new WebSocket(url)

    await new Promise((resolve, reject) => {
      ws.onopen = () => {
        results.connection = true
        results.handshake = Date.now() - start
        resolve()
      }
      ws.onerror = reject
      setTimeout(() => reject(new Error('Timeout')), 5000)
    })

    // Test protocol
    const msgStart = Date.now()
    let received = 0

    ws.onmessage = () => received++

    // Send pubkey request
    ws.send(await apds.pubkey())

    await new Promise(r => setTimeout(r, 2000))

    results.protocol = received > 0
    results.latency = (Date.now() - msgStart) / Math.max(received, 1)
    results.messagesReceived = received

    ws.close()
  } catch (err) {
    results.error = err.message
  }

  return results
}
```

## Well-Known Peers

Public APDS peers for the ANProto network:

| URL | Description |
|-----|-------------|
| `wss://apds.anproto.com` | Official ANProto server |

## Example Sessions

### List peers
```
> /apds-peers

Known Peers (3):
┌────┬──────────────────────────────┬────────────┬──────────────┐
│ #  │ URL                          │ Status     │ Last Seen    │
├────┼──────────────────────────────┼────────────┼──────────────┤
│ 1  │ wss://apds.anproto.com       │ connected  │ just now     │
│ 2  │ ws://localhost:8080          │ offline    │ 2 hours ago  │
│ 3  │ wss://peer.example.com       │ connected  │ 5 mins ago   │
└────┴──────────────────────────────┴────────────┴──────────────┘

Connected: 2/3 | Messages synced: 1,234
```

### Add peer
```
> /apds-peers add wss://new-relay.example.com

Connecting to wss://new-relay.example.com...
  ✓ Connection established (67ms)
  ✓ Protocol handshake OK
  ✓ Added to peer list

Syncing messages...
  → Received 45 new messages

Peer added and synced.
```

### Test connectivity
```
> /apds-peers test wss://apds.anproto.com

Testing wss://apds.anproto.com...

Results:
  Connection:     ✓ OK (34ms)
  Protocol:       ✓ OK
  Messages:       47 received in 2s
  Avg Latency:    42ms

Status: Healthy
```
