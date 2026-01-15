---
description: Guide developers through setting up APDS (ANProto Personal Data Server) - server setup, CLI client, relay, or browser client configuration
user_invocable: true
---

# APDS Setup Assistant

You are helping a developer set up APDS (ANProto Personal Data Server). APDS is a Deno-based personal data server for the ANProto decentralized social protocol.

## Setup Types

Ask the user which type of setup they need:

1. **Server** - Full APDS server with HTTP/WebSocket on port 9000
2. **CLI Client** - Terminal-based chat client
3. **Relay** - Simple WebSocket message relay
4. **Browser Client** - Web application integration

## Prerequisites

Before any setup, verify:
- Deno is installed (`deno --version`)
- If not installed, guide them to https://deno.land/#installation

## Server Setup

For a full APDS server:

```bash
# Clone the repository
git clone https://github.com/evbogue/apds.git
cd apds

# Run the server (port 9000)
deno run -A serve.js
```

The server will:
- Generate a keypair on first run
- Store data in the `apdsv1` namespace
- Expose HTTP endpoints at `http://localhost:9000`
- Accept WebSocket connections for real-time sync

### Server Endpoints
- `GET /all` - All messages
- `GET /latest` - Messages from last 5 minutes
- `GET /<hash>` - Specific message by hash
- `GET /<pubkey>` - Messages from specific author
- `WS /` - WebSocket for real-time sync

### Optional Configuration
```bash
# Set VAPID subject for Web Push (optional)
export VAPID_SUBJECT="mailto:you@example.com"
deno run -A serve.js
```

## CLI Client Setup

For the interactive terminal client:

```bash
# Clone if not already done
git clone https://github.com/evbogue/apds.git
cd apds

# Run CLI with optional namespace
deno run -A cli.js [namespace]
```

### CLI Commands
- `/nick <name>` - Set your display name
- `/connect <url>` - Connect to a WebSocket peer (e.g., `wss://apds.anproto.com`)
- `/clear` - Erase all local data
- `/exit` - Exit the CLI

### Example Session
```
$ deno run -A cli.js
ANPROTO APDS ACTIVATED -- Your pubkey is: abc123...

abc123...> /nick Alice
You are now known as "Alice"

abc123... Alice> /connect wss://apds.anproto.com
Connecting to wss://apds.anproto.com
Connected to: wss://apds.anproto.com

abc123... Alice> Hello, ANProto world!
```

## Relay Setup

For a simple WebSocket relay:

```bash
git clone https://github.com/evbogue/apds.git
cd apds

# Run relay on default port 8080
deno run -A relay.js

# Or specify a custom port
deno run -A relay.js 3000
```

The relay broadcasts all received messages to all connected peers.

## Browser Client Setup

For integrating APDS into a web application:

### Minimal Example
```html
<!DOCTYPE html>
<html>
<head>
  <title>APDS Client</title>
</head>
<body>
  <script type="module">
    import { apds } from './apds.js'

    // Initialize with app namespace
    await apds.start('myapp')

    // Generate keypair if needed
    if (!await apds.pubkey()) {
      const keypair = await apds.generate()
      await apds.put('keypair', keypair)
      console.log('New keypair generated:', await apds.pubkey())
    } else {
      console.log('Using existing keypair:', await apds.pubkey())
    }

    // Set user profile
    await apds.put('name', 'MyUsername')

    // Compose and sign a message
    const hash = await apds.compose('Hello from the browser!')
    console.log('Message published:', hash)

    // Query messages
    const messages = await apds.query()
    console.log('All messages:', messages)
  </script>
</body>
</html>
```

### Required Files
Copy these from the APDS repo to your project:
- `apds.js` - Core library
- `lib/` directory - Supporting libraries

### Connecting to a Server
```javascript
// Connect via WebSocket
const ws = new WebSocket('wss://apds.anproto.com')

ws.onopen = () => {
  console.log('Connected to APDS server')
}

ws.onmessage = async (event) => {
  const data = event.data
  // Store incoming messages
  await apds.make(data)
  await apds.add(data)
}

// Send a message
const hash = await apds.compose('Hello!')
const msg = await apds.get(hash)
ws.send(msg)
```

## Troubleshooting

### Common Issues

1. **Permission errors**: Ensure you're using `-A` flag or specific permissions:
   ```bash
   deno run --allow-net --allow-read --allow-write serve.js
   ```

2. **Port in use**: Change the port in serve.js or use a different port

3. **Connection refused**: Check if the server is running and the URL is correct

4. **No keypair**: First run auto-generates a keypair; check storage permissions

### Verify Installation
```bash
# Check Deno version
deno --version

# Test the server
curl http://localhost:9000/all

# Test WebSocket (requires websocat or similar)
websocat ws://localhost:9000
```

## Next Steps

After setup, you might want to:
- Connect to the public server at `wss://apds.anproto.com`
- Set up Web Push notifications (server mode)
- Build a custom UI using the `composer.js` and `profile.js` examples
- Run multiple instances for testing P2P sync
