# indexed-db-and-dixie

IndexedDB is a low-level API for client-side storage of significant amounts of structured data, including files/blobs. It's a powerful browser API that allows you to store large amounts of data locally, but working with it directly can be quite verbose and complex.

That's where Dexie.js comes in. Dexie is a minimalistic wrapper library around IndexedDB that makes it much more pleasant to work with. It provides a clean, promise-based API that feels more like working with a modern database.

## Why IndexedDB?

- **Large storage capacity**: Unlike localStorage (which is limited to ~5-10MB), IndexedDB can store much larger amounts of data
- **Structured data**: It's a NoSQL database that stores objects, not just strings
- **Indexed queries**: You can create indexes on properties for fast lookups
- **Transactions**: Built-in support for atomic operations
- **Asynchronous**: All operations are async, so they don't block the main thread

## Why Dexie?

The native IndexedDB API is quite verbose. Here's what a simple query looks like with native IndexedDB vs Dexie:

**Native IndexedDB** (verbose):
```javascript
const request = indexedDB.open('myDB', 1);
request.onsuccess = (event) => {
  const db = event.target.result;
  const transaction = db.transaction(['notes'], 'readonly');
  const store = transaction.objectStore('notes');
  const getRequest = store.get(1);
  getRequest.onsuccess = (event) => {
    console.log(event.target.result);
  };
};
```

**Dexie** (clean):
```javascript
const db = new Dexie('myDB');
db.version(1).stores({
  notes: '++id, title, content, createdAt'
});

const note = await db.notes.get(1);
```

## Key Dexie Features

- **Schema definition**: Define your database schema in a clean, declarative way
- **Promise-based**: All operations return promises, making async/await easy
- **TypeScript support**: Great TypeScript definitions available
- **Migration support**: Easy versioning and schema migrations
- **Query building**: Chainable query methods like `.where()`, `.filter()`, `.sort()`

## Common Use Cases

- Offline-first applications that need to store data locally
- Caching large amounts of data from APIs
- Progressive Web Apps (PWAs) that work offline
- Applications that need fast local search and filtering

## Example Setup

```javascript
import Dexie from 'dexie';

const db = new Dexie('NotesDB');

db.version(1).stores({
  notes: '++id, title, content, createdAt, updatedAt'
});

// Add a note
await db.notes.add({
  title: 'My Note',
  content: 'Note content here',
  createdAt: new Date(),
  updatedAt: new Date()
});

// Query notes
const allNotes = await db.notes.toArray();
const recentNotes = await db.notes
  .where('createdAt')
  .above(new Date(Date.now() - 7 * 24 * 60 * 60 * 1000))
  .toArray();
```

Links:

- [MDN - IndexedDB API](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API)
- [Dexie.js Documentation](https://dexie.org/)
- [Dexie.js GitHub](https://github.com/dexie/Dexie.js)

