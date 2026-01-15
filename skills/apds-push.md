---
description: Set up Web Push notifications for APDS - VAPID keys, service workers, subscriptions, and notification handling
user_invocable: true
---

# APDS Web Push Notifications

You are helping developers set up Web Push notifications for APDS. The server supports push notifications to alert users when new messages arrive.

## Web Push Architecture

```
┌──────────────┐    Subscribe    ┌──────────────┐
│   Browser    │────────────────►│ APDS Server  │
│ (ServiceWorker)                │  (serve.js)  │
└──────────────┘                 └──────────────┘
       ▲                                │
       │                                │ New message arrives
       │         Push Service           │
       │    (Google/Mozilla/Apple)      ▼
       │◄─────────────────────── Send notification
```

## Server Endpoints

APDS exposes these Web Push endpoints:

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/push/vapidPublicKey` | GET | Get VAPID public key for subscription |
| `/push/subscribe` | POST | Store browser push subscription |
| `/push/test` | POST | Send test notification to all subscribers |

## Server Setup

### 1. Configure VAPID Subject

Set an identifier for your server (required for Web Push):

```bash
# Email format
export VAPID_SUBJECT="mailto:admin@example.com"

# Or URL format
export VAPID_SUBJECT="https://your-apds-server.com"

# Run server
deno run -A serve.js
```

### 2. VAPID Keys (Auto-Generated)

The server automatically generates and stores VAPID keys:
- `vapid.json` - Contains public and private keys
- Generated on first `/push/vapidPublicKey` request

## Browser Client Setup

### 1. Register Service Worker

```javascript
// main.js
if ('serviceWorker' in navigator) {
  const registration = await navigator.serviceWorker.register('/sw.js')
  console.log('Service Worker registered:', registration.scope)
}
```

### 2. Get VAPID Public Key

```javascript
async function getVapidKey() {
  const response = await fetch('/push/vapidPublicKey')
  const { publicKey } = await response.json()
  return publicKey
}
```

### 3. Subscribe to Push

```javascript
async function subscribeToPush() {
  const registration = await navigator.serviceWorker.ready

  // Get VAPID key from server
  const vapidKey = await getVapidKey()

  // Convert base64url to Uint8Array
  const applicationServerKey = urlBase64ToUint8Array(vapidKey)

  // Subscribe
  const subscription = await registration.pushManager.subscribe({
    userVisibleOnly: true,
    applicationServerKey
  })

  // Send subscription to server
  await fetch('/push/subscribe', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(subscription)
  })

  console.log('Push subscription active')
  return subscription
}

// Helper: Convert base64url to Uint8Array
function urlBase64ToUint8Array(base64String) {
  const padding = '='.repeat((4 - base64String.length % 4) % 4)
  const base64 = (base64String + padding)
    .replace(/-/g, '+')
    .replace(/_/g, '/')

  const rawData = atob(base64)
  const outputArray = new Uint8Array(rawData.length)

  for (let i = 0; i < rawData.length; ++i) {
    outputArray[i] = rawData.charCodeAt(i)
  }
  return outputArray
}
```

### 4. Create Service Worker

```javascript
// sw.js - Service Worker

self.addEventListener('push', (event) => {
  const data = event.data?.json() || {}

  // APDS sends { type: "latest" } or { type: "anproto", sigs: [...] }
  if (data.type === 'latest') {
    // Fetch latest messages
    event.waitUntil(
      fetchLatestMessages().then(() => {
        return self.registration.showNotification('New APDS Message', {
          body: 'New messages available',
          icon: '/icon.png',
          badge: '/badge.png',
          tag: 'apds-new-message'
        })
      })
    )
  }

  if (data.type === 'anproto' && data.sigs) {
    // Process specific message signatures
    event.waitUntil(
      processMessages(data.sigs).then((messages) => {
        return self.registration.showNotification('APDS Update', {
          body: `${messages.length} new message(s)`,
          icon: '/icon.png',
          data: { sigs: data.sigs }
        })
      })
    )
  }
})

// Handle notification click
self.addEventListener('notificationclick', (event) => {
  event.notification.close()

  event.waitUntil(
    clients.matchAll({ type: 'window' }).then((clientList) => {
      // Focus existing window or open new one
      for (const client of clientList) {
        if (client.url.includes('/') && 'focus' in client) {
          return client.focus()
        }
      }
      return clients.openWindow('/')
    })
  )
})

// Fetch latest from server
async function fetchLatestMessages() {
  const response = await fetch('/latest')
  const messages = await response.json()
  // Cache or process messages
  return messages
}

// Process specific signatures
async function processMessages(sigs) {
  const messages = []
  for (const sig of sigs) {
    const response = await fetch(`/${sig}`)
    if (response.ok) {
      messages.push(await response.json())
    }
  }
  return messages
}
```

## Notification Payload Types

### Type: "latest"
```json
{
  "type": "latest"
}
```
Tells the service worker to fetch `/latest` for recent messages.

### Type: "anproto"
```json
{
  "type": "anproto",
  "sigs": ["signature1==", "signature2=="]
}
```
Contains specific message signatures (batched and throttled).

## Testing Push Notifications

### Send Test Push
```bash
curl -X POST http://localhost:9000/push/test
```

### Check Subscriptions
Subscriptions are stored in `subscriptions.json`:
```json
[
  {
    "endpoint": "https://fcm.googleapis.com/fcm/send/...",
    "keys": {
      "p256dh": "...",
      "auth": "..."
    }
  }
]
```

## Complete Browser Example

```html
<!DOCTYPE html>
<html>
<head>
  <title>APDS Push Demo</title>
</head>
<body>
  <button id="subscribe">Enable Notifications</button>
  <button id="test">Send Test</button>

  <script>
    // Register service worker
    navigator.serviceWorker.register('/sw.js')

    // Subscribe button
    document.getElementById('subscribe').onclick = async () => {
      // Request permission
      const permission = await Notification.requestPermission()
      if (permission !== 'granted') {
        alert('Notification permission denied')
        return
      }

      // Get VAPID key
      const res = await fetch('/push/vapidPublicKey')
      const { publicKey } = await res.json()

      // Subscribe
      const reg = await navigator.serviceWorker.ready
      const sub = await reg.pushManager.subscribe({
        userVisibleOnly: true,
        applicationServerKey: urlBase64ToUint8Array(publicKey)
      })

      // Store subscription on server
      await fetch('/push/subscribe', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(sub)
      })

      alert('Subscribed to push notifications!')
    }

    // Test button
    document.getElementById('test').onclick = async () => {
      await fetch('/push/test', { method: 'POST' })
      alert('Test notification sent!')
    }

    function urlBase64ToUint8Array(base64String) {
      const padding = '='.repeat((4 - base64String.length % 4) % 4)
      const base64 = (base64String + padding)
        .replace(/-/g, '+')
        .replace(/_/g, '/')
      const rawData = atob(base64)
      return Uint8Array.from(rawData, c => c.charCodeAt(0))
    }
  </script>
</body>
</html>
```

## Troubleshooting

### Notifications Not Appearing

1. **Check permission**: `Notification.permission` should be `'granted'`
2. **Check service worker**: Must be registered and active
3. **Check subscription**: Verify stored in `subscriptions.json`
4. **Check VAPID**: Ensure keys match between client and server

### "Push service error"

- VAPID subject may be invalid
- Keys may be corrupted - delete `vapid.json` to regenerate

### Service Worker Not Updating

```javascript
// Force update in development
registration.update()

// Or unregister and re-register
navigator.serviceWorker.getRegistrations().then(registrations => {
  registrations.forEach(r => r.unregister())
})
```

### Subscription Expired

Push subscriptions can expire. Handle in service worker:

```javascript
self.addEventListener('pushsubscriptionchange', (event) => {
  event.waitUntil(
    // Re-subscribe and update server
    self.registration.pushManager.subscribe(event.oldSubscription.options)
      .then(subscription => {
        return fetch('/push/subscribe', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(subscription)
        })
      })
  )
})
```

## Security Considerations

1. **HTTPS Required**: Web Push only works over HTTPS (except localhost)
2. **VAPID Keys**: Keep `vapid.json` private - don't commit to git
3. **Subscription Data**: Contains sensitive endpoint URLs
4. **User Consent**: Always request permission explicitly

## Best Practices

1. **Batch notifications** - Don't spam users with every message
2. **Meaningful content** - Show useful info in notifications
3. **Handle clicks** - Open relevant content when notification clicked
4. **Graceful degradation** - App should work without push
5. **Respect user choice** - Don't repeatedly ask for permission
