# Core Observable API

## Creating Observables

```js
import { observable } from "@legendapp/state"

// Deep nested object
const state$ = observable({
  fname: 'Annyong',
  lname: 'Bluth',
  // Computed function
  name: () => state$.fname.get() + ' ' + state$.lname.get(),
  // Action function
  setName: (name) => {
    const [fname, lname] = name.split(' ')
    state$.assign({ fname, lname })
  }
})

// Individual atoms
const fname$ = observable('Annyong')
const lname$ = observable('Bluth')

// Within React components
function App() {
  const store$ = useObservable({ profile: { name: "hi" } })
  return <Profile profile={store$.profile} />
}
```

## Observable Methods

### get()
Returns the raw value. Inside an observing context, automatically tracks for changes.

```js
const name = state$.profile.name.get()
state$.get(true) // Shallow listener (only keys added/removed)
```

### peek()
Returns the raw value WITHOUT tracking. Use when you don't want re-renders.

```js
const current = state$.name.peek()
```

### set()
Modify at any path. Fills in undefined paths automatically.

```js
state$.text.set("hello")
state$.text.set((prev) => prev + " there")
state$.otherKey.otherProp.set("hi") // Works even if undefined
```

### assign()
Shallow merge, like Object.assign. Batches all changes.

```js
state$.assign({ text: "hi!", text2: "there!" })
```

### delete()
Remove a key from an object or element from an array.

```js
state$.text.delete()
state$[0].delete() // Works on arrays too
```

## Computed Observables

### Functions in observables
Any function becomes a computed observable when accessed via `.get()`.

```js
const state$ = observable({
  fname: 'Annyong',
  lname: 'Bluth',
  fullName: () => state$.fname.get() + ' ' + state$.lname.get()
})

// As function (recomputes each call)
const name1 = state$.fullName()

// As computed observable (caches, recomputes on dependency change)
const name2 = state$.fullName.get()
```

### Root computed
```js
const name$ = observable(() => state$.fname.get() + ' ' + state$.lname.get())
```

### Lookup tables
Functions with a single string parameter create computed observables by key.

```js
const state$ = observable({
  items: {
    test1: { text: 'hi' },
    test2: { text: 'hello' }
  },
  texts: (key) => state$.items[key].text
})

state$.texts['test1'].get() // 'hi'
state$.texts.test1.set('hello') // Goes through to linked observable
```

## Async Observables

Promises initialize to `undefined`, then update when resolved.

```js
const data$ = observable(() => fetch('/api').then(r => r.json()))

observe(() => {
  const data = data$.get() // Activates fetch, updates when resolved
})

// Or await with when()
const data = await when(data$)
```

Check status with `syncState`:
```js
import { syncState } from "@legendapp/state"
const status$ = syncState(data$)
const { isLoaded, error } = status$.get()
```

## Linked Observables

Two-way binding with get/set functions.

```js
import { linked } from "@legendapp/state"

const selected$ = observable([false, false, false])
const selectedAll$ = observable(linked({
  get: () => selected$.every(v => v.get()),
  set: (value) => selected$.forEach(v => v.set(value))
}))
```

With initial value (useful for async):
```js
const data$ = observable(linked({
  get: () => fetch('/api').then(r => r.json()),
  initial: { users: [] }
}))
```

## Events

```js
import { event } from "@legendapp/state"

const onClosed$ = event()
onClosed$.onChange(() => { ... })
onClosed$.fire()
```

## Array Methods

Array looping functions provide observables in callbacks and set up shallow tracking:
- `every`, `filter`, `find`, `findIndex`, `forEach`, `includes`, `join`, `map`, `some`

```js
// filter returns array of observables
state$.todos.filter(todo$ => todo$.completed.get())

// find returns observable (or undefined)
state$.todos.find(todo$ => todo$.id.get() === 1)
```

## Safety

Direct assignment is prevented by default:

```js
state$.text = "hi"        // Error
state$.text.set("hi")     // Correct
state$.obj = {}           // Error
state$.set({ obj: {} })   // Correct
```

Enable `$` shorthand with `enable$GetSet()`:
```js
import { enable$GetSet } from "@legendapp/state/config/enable$GetSet"
enable$GetSet()

state$.text.$ = "hi"      // Shorthand for set()
const val = state$.text.$ // Shorthand for get()
```

## undefined Handling

Observables point to paths, not data. Accessing undefined paths is safe.

```js
const state$ = observable({ user: undefined })
state$.user.profile.name.get()        // Returns undefined, no error
state$.user.profile.name.set("Hi")    // Fills in the object tree
// state$ === { user: { profile: { name: 'Hi' } } }
```

## Observable Hints

```js
import { ObservableHint } from '@legendapp/state'

// Mark as opaque (treat as primitive, no child tracking)
state$.body = ObservableHint.opaque(document.body)

// Mark as plain (no child functions/observables to scan for)
state$.data = ObservableHint.plain(bigObject)
```

## mergeIntoObservable

Deep merge while retaining observables and listeners.

```js
import { mergeIntoObservable } from "@legendapp/state"
mergeIntoObservable(state$, { profile: { name: 'New' } })
```
