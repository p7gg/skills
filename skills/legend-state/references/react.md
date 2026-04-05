# React Integration

## Primary Pattern: useValue

**This is the recommended way to consume observables in React v3.** Required for React Compiler compatibility.

```jsx
import { useValue, useObservable } from "@legendapp/state/react"

function Component() {
  // Track a single observable
  const theme = useValue(store$.settings.theme)

  // Track a computation function
  const completedCount = useValue(() =>
    store$.todos.get().filter(t => t.completed).length
  )

  // Local observable
  const local$ = useObservable({ count: 0 })
  const count = useValue(local$.count)

  // Computed local observable
  const fullname$ = useObservable(() =>
    `${local$.first.get()} ${local$.last.get()}`
  )

  return <div>{theme} — {count}</div>
}
```

### With Suspense

```jsx
const value = useValue(state$, { suspense: true })
```

## observer (Optimization)

Merges multiple `useValue` calls into a single hook. Optional but useful for components with many observables.

```jsx
import { observer, useValue } from "@legendapp/state/react"

const Component = observer(function Component() {
  const v1 = useValue(state$.v1)
  const v2 = useValue(state$.v2)
  // ... all tracked in one hook
})
```

## Hooks

### useObserve
Like `useEffect` but runs when observables change (not deps array). Runs during render.

```jsx
useObserve(() => {
  document.title = `${profile$.name.get()} - Profile`
})

// With callback for single observable or event
useObserve(profile$.name, ({ value }) => {
  document.title = `${value} - Profile`
})
```

### useObserveEffect
Same as `useObserve` but runs after mount (like `useEffect`).

### useObservableReducer
Like `useReducer` but sets an observable.

```jsx
const [age$, dispatch] = useObservableReducer(reducer, { age: 42 })
```

### useWhen / useWhenReady
Hook versions of `when()`.

### useEffectOnce / useMount / useUnmount
Convenience hooks for mount/unmount lifecycle.

### usePauseProvider
Creates context with `paused$` observable to pause all Legend-State rendering.

## Context with Observables

Observables are stable objects, perfect for Context.

```tsx
const StateContext = createContext<Observable<UserState>>(undefined as any)

function App() {
  const state$ = useObservable({ profile: { name: "" } })
  return (
    <StateContext.Provider value={state$}>
      <Sidebar />
    </StateContext.Provider>
  )
}

function Sidebar() {
  const state$ = useContext(StateContext) // Never causes re-render
  return <Memo>{state$.profile.name}</Memo> // Self-updating
}
```

## Fine-Grained Reactivity

### Memo

Self-updating element. Parent never re-renders when the value changes.

```jsx
import { Memo } from "@legendapp/state/react"

function Component() {
  return (
    <div>
      Count: <Memo>{count$}</Memo>
    </div>
  )
}
```

With a selector function:
```jsx
<Memo>{() => <div>{state$.value.get()}</div>}</Memo>
```

### Computed

Extracts children so their changes don't affect parent, but parent changes still re-render them. Use when children depend on both observables and local parent state.

```jsx
<Computed>
  {() => state$.messages.map((message) => (
    <div key={message.id}>{message.text} {localVar}</div>
  ))}
</Computed>
```

### Show

Conditional rendering without parent re-render.

```jsx
<Show if={state$.showModal} else={() => <div>Nothing</div>}>
  {() => <Modal />}
</Show>

// With animation wrapper
<Show if={state$.show} wrap={AnimatePresence}>
  {() => <Modal />}
</Show>

// ifReady: won't render if value is empty object/array
<Show ifReady={state$.data}>
  {() => <DataView data={state$.data.get()} />}
</Show>
```

### Switch

Render one child based on value.

```jsx
<Switch value={state$.page}>
  {{
    0: () => <Tab1 />,
    1: () => <Tab2 />,
    default: () => <Error />,
  }}
</Switch>
```

### For

Optimized array rendering. Parent doesn't re-render when array elements change.

```jsx
import { For } from "@legendapp/state/react"

function Row({ item$ }) {
  const text = useValue(item$.text)
  return <div>{text}</div>
}

function List() {
  return <For each={state$.arr} item={Row} />
  // Or with render function:
  return (
    <For each={state$.arr} optimized>
      {(item$) => <div>{item$.text.get()}</div>}
    </For>
  )
}
```

**Important**: Arrays need unique `id` or `key` fields. Custom key extractor:

```js
const data$ = observable({
  arr: [],
  arr_keyExtractor: (item) => item.idObject._id,
})
```

## Reactive Components

### Web ($React)

```jsx
import { $React } from "@legendapp/state/react-web"

function Component() {
  const state$ = useObservable({ name: '', age: 18 })

  return (
    <div>
      <$React.div
        $style={() => ({
          color: state$.age.get() > 5 ? 'green' : 'red'
        })}
        $className={() => state$.age.get() > 5 ? 'kid' : 'baby'}
      />
      <$React.input $value={state$.name} />
      <$React.textarea $value={state$.name} />
    </div>
  )
}
```

### React Native

```jsx
import { $View, $Text, $TextInput } from "@legendapp/state/react-native"

function Component() {
  const state$ = useObservable({ name: '', age: 18 })

  return (
    <View>
      <$View $style={() => ({ backgroundColor: state$.age.get() > 5 ? '#22c55e' : '#ef4444' })} />
      <$Text>{() => state$.age.get() > 5 ? 'child' : 'baby'}</$Text>
      <$TextInput $value={state$.name} />
    </View>
  )
}
```

### Creating Custom Reactive Components

```jsx
import { reactive } from "@legendapp/state/react"

const Component = reactive(function Component({ message }) {
  return <div>{message}</div>
})

// Usage
<Component $message={() => isSignedIn$.get() ? "Hello" : "Goodbye"} />
```

`reactiveObserver` combines `observer` + `reactive`:

```jsx
const Component = reactiveObserver(function Component({ message }) {
  const name = useValue(name$)
  return <div>{message} {name}</div>
})
```

## Babel Plugin

Makes `Computed`, `Memo`, `Show` syntax less verbose.

```js
// babel.config.js
module.exports = {
  plugins: ["@legendapp/state/babel"],
}
```

```jsx
// You write:
<Computed><div>Count: {state$.count.get()}</div></Computed>
// Babel transforms to:
<Computed>{() => <div>Count: {state$.count.get()}</div>}</Computed>
```

## Tracing Hooks

Debug performance issues:

```jsx
import { useTraceListeners, useTraceUpdates, useVerifyNotTracking, useVerifyOneRender } from "@legendapp/state/trace"

function Component() {
  useTraceListeners()      // Logs all tracked observables
  useTraceUpdates()        // Logs what caused each render
  useVerifyNotTracking()   // Errors if tracking anything
  useVerifyOneRender()     // Errors if renders more than once
}
```
