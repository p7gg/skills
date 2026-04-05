# Persistence and Sync

## Core Sync System

### configureSynced

Set defaults to reduce duplication:

```js
import { configureSynced } from "@legendapp/state/sync"
import { ObservablePersistLocalStorage } from "@legendapp/state/persist-plugins/local-storage"

const syncPlugin = configureSynced({
  persist: {
    plugin: ObservablePersistLocalStorage
  }
})

// Use with merged options
const state$ = observable(syncPlugin({
  persist: { name: 'test' }
}))
```

### synced

Lazy computed — activates on `get()`.

```js
import { synced } from '@legendapp/state/sync'

const state$ = observable(synced({
  get: () => fetch('https://url.to.get').then(r => r.json()),
  set: ({ value }) =>
    fetch('https://url.to.set', { method: 'POST', body: JSON.stringify(value) }),
  persist: { name: 'test' },
  initial: { numUsers: 0, messages: [] },
  mode: 'set', // 'set' | 'assign' | 'merge' | 'append' | 'prepend'
  debounceSet: 500,
  retry: { infinite: true, backoff: 'exponential', maxDelay: 30 },
  subscribe: ({ refresh, update }) => {
    const unsubscribe = someService.subscribe(data => update(data))
    return unsubscribe
  },
}))
```

### syncObservable

Set up sync after observable creation:

```js
import { syncObservable } from '@legendapp/state/sync'

const state$ = observable({ initial: 'value' })

syncObservable(state$, {
  get: () => fetch('https://url.to.get').then(r => r.json()),
  set: ({ value }) => fetch('https://url.to.set', { method: 'POST', body: JSON.stringify(value) }),
  persist: { name: 'test' }
})
```

### syncState

Check sync status:

```js
import { syncState } from "@legendapp/state"

const status$ = syncState(obs$)
const { isPersistLoaded, isLoaded, error, syncCount, lastSync } = status$.get()

// Methods
await status$.clearPersist()
await status$.sync()
const pending = status$.getPendingChanges()
```

## Persist Plugins

### Local Storage (Web)

```js
import { ObservablePersistLocalStorage } from '@legendapp/state/persist-plugins/local-storage'

syncObservable(state$, {
  persist: { name: "documents", plugin: ObservablePersistLocalStorage }
})
```

### IndexedDB (Web)

```js
import { observablePersistIndexedDB } from "@legendapp/state/persist-plugins/indexeddb"

const persistOptions = configureSynced({
  persist: {
    plugin: observablePersistIndexedDB({
      databaseName: "Legend",
      version: 1,
      tableNames: ["documents", "store"]
    })
  }
})

// Mode 1: Dictionary (each value has id field)
syncObservable(state$, persistOptions({ persist: { name: "documents" } }))

// Mode 2: Single object with itemID
syncObservable(settings$, persistOptions({
  persist: { name: "store", indexedDB: { itemID: "settings" } }
}))
```

### MMKV (React Native)

```js
import { ObservablePersistMMKV } from '@legendapp/state/persist-plugins/mmkv'

syncObservable(state$, {
  persist: { name: "documents", plugin: ObservablePersistMMKV }
})
```

### AsyncStorage (React Native)

```js
import { observablePersistAsyncStorage } from '@legendapp/state/persist-plugins/async-storage'
import AsyncStorage from '@react-native-async-storage/async-storage'

const persistOptions = configureSynced({
  persist: {
    plugin: observablePersistAsyncStorage({ AsyncStorage })
  }
})
syncObservable(state$, persistOptions({ persist: { name: 'store' } }))
```

## syncedFetch

Simple fetch wrapper:

```js
import { syncedFetch } from '@legendapp/state/sync-plugins/fetch'

const state$ = observable(syncedFetch({
  get: 'https://url.to.get',
  set: 'https://url.to.set',
  onSaved: (value) => ({ updatedAt: value.updatedAt })
}))
```

## syncedCrud

Full CRUD operations:

```js
import { syncedCrud } from '@legendapp/state/sync-plugins/crud'

// Single item (get pattern)
const profile$ = observable(syncedCrud({
  get: getProfile,
  create: createProfile,
  update: updateProfile,
  delete: deleteProfile,
}))

// Multiple items (list pattern)
const profiles$ = observable(syncedCrud({
  list: listProfiles,
  create: createProfile,
  update: updateProfile,
  delete: deleteProfile,
}))
```

Key options:
- `as`: `'object'` | `'array'` | `'Map'` | `'value'`
- `fieldCreatedAt` / `fieldUpdatedAt` / `fieldDeleted`
- `updatePartial`: send only changed fields
- `changesSince`: `'all'` | `'last-sync'`
- `generateId`: custom ID generator
- `subscribe`: realtime connection
- `onSaved`: merge server response back
- `onSavedUpdate`: `'createdUpdatedAt'` auto-saves timestamp fields

## syncedKeel

```js
import { syncedKeel } from '@legendapp/state/sync-plugins/keel'
import KSUID from 'ksuid'

const sync = configureSynced(syncedKeel, {
  client,
  persist: { plugin: ObservablePersistLocalStorage, retrySync: true },
  debounceSet: 500,
  retry: { infinite: true },
  changesSince: 'last-sync',
  waitFor: isAuthed$,
})

const messages$ = observable(sync({
  list: queries.getMessages,
  create: mutations.createMessage,
  update: mutations.updateMessage,
  delete: mutations.deleteMessage,
  persist: { name: 'messages' },
}))

// Create with local ID
function addMessage(text) {
  const id = KSUID.randomSync().string
  messages$[id].set({ id, text, createdAt: undefined, updatedAt: undefined })
}
```

## syncedSupabase

```js
import { syncedSupabase, configureSyncedSupabase } from '@legendapp/state/sync-plugins/supabase'

configureSyncedSupabase({ generateId: () => uuidv4() })

const messages$ = observable(syncedSupabase({
  supabase,
  collection: 'messages',
  filter: (select) => select.eq('user_id', uid),
  realtime: { filter: `user_id=eq.${uid}` },
  persist: { name: 'messages', retrySync: true },
  changesSince: 'last-sync',
}))
```

## syncedQuery (TanStack Query)

### React Hook

```tsx
import { useObservableSyncedQuery } from '@legendapp/state/sync-plugins/tanstack-react-query'

function Component() {
  const state$ = useObservableSyncedQuery({
    query: {
      queryKey: ['user'],
      queryFn: () => fetch('/api/user').then(r => r.json()),
    },
    mutation: {
      mutationFn: (variables) =>
        fetch('/api/user', { body: JSON.stringify(variables), method: 'POST' }),
    },
  })

  const state = useValue(state$) // Starts sync
  return <$React.input $value={state$.first_name} />
}
```

### Outside React

```js
import { syncedQuery } from '@legendapp/state/sync-plugins/tanstack-query'
import { QueryClient } from '@tanstack/react-query'

const queryClient = new QueryClient()
const state$ = observable(syncedQuery({
  queryClient,
  query: { queryKey: ['user'], queryFn: () => fetch('/api/user').then(r => r.json()) },
  mutation: { mutationFn: (v) => fetch('/api/user', { body: JSON.stringify(v), method: 'POST' }) },
}))
```

## Transform Data

```js
import { combineTransforms, transformStringifyDates, transformStringifyKeys } from '@legendapp/state/sync'

const state$ = observable(synced({
  get: () => {/* ... */},
  transform: combineTransforms(
    transformStringifyDates(),
    transformStringifyKeys('jsonData', 'messagesArr'),
    {
      load: async (value) => { value.localBool = value.serverOption !== 'no'; return value },
      save: async (value) => { value.serverOption = value.localBool ? 'yes' : 'no'; return value }
    }
  )
}))
```

## Offline-First Pattern

```js
const profile$ = observable(syncedCrud({
  list: () => {/*...*/},
  create: () => {/*...*/},
  update: () => {/*...*/},
  subscribe: ({ refresh }) => realtime.subscribe(() => refresh()),
  persist: { plugin: ObservablePersistLocalStorage, name: 'profile', retrySync: true },
  retry: { infinite: true },
  changesSince: 'last-sync',
  fieldUpdatedAt: 'updatedAt'
}))
```
