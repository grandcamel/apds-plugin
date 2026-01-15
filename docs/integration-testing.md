# APDS Integration Testing Guide

This guide covers testing APDS applications against live servers and verifying end-to-end functionality.

## Prerequisites

- Deno installed (`deno --version`)
- Network access to test servers
- APDS plugin installed in Claude Code

## Test Environments

### Local Development
```bash
# Start local server
deno run -A serve.js

# Server available at:
# HTTP: http://localhost:9000
# WebSocket: ws://localhost:9000
```

### Public Test Server
```
URL: https://apds.anproto.com
WebSocket: wss://apds.anproto.com
```

## Integration Test Scenarios

### 1. Server Connectivity

**Test**: Verify server is reachable and responding

```bash
# HTTP health check
curl -s http://localhost:9000/all | jq 'type'
# Expected: "array"

# Check message count
curl -s http://localhost:9000/all | jq 'length'
# Expected: number >= 0

# Test latest endpoint
curl -s http://localhost:9000/latest | jq 'length'
# Expected: number (messages from last 5 mins)
```

**Against live server:**
```bash
curl -s https://apds.anproto.com/all | jq 'length'
```

### 2. WebSocket Connection

**Test**: Establish WebSocket and receive data

```javascript
// test-websocket.js
const ws = new WebSocket('ws://localhost:9000')

ws.onopen = () => {
  console.log('✓ WebSocket connected')

  // Request data by sending a pubkey
  ws.send('TEST_PUBKEY_44_CHARACTERS_HERE_AAAAAAAAAA==')
}

ws.onmessage = (event) => {
  console.log('✓ Received:', event.data.substring(0, 50) + '...')
}

ws.onerror = (err) => {
  console.log('✗ Error:', err.message)
}

// Run for 5 seconds
setTimeout(() => {
  ws.close()
  console.log('Test complete')
}, 5000)
```

```bash
deno run -A test-websocket.js
```

### 3. Message Signing & Verification

**Test**: Create, sign, and verify a message

```javascript
// test-signing.js
import { apds } from './apds.js'

await apds.start('test-namespace')

// Generate keypair if needed
if (!await apds.pubkey()) {
  const kp = await apds.generate()
  await apds.put('keypair', kp)
}

console.log('Pubkey:', await apds.pubkey())

// Sign a message
const testContent = `Test message at ${Date.now()}`
const hash = await apds.sign(testContent)
console.log('✓ Signed, hash:', hash)

// Retrieve and verify
const msg = await apds.get(hash)
console.log('✓ Retrieved:', msg.substring(0, 50) + '...')

const opened = await apds.open(msg)
if (opened) {
  console.log('✓ Signature verified')
  console.log('  Timestamp:', opened.substring(0, 13))
  console.log('  Content hash:', opened.substring(13))
} else {
  console.log('✗ Signature verification failed')
}
```

### 4. Message Composition with YAML

**Test**: Compose message with frontmatter

```javascript
// test-compose.js
import { apds } from './apds.js'

await apds.start('test-compose')

// Setup profile
await apds.put('name', 'TestUser')

// Generate keypair
if (!await apds.pubkey()) {
  await apds.put('keypair', await apds.generate())
}

// Compose with metadata
const hash = await apds.compose('Hello from integration test!')
console.log('✓ Composed message:', hash)

// Verify YAML structure
const msg = await apds.get(hash)
const sig = await apds.get(msg) // Get the signed content
const yaml = await apds.parseYaml(sig)

console.log('✓ Parsed YAML:')
console.log('  name:', yaml.name)
console.log('  body:', yaml.body)
console.log('  previous:', yaml.previous || '(none)')
```

### 5. Peer Synchronization

**Test**: Sync messages between two instances

```javascript
// test-sync.js
// Run two terminals:
// Terminal 1: deno run -A serve.js
// Terminal 2: Run this script

import { apds } from './apds.js'

await apds.start('sync-test')

if (!await apds.pubkey()) {
  await apds.put('keypair', await apds.generate())
}

const ws = new WebSocket('ws://localhost:9000')
const received = []

ws.onopen = async () => {
  console.log('✓ Connected to server')

  // Create and send a message
  const hash = await apds.compose('Sync test message')
  const msg = await apds.get(hash)

  ws.send(msg)
  ws.send(hash)
  console.log('✓ Sent message:', hash)
}

ws.onmessage = async (event) => {
  received.push(event.data)
  console.log('✓ Received data:', event.data.length, 'chars')

  // Store received messages
  if (event.data.length !== 44) {
    await apds.make(event.data)
    await apds.add(event.data)
  }
}

// Check results after 5 seconds
setTimeout(async () => {
  console.log('\nSync Results:')
  console.log('  Messages received:', received.length)
  console.log('  Local message count:', (await apds.getHashLog()).length)
  ws.close()
}, 5000)
```

### 6. Web Push Notifications

**Test**: Verify push notification setup

```bash
# Check VAPID key
curl -s http://localhost:9000/push/vapidPublicKey | jq '.publicKey'
# Expected: 87-character base64url string

# Check if vapid.json exists
ls -la vapid.json

# Check subscriptions
cat subscriptions.json 2>/dev/null | jq 'length' || echo "No subscriptions yet"

# Send test notification
curl -X POST http://localhost:9000/push/test
# Expected: Notifications sent to subscribers
```

### 7. Query and Search

**Test**: Query messages by various criteria

```javascript
// test-query.js
import { apds } from './apds.js'

await apds.start('query-test')

// Get all messages
const all = await apds.query()
console.log('Total messages:', all.length)

// Get unique authors
const authors = await apds.getPubkeys()
console.log('Unique authors:', authors.length)

if (authors[0]) {
  // Query by author
  const byAuthor = await apds.query(authors[0])
  console.log(`Messages from ${authors[0].substring(0, 10)}:`, byAuthor.length)
}

// Search (if messages exist)
if (all.length > 0) {
  const search = await apds.query('?test')
  console.log('Messages containing "test":', search.length)
}
```

## Automated Test Runner

Create a comprehensive test script:

```javascript
// run-tests.js
const tests = []
let passed = 0
let failed = 0

function test(name, fn) {
  tests.push({ name, fn })
}

async function run() {
  console.log('APDS Integration Tests\n' + '='.repeat(50))

  for (const t of tests) {
    try {
      await t.fn()
      console.log(`✓ ${t.name}`)
      passed++
    } catch (err) {
      console.log(`✗ ${t.name}`)
      console.log(`  Error: ${err.message}`)
      failed++
    }
  }

  console.log('\n' + '='.repeat(50))
  console.log(`Results: ${passed} passed, ${failed} failed`)

  Deno.exit(failed > 0 ? 1 : 0)
}

// Define tests
test('Server responds to /all', async () => {
  const res = await fetch('http://localhost:9000/all')
  if (!res.ok) throw new Error('Server not responding')
  const data = await res.json()
  if (!Array.isArray(data)) throw new Error('Expected array')
})

test('Server responds to /latest', async () => {
  const res = await fetch('http://localhost:9000/latest')
  if (!res.ok) throw new Error('Server not responding')
})

test('VAPID key available', async () => {
  const res = await fetch('http://localhost:9000/push/vapidPublicKey')
  if (!res.ok) throw new Error('VAPID endpoint not available')
  const { publicKey } = await res.json()
  if (!publicKey || publicKey.length < 80) throw new Error('Invalid VAPID key')
})

test('WebSocket connects', async () => {
  const ws = new WebSocket('ws://localhost:9000')
  await new Promise((resolve, reject) => {
    ws.onopen = resolve
    ws.onerror = reject
    setTimeout(() => reject(new Error('Timeout')), 3000)
  })
  ws.close()
})

// Run all tests
run()
```

```bash
# Start server first, then:
deno run -A run-tests.js
```

## CI/CD Integration

### GitHub Actions Example

```yaml
# .github/workflows/test.yml
name: APDS Integration Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: denoland/setup-deno@v1
        with:
          deno-version: v1.x

      - name: Start APDS server
        run: |
          deno run -A serve.js &
          sleep 3

      - name: Run integration tests
        run: deno run -A run-tests.js

      - name: Test against live server
        run: |
          curl -sf https://apds.anproto.com/all > /dev/null
          echo "Live server reachable"
```

## Troubleshooting Failed Tests

### Server Not Responding
```bash
# Check if server is running
lsof -i :9000

# Check server logs
deno run -A serve.js 2>&1 | tee server.log
```

### WebSocket Failures
```bash
# Test with websocat if available
websocat ws://localhost:9000

# Check for firewall issues
nc -zv localhost 9000
```

### Signature Verification Fails
```javascript
// Debug signature
const msg = await apds.get(hash)
console.log('Message length:', msg.length)
console.log('Ends with ==:', msg.endsWith('=='))
console.log('First 44 chars (pubkey):', msg.substring(0, 44))
```

## Performance Benchmarks

```javascript
// benchmark.js
import { apds } from './apds.js'

await apds.start('benchmark')
await apds.put('keypair', await apds.generate())

const iterations = 100

// Benchmark signing
console.log(`Signing ${iterations} messages...`)
const signStart = Date.now()
for (let i = 0; i < iterations; i++) {
  await apds.sign(`Message ${i}`)
}
const signTime = Date.now() - signStart
console.log(`  Total: ${signTime}ms`)
console.log(`  Per message: ${(signTime / iterations).toFixed(2)}ms`)

// Benchmark queries
console.log(`\nQuerying ${iterations} times...`)
const queryStart = Date.now()
for (let i = 0; i < iterations; i++) {
  await apds.query()
}
const queryTime = Date.now() - queryStart
console.log(`  Total: ${queryTime}ms`)
console.log(`  Per query: ${(queryTime / iterations).toFixed(2)}ms`)
```
