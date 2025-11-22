# offline-web-app-with-inertia-and-react

Building an offline-first web application with Laravel (Inertia.js) and React was an interesting challenge. The goal was to create a note-taking app that works seamlessly offline and syncs when the connection is restored.

## The Stack

- **Backend**: Laravel (PHP)
- **Frontend**: React with Inertia.js
- **Offline Storage**: IndexedDB with Dexie.js
- **Service Worker**: For caching and offline support
- **PWA**: Web App Manifest for installability

## Key Challenges

### 1. Dual Storage Strategy

The app needs to work with two data sources:
- **Local (IndexedDB)**: For immediate access and offline functionality
- **Remote (Laravel API)**: For persistence and synchronization

The trick is keeping them in sync without creating conflicts or losing data.

### 2. Service Worker Integration

The service worker needs to:
- Cache the app shell (HTML, CSS, JS)
- Cache API responses for offline access
- Handle background sync when connection is restored
- Use appropriate caching strategies (cache-first for assets, network-first for data)

### 3. Inertia.js Considerations

Inertia.js is designed for server-rendered apps, which means:
- Initial page loads come from the server
- Subsequent navigation uses AJAX requests
- We need to ensure the service worker caches these responses properly

## Implementation Approach

### Local-First Architecture

1. **Write to IndexedDB first**: All user actions (create, update, delete) immediately write to IndexedDB
2. **Queue for sync**: Changes are queued for synchronization when online
3. **Background sync**: Use the Background Sync API to sync when connection is restored
4. **Conflict resolution**: Handle cases where local and remote data differ

### Service Worker Strategy

```javascript
// Cache-first for static assets
workbox.strategies.cacheFirst({
  cacheName: 'app-shell',
  plugins: [/* ... */]
});

// Network-first for API calls
workbox.strategies.networkFirst({
  cacheName: 'api-cache',
  plugins: [/* ... */]
});
```

### Sync Queue Pattern

```javascript
// When offline, queue operations
const syncQueue = [];

async function createNote(note) {
  // Write to IndexedDB immediately
  await db.notes.add(note);
  
  // Queue for sync
  syncQueue.push({
    type: 'create',
    data: note,
    timestamp: Date.now()
  });
  
  // Try to sync if online
  if (navigator.onLine) {
    await syncQueue();
  }
}
```

## Lessons Learned

1. **User experience matters**: Users should never feel like they're "offline" - the app should work seamlessly
2. **Sync conflicts are hard**: Deciding which version wins in a conflict requires careful consideration
3. **Service workers are powerful but complex**: They can do a lot, but debugging can be tricky
4. **IndexedDB is your friend**: For storing large amounts of structured data locally, it's the best option

## Real-World Application

I built this as part of the [proesc-notes](https://github.com/lumamontes/proesc-notes) project - a note-taking app for collaborators. The app demonstrates:
- Offline-first architecture
- Service worker implementation
- IndexedDB usage with Dexie
- Laravel + Inertia.js + React integration
- PWA features (installability, offline support)

Links:

- [Proesc Notes Repository](https://github.com/lumamontes/proesc-notes)
- [Inertia.js Documentation](https://inertiajs.com/)
- [MDN - Service Worker API](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API)
- [MDN - Background Sync API](https://developer.mozilla.org/en-US/docs/Web/API/Background_Sync_API)

