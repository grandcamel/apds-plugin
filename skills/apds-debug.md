---
description: Debug APDS issues - WebSocket connections, message sync, gossip protocol, and storage problems
user_invocable: true
---

# APDS Debug Assistant

You are helping debug issues with APDS (ANProto Personal Data Server). Use this guide to systematically diagnose problems.

## Common Issue Categories

### 1. Connection Issues

**Symptoms**: Can't connect to server, WebSocket errors, timeouts

**Diagnosis Steps**:
```bash
# Check if server is running
curl -s http://localhost:9000/all

# Test WebSocket (if websocat installed)
websocat ws://localhost:9000

# Check port availability
lsof -i :9000
```

**Common Causes**:
- Server not running
- Wrong port or URL
- Firewall blocking connections
- SSL/TLS issues with wss:// URLs

### 2. Message Sync Issues

**Symptoms**: Messages not appearing, missing data, stale content

**Diagnosis Steps**:
1. Check local message count:
   ```javascript
   const log = await apds.getOpenedLog()
   console.log('Local messages:', log.length)
   ```

2. Check gossip queue status (in serve.js):
   - Missing messages are tracked in the gossip queue
   - Check if `missing.size` is growing

3. Verify message format:
   ```javascript
   const msg = await apds.get(hash)
   const opened = await apds.open(msg)
   console.log('Opened:', opened)
   ```

**Common Causes**:
- Gossip interval too slow (default 10s)
- Peer disconnected before sync completed
- Invalid message signatures
- Storage namespace mismatch

### 3. Keypair Issues

**Symptoms**: "No keypair found", signature errors, different pubkey than expected

**Diagnosis Steps**:
```javascript
// Check keypair exists
const kp = await apds.keypair()
console.log('Keypair:', kp ? 'exists' : 'missing')
console.log('Pubkey:', await apds.pubkey())
```

**Common Causes**:
- Different storage namespace
- Storage was cleared
- Browser storage deleted
- Running in different directory

### 4. YAML Parsing Errors

**Symptoms**: Frontmatter not parsed, missing name/image, malformed messages

**Check Message Format**:
```javascript
const msg = await apds.get(hash)
const yaml = await apds.parseYaml(msg)
console.log('Parsed:', yaml)
```

**Expected Format**:
```yaml
---
name: Username
image: hash==
previous: hash==
---
Message body here
```

### 5. Storage Issues

**Symptoms**: Data not persisting, messages lost on restart

**Diagnosis**:
```javascript
// Check storage namespace
// serve.js uses 'apdsv1'
// cli.js uses 'default' or first arg

// Test storage
await apds.put('test', 'value')
const got = await apds.get('test')
console.log('Storage test:', got === 'value' ? 'OK' : 'FAIL')
```

**Common Causes**:
- Write permissions
- Disk space
- Browser storage limits
- IndexedDB issues

## Debug Logging

Add console logging to trace issues:

```javascript
// In serve.js, after line 48
ws.onmessage = async (m) => {
  console.log('RECEIVED:', m.data.substring(0, 50) + '...')
  console.log('Length:', m.data.length)
  // ... rest of handler
}
```

## Network Debugging

### Monitor WebSocket Traffic
In browser DevTools:
1. Network tab → WS filter
2. Click on WebSocket connection
3. View Messages tab for sent/received

### Check Gossip Activity
The gossip queue logs missing hashes and resolution:
```javascript
// In gossip.js tick function
console.log('Missing hashes:', missing.size)
console.log('Connected peers:', list.length)
```

## Quick Fixes

### Reset Everything
```javascript
await apds.clear()  // Deletes all local data
```

### Regenerate Keypair
```javascript
await apds.deletekey()
const newKey = await apds.generate()
await apds.put('keypair', newKey)
```

### Force Resync
Connect to a peer and request all their messages:
```javascript
// Send pubkey to request their latest
ws.send(peerPubkey)
```

## Performance Issues

### Slow Queries
- Check `openedLog` size
- Consider pagination for large datasets
- Sort interval is 20 seconds (line 65 in apds.js)

### Memory Growth
- Hash log and opened log grow unbounded
- Consider periodic cleanup for long-running servers

## Getting More Help

1. Check server console for error messages
2. Enable verbose logging in Deno: `deno run -A --log-level=debug serve.js`
3. Test with the live server: `wss://apds.anproto.com`
4. Compare behavior with cli.js for reference
