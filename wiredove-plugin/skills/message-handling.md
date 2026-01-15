---
description: Use this skill when developers need to understand message handling in Wiredove - including message creation, verification, rendering, replies, edits, and the message lifecycle. Covers the complete flow from composition to display.
---

# Message Handling in Wiredove

Messages in Wiredove are cryptographically signed YAML documents distributed via P2P and WebSocket networks.

## Message Structure

### Signed Message (Signature Blob)
```
[author_pubkey:44][signature:86][timestamp:13][content_hash:43]
```
- **author_pubkey**: 44-character base64 public key
- **signature**: 86-character cryptographic signature
- **timestamp**: 13-digit Unix timestamp (milliseconds)
- **content_hash**: 43-44 character hash of content

### Content Blob (YAML)
```yaml
name: "Author Name"
image: "base64_hash_of_avatar"
body: "Message content in **markdown**"
previous: "hash_of_authors_previous_message"
reply: "hash_of_parent_if_this_is_reply"
replyto: "pubkey_of_parent_author"
edit: "hash_of_original_if_this_is_edit"
```

## Creating Messages

### Basic Message
```javascript
import { apds } from 'apds'

const hash = await apds.compose('Hello, Wiredove!')
const signed = await apds.get(hash)  // Get the signature blob
```

### Reply to Message
```javascript
const replyOptions = {
  reply: parentMessageHash,
  replyto: parentAuthorPubkey
}

const hash = await apds.compose('This is my reply', replyOptions)
```

### Edit a Message
```javascript
const editOptions = {
  edit: originalMessageHash
}

const hash = await apds.compose('Updated content', editOptions)
```

### Full Composition Flow (from composer.js)
```javascript
const publish = async (text, options = {}) => {
  // Create signed message
  const hash = await apds.compose(text, options)
  const signed = await apds.get(hash)
  const opened = await apds.open(signed)
  const content = await apds.get(opened.substring(13))

  // Broadcast via ntfy.sh
  await ntfy(signed)
  await ntfy(content)

  // Queue for network distribution
  await send(signed)
  await send(content)

  // Handle embedded images
  const images = content.match(/!\[.*?\]\((.*?)\)/g)
  if (images) {
    for (const img of images) {
      const src = img.match(/!\[.*?\]\((.*?)\)/)[1]
      const imgBlob = await apds.get(src)
      if (imgBlob) { await send(imgBlob) }
    }
  }

  // Render locally
  await render.blob(signed)
}
```

## Receiving Messages

### Processing Incoming Messages
```javascript
const processIncoming = async (blob) => {
  // 1. Remove from pending queue (avoid duplicates)
  noteReceived(blob)

  // 2. Check if we should render it
  await render.shouldWe(blob)

  // 3. Verify and store
  await apds.make(blob)

  // 4. Add to author's message log
  await apds.add(blob)

  // 5. Render in UI
  await render.blob(blob)
}
```

### Determine if Message Should Display
```javascript
// From render.js - shouldWe()
const shouldWe = async (blob) => {
  const [opened, hash] = await Promise.all([
    apds.open(blob),
    apds.hash(blob)
  ])

  if (!opened) return  // Invalid signature

  // Already in DOM?
  if (document.getElementById(hash)) return

  // Track activity
  noteSeen(blob.substring(0, 44))

  // Get current view context
  const src = window.location.hash.substring(1)

  // Check if this message belongs in current view
  const authorKey = blob.substring(0, 44)
  const yaml = await apds.parseYaml(await apds.get(opened.substring(13)))

  // Handle replies
  if (yaml && yaml.reply) {
    // Add to parent's reply list
    indexReply(yaml.reply, hash, timestamp)
    return
  }

  // Insert in feed if viewing this author or all
  if (src === '' || src === authorKey) {
    const ts = opened.substring(0, 13)
    insertByTimestamp(scroller, hash, ts)
    await render.blob(blob)
  }
}
```

## Message Verification

```javascript
const verifyMessage = async (signed) => {
  // Open verifies signature and returns timestamp+hash
  const opened = await apds.open(signed)

  if (!opened) {
    console.log('Invalid signature')
    return null
  }

  const timestamp = opened.substring(0, 13)
  const contentHash = opened.substring(13)
  const author = signed.substring(0, 44)

  return { timestamp, contentHash, author }
}
```

## Rendering Messages

### Main Render Entry
```javascript
render.blob = async (blob) => {
  const [hash, opened] = await Promise.all([
    apds.hash(blob),
    apds.open(blob)
  ])

  const div = document.getElementById(hash)

  if (opened && div) {
    // It's a signed message - render with metadata
    await render.meta(blob, opened, hash, div)
  } else if (div) {
    // It's content (YAML) - render body
    await render.content(hash, blob, div)
  }
}
```

### Render Message Metadata
```javascript
render.meta = async (blob, opened, hash, div) => {
  const timestamp = opened.substring(0, 13)
  const contentHash = opened.substring(13)
  const author = blob.substring(0, 44)

  const [humanTime, contentBlob, img] = await Promise.all([
    apds.human(timestamp),
    apds.get(contentHash),
    apds.visual(author)  // Generated avatar
  ])

  const yaml = await apds.parseYaml(contentBlob)

  // Check if this is an edit
  if (yaml && yaml.edit) {
    // Render as edit indicator
    renderEditMessage(...)
    return
  }

  // Build message UI
  const message = h('div', {classList: 'message'}, [
    buildMetaRow(author, hash, timestamp),
    h('div', {classList: 'message-main'}, [
      avatarLink(author, img),
      h('div', {classList: 'message-stack'}, [
        nameLink(author, yaml.name),
        contentDiv(contentBlob, yaml)
      ])
    ])
  ])

  div.replaceWith(message)
}
```

## Reply Threading

### Build Reply Index
```javascript
const replyIndex = new Map()

const buildReplyIndex = async () => {
  const log = await apds.getOpenedLog()

  for (const msg of log) {
    const yaml = await apds.parseYaml(msg.text)
    const parent = yaml?.reply || yaml?.replyHash
    if (parent) {
      const list = replyIndex.get(parent) || []
      list.push({ hash: msg.hash, ts: msg.ts })
      replyIndex.set(parent, list)
    }
  }
}
```

### Lazy Load Replies
```javascript
// Use IntersectionObserver for lazy loading
const observeReplies = (wrapper, parentHash) => {
  replyObserver.observe(wrapper)
  wrapper.dataset.replyParent = parentHash
}

// When visible, load replies
replyObserver = new IntersectionObserver((entries) => {
  entries.forEach(async entry => {
    if (!entry.isIntersecting) return
    if (entry.target.dataset.repliesLoaded) return

    const parentHash = entry.target.dataset.replyParent
    const replies = replyIndex.get(parentHash) || []

    for (const reply of replies) {
      await appendReply(parentHash, reply.hash, reply.ts)
    }

    entry.target.dataset.repliesLoaded = 'true'
  })
})
```

## Edit History

### Track Edits
```javascript
const fetchEditsForMessage = async (hash, author) => {
  const log = await apds.query(author)
  const edits = []

  for (const msg of log) {
    const yaml = await apds.parseYaml(msg.text)
    if (yaml && yaml.edit === hash) {
      edits.push({
        hash: msg.hash,
        author: msg.author,
        ts: msg.ts,
        yaml
      })
    }
  }

  return edits.sort((a, b) => a.ts - b.ts)
}
```

### Navigate Edit History
```javascript
render.stepEdit = async (hash, delta) => {
  const state = getEditState(hash)
  const edits = await fetchEditsForMessage(hash, state.author)
  const total = edits.length + 1  // +1 for original

  const nextIndex = Math.max(0, Math.min(
    state.currentIndex + delta,
    total - 1
  ))

  state.currentIndex = nextIndex
  await render.refreshEdits(hash)
}
```

## Key Files

- `composer.js` - Message creation UI
- `render.js` - All rendering logic
- `send.js` / `network_queue.js` - Network distribution
- `gossip.js:51-58` - P2P message reception
- `websocket.js:112-126` - WebSocket message reception
