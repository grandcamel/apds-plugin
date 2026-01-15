---
name: anproto-init
description: Initialize ANProto in a project with proper setup for Deno, Node.js, or browser
version: "1.1.0"
author: "grandcamel"
license: "MIT"
category: "setup"
keywords:
  - anproto
  - initialize
  - setup
  - deno
  - nodejs
  - browser
---

# ANProto Project Initialization

You are helping the user set up ANProto in their project. Guide them through the process conversationally.

## Step 1: Detect Runtime Environment

First, check what kind of project this is:

```bash
# Check for Deno
ls deno.json deno.jsonc 2>/dev/null && echo "Deno project detected"

# Check for Node.js
ls package.json 2>/dev/null && echo "Node.js project detected"

# Check for browser/HTML
ls index.html 2>/dev/null && echo "Browser project detected"
```

Ask the user which runtime they're targeting if it's not clear:

"I'll help you set up ANProto in your project. Which runtime are you using?
1. **Deno** - Simplest setup, native ES modules
2. **Node.js** - Requires crypto polyfills
3. **Browser** - Direct ES modules or bundled"

## Step 2: Create Directory Structure

```bash
mkdir -p lib
```

## Step 3: Download ANProto Files

```bash
# Core library
curl -sL -o an.js https://raw.githubusercontent.com/evbogue/anproto/main/an.js

# Dependencies
curl -sL -o lib/base64.js https://raw.githubusercontent.com/evbogue/anproto/main/lib/base64.js
curl -sL -o lib/nacl-fast-es.js https://raw.githubusercontent.com/evbogue/anproto/main/lib/nacl-fast-es.js
```

## Step 4: Runtime-Specific Setup

### For Node.js Only

Create the polyfill wrapper:

```bash
cat > an-node.js << 'EOF'
// an-node.js - Node.js wrapper with crypto polyfills
import { createRequire } from 'module';
const require = createRequire(import.meta.url);

globalThis.self = {
  crypto: {
    getRandomValues: (buf) => {
      require('crypto').randomFillSync(buf);
      return buf;
    }
  }
};

if (typeof globalThis.crypto === 'undefined' || typeof globalThis.crypto.subtle === 'undefined') {
  globalThis.crypto = globalThis.crypto || {};
  globalThis.crypto.subtle = require('crypto').webcrypto.subtle;
}

export { an } from './an.js';
EOF
```

Check/update package.json for ES modules:

```bash
# Check if package.json exists and has type: module
node -e "const p=require('./package.json'); if(p.type!=='module') console.log('Add type:module to package.json')" 2>/dev/null || echo "No package.json found"
```

If needed, update package.json:

```bash
# If package.json exists but missing type: module
node -e "
const fs = require('fs');
const pkg = JSON.parse(fs.readFileSync('package.json'));
if (pkg.type !== 'module') {
  pkg.type = 'module';
  fs.writeFileSync('package.json', JSON.stringify(pkg, null, 2));
  console.log('Added type: module to package.json');
}
" 2>/dev/null
```

## Step 5: Create Example File

### Deno Example

```bash
cat > anproto-example.ts << 'EOF'
import { an } from './an.js'

async function main() {
  // Generate a keypair
  const keypair = await an.gen()
  console.log('Generated keypair')
  console.log('Public key:', keypair.substring(0, 44))

  // Hash some content
  const content = "Hello, ANProto!"
  const hash = await an.hash(content)
  console.log('Content hash:', hash)

  // Sign the hash
  const signed = await an.sign(hash, keypair)
  console.log('Signed message:', signed.substring(0, 60) + '...')

  // Verify and open
  const opened = await an.open(signed)
  console.log('Verification:', opened ? 'SUCCESS' : 'FAILED')

  if (opened) {
    console.log('Timestamp:', opened.substring(0, 13))
    console.log('Hash matches:', opened.substring(13) === hash)
  }
}

main()
EOF

echo "Run with: deno run anproto-example.ts"
```

### Node.js Example

```bash
cat > anproto-example.js << 'EOF'
import { an } from './an-node.js'

async function main() {
  // Generate a keypair
  const keypair = await an.gen()
  console.log('Generated keypair')
  console.log('Public key:', keypair.substring(0, 44))

  // Hash some content
  const content = "Hello, ANProto!"
  const hash = await an.hash(content)
  console.log('Content hash:', hash)

  // Sign the hash
  const signed = await an.sign(hash, keypair)
  console.log('Signed message:', signed.substring(0, 60) + '...')

  // Verify and open
  const opened = await an.open(signed)
  console.log('Verification:', opened ? 'SUCCESS' : 'FAILED')

  if (opened) {
    console.log('Timestamp:', opened.substring(0, 13))
    console.log('Hash matches:', opened.substring(13) === hash)
  }
}

main()
EOF

echo "Run with: node anproto-example.js"
```

### Browser Example

```bash
cat > index.html << 'EOF'
<!DOCTYPE html>
<html>
<head>
  <title>ANProto Demo</title>
  <style>
    body { font-family: monospace; padding: 20px; }
    pre { background: #f0f0f0; padding: 10px; }
    button { margin: 10px 0; padding: 10px 20px; }
  </style>
</head>
<body>
  <h1>ANProto Demo</h1>
  <button id="run">Run Demo</button>
  <pre id="output">Click "Run Demo" to start</pre>

  <script type="module">
    import { an } from './an.js'

    const output = document.getElementById('output')

    document.getElementById('run').onclick = async () => {
      output.textContent = 'Running...\n'

      const keypair = await an.gen()
      output.textContent += 'Generated keypair\n'
      output.textContent += 'Public key: ' + keypair.substring(0, 44) + '\n'

      const content = "Hello, ANProto!"
      const hash = await an.hash(content)
      output.textContent += 'Content hash: ' + hash + '\n'

      const signed = await an.sign(hash, keypair)
      output.textContent += 'Signed (truncated): ' + signed.substring(0, 60) + '...\n'

      const opened = await an.open(signed)
      output.textContent += 'Verification: ' + (opened ? 'SUCCESS' : 'FAILED') + '\n'

      if (opened) {
        output.textContent += 'Timestamp: ' + opened.substring(0, 13) + '\n'
        output.textContent += 'Hash matches: ' + (opened.substring(13) === hash) + '\n'
      }
    }
  </script>
</body>
</html>
EOF

echo "Open index.html in a browser (may need a local server for ES modules)"
```

## Step 6: Verify Installation

Run the example to verify everything works:

**Deno:**
```bash
deno run anproto-example.ts
```

**Node.js:**
```bash
node anproto-example.js
```

**Browser:**
```bash
# Start a simple server
python3 -m http.server 8000
# Then open http://localhost:8000 in browser
```

## Step 7: Add to .gitignore

Remind the user about security:

```bash
echo "# ANProto secrets" >> .gitignore
echo ".env" >> .gitignore
echo ".keys/" >> .gitignore
echo "*.key" >> .gitignore
```

## Completion Message

"ANProto is now set up in your project! Here's what was created:

```
./
├── an.js              # Core ANProto library
├── an-node.js         # Node.js wrapper (if applicable)
├── lib/
│   ├── base64.js      # Base64 encoding
│   └── nacl-fast-es.js # TweetNaCl crypto
└── anproto-example.*  # Example file for your runtime
```

**Next steps:**
- Run the example to verify the setup
- Generate a keypair for your application
- Store keypairs securely (environment variables or key files)

Need help? Just ask about signing, verifying, or key management!"
