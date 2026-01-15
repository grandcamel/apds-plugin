# Wiredove Development Patterns

This document describes common patterns used throughout the Wiredove codebase to help developers understand conventions and contribute effectively.

## DOM Creation Pattern

Wiredove uses a custom `h()` helper from APDS for creating DOM elements:

```javascript
import { h } from 'h'

// Basic element
const div = h('div', {classList: 'message'}, ['Text content'])

// With event handlers
const button = h('button', {
  onclick: async () => {
    await doSomething()
  }
}, ['Click me'])

// Nested elements
const card = h('div', {classList: 'card'}, [
  h('span', {classList: 'title'}, ['Title']),
  h('div', {classList: 'content'}, [bodyContent])
])
```

**Pattern**: Always use `h()` instead of `document.createElement()`.

## Async Module Pattern

All modules export async functions and use top-level await:

```javascript
// Module initialization
import { apds } from 'apds'

await apds.start('dbname')

// Exported async function
export const myFunction = async () => {
  const data = await apds.get('key')
  return data
}
```

## Message Handling Pattern

Messages flow through a consistent pipeline:

```javascript
// 1. Receive raw data
const blob = receivedData

// 2. Verify and parse
const opened = await apds.open(blob)
if (!opened) return // Invalid signature

// 3. Extract components
const timestamp = opened.substring(0, 13)
const contentHash = opened.substring(13)
const author = blob.substring(0, 44)

// 4. Get content
const content = await apds.get(contentHash)
const yaml = await apds.parseYaml(content)

// 5. Process fields
if (yaml.body) { /* render message */ }
if (yaml.edit) { /* handle edit */ }
if (yaml.reply) { /* handle reply */ }
```

## Network Send Pattern

All network sends go through the unified queue:

```javascript
import { queueSend } from './network_queue.js'

// Send to both WebSocket and Trystero
queueSend(message, 'both')

// Send to WebSocket only
queueSend(message, 'ws')

// Send to Trystero only
queueSend(message, 'gossip')
```

**Pattern**: Never call WebSocket or Trystero directly. Always use `queueSend()` or the `send()` wrapper.

## Hash-Based Content Addressing

Content is addressed by its hash (base64, 44 characters):

```javascript
// Store and retrieve by hash
const hash = await apds.hash(content)
await apds.put(hash, content)  // Usually done by apds.make()
const retrieved = await apds.get(hash)

// Request from network if missing
if (!retrieved) {
  send(hash)  // Network peers will respond with content
}
```

## IntersectionObserver Pattern

Lazy loading is done via IntersectionObserver:

```javascript
let observer = null

const observe = (element, onVisible) => {
  if (!observer) {
    observer = new IntersectionObserver((entries) => {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          onVisible(entry.target)
        }
      })
    })
  }
  observer.observe(element)
}

// Usage for lazy loading replies
observe(wrapper, async (target) => {
  const hash = target.dataset.replyParent
  await loadReplies(hash)
  target.dataset.repliesLoaded = 'true'
})
```

## Timestamp Insertion Pattern

Messages are inserted in chronological order:

```javascript
const insertByTimestamp = (container, hash, ts) => {
  const stamp = parseInt(ts, 10)
  const div = createPlaceholder(hash)
  div.dataset.ts = stamp.toString()

  // Find correct position (newest first)
  const children = Array.from(container.children)
  for (const child of children) {
    const childTs = parseInt(child.dataset.ts, 10)
    if (childTs < stamp) {
      container.insertBefore(div, child)
      return div
    }
  }
  container.appendChild(div)
  return div
}
```

## Edit Tracking Pattern

Edits link back to original messages:

```javascript
// Creating an edit
const editObj = {
  edit: originalHash,  // Links to original
  body: newContent
}
await apds.compose(newContent, editObj)

// Finding edits for a message
const log = await apds.query(author)
const edits = log.filter(msg => {
  const yaml = await apds.parseYaml(msg.text)
  return yaml && yaml.edit === originalHash
})
edits.sort((a, b) => a.ts - b.ts)
```

## Reply Threading Pattern

Replies reference parent messages:

```javascript
// Creating a reply
const replyObj = {
  reply: parentHash,
  replyto: parentAuthor
}
await apds.compose(content, replyObj)

// Building reply index
const replyIndex = new Map()
for (const msg of log) {
  const yaml = await apds.parseYaml(msg.text)
  const parent = yaml.replyHash || yaml.reply
  if (parent) {
    const list = replyIndex.get(parent) || []
    list.push({ hash: msg.hash, ts: msg.ts })
    replyIndex.set(parent, list)
  }
}
```

## Tiered Sync Pattern

Sync prioritizes active content:

```javascript
const HOT_INTEREST_MS = 15 * 60 * 1000   // 15 minutes
const HOT_SEEN_MS = 24 * 60 * 60 * 1000  // 24 hours

const classify = (entry, now) => {
  const seenAge = now - entry.lastSeen
  const interestAge = now - entry.lastInterest

  if (interestAge <= HOT_INTEREST_MS || seenAge <= HOT_SEEN_MS) {
    return 'hot'  // Sync frequently
  }
  if (interestAge <= WARM_INTEREST_MS || seenAge <= WARM_SEEN_MS) {
    return 'warm'  // Sync occasionally
  }
  return 'cold'  // Sync rarely
}
```

## Trystero Room Pattern

P2P rooms are keyed by public key or topic:

```javascript
import { joinRoom } from './trystero-torrent.min.js'

const makeRoom = async (roomId) => {
  const room = joinRoom({
    appId: 'wiredovetestnet',
    password: 'shared-secret'
  }, roomId)

  // Create action channels
  const [sendHash, onHash] = room.makeAction('hash')
  const [sendBlob, onBlob] = room.makeAction('blob')

  // Handle incoming
  onHash(async (hash, peerId) => {
    const data = await apds.get(hash)
    if (data) sendBlob(data, peerId)
  })

  onBlob(async (blob) => {
    await processBlob(blob)
  })

  // Peer lifecycle
  room.onPeerJoin(async (id) => {
    // Send latest state to new peer
  })

  return room
}
```

## WebSocket Reconnection Pattern

WebSocket connections use exponential backoff:

```javascript
const wsBackoff = new Map()

const getBackoff = (url) => {
  let state = wsBackoff.get(url)
  if (!state) {
    state = { delayMs: 1000, timer: null }
    wsBackoff.set(url, state)
  }
  return state
}

const scheduleReconnect = (url, connectFn) => {
  const state = getBackoff(url)
  if (state.timer) return

  state.timer = setTimeout(() => {
    state.timer = null
    connectFn()
    state.delayMs *= 2  // Exponential backoff
  }, state.delayMs)
}

const resetBackoff = (url) => {
  const state = getBackoff(url)
  clearTimeout(state.timer)
  state.timer = null
  state.delayMs = 1000
}
```

## Material Icons Pattern

Icons use Google Material Symbols:

```javascript
// Icon element
const icon = h('span', {
  classList: 'material-symbols-outlined'
}, ['Share'])

// Clickable icon
const button = h('a', {
  classList: 'material-symbols-outlined',
  onclick: handleClick
}, ['Edit'])
```

Common icons used: `Share`, `Edit`, `Chat_Bubble`, `Qr_Code`, `Code`, `Cancel`, `Subdirectory_Arrow_left`, `Chevron_Left`, `Chevron_Right`

## URL Hash Routing Pattern

Navigation uses URL hashes:

```javascript
// Navigate programmatically
window.location.hash = '#' + destination

// Listen for changes
window.onhashchange = async () => {
  // Clear current view
  document.getElementById('scroller')?.remove()
  // Render new view
  await route()
}

// Parse current route
const src = window.location.hash.substring(1)
if (src === '') { /* home */ }
else if (src === 'settings') { /* settings */ }
else if (src.length === 44) { /* profile by pubkey */ }
else if (src.length > 44) { /* message view */ }
```

## Error Handling Pattern

Errors are generally logged but don't crash:

```javascript
try {
  const result = await riskyOperation()
  // Continue with result
} catch (err) {
  console.log(err)  // or console.warn(err)
  // Gracefully continue or return default
}
```

**Note**: The codebase prefers resilience over strict error handling. Network failures and missing data are expected in a decentralized system.
