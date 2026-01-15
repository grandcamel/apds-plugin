---
description: Use this skill when developers need to understand the feed synchronization system in Wiredove - including tiered sync, activity tracking, network queue management, and sync scheduling. Covers how messages are synchronized across peers.
---

# Feed Synchronization in Wiredove

Wiredove uses a tiered synchronization system that prioritizes active content while still syncing older messages.

## Sync Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                     sync.js (Scheduler)                      │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                  │
│  │   HOT    │  │   WARM   │  │   COLD   │                  │
│  │ 4/tick   │  │ 2/tick   │  │ 1/tick   │                  │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘                  │
│       │             │             │                         │
│       └─────────────┴─────────────┘                         │
│                     │                                        │
└─────────────────────┼────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│                 network_queue.js                             │
│  ┌──────────────────────────────────────────────────────┐   │
│  │                   Queue                               │   │
│  │  [msg1] [msg2] [msg3] ...                            │   │
│  └──────────────────────────────────────────────────────┘   │
│                      │                                       │
│         ┌────────────┴────────────┐                         │
│         ▼                         ▼                         │
│  ┌─────────────┐          ┌─────────────┐                  │
│  │  WebSocket  │          │   Trystero  │                  │
│  │  (ws)       │          │   (gossip)  │                  │
│  └─────────────┘          └─────────────┘                  │
└─────────────────────────────────────────────────────────────┘
```

## Tiered Sync System

### Tier Classification

```javascript
// Time thresholds
const HOT_INTEREST_MS = 15 * 60 * 1000      // 15 minutes
const HOT_SEEN_MS = 24 * 60 * 60 * 1000     // 24 hours
const WARM_INTEREST_MS = 24 * 60 * 60 * 1000 // 24 hours
const WARM_SEEN_MS = 30 * 24 * 60 * 60 * 1000 // 30 days

// Classification logic
const classify = (entry, now) => {
  const seenAge = entry.lastSeen ? now - entry.lastSeen : Infinity
  const interestAge = entry.lastInterest ? now - entry.lastInterest : Infinity

  // HOT: Recently viewed or recently active
  if (interestAge <= HOT_INTEREST_MS || seenAge <= HOT_SEEN_MS) {
    return 'hot'
  }

  // WARM: Somewhat recent activity
  if (interestAge <= WARM_INTEREST_MS || seenAge <= WARM_SEEN_MS) {
    return 'warm'
  }

  // COLD: Everything else
  return 'cold'
}
```

### Sync Rates

| Tier | Messages per Tick | Min Request Interval |
|------|-------------------|---------------------|
| HOT  | 4                 | 15 seconds          |
| WARM | 2                 | 20 minutes          |
| COLD | 1                 | 6 hours             |

### Activity Tracking

```javascript
const activity = new Map()

// Entry structure
const getEntry = (pubkey) => {
  if (activity.has(pubkey)) return activity.get(pubkey)

  const entry = {
    lastSeen: 0,       // Timestamp of last message from this author
    lastInterest: 0,   // Timestamp when user viewed this author
    lastRequested: 0   // Timestamp of last sync request
  }
  activity.set(pubkey, entry)
  return entry
}

// Mark when we receive a message from an author
export const noteSeen = (pubkey) => {
  const entry = getEntry(pubkey)
  entry.lastSeen = Date.now()
  needsRebuild = true
}

// Mark when user views an author's profile/messages
export const noteInterest = (pubkey) => {
  const entry = getEntry(pubkey)
  entry.lastInterest = Date.now()
  needsRebuild = true
}
```

## Sync Scheduler

### Starting the Sync Loop

```javascript
const TICK_MS = 15 * 1000      // 15 seconds between ticks
const REFRESH_MS = 5 * 60 * 1000 // 5 minutes between pubkey refreshes

let syncTimer = null
let pubkeys = []
let tiered = { hot: [], warm: [], cold: [] }

export const startSync = async (sendFn) => {
  if (syncTimer) return

  await refreshPubkeys()

  syncTimer = setInterval(async () => {
    const now = Date.now()

    // Refresh pubkey list periodically
    if (now - lastRefresh > REFRESH_MS) {
      await refreshPubkeys()
    }

    // Rebuild tier classification if needed
    if (needsRebuild) {
      rebuildTiers()
    }

    // Pick candidates from each tier
    const hot = pickCandidates('hot', BATCH.hot, now)
    const warm = pickCandidates('warm', BATCH.warm, now)
    const cold = pickCandidates('cold', BATCH.cold, now)

    // Send sync requests
    const batch = [...hot, ...warm, ...cold]
    batch.forEach(pubkey => {
      const entry = getEntry(pubkey)
      entry.lastRequested = now
      sendFn(pubkey)
    })

  }, TICK_MS)
}
```

### Picking Candidates

```javascript
const pickCandidates = (tier, count, now) => {
  const list = tiered[tier]
  if (!list.length) return []

  const picked = []
  let attempts = 0

  while (picked.length < count && attempts < list.length) {
    // Round-robin through the tier
    const idx = tierIndex[tier] % list.length
    tierIndex[tier] = (tierIndex[tier] + 1) % list.length

    const pubkey = list[idx]
    const entry = getEntry(pubkey)

    // Respect minimum request interval
    if (now - entry.lastRequested >= MIN_REQUEST_MS[tier]) {
      picked.push(pubkey)
    }

    attempts += 1
  }

  return picked
}
```

## Network Queue

### Queue Structure

```javascript
const queue = []
const pending = new Map()  // Deduplication

const queueSend = (msg, targets = 'both') => {
  const key = typeof msg === 'string' ? msg : null
  const targetFlags = normalizeTargets(targets)

  // Check for duplicates
  if (key && pending.has(key)) {
    const item = pending.get(key)
    item.targets.ws = item.targets.ws || targetFlags.ws
    item.targets.gossip = item.targets.gossip || targetFlags.gossip
    return
  }

  // Add to queue
  const item = {
    msg,
    key,
    targets: targetFlags,
    sent: { ws: false, gossip: false }
  }

  queue.push(item)
  if (key) pending.set(key, item)

  scheduleDrain()
}
```

### Drain Process

```javascript
const SEND_DELAY_MS = 100  // 100ms between sends

const drainQueue = () => {
  for (let i = 0; i < queue.length; i++) {
    const item = queue[i]

    // Check transport availability
    const wsReady = item.targets.ws && !item.sent.ws && isTargetReady('ws')
    const gossipReady = item.targets.gossip && !item.sent.gossip && isTargetReady('gossip')

    if (!wsReady && !gossipReady) continue

    // Send to available transports
    if (wsReady) {
      sendToTarget('ws', item.msg)
      item.sent.ws = true
    }
    if (gossipReady) {
      sendToTarget('gossip', item.msg)
      item.sent.gossip = true
    }

    // Remove if fully sent
    if (wsDone && gossipDone) {
      cleanupItem(item, i)
    }

    break  // One item per drain cycle
  }

  if (queue.length) {
    setTimeout(drainQueue, SEND_DELAY_MS)
  }
}
```

### Noting Received Messages

```javascript
// When a message is received, remove from queue
export const noteReceived = (msg) => {
  const key = typeof msg === 'string' ? msg : null
  if (!key) return

  const item = pending.get(key)
  if (!item) return

  pending.delete(key)
  const idx = queue.indexOf(item)
  if (idx >= 0) queue.splice(idx, 1)
}
```

## Bootstrap Activity

When sync starts, seed activity data from stored messages:

```javascript
const BOOTSTRAP_MAX = 20

const bootstrapActivity = async () => {
  let count = 0

  for (const pubkey of pubkeys) {
    if (count >= BOOTSTRAP_MAX) break

    const entry = getEntry(pubkey)
    if (entry.lastSeen) continue  // Already has activity

    const latest = await apds.getLatest(pubkey)
    if (latest?.opened) {
      const ts = parseInt(latest.opened.substring(0, 13), 10)
      if (ts) entry.lastSeen = ts
    }

    count += 1
  }
}
```

## Integration Points

### Route.js - Noting Interest
```javascript
// When user views a profile
noteInterest(pubkey)

// Request latest messages
await send(pubkey)
```

### Render.js - Noting Seen
```javascript
// When receiving a new message
noteSeen(blob.substring(0, 44))  // Author's pubkey
```

### Connect.js - Starting Sync
```javascript
import { startSync } from './sync.js'
import { send } from './send.js'

await startSync(send)
```

## Key Files

- `sync.js` - Tiered sync scheduler
- `network_queue.js` - Unified send queue
- `send.js` - Simple send wrapper
- `gossip.js` - Trystero transport
- `websocket.js` - WebSocket transport
- `connect.js` - Transport initialization
