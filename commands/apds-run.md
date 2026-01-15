---
description: Start APDS server, CLI, or relay with the correct Deno permissions
user_invocable: true
arguments:
  - name: mode
    description: "Which component to run: server, cli, or relay"
    required: false
    default: "server"
---

# APDS Run Command

Quickly start APDS components with correct Deno permissions.

## Usage

```
/apds-run [mode]
```

## Modes

### server (default)
Starts the full APDS server on port 9000:
```bash
deno run -A serve.js
```

### cli
Starts the interactive CLI client:
```bash
deno run -A cli.js
```

### relay
Starts a WebSocket relay on port 8080:
```bash
deno run -A relay.js
```

## Implementation

Before running, check if the user is in an APDS directory by looking for `apds.js` or `serve.js`.

If not in an APDS directory, offer to:
1. Clone the APDS repo
2. Navigate to an existing APDS directory

Run the appropriate command based on the mode argument.
