# ANProto Plugin Implementation Plan

## Research Summary

The deep analysis revealed:
- **Extremely minimal core library** (37 lines of JavaScript)
- **4 functions**: `gen()`, `hash()`, `sign()`, `open()`
- **Key pain points**: Node.js setup, message parsing, key management, sparse docs
- **Cross-language support**: JS, Go, Rust, Python implementations exist

---

## Phase 1: Deep Research ✅ COMPLETE

See [RESEARCH.md](./RESEARCH.md) for detailed findings.

---

## Phase 2: Plugin Design

### Skills

| Skill ID | Name | Trigger Phrases | Purpose |
|----------|------|-----------------|---------|
| `anproto-explain` | ANProto Explainer | "what is anproto", "explain anproto", "how does anproto work" | Explain protocol concepts and architecture |
| `anproto-keygen` | Key Generator | "generate keypair", "create anproto keys", "new identity" | Guide keypair generation and storage |
| `anproto-sign` | Message Signer | "sign message", "create signature", "anproto sign" | Help with signing workflow |
| `anproto-verify` | Message Verifier | "verify message", "check signature", "anproto open" | Verify and parse signed messages |
| `anproto-debug` | Message Debugger | "debug message", "parse anproto", "message format" | Parse and explain message components |
| `anproto-setup` | Project Setup | "setup anproto", "init anproto", "add anproto" | Initialize ANProto in a project |

### Commands

| Command | Arguments | Purpose |
|---------|-----------|---------|
| `/anproto-init` | `--runtime <deno\|node\|browser>` | Scaffold ANProto project |
| `/anproto-parse` | `<message>` | Parse and explain a message |
| `/anproto-scaffold` | `--lang <js\|go\|rust\|python>` | Generate integration code |

### Agents

| Agent ID | Name | Purpose | Tools |
|----------|------|---------|-------|
| `anproto-explorer` | Codebase Explorer | Analyze ANProto usage in a project | Glob, Grep, Read |
| `anproto-reviewer` | Code Reviewer | Review ANProto implementations | Glob, Grep, Read |

---

## Phase 3: Implementation Details

### Skill: `anproto-explain`

**Description**: Explains ANProto protocol concepts, comparing to alternatives.

**Knowledge Base**:
- Protocol overview (ed25519 + SHA-256 + base64)
- Comparison to ATProto, Nostr, Scuttlebot
- Network-agnostic design philosophy
- Message format specification

### Skill: `anproto-setup`

**Description**: Helps set up ANProto in different runtimes.

**Workflows**:

1. **Deno Setup**
   - Copy `an.js` and `lib/` folder
   - Import directly: `import { an } from './an.js'`

2. **Node.js Setup**
   - Copy files + add crypto polyfills
   - Provide `node_ex.js` template

3. **Browser Setup**
   - Copy files as ES modules
   - Configure bundler if needed

### Skill: `anproto-debug`

**Description**: Parse and explain ANProto message components.

**Parsing Logic**:
```
Keypair (132 chars):
├─ [0-43]   Public Key (44 chars, base64)
└─ [44-131] Secret Key (88 chars, base64)

Signed Message:
├─ [0-43]   Public Key (44 chars)
└─ [44+]    Signed Payload (variable)

Opened Message:
├─ [0-12]   Timestamp (13 digits, Unix ms)
└─ [13+]    Hash (44 chars, base64 SHA-256)
```

### Command: `/anproto-init`

**Template Structure**:
```
project/
├── an.js
├── lib/
│   ├── base64.js
│   └── nacl-fast-es.js
├── index.js (runtime-specific)
└── README.md
```

### Command: `/anproto-scaffold`

**Language Templates**:

- **JavaScript**: Full workflow example with comments
- **Go**: Using `github.com/vic/goan`
- **Rust**: Using `anproto` crate
- **Python**: Using `ANproto-Python`

---

## Phase 4: File Structure

```
anproto-plugin/
├── plugin.json
├── README.md
├── RESEARCH.md
├── PLAN.md
├── skills/
│   ├── anproto-explain.md
│   ├── anproto-keygen.md
│   ├── anproto-sign.md
│   ├── anproto-verify.md
│   ├── anproto-debug.md
│   └── anproto-setup.md
├── commands/
│   ├── anproto-init.md
│   ├── anproto-parse.md
│   └── anproto-scaffold.md
├── agents/
│   ├── anproto-explorer.md
│   └── anproto-reviewer.md
└── templates/
    ├── deno/
    ├── node/
    ├── browser/
    ├── go/
    ├── rust/
    └── python/
```

---

## Phase 5: Implementation Priority

### P0 - Core (Week 1)
1. `plugin.json` manifest
2. `anproto-explain` skill
3. `anproto-setup` skill
4. `/anproto-init` command

### P1 - Essential (Week 2)
1. `anproto-debug` skill
2. `/anproto-parse` command
3. `anproto-keygen` skill
4. `anproto-sign` skill
5. `anproto-verify` skill

### P2 - Extended (Week 3)
1. `/anproto-scaffold` command
2. `anproto-explorer` agent
3. `anproto-reviewer` agent
4. Templates for Go/Rust/Python

---

## Open Questions (Resolved)

| Question | Decision |
|----------|----------|
| Live crypto operations? | No - guide users, don't execute crypto |
| Which runtimes first? | Deno > Node > Browser |
| Cross-language priority? | JS first, then Go/Rust/Python |

---

## Success Criteria

1. Developer can set up ANProto in < 5 minutes
2. Message debugging provides clear, actionable output
3. Cross-language scaffolding works correctly
4. Skills trigger appropriately on natural language
