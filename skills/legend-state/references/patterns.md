# State Patterns

Legend-State is unopinionated — use it however your team prefers.

## One Large Global Store

```js
const store$ = observable({
  UI: {
    windowSize: undefined,
    activeTab: 'home',
  },
  settings: {
    theme: 'light',
    fontSize: 14,
  },
  todos: []
})
```

## Multiple Individual Atoms

```ts
// Settings
export const theme$ = observable<'light' | 'dark'>('light')
export const fontSize$ = observable(14)

// UI State
export const uiState$ = observable({
  windowSize: undefined as { width: number, height: number } | undefined,
  activeTab: 'home' as 'home' | 'user' | 'profile',
})
```

## Within React Components

Use `useObservable` for component-local state, pass through props or Context.

```jsx
function App() {
  const store$ = useObservable({
    profile: { name: "hi" },
  })

  return (
    <div>
      <Profile profile={store$.profile} />
    </div>
  )
}

function Profile({ profile }) {
  return <div>{profile.name}</div>
}
```

## Choosing a Pattern

- **Global store**: Good for small-medium apps, easy to persist/sync everything at once
- **Individual atoms**: Good for tree-shaking, clear separation of concerns
- **Component state**: Good for ephemeral UI state, avoids global pollution
- **Mixed**: Most real apps use all three — global for app-wide state, atoms for domain-specific state, component state for ephemeral UI
