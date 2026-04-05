# Performance Guide

Legend-State is already optimized by default, but follow these patterns for best results.

## Batching

Multiple changes in a row can trigger multiple re-renders. Batch them.

```js
import { batch } from "@legendapp/state"

const state$ = observable({ items: [] })

function addItems() {
  for (let i = 0; i < 1000; i++) {
    state$.items.push({ text: `Item ${i}` })
  }
}

// Bad: can render 1000 times
addItems()

// Good: renders once
batch(addItems)
```

### When persisting

If using `synced` or `syncObservable`, batching prevents excessive writes to storage. Pushing to an array 1000 times could save 1000 times without batching.

## Iterating Creates Proxies

Accessing objects/arrays in observables creates Proxies. For large data that doesn't need tracking, call `.get()` first.

```js
const state$ = observable({ items: [{ data: { value: 10 }}, ...] })

// Bad: creates proxies for each element
state$.items.forEach(item => sum += item.data.value.get())

// Good: no proxy creation
state$.items.get().forEach(item => sum += item.data.value)
```

## Array Optimizations

### Unique id required

Arrays of objects need `id` or `key` fields for optimized rendering.

```js
const data$ = observable({
  arr: [],
  arr_keyExtractor: (item) => item.idObject._id,
})
```

Legend-State uses ids to handle elements being moved and keeps observables as stable references.

### Use the For component

`For` extracts array items into separate tracking contexts so the parent doesn't re-render.

```jsx
import { For } from "@legendapp/state/react"

function Row({ item$ }) {
  return <div>{item$.text}</div>
}

function List() {
  return <For each={state$.arr} item={Row} />
}
```

### Don't get() while mapping

`map` sets up a shallow listener. Don't access observable properties while mapping — use `peek()` for keys.

```jsx
function List() {
  return state$.arr.map((item) => (
    <Row key={item.peek().id} item={item} />
  ))
}
```

### Optimized rendering

The `optimized` prop reuses React nodes instead of replacing them. Best for drag/drop, sorting, swapping.

```jsx
<For each={list} item={Row} optimized />
```

May have unexpected behavior with animations or external DOM manipulation.

## Why Legend-State Is Fast

- **Proxy to path**: Each proxy node stores minimal metadata, never modifies underlying data
- **Listeners at each node**: `Set` of listeners per node, only affected callbacks fire
- **Granular renders**: Components render only when their specific state changes
- **Shallow listeners**: Parent components don't re-render when children change
- **Minimal closures**: Observable functions created only once
- **No boilerplate**: Smaller bundle, faster apps
