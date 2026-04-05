# Helper Functions and Configuration

## Helper Observables

### Time helpers

```js
import { currentDay, currentTime } from "@legendapp/state/helpers/time"

observe(() => {
  console.log('Today is:', currentDay.get())
  console.log('Time is:', currentTime.get())
})
```

### Page Hash (web)

```js
import { pageHash, configurePageHash } from '@legendapp/state/helpers/pageHash'
import { pageHashParams, configurePageHashParams } from '@legendapp/state/helpers/pageHashParams'

configurePageHash({ setter: 'pushState' })

observe(() => {
  console.log('hash changed to:', pageHash.get())
})

pageHash.set('value=test')
pageHashParams.userid.set('newuser')
```

## React Hooks

### useHover (web)

```jsx
import { useHover } from "@legendapp/state/react-hooks/useHover"

function ButtonWithTooltip() {
  const refButton = useRef()
  const isHovered = useHover(refButton)

  return (
    <div>
      <button ref={refButton}>Click me</button>
      <Show if={isHovered}>
        {() => <Tooltip text="Tooltip!" target={refButton} />}
      </Show>
    </div>
  )
}
```

### useIsMounted

```jsx
import { useIsMounted } from "@legendapp/state/react"

function Component() {
  const isMounted = useIsMounted()
  const onClick = () => {
    setTimeout(() => {
      if (isMounted.get()) console.log("Clicked")
    }, 100)
  }
  return <button onClick={onClick}>Click me</button>
}
```

### useMeasure (web)

```jsx
import { useMeasure } from "@legendapp/state/react-hooks/useMeasure"

function Component() {
  const ref = useRef()
  const { width, height } = useMeasure(ref)
  return <div ref={ref}>Width: {width}, Height: {height}</div>
}
```

### createObservableHook

Convert existing hooks to return observables:

```js
import { createObservableHook } from "@legendapp/state/react-hooks/createObservableHook"

const useMyHookObservable = createObservableHook(useMyHook)

function Component() {
  const value = useMyHookObservable()
}
```

## trackHistory

```js
import { trackHistory } from '@legendapp/state/helpers/trackHistory'

const state$ = observable({ profile: { name: 'Hello' }})
const history = trackHistory(state$)

state$.profile.name.set('Annyong')
// history: { 1666593133018: { profile: { name: 'Hello' } } }
```

## undoRedo

```js
import { undoRedo } from '@legendapp/state/helpers/undoRedo'

const state$ = observable({ todos: ["Get milk"] })
const { undo, redo, getHistory } = undoRedo(state$.todos, { limit: 100 })

state$.todos.push("Pick up bread")
undo()  // Back to ["Get milk"]
redo()  // Back to ["Get milk", "Pick up bread"]
```

## Configuration

### enable$GetSet

Shorthand `$` for get/set:

```js
import { enable$GetSet } from "@legendapp/state/config/enable$GetSet"
enable$GetSet()

const val = state$.test.$     // get()
state$.test.$ = "hello"       // set()
state$.$ = { test: "hello" }  // set whole object
```

### enable_PeekAssign

Shorthand `._` for peek/assign without tracking:

```js
import { enable_PeekAssign } from "@legendapp/state/config/enable_PeekAssign"
enable_PeekAssign()

const val = state$.test._     // peek()
state$.test._ = "hello"       // assign without notifying
```

### enableReactTracking

Warn when `get()` is called without `useValue`:

```js
import { enableReactTracking } from "@legendapp/state/config/enableReactTracking"
enableReactTracking({ warnMissingUse: true })
```
