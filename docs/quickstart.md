# APDS Quick Start Guide

Get up and running with APDS in 5 minutes.

## What is APDS?

APDS (ANProto Personal Data Server) is a decentralized personal data server for the ANProto social protocol. It enables:

- **Self-sovereign identity** via Ed25519 keypairs
- **Signed messages** with cryptographic verification
- **P2P synchronization** via WebSocket gossip
- **Push notifications** for real-time updates

## Prerequisites

Install Deno:
```bash
# macOS/Linux
curl -fsSL https://deno.land/install.sh | sh

# Windows
irm https://deno.land/install.ps1 | iex

# Verify installation
deno --version
```

## Option 1: Run a Server (2 minutes)

```bash
# Clone APDS
git clone https://github.com/evbogue/apds.git
cd apds

# Start the server
deno run -A serve.js
```

Your server is now running at:
- **HTTP**: http://localhost:9000
- **WebSocket**: ws://localhost:9000

Test it:
```bash
curl http://localhost:9000/all
```

## Option 2: Use the CLI (3 minutes)

```bash
# Clone APDS (if not already)
git clone https://github.com/evbogue/apds.git
cd apds

# Start CLI
deno run -A cli.js
```

You'll see:
```
ANPROTO APDS ACTIVATED -- Your pubkey is: ABC123...
ABC123...>
```

Try these commands:
```
/nick YourName          # Set your display name
/connect wss://apds.anproto.com   # Connect to public server
Hello, world!           # Send a message
/exit                   # Quit
```

## Option 3: Browser App (5 minutes)

Create `index.html`:
```html
<!DOCTYPE html>
<html>
<head><title>APDS Demo</title></head>
<body>
  <div id="messages"></div>
  <input id="input" placeholder="Type a message...">
  <button id="send">Send</button>

  <script type="module">
    import { apds } from './apds.js'

    // Initialize
    await apds.start('demo')

    // Generate keypair if needed
    if (!await apds.pubkey()) {
      await apds.put('keypair', await apds.generate())
    }

    console.log('Your pubkey:', await apds.pubkey())

    // Display messages
    const messagesDiv = document.getElementById('messages')
    const log = await apds.query()
    log.forEach(msg => {
      messagesDiv.innerHTML += `<p>${msg.author.slice(0,8)}: ${msg.text}</p>`
    })

    // Send message
    document.getElementById('send').onclick = async () => {
      const input = document.getElementById('input')
      const hash = await apds.compose(input.value)
      console.log('Sent:', hash)
      input.value = ''
      location.reload()
    }
  </script>
</body>
</html>
```

Serve it:
```bash
deno run -A --watch https://deno.land/std/http/file_server.ts
```

Open http://localhost:4507

## Connect to the Network

Connect to the public APDS server:

**CLI:**
```
/connect wss://apds.anproto.com
```

**JavaScript:**
```javascript
const ws = new WebSocket('wss://apds.anproto.com')
ws.onopen = () => console.log('Connected!')
ws.onmessage = (e) => console.log('Received:', e.data)
```

## Key Concepts

### Identity = Keypair

Your identity is your Ed25519 keypair. The first 44 characters of a signature is your public key (your ID).

```javascript
const pubkey = await apds.pubkey()  // Your ID
// Example: "ABC123XYZ789..."
```

### Messages are Signed

Every message is cryptographically signed. Others can verify you authored it.

```javascript
const hash = await apds.compose('Hello!')
// Returns: content-addressed hash
```

### Content-Addressed Storage

Everything is stored by its hash. Same content = same hash = deduplication.

```javascript
const hash = await apds.make('some data')
const data = await apds.get(hash)
// data === 'some data'
```

## Using the Plugin

With the APDS plugin installed in Claude Code:

```
# Get setup help
/apds-setup

# Run the server
/apds-run server

# Test everything works
/apds-test all

# Manage peer connections
/apds-peers list
```

## Next Steps

1. **Read the full docs**: See `docs/` folder
2. **Explore the code**: Run `/apds-setup` for guidance
3. **Build something**: Chat app, social feed, or collaborative tool
4. **Join the network**: Connect to `wss://apds.anproto.com`

## Common Issues

**"Module not found"**
```bash
# Make sure you're in the apds directory
cd apds
deno run -A serve.js
```

**"Port 9000 in use"**
```bash
# Find what's using it
lsof -i :9000

# Kill it or change port in serve.js
```

**"No keypair found"**
```javascript
// Generate a new one
const kp = await apds.generate()
await apds.put('keypair', kp)
```

## Resources

- **APDS Repository**: https://github.com/evbogue/apds
- **Live Server**: https://apds.anproto.com
- **Deno**: https://deno.land
