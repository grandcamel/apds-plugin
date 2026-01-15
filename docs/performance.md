# APDS Performance Optimization Guide

Optimize your APDS applications for speed, memory efficiency, and scalability.

## Overview

APDS performance depends on:
1. **Storage operations** - Read/write to cache/database
2. **Cryptographic operations** - Signing and verification
3. **Network operations** - WebSocket sync and gossip
4. **Query operations** - Searching and filtering messages

## Storage Optimization

### Use Appropriate Namespaces

```javascript
// Separate namespaces for different data types
await apds.start('messages')    // Main message store
await apds.start('media')       // Large media files
await apds.start('cache')       // Temporary data
```

### Batch Operations

```javascript
// ❌ Slow: Individual operations
for (const msg of messages) {
  await apds.add(msg)
}

// ✓ Better: Batch when possible
await Promise.all(messages.map(msg => apds.add(msg)))
```

### Limit Stored Data

```javascript
// Implement message expiry
async function cleanOldMessages(maxAge = 7 * 24 * 60 * 60 * 1000) {
  const cutoff = Date.now() - maxAge
  const log = await apds.getOpenedLog()

  for (const msg of log) {
    if (parseInt(msg.ts) < cutoff) {
      await apds.rm(msg.hash)
    }
  }
}
```

## Cryptographic Optimization

### Cache Keypair Reference

```javascript
// ❌ Slow: Fetch keypair each time
async function signMessage(content) {
  const keypair = await apds.keypair()  // DB lookup each time
  return await an.sign(content, keypair)
}

// ✓ Better: Cache in memory
let cachedKeypair = null

async function signMessage(content) {
  if (!cachedKeypair) {
    cachedKeypair = await apds.keypair()
  }
  return await an.sign(content, cachedKeypair)
}
```

### Batch Signature Verification

```javascript
// ✓ Verify multiple signatures in parallel
async function verifyBatch(messages) {
  const results = await Promise.all(
    messages.map(async (msg) => ({
      msg,
      valid: !!(await apds.open(msg.sig))
    }))
  )
  return results.filter(r => r.valid).map(r => r.msg)
}
```

## Network Optimization

### Connection Pooling

```javascript
// Maintain a pool of connections
const connectionPool = new Map()
const MAX_CONNECTIONS = 5

async function getConnection(url) {
  if (connectionPool.has(url)) {
    const ws = connectionPool.get(url)
    if (ws.readyState === WebSocket.OPEN) {
      return ws
    }
  }

  if (connectionPool.size >= MAX_CONNECTIONS) {
    // Close oldest connection
    const [oldestUrl] = connectionPool.keys()
    connectionPool.get(oldestUrl).close()
    connectionPool.delete(oldestUrl)
  }

  const ws = new WebSocket(url)
  await new Promise(r => ws.onopen = r)
  connectionPool.set(url, ws)
  return ws
}
```

### Throttle Broadcasts

```javascript
// ❌ Spam: Broadcast every message immediately
ws.send(message)

// ✓ Better: Batch and throttle
const broadcastQueue = []
let broadcastTimer = null

function queueBroadcast(data) {
  broadcastQueue.push(data)

  if (!broadcastTimer) {
    broadcastTimer = setTimeout(flushBroadcasts, 100)
  }
}

function flushBroadcasts() {
  const batch = broadcastQueue.splice(0)
  batch.forEach(data => {
    sockets.forEach(ws => ws.send(data))
  })
  broadcastTimer = null
}
```

### Optimize Gossip Interval

```javascript
// Default: 10 seconds
const gossipQueue = createGossip({
  intervalMs: 10000,  // Adjust based on needs
  // ...
})

// For real-time apps: shorter interval
intervalMs: 2000

// For battery/bandwidth savings: longer interval
intervalMs: 30000
```

## Query Optimization

### Index Frequently Accessed Data

```javascript
// Build in-memory indexes
const authorIndex = new Map()  // pubkey -> [hashes]
const tagIndex = new Map()     // tag -> [hashes]

async function indexMessage(msg) {
  // Index by author
  if (!authorIndex.has(msg.author)) {
    authorIndex.set(msg.author, [])
  }
  authorIndex.get(msg.author).push(msg.hash)

  // Index by tags
  const yaml = await apds.parseYaml(msg.text)
  if (yaml.tags) {
    for (const tag of yaml.tags) {
      if (!tagIndex.has(tag)) {
        tagIndex.set(tag, [])
      }
      tagIndex.get(tag).push(msg.hash)
    }
  }
}

// Fast query by author
function getByAuthor(pubkey) {
  return authorIndex.get(pubkey) || []
}
```

### Paginate Large Results

```javascript
// ❌ Slow: Load all messages
const all = await apds.query()
renderMessages(all)

// ✓ Better: Paginate
async function getPage(offset = 0, limit = 20) {
  const all = await apds.query()
  return all.slice(offset, offset + limit)
}

// Usage
let page = 0
const pageSize = 20

async function loadMore() {
  const messages = await getPage(page * pageSize, pageSize)
  renderMessages(messages)
  page++
}
```

### Use Selective Sync

```javascript
// Only sync messages from followed users
const following = new Set(['pubkey1', 'pubkey2'])

ws.onmessage = async (event) => {
  if (event.data.length === 44) {
    // It's a hash - check if we want it
    const stored = await apds.get(event.data)
    if (stored) {
      const author = stored.substring(0, 44)
      if (following.has(author)) {
        ws.send(stored)
      }
    }
  }
}
```

## Memory Optimization

### Limit Log Size

```javascript
// apds.js stores all messages in memory
// Implement circular buffer for high-volume apps

const MAX_LOG_SIZE = 10000

async function addWithLimit(msg) {
  await apds.add(msg)

  const log = await apds.getHashLog()
  if (log.length > MAX_LOG_SIZE) {
    // Remove oldest 10%
    const toRemove = log.slice(0, Math.floor(MAX_LOG_SIZE * 0.1))
    for (const hash of toRemove) {
      await apds.rm(hash)
    }
  }
}
```

### Stream Large Files

```javascript
// ❌ Bad: Load entire file into memory
const imageData = await Deno.readFile('large-image.png')
const hash = await apds.make(imageData)

// ✓ Better: Stream and chunk
async function storeFile(path, chunkSize = 1024 * 1024) {
  const file = await Deno.open(path)
  const chunks = []

  const buffer = new Uint8Array(chunkSize)
  let bytesRead

  while ((bytesRead = await file.read(buffer)) !== null) {
    const chunk = buffer.slice(0, bytesRead)
    const hash = await apds.make(new TextDecoder().decode(chunk))
    chunks.push(hash)
  }

  file.close()

  // Store manifest
  return await apds.make(JSON.stringify({ chunks }))
}
```

## Server Optimization

### Use Worker Threads

```javascript
// For CPU-intensive operations
const worker = new Worker(new URL('./crypto-worker.js', import.meta.url).href, {
  type: 'module'
})

worker.postMessage({ action: 'sign', data: content })
worker.onmessage = (e) => {
  console.log('Signed:', e.data.hash)
}
```

### Implement Rate Limiting

```javascript
const rateLimit = new Map()  // IP -> { count, resetTime }

function checkRateLimit(ip, maxRequests = 100, windowMs = 60000) {
  const now = Date.now()
  const record = rateLimit.get(ip) || { count: 0, resetTime: now + windowMs }

  if (now > record.resetTime) {
    record.count = 0
    record.resetTime = now + windowMs
  }

  record.count++
  rateLimit.set(ip, record)

  return record.count <= maxRequests
}

// In request handler
if (!checkRateLimit(request.headers.get('x-forwarded-for'))) {
  return new Response('Rate limited', { status: 429 })
}
```

## Benchmarking

### Measure Operation Times

```javascript
async function benchmark(name, fn, iterations = 100) {
  const times = []

  for (let i = 0; i < iterations; i++) {
    const start = performance.now()
    await fn()
    times.push(performance.now() - start)
  }

  const avg = times.reduce((a, b) => a + b) / times.length
  const min = Math.min(...times)
  const max = Math.max(...times)

  console.log(`${name}:`)
  console.log(`  Avg: ${avg.toFixed(2)}ms`)
  console.log(`  Min: ${min.toFixed(2)}ms`)
  console.log(`  Max: ${max.toFixed(2)}ms`)
}

// Usage
await benchmark('Sign message', () => apds.sign('test'))
await benchmark('Query all', () => apds.query())
await benchmark('Get by hash', () => apds.get(someHash))
```

### Profile Memory Usage

```javascript
// Deno memory profiling
function logMemory(label) {
  const mem = Deno.memoryUsage()
  console.log(`${label}:`)
  console.log(`  Heap used: ${(mem.heapUsed / 1024 / 1024).toFixed(2)} MB`)
  console.log(`  Heap total: ${(mem.heapTotal / 1024 / 1024).toFixed(2)} MB`)
}

logMemory('Before load')
const messages = await apds.query()
logMemory('After query')
```

## Quick Wins Checklist

- [ ] Cache keypair in memory
- [ ] Use Promise.all for parallel operations
- [ ] Implement message pagination
- [ ] Throttle WebSocket broadcasts
- [ ] Set appropriate gossip interval
- [ ] Add indexes for common queries
- [ ] Implement message expiry
- [ ] Rate limit incoming requests
- [ ] Use connection pooling
- [ ] Profile before optimizing
