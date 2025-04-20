# ⚛️ React + Service Worker (PWA) Setup Guide

A comprehensive guide for implementing Service Workers in React applications, enabling offline support and PWA capabilities.

## 🌐 What is a Service Worker?

A Service Worker is a script (written in JavaScript) that runs in the background, separate from the main browser thread. It gives your web app powerful features like:

- Offline support
- Caching assets and API responses
- Background sync
- Push notifications

Think of it as a programmable network proxy between your app and the internet.

### ⚙️ How Service Workers Work (Basics)

1. **Registration**
   - The browser needs to register the service worker file
   - Usually done in your main `index.js` or `main.tsx`

2. **Installation**
   - First time it's downloaded, the browser runs the install event
   - You can pre-cache files here

3. **Activation**
   - Next, it runs the activate event
   - Clean up old caches during this phase

4. **Fetch Interception**
   - Once active, the service worker can intercept every network request
   - Serve cached assets via the fetch event

## 🌟 Features

- ✅ Complete offline support
- 🚀 Performance optimization with caching strategies
- 🔄 Automatic asset and API caching
- 📱 PWA-ready configuration
- 🌐 Fallback for offline network requests

## 🛠️ Implementation Guide

### 1. Initial Setup

```bash
# Create a new React app
npx create-react-app my-app
cd my-app
```

### 2. Enable Service Worker

In `src/index.js`, replace:
```javascript
serviceWorkerRegistration.unregister();
```
with:
```javascript
import * as serviceWorkerRegistration from './serviceWorkerRegistration';
serviceWorkerRegistration.register();
```

### 3. Custom Service Worker Setup

Create `public/custom-sw.js`:
```javascript
const CACHE_NAME = 'my-react-app-v1';
const urlsToCache = ['/', '/index.html', '/favicon.ico'];

// Install event
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME).then((cache) => cache.addAll(urlsToCache))
  );
});

// Activate event
self.addEventListener('activate', (event) => {
  event.waitUntil(
    caches.keys().then((keys) =>
      Promise.all(
        keys.map((key) => {
          if (key !== CACHE_NAME) {
            return caches.delete(key);
          }
        })
      )
    )
  );
});

// Fetch event
self.addEventListener('fetch', (event) => {
  if (event.request.method !== 'GET') return;
  event.respondWith(
    caches.match(event.request).then((cached) => {
      return (
        cached ||
        fetch(event.request)
          .then((res) => {
            return caches.open(CACHE_NAME).then((cache) => {
              cache.put(event.request, res.clone());
              return res;
            });
          })
          .catch(() => caches.match('/offline.html'))
      );
    })
  );
});
```

### 4. Register Service Worker

Add to `public/index.html`:
```html
<script>
  if ('serviceWorker' in navigator) {
    navigator.serviceWorker.register('/custom-sw.js').then(
      function(reg) {
        console.log('SW registered!', reg);
      },
      function(err) {
        console.error('SW registration failed:', err);
      }
    );
  }
</script>
```

### 5. Auto-Update Configuration

Add to `index.js`:
```javascript
serviceWorkerRegistration.register({
  onUpdate: (registration) => {
    if (window.confirm('New version available. Refresh now?')) {
      registration.waiting?.postMessage({ type: 'SKIP_WAITING' });
      window.location.reload();
    }
  },
});
```

### 6. PWA Configuration

Create `public/manifest.json`:
```json
{
  "short_name": "ReactPWA",
  "name": "React App with PWA Support",
  "icons": [
    {
      "src": "icon-192.png",
      "type": "image/png",
      "sizes": "192x192"
    },
    {
      "src": "icon-512.png",
      "type": "image/png",
      "sizes": "512x512"
    }
  ],
  "start_url": ".",
  "display": "standalone",
  "theme_color": "#000000",
  "background_color": "#ffffff"
}
```

Add to `public/index.html`:
```html
<link rel="manifest" href="%PUBLIC_URL%/manifest.json" />
<link rel="apple-touch-icon" href="%PUBLIC_URL%/icon-192.png" />
<meta name="theme-color" content="#000000" />
```

## 🧪 Testing

```bash
# Build for production
npm run build

# Serve locally
npx serve -s build
```

Visit http://localhost:3000 and test offline functionality.

## 🔧 Development Tools

1. Open Chrome DevTools
2. Navigate to Application → Service Workers
3. Use available controls:
   - Update
   - Unregister
   - Offline mode

## 🗑️ Removing Service Workers

To remove service workers:

```javascript
// Revert to unregister
serviceWorkerRegistration.unregister();

// Force unregister all service workers
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.getRegistrations().then((regs) => {
    regs.forEach((reg) => reg.unregister());
  });
}
```

## ⚠️ Important Notes

- Service Workers require HTTPS in production
- localhost is allowed for development
- External API caching needs CORS configuration
- Service Workers require manual reload for updates
- Create an `offline.html` fallback page in `public/` directory
