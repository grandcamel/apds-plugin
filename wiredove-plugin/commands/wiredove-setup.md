---
description: Initialize a Wiredove development environment. Clones the repository, sets up dependencies, and starts the development server.
allowed-tools: Bash, Read, Write, Glob, Grep, AskUserQuestion
---

# Wiredove Development Environment Setup

You are setting up a Wiredove development environment. Follow these steps:

## Step 1: Determine Setup Location

Ask the user where they want to set up the Wiredove project:

```
Where would you like to set up the Wiredove project?
- Current directory (clone into ./wiredove)
- Specify a custom path
```

## Step 2: Check Prerequisites

Verify the following are available:
- `git` - for cloning the repository
- `node` / `npx` - for serving locally (optional)
- `docker` - for Docker-based setup (optional)

Run these checks:
```bash
git --version
node --version 2>/dev/null || echo "Node.js not installed (optional)"
docker --version 2>/dev/null || echo "Docker not installed (optional)"
```

## Step 3: Clone Repository

If the wiredove directory doesn't exist at the target location:
```bash
git clone https://github.com/evbogue/wiredove.git [target-path]
```

If it already exists, ask if they want to pull latest changes.

## Step 4: Choose Run Method

Ask the user how they want to run Wiredove:

**Option 1: Docker (Recommended)**
```bash
cd [wiredove-path]
docker compose up --build
```
- Runs on http://localhost:8000
- No Node.js required
- Isolated environment

**Option 2: npx serve**
```bash
cd [wiredove-path]
npx serve .
```
- Runs on http://localhost:3000
- Requires Node.js
- Quick and simple

**Option 3: Just clone (manual setup)**
- Clone only, user will set up manually

## Step 5: Verify Setup

After starting the server:
1. Confirm the server is running
2. Provide the URL to access Wiredove
3. Explain next steps for development

## Step 6: Development Tips

After setup, provide these tips:

### Key Files to Explore
- `app.js` - Entry point
- `render.js` - Message rendering (largest file)
- `composer.js` - Message composition
- `sync.js` - Feed synchronization
- `gossip.js` - P2P connections

### Making Changes
- All code is vanilla JavaScript with ES modules
- No build step required - just edit and refresh
- Uses importmap in `index.html` for dependencies

### Testing Locally
- Open multiple browser tabs to test P2P
- Use different browser profiles for different identities
- Check browser console for connection logs

### External Services
- **Pub server**: wss://pub.wiredove.net/
- **Notifications**: https://ntfy.sh/wiredove
- **Directory**: https://pub.wiredove.net/[username]

## Output Format

Provide clear status updates at each step:
```
[1/6] Checking prerequisites...
[2/6] Cloning repository...
[3/6] Setting up environment...
...
```

End with a summary:
```
Wiredove is ready for development!

Location: /path/to/wiredove
Server: http://localhost:8000
Run command: docker compose up

Next steps:
- Open http://localhost:8000 in your browser
- Generate a keypair in Settings
- Start exploring the codebase!
```
