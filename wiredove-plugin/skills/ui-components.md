---
description: Use this skill when developers need to work with Wiredove UI components - including message rendering, the composer, routing, navigation, profiles, and the DOM helper. Covers the client-side UI implementation.
---

# Wiredove UI Components

Wiredove's UI is built entirely in JavaScript using a custom DOM helper, with hash-based routing and lazy loading.

## DOM Helper (h)

All UI elements are created using the `h()` helper from APDS:

```javascript
import { h } from 'h'

// Basic element
const div = h('div', {classList: 'message'}, ['Text content'])

// With attributes
const link = h('a', {href: '#author', classList: 'avatarlink'}, ['Click me'])

// With event handlers
const button = h('button', {
  onclick: async () => {
    await doSomething()
  }
}, ['Submit'])

// Nested elements
const card = h('div', {classList: 'card'}, [
  h('span', {classList: 'title'}, ['Title']),
  h('div', {classList: 'content'}, [content])
])
```

## Hash-Based Routing

### Route Patterns (route.js)

```javascript
export const route = async () => {
  const src = window.location.hash.substring(1)
  const scroller = h('div', {id: 'scroller'})
  document.body.appendChild(scroller)

  if (src === '') {
    // Home feed - all messages
    const log = await apds.query()
    adder(log, src, scroller)
  }
  else if (src === 'settings') {
    // Settings page
    scroller.appendChild(await settings())
  }
  else if (src === 'import') {
    // Import data page
    scroller.appendChild(await importBlob())
  }
  else if (src.length < 44 && !src.startsWith('?')) {
    // Username lookup
    const ar = await fetch('https://pub.wiredove.net/' + src)
      .then(r => r.json())
    // Fetch messages for each pubkey...
  }
  else if (src.length === 44) {
    // Author profile (pubkey)
    noteInterest(src)
    const log = await apds.query(src)
    adder(log, src, scroller)
  }
  else if (src.startsWith('?')) {
    // Search query
    const log = await apds.query(src)
    adder(log, src, scroller)
  }
  else if (src.length > 44) {
    // Direct message/blob view
    await apds.add(src)
    await render.blob(src)
  }
}

// Handle navigation
window.onhashchange = async () => {
  document.getElementById('scroller')?.remove()
  await route()
}
```

## Message Rendering

### Message Structure

```html
<div id="{hash}" class="message-wrapper" data-ts="{timestamp}">
  <div class="message-shell">
    <div class="message">
      <span class="message-meta">
        <span class="pubkey">{author:6}</span>
        <a class="material-symbols-outlined">Qr_Code</a>
        <a class="material-symbols-outlined">Share</a>
        <a class="material-symbols-outlined">Code</a>
        <a href="#{hash}">{humanTime}</a>
      </span>
      <div class="message-main">
        <a href="#{author}"><img class="avatar"></a>
        <div class="message-stack">
          <a href="#{author}">{name}</a>
          <div class="message-body">
            <div class="content">{markdown body}</div>
            <div class="message-actions">
              <span class="message-actions-reply">
                <a class="material-symbols-outlined">Chat_Bubble</a>
                <span>{replyCount}</span>
              </span>
              <span class="message-actions-edit">
                <a class="material-symbols-outlined">Edit</a>
                <span class="edit-hint">edit: 2h ago</span>
                <span class="edit-nav">...</span>
              </span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
  <div class="message-replies">
    <!-- Nested replies -->
  </div>
</div>
```

### Render Pipeline

```javascript
// Main entry point
render.blob = async (blob) => {
  const [hash, opened] = await Promise.all([
    apds.hash(blob),
    apds.open(blob)
  ])

  const div = document.getElementById(hash)

  if (opened && div) {
    // Signed message - render with full metadata
    await render.meta(blob, opened, hash, div)
  } else if (div) {
    // Content blob - render body only
    await render.content(hash, blob, div)
  }
}

// Insert in chronological order
render.insertByTimestamp = (container, hash, ts) => {
  const stamp = parseInt(ts, 10)
  const div = render.hash(hash)  // Create placeholder
  div.dataset.ts = stamp.toString()

  const children = Array.from(container.children)
  for (const child of children) {
    if (parseInt(child.dataset.ts, 10) < stamp) {
      container.insertBefore(div, child)
      return div
    }
  }
  container.appendChild(div)
  return div
}
```

## Composer Component

### Basic Composer

```javascript
export const composer = async (sig, options = {}) => {
  const textarea = h('textarea', {placeholder: 'Write a message'})

  const publishButton = h('button', {
    onclick: async (e) => {
      e.target.disabled = true
      e.target.textContent = 'Publishing...'

      const published = await apds.compose(textarea.value, replyObj)
      textarea.value = ''

      const signed = await apds.get(published)
      await ntfy(signed)
      await send(signed)
      await render.blob(signed)
    }
  }, ['Publish'])

  const previewButton = h('button', {
    onclick: async () => {
      preview.innerHTML = await markdown(textarea.value)
      textareaDiv.style.display = 'none'
      previewDiv.style.display = 'block'
    }
  }, ['Preview'])

  // ... rest of composer UI
}
```

### Composer Features

1. **Reply context** - Shows parent message info
2. **Edit mode** - Pre-fills content for editing
3. **Preview** - Markdown preview before publishing
4. **Image upload** - Inline image support
5. **Cancel** - Dismisses without posting

## Profile Components

### Avatar Span

```javascript
import { apds } from 'apds'
import { h } from 'h'

export const avatarSpan = async () => {
  const pubkey = await apds.pubkey()
  const img = await apds.visual(pubkey)
  img.classList = 'avatar'

  // Add image upload functionality
  const input = h('input', {
    type: 'file',
    accept: 'image/*',
    onchange: async (e) => {
      const file = e.target.files[0]
      // Process and store image...
    }
  })

  return h('span', [img, input])
}
```

### Name Span

```javascript
export const nameSpan = async () => {
  const name = await apds.get('name')
  return h('span', [name || 'Anonymous'])
}

export const nameDiv = async () => {
  const name = await apds.get('name')

  const input = h('input', {placeholder: name || 'Name yourself'})
  const saveBtn = h('button', {
    onclick: async () => {
      await apds.put('name', input.value)
      input.placeholder = input.value
      input.value = ''
    }
  }, ['Save'])

  return h('span', [input, saveBtn])
}
```

## Navigation Bar

```javascript
export const navbar = async () => {
  const pubkey = await apds.pubkey()

  const home = h('a', {href: '#'}, [
    h('span', {classList: 'material-symbols-outlined'}, ['Home'])
  ])

  const search = h('a', {
    id: 'search',
    classList: 'material-symbols-outlined',
    onclick: () => {
      // Toggle search input
    }
  }, ['Search'])

  const compose = h('a', {
    onclick: async () => {
      if (pubkey) {
        document.body.appendChild(await composer())
      }
    }
  }, [h('span', {classList: 'material-symbols-outlined'}, ['Add'])])

  const settings = h('a', {href: '#settings'}, [
    h('span', {classList: 'material-symbols-outlined'}, ['Settings'])
  ])

  return h('nav', [home, search, compose, settings])
}
```

## Lazy Loading with IntersectionObserver

### Timestamp Refresh

```javascript
const visibleTimestamps = new Map()
let timestampObserver = null

const observeTimestamp = (element, timestamp) => {
  if (!timestampObserver) {
    timestampObserver = new IntersectionObserver((entries) => {
      entries.forEach(entry => {
        const ts = entry.target.dataset.timestamp
        if (entry.isIntersecting) {
          // Start refreshing
          refreshTimestamp(entry.target, ts)
          const intervalId = setInterval(() => {
            refreshTimestamp(entry.target, ts)
          }, 60000)  // Every minute
          visibleTimestamps.set(entry.target, { intervalId })
        } else {
          // Stop refreshing
          const stored = visibleTimestamps.get(entry.target)
          if (stored) clearInterval(stored.intervalId)
          visibleTimestamps.delete(entry.target)
        }
      })
    })
  }
  element.dataset.timestamp = timestamp
  timestampObserver.observe(element)
}
```

### Reply Loading

```javascript
let replyObserver = null

const observeReplies = (wrapper, parentHash) => {
  if (!replyObserver) {
    replyObserver = new IntersectionObserver((entries) => {
      entries.forEach(entry => {
        if (!entry.isIntersecting) return
        if (entry.target.dataset.repliesLoaded === 'true') return

        const hash = entry.target.dataset.replyParent
        const replies = replyIndex.get(hash) || []

        for (const reply of replies) {
          appendReply(hash, reply.hash, reply.ts)
        }

        entry.target.dataset.repliesLoaded = 'true'
      })
    })
  }
  wrapper.dataset.replyParent = parentHash
  replyObserver.observe(wrapper)
}
```

## QR Code Generation

```javascript
render.qr = (hash, blob, target) => {
  return h('a', {
    classList: 'material-symbols-outlined',
    onclick: () => {
      const qrTarget = target || document.getElementById('qr-target' + hash)
      if (!qrTarget.firstChild) {
        const canvas = document.createElement('canvas')
        qrTarget.appendChild(canvas)

        const darkMode = window.matchMedia('(prefers-color-scheme: dark)').matches
        new QRious({
          element: canvas,
          value: location.href + blob,
          size: 160,
          background: darkMode ? '#222' : '#f8f8f8',
          foreground: darkMode ? '#ccc' : '#444'
        })
      } else {
        qrTarget.innerHTML = ''  // Toggle off
      }
    }
  }, ['Qr_Code'])
}
```

## Material Icons

Common icons used:
- `Home`, `Search`, `Add`, `Settings` - Navigation
- `Share`, `Qr_Code`, `Code` - Message actions
- `Chat_Bubble` - Reply
- `Edit` - Edit message
- `Cancel` - Close/dismiss
- `Subdirectory_Arrow_left` - Reply indicator
- `Chevron_Left`, `Chevron_Right` - Edit navigation
- `notifications`, `notifications_active` - Notifications toggle

## Key Files

- `render.js` - Message rendering (928 lines)
- `composer.js` - Message composition (200 lines)
- `route.js` - URL routing (110 lines)
- `navbar.js` - Navigation bar
- `profile.js` - Avatar and name components
- `adder.js` - Feed pagination
- `markdown.js` - Markdown rendering
- `style.css` - All styling
