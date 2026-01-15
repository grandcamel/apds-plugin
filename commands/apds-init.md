---
description: Scaffold a new APDS project with server, client, or full-stack templates
user_invocable: true
arguments:
  - name: template
    description: "Project template: server, client, relay, or fullstack"
    required: false
    default: "fullstack"
  - name: name
    description: "Project name (directory name)"
    required: false
---

# APDS Project Initializer

Initialize a new APDS project with the appropriate template and structure.

## Usage

```
/apds-init [template] [name]
```

## Templates

### fullstack (default)
Complete APDS setup with server, client, and relay:
```
my-apds-app/
├── server/
│   ├── serve.js        # Main server
│   ├── apds.js         # Core library
│   ├── gossip.js       # P2P sync
│   └── db.json         # Initial config
├── client/
│   ├── index.html      # Browser client
│   ├── app.js          # Client logic
│   ├── composer.js     # Message UI
│   ├── profile.js      # Profile UI
│   └── render.js       # Message rendering
├── relay/
│   └── relay.js        # WebSocket relay
├── lib/                # Shared libraries
├── public/
│   └── sw.js           # Service worker
├── deno.json           # Deno config
└── README.md
```

### server
Server-only setup:
```
my-apds-server/
├── serve.js
├── apds.js
├── gossip.js
├── lib/
├── db.json
├── deno.json
└── README.md
```

### client
Browser client only:
```
my-apds-client/
├── index.html
├── app.js
├── apds.js
├── lib/
└── README.md
```

### relay
Simple relay server:
```
my-apds-relay/
├── relay.js
├── apds.js
├── lib/
├── deno.json
└── README.md
```

## Implementation Steps

1. **Ask for project name** if not provided
2. **Create project directory**
3. **Clone or copy APDS source files** based on template
4. **Generate initial configuration**:
   - Create `deno.json` with tasks
   - Create `.gitignore`
   - Create `README.md` with setup instructions
5. **Initialize git repository**
6. **Display next steps**

## Generated Files

### deno.json
```json
{
  "tasks": {
    "server": "deno run -A server/serve.js",
    "client": "deno run -A --watch client/app.js",
    "relay": "deno run -A relay/relay.js",
    "dev": "deno task server"
  }
}
```

### .gitignore
```
# Storage
*.json
!deno.json
!db.json

# Credentials
vapid.json
subscriptions.json

# OS
.DS_Store

# Deno
.deno/
```

### README.md (generated)
```markdown
# {project-name}

APDS-based application.

## Quick Start

# Start the server
deno task server

# Or run directly
deno run -A server/serve.js

## Endpoints

- http://localhost:9000 - Server
- http://localhost:9000/all - All messages
- ws://localhost:9000 - WebSocket

## Configuration

Set VAPID_SUBJECT for Web Push:
export VAPID_SUBJECT="mailto:you@example.com"
```

## Example Session

```
> /apds-init fullstack my-chat-app

Creating APDS project: my-chat-app
Template: fullstack

✓ Created directory: my-chat-app/
✓ Created server files
✓ Created client files
✓ Created relay files
✓ Created shared libraries
✓ Generated deno.json
✓ Generated .gitignore
✓ Generated README.md
✓ Initialized git repository

Next steps:
  cd my-chat-app
  deno task server

Your server will start at http://localhost:9000
```
