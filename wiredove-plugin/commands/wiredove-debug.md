---
description: Debug Wiredove P2P connections, message flow, WebSocket status, and synchronization issues. Provides diagnostic tools and troubleshooting guidance.
allowed-tools: Bash, Read, Write, Glob, Grep, AskUserQuestion, mcp__plugin_playwright_playwright__browser_navigate, mcp__plugin_playwright_playwright__browser_snapshot, mcp__plugin_playwright_playwright__browser_console_messages, mcp__plugin_playwright_playwright__browser_network_requests, mcp__plugin_playwright_playwright__browser_evaluate
---

# Wiredove Debug Command

You are helping debug a Wiredove application. This command provides diagnostic tools for P2P connections, message flow, and synchronization issues.

## Step 1: Identify Debug Target

Ask the user what they want to debug:

1. **Connection Issues** - WebSocket or P2P not connecting
2. **Message Sync** - Messages not appearing or syncing
3. **Identity/Keys** - Keypair or authentication problems
4. **Performance** - Slow sync or UI responsiveness
5. **Full Diagnostic** - Run all checks

## Step 2: Gather Context

Ask for:
- Wiredove instance URL (default: http://localhost:8000 or https://wiredove.net)
- Browser being used
- Any error messages seen

## Debug Procedures

### A. Connection Diagnostics

#### Check WebSocket Status
Provide this browser console snippet:
```javascript
// Check WebSocket connections
console.log('=== WebSocket Status ===');
console.log('Active connections:', typeof pubs !== 'undefined' ? pubs.size : 'pubs not accessible');

// Test WebSocket connectivity
const testWs = new WebSocket('wss://pub.wiredove.net/');
testWs.onopen = () => { console.log('✓ WebSocket to pub.wiredove.net: CONNECTED'); testWs.close(); };
testWs.onerror = () => console.log('✗ WebSocket to pub.wiredove.net: FAILED');
```

#### Check Trystero P2P Status
```javascript
// Check Trystero room status
console.log('=== Trystero P2P Status ===');
console.log('Room active:', typeof chan !== 'undefined' && chan !== null);
if (typeof chan !== 'undefined' && chan) {
  console.log('Room details:', chan);
}
```

#### Network Connectivity Checks
```bash
# Check if pub server is reachable
curl -I https://pub.wiredove.net/ 2>/dev/null | head -1

# Check WebSocket upgrade support
curl -i -N -H "Connection: Upgrade" -H "Upgrade: websocket" \
  -H "Sec-WebSocket-Key: test" -H "Sec-WebSocket-Version: 13" \
  https://pub.wiredove.net/ 2>/dev/null | head -5
```

### B. Message Sync Diagnostics

#### Check Local Database
```javascript
// Check APDS database status
console.log('=== APDS Database Status ===');

(async () => {
  const pubkeys = await apds.getPubkeys();
  console.log('Known pubkeys:', pubkeys?.length || 0);

  const log = await apds.query();
  console.log('Total messages:', log?.length || 0);

  const myKey = await apds.pubkey();
  console.log('My pubkey:', myKey ? myKey.substring(0, 20) + '...' : 'Not set');

  if (myKey) {
    const myMsgs = await apds.query(myKey);
    console.log('My messages:', myMsgs?.length || 0);
  }
})();
```

#### Check Sync Scheduler Status
```javascript
// Check sync tier distribution (if accessible)
console.log('=== Sync Status ===');
console.log('Check browser Network tab for:');
console.log('- WebSocket messages (wss://pub.wiredove.net/)');
console.log('- Look for 44-char hashes (pubkey requests)');
console.log('- Look for longer blobs (message content)');
```

#### Verify Message Structure
```javascript
// Validate a specific message
async function debugMessage(hashOrSig) {
  console.log('=== Message Debug ===');
  console.log('Input:', hashOrSig.substring(0, 30) + '...');
  console.log('Length:', hashOrSig.length);

  if (hashOrSig.length === 44) {
    console.log('Type: Hash/Pubkey');
    const content = await apds.get(hashOrSig);
    console.log('Content exists:', !!content);
    if (content) {
      console.log('Content preview:', content.substring(0, 100));
    }
  } else if (hashOrSig.length > 44) {
    console.log('Type: Signed message or blob');
    const opened = await apds.open(hashOrSig);
    if (opened) {
      console.log('✓ Valid signature');
      console.log('Timestamp:', opened.substring(0, 13));
      console.log('Content hash:', opened.substring(13));
      const author = hashOrSig.substring(0, 44);
      console.log('Author:', author.substring(0, 20) + '...');
    } else {
      console.log('✗ Invalid signature or not a signed message');
      // Try parsing as YAML content
      try {
        const yaml = await apds.parseYaml(hashOrSig);
        if (yaml) {
          console.log('Parsed as YAML:', yaml);
        }
      } catch (e) {
        console.log('Not valid YAML either');
      }
    }
  }
}

// Usage: debugMessage('paste-hash-or-sig-here');
```

### C. Identity Diagnostics

```javascript
// Check identity status
console.log('=== Identity Status ===');

(async () => {
  const pubkey = await apds.pubkey();
  const keypair = await apds.keypair();
  const name = await apds.get('name');

  console.log('Pubkey:', pubkey || 'NOT SET');
  console.log('Keypair exists:', !!keypair);
  console.log('Display name:', name || 'NOT SET');

  if (pubkey) {
    const latest = await apds.getLatest(pubkey);
    console.log('Latest message:', latest ? 'EXISTS' : 'NONE');
    if (latest) {
      console.log('  Signature:', latest.sig?.substring(0, 30) + '...');
    }
  }
})();
```

### D. Performance Diagnostics

```javascript
// Performance metrics
console.log('=== Performance Metrics ===');

(async () => {
  const start = performance.now();

  // Database query speed
  const queryStart = performance.now();
  const log = await apds.query();
  const queryTime = performance.now() - queryStart;
  console.log(`Query all messages: ${queryTime.toFixed(2)}ms (${log?.length || 0} messages)`);

  // Pubkey lookup speed
  const pkStart = performance.now();
  const pubkeys = await apds.getPubkeys();
  const pkTime = performance.now() - pkStart;
  console.log(`Get pubkeys: ${pkTime.toFixed(2)}ms (${pubkeys?.length || 0} keys)`);

  // DOM element count
  const msgCount = document.querySelectorAll('.message').length;
  const replyCount = document.querySelectorAll('.message-replies').length;
  console.log(`DOM: ${msgCount} messages, ${replyCount} reply containers`);

  // Memory (if available)
  if (performance.memory) {
    const mb = (performance.memory.usedJSHeapSize / 1024 / 1024).toFixed(2);
    console.log(`JS Heap: ${mb} MB`);
  }
})();
```

### E. Full Diagnostic Report

Run all checks and generate a report:

```javascript
// Full diagnostic report
async function wiredoveDiagnostic() {
  const report = {
    timestamp: new Date().toISOString(),
    url: window.location.href,
    userAgent: navigator.userAgent,
    connection: {},
    database: {},
    identity: {},
    performance: {}
  };

  // Connection
  report.connection.webSocketSupport = 'WebSocket' in window;
  report.connection.webRTCSupport = 'RTCPeerConnection' in window;

  // Database
  try {
    const pubkeys = await apds.getPubkeys();
    const log = await apds.query();
    report.database.pubkeyCount = pubkeys?.length || 0;
    report.database.messageCount = log?.length || 0;
    report.database.status = 'OK';
  } catch (e) {
    report.database.status = 'ERROR: ' + e.message;
  }

  // Identity
  try {
    report.identity.hasPubkey = !!(await apds.pubkey());
    report.identity.hasKeypair = !!(await apds.keypair());
    report.identity.hasName = !!(await apds.get('name'));
  } catch (e) {
    report.identity.status = 'ERROR: ' + e.message;
  }

  // Performance
  report.performance.domMessages = document.querySelectorAll('.message').length;
  if (performance.memory) {
    report.performance.heapMB = (performance.memory.usedJSHeapSize / 1024 / 1024).toFixed(2);
  }

  console.log('=== WIREDOVE DIAGNOSTIC REPORT ===');
  console.log(JSON.stringify(report, null, 2));

  return report;
}

wiredoveDiagnostic();
```

## Common Issues & Solutions

### Issue: "No messages appearing"
1. Check if WebSocket is connected (Network tab)
2. Verify pubkey is set (Settings page)
3. Check console for errors
4. Try: `await apds.getPubkeys()` - should return array

### Issue: "P2P not connecting"
1. Check if WebRTC is supported: `'RTCPeerConnection' in window`
2. Firewall/NAT may block BitTorrent DHT
3. Try different network (mobile hotspot)
4. Check if Trystero room is created in console

### Issue: "Messages not syncing between tabs"
1. Both tabs need same Trystero room ID
2. Check WebSocket is connected in both
3. Verify pubkeys are being requested
4. Look for "SENDING" logs in console

### Issue: "Keypair not saving"
1. Check IndexedDB in DevTools > Application
2. Look for "wiredovedbversion1" database
3. Try clearing and regenerating
4. Check for storage quota errors

### Issue: "Slow performance"
1. Too many messages in DOM - check message count
2. Large images not cached - check Network tab
3. Sync scheduler overloaded - reduce pubkey count
4. Try clearing old data: Settings > Delete everything

## Browser DevTools Tips

1. **Console Tab**: Filter by "OPEN", "CLOSED", "SENDING", "Received"
2. **Network Tab**: Filter by "WS" for WebSocket traffic
3. **Application Tab**: Check IndexedDB > wiredovedbversion1
4. **Performance Tab**: Record during sync to find bottlenecks

## Output Format

Provide clear diagnostic results:
```
=== Wiredove Diagnostic Results ===

Connection:
  ✓ WebSocket: Connected to pub.wiredove.net
  ✓ Trystero: Room active (wiredovev1)

Database:
  ✓ APDS: Initialized
  ✓ Messages: 247
  ✓ Pubkeys: 12

Identity:
  ✓ Pubkey: evSFOKnXaF9Z...
  ✓ Name: ev

Issues Found:
  ⚠ No P2P peers connected (normal if alone)

Recommendations:
  1. Subscribe to ntfy.sh/wiredove for new message alerts
  2. Keep tab open for P2P discovery
```
