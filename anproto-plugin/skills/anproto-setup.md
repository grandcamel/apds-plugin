---
name: anproto-setup
description: Use this skill when the user asks to "setup anproto", "init anproto", "add anproto to project", "install anproto", or needs help integrating ANProto into a Deno, Node.js, or browser project.
version: "1.1.0"
author: "grandcamel"
license: "MIT"
allowed-tools:
  - Bash
  - Read
  - Write
  - Glob
---

# ANProto Project Setup

Help users integrate ANProto into their projects for different JavaScript runtimes.

## Runtime Detection

First, identify the user's runtime environment:

1. **Deno**: Look for `deno.json`, `deno.jsonc`, or `.ts` files with Deno imports
2. **Node.js**: Look for `package.json`, `node_modules/`
3. **Browser**: Look for `index.html`, bundler configs (webpack, vite, etc.)

## Deno Setup (Simplest)

Deno works directly with ANProto's ES modules.

### Step 1: Copy the library files

```bash
# Create lib directory
mkdir -p lib

# Download files from the ANProto repo
curl -o an.js https://raw.githubusercontent.com/evbogue/anproto/main/an.js
curl -o lib/base64.js https://raw.githubusercontent.com/evbogue/anproto/main/lib/base64.js
curl -o lib/nacl-fast-es.js https://raw.githubusercontent.com/evbogue/anproto/main/lib/nacl-fast-es.js
```

### Step 2: Use in your code

```typescript
import { an } from './an.js'

const keypair = await an.gen()
console.log('Public key:', keypair.substring(0, 44))
```

### Step 3: Run with Deno

```bash
deno run --allow-read your-script.ts
```

## Node.js Setup (Requires Polyfills)

Node.js needs crypto polyfills for Web Crypto API compatibility.

### Step 1: Copy library files (same as Deno)

```bash
mkdir -p lib
curl -o an.js https://raw.githubusercontent.com/evbogue/anproto/main/an.js
curl -o lib/base64.js https://raw.githubusercontent.com/evbogue/anproto/main/lib/base64.js
curl -o lib/nacl-fast-es.js https://raw.githubusercontent.com/evbogue/anproto/main/lib/nacl-fast-es.js
```

### Step 2: Create the polyfill wrapper

Create `an-node.js`:

```javascript
// an-node.js - Node.js wrapper with crypto polyfills
import { createRequire } from 'module';
const require = createRequire(import.meta.url);

// Polyfill for nacl's getRandomValues
globalThis.self = {
  crypto: {
    getRandomValues: (buf) => {
      require('crypto').randomFillSync(buf);
      return buf;
    }
  }
};

// Polyfill for Web Crypto API
if (typeof globalThis.crypto === 'undefined' || typeof globalThis.crypto.subtle === 'undefined') {
  globalThis.crypto = globalThis.crypto || {};
  globalThis.crypto.subtle = require('crypto').webcrypto.subtle;
}

// Re-export the main module
export { an } from './an.js';
```

### Step 3: Update package.json

Ensure your `package.json` has:

```json
{
  "type": "module"
}
```

### Step 4: Use in your code

```javascript
import { an } from './an-node.js'

const keypair = await an.gen()
console.log('Public key:', keypair.substring(0, 44))
```

## Browser Setup

For browsers, ANProto works as native ES modules.

### Option A: Direct ES Modules (No Bundler)

```html
<!DOCTYPE html>
<html>
<head>
  <title>ANProto App</title>
</head>
<body>
  <script type="module">
    import { an } from './an.js'

    async function init() {
      const keypair = await an.gen()
      console.log('Generated keypair')

      const hash = await an.hash('Hello World')
      console.log('Hash:', hash)
    }

    init()
  </script>
</body>
</html>
```

### Option B: With Vite/Bundler

1. Copy the library files to your `src/lib/` directory
2. Import normally:

```javascript
import { an } from './lib/an.js'
```

3. Vite and most modern bundlers handle ES modules automatically

## Verification

After setup, verify the installation works:

```javascript
import { an } from './an.js'  // or './an-node.js' for Node

async function verify() {
  try {
    // Test key generation
    const keypair = await an.gen()
    console.log('✓ Key generation works')
    console.log('  Keypair length:', keypair.length, '(should be 132)')

    // Test hashing
    const hash = await an.hash('test')
    console.log('✓ Hashing works')
    console.log('  Hash length:', hash.length, '(should be 44)')

    // Test signing
    const signed = await an.sign(hash, keypair)
    console.log('✓ Signing works')

    // Test verification
    const opened = await an.open(signed)
    console.log('✓ Verification works')
    console.log('  Timestamp:', opened.substring(0, 13))
    console.log('  Hash:', opened.substring(13))

    console.log('\n✓ ANProto setup complete!')
  } catch (error) {
    console.error('✗ Setup failed:', error.message)
  }
}

verify()
```

## Troubleshooting

| Error | Cause | Solution |
|-------|-------|----------|
| `crypto is not defined` | Missing Web Crypto polyfill | Use `an-node.js` wrapper in Node.js |
| `self is not defined` | NaCl expects browser globals | Add `globalThis.self` polyfill |
| `Cannot use import statement` | CommonJS mode | Add `"type": "module"` to package.json |
| `ERR_MODULE_NOT_FOUND` | Wrong file extension | Ensure `.js` extension in imports |

## File Structure After Setup

```
your-project/
├── an.js              # Core ANProto library
├── an-node.js         # Node.js wrapper (if using Node)
├── lib/
│   ├── base64.js      # Base64 encoding
│   └── nacl-fast-es.js # TweetNaCl crypto
├── package.json       # With "type": "module"
└── your-app.js        # Your application
```
