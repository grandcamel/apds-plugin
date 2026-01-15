---
description: Test APDS functionality - push notifications, server connectivity, message signing, and peer sync
user_invocable: true
arguments:
  - name: test_type
    description: "What to test: push, server, sign, sync, or all"
    required: false
    default: "all"
  - name: url
    description: "Server URL to test against"
    required: false
    default: "http://localhost:9000"
---

# APDS Test Command

Run diagnostic tests on APDS components to verify they're working correctly.

## Usage

```
/apds-test [test_type] [url]
```

## Test Types

### push
Test Web Push notification setup:

```bash
# Check VAPID key endpoint
curl -s ${URL}/push/vapidPublicKey

# Verify vapid.json exists
ls -la vapid.json

# Send test notification
curl -X POST ${URL}/push/test

# Check subscriptions
cat subscriptions.json | jq 'length'
```

**Expected output:**
```
VAPID Public Key: ✓ Available (87 chars)
VAPID File: ✓ Exists
Subscriptions: 2 registered
Test Push: ✓ Sent to 2 subscribers
```

### server
Test server connectivity and endpoints:

```bash
# Health check
curl -s ${URL}/all | head -c 100

# Check latest endpoint
curl -s ${URL}/latest | jq 'length'

# Test WebSocket upgrade
curl -s -i -N \
  -H "Connection: Upgrade" \
  -H "Upgrade: websocket" \
  -H "Sec-WebSocket-Version: 13" \
  -H "Sec-WebSocket-Key: test" \
  ${URL} 2>&1 | head -5
```

**Expected output:**
```
Server Status: ✓ Running at ${URL}
/all endpoint: ✓ Returns JSON array
/latest endpoint: ✓ Returns 5 messages
WebSocket: ✓ Upgrade supported
```

### sign
Test message signing and verification:

```javascript
// Generate test keypair
const keypair = await apds.generate()
await apds.put('keypair', keypair)

// Sign a test message
const hash = await apds.sign('Test message')

// Verify signature
const msg = await apds.get(hash)
const opened = await apds.open(msg)

console.log('Keypair:', keypair ? '✓' : '✗')
console.log('Signature:', msg ? '✓' : '✗')
console.log('Verification:', opened ? '✓' : '✗')
```

**Expected output:**
```
Keypair Generation: ✓ 88 chars
Message Signing: ✓ Hash generated
Signature Verification: ✓ Opens correctly
Author Pubkey: ABC123...
```

### sync
Test peer synchronization:

```javascript
// Connect to server
const ws = new WebSocket(URL.replace('http', 'ws'))

// Track sync status
let received = 0
ws.onmessage = () => received++

// Request sync
ws.send(await apds.pubkey())

// Wait and report
setTimeout(() => {
  console.log('Messages received:', received)
}, 3000)
```

**Expected output:**
```
WebSocket Connection: ✓ Connected
Sync Request: ✓ Sent pubkey
Messages Received: 12 in 3s
Gossip Queue: 0 pending
```

### all
Run all tests in sequence.

## Implementation

When this command is invoked:

1. **Detect environment**
   - Check if in APDS project directory
   - Find server URL from config or use default

2. **Run selected tests**
   - Execute test commands via Bash
   - Parse and format results

3. **Report results**
   - Show pass/fail for each check
   - Provide troubleshooting hints for failures

## Example Session

```
> /apds-test all

APDS Test Suite
===============
Server: http://localhost:9000

[Server Tests]
  ✓ Server responding (23ms)
  ✓ /all endpoint returns JSON
  ✓ /latest endpoint works
  ✓ WebSocket upgrade supported

[Push Tests]
  ✓ VAPID public key available
  ✓ vapid.json exists
  ✓ 2 push subscriptions registered
  ✓ Test notification sent

[Signing Tests]
  ✓ Keypair generation works
  ✓ Message signing works
  ✓ Signature verification works

[Sync Tests]
  ✓ WebSocket connects
  ✓ Peer responds to requests
  ✓ Messages sync correctly

Results: 13/13 passed
```

## Failure Examples

```
> /apds-test server

[Server Tests]
  ✗ Server not responding
    → Is the server running? Try: deno run -A serve.js

  ✗ Port 9000 in use by another process
    → Kill existing process: lsof -ti:9000 | xargs kill
```

```
> /apds-test push

[Push Tests]
  ✗ VAPID public key not available
    → Server may not have push enabled
    → Check if /push/vapidPublicKey endpoint exists

  ✗ No subscriptions registered
    → Subscribe a browser first using the push API
```

## Quick Checks

For rapid verification without full test suite:

```bash
# Is server up?
curl -sf http://localhost:9000/all > /dev/null && echo "✓ Server OK" || echo "✗ Server down"

# Any messages?
curl -s http://localhost:9000/all | jq 'length'

# Push ready?
curl -s http://localhost:9000/push/vapidPublicKey | jq -r '.publicKey // "not configured"'
```
