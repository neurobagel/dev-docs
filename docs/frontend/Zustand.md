---
title: Zustand
tags: [React, Typescript, Frontend, Store, State Management]

---


## Zustand design principles

Some basic principles we use to organize our stores. With sources where possible.

### Normalize store as much as possible, but no more

Store objects should have a single theme or purpose. Or: all information one "thing" should be located
in one place and not duplicated. When designing or expanding a store, we're following loosely the rules
for relational database modeling, with some practical exceptions:

- if information about two "entities" is always handled and changed together (e.g. a standardized variable and its URI)
then they can be modeled in the same object
- we don't normalize many-to-many relations with associative tables/objects. In such a case, we just represent
the relation with an array of foreign keys

Reason:

- looking up and changing data is more straightforward
- store objects are less deeply nested, making mutations simpler
- the store is (hopefully) easier to read (more in the source)

Source:

- [https://redux.js.org/usage/structuring-reducers/normalizing-state-shape](https://redux.js.org/usage/structuring-reducers/normalizing-state-shape)

### Actions have semantic names

Store actions should reflect a real action that a user takes (e.g. `userSelectedConfig`)
or an event that has happened (e.g. `describedColumn`). The action should not just be a wrapper
for `set()` of a specific variable (i.e. not `"setDataTable"`). That also means that a single
store variable may be "written to" by more than one action.

Example:

```js
tbd
```

Sources:

- [https://redux.js.org/style-guide/#model-actions-as-events-not-setters](https://redux.js.org/style-guide/#model-actions-as-events-not-setters)
- [https://tkdodo.eu/blog/working-with-zustand#model-actions-as-events-not-setters](https://tkdodo.eu/blog/working-with-zustand#model-actions-as-events-not-setters)

### (somewhat) complex getters are custom hooks

If my component needs access to a computed value that is derived from other values in the store,
then we put the logic to generate the computed value inside of a custom hook, not inside the store.

!!! note "Store actions can't use hooks, if you want to then that's a signal to factor out"

    Hooks (custom and otherwise) are [only usable inside a React function component](https://react.dev/reference/rules/rules-of-hooks#only-call-hooks-from-react-functions).
    When you find that you want to use a custom hook in an action, that is a signal to consider 
    factoring (some of) the logic in the hook out into a regular JS utility function. 
    Then you can use the utility function inside the hook and also inside the action.

Reason:

- This keeps the store simple and focused on state variables and actions
- Keeps businesslogic separate and easily testable

Example:

```js
todo
```

### Actions can be async (e.g. as fetch)

Actions are usually there to handle some app event and respond to it by setting state.
You can also make the action responsible for fetching external data, processing it, and then setting
the app state. This is useful e.g. for fetching data from an API and then putting it in the state.

Sources:

- [https://github.com/pmndrs/zustand?tab=readme-ov-file#async-actions](https://github.com/pmndrs/zustand?tab=readme-ov-file#async-actions)

### Expose store variables with hooks only

We're following a suggestion by [tktodo](https://tkdodo.eu/blog/working-with-zustand#only-export-custom-hooks)
to expose atomic selectors with custom hooks instead of exposing the entire store.

Reason

- You are less likely to accidentally subscribe the entire store when you don't need to, avoiding
rerenders
- the custom hooks are more clearly named than destructing the store hook in the component

Example:

```js
// Export the selector, not the entire store
export const useBears = () => useBearStore((state) => state.bears)
```

Sources:

- [https://tkdodo.eu/blog/working-with-zustand#only-export-custom-hooks](https://tkdodo.eu/blog/working-with-zustand#only-export-custom-hooks)

### Controlled components do not access store directly, their parents do

This is a "house rule" for us that we may revisit. If a component has state managed by a parent
componet (i.e. is primarily a presentational component that gets props), then this component will
not directly access the store. Instead this responsibility falls to the parent who will prop-forward
both the store variables as well as store actions (in the form of handlers) to the child.

This pattern will likely have limits. E.g. we don't want to do very long prop-drilling chains.
But it will help keep responsibility clear and allow us to make more reusable components without
the close coupling that direct store access would entail.

Reasons

- Easier to write reusable components, because they are not coupled to store access
- Separation of concerns. As a non-page component, I only need to worry about my props

### Actions should not call other actions

Actions [represent events](#actions-have-semantic-names), and are not just variable setters. That
also means that actions should not directly call other actions to simply avoid duplicating code.
Instead, the component should dispatch the appropriate actions, and each action is then only
concerned with processing its own event.

There may be scenarios when one action will always lead to another action being triggered. For now
we would expect this situation to be handled in the component and not in the actions.

## Lifting state up

To collect data from multiple children, or to have two or more child components communicate with each other, declare the shared state in their parent component instead. The parent component can pass that state back down to the children via props. This keeps the child components in sync with each other and with their parent.

## Updating state

For flat updates we can simply use the `set` function provided with the new state and it will be shallowly merged with the existing state i.e., merging the new state object with the existing one at the top level only.

For deeply nested updates we have couple options:
- Normal approach: copy all ancestors until you get to the nested state and update it then

```JS
type State = {
  deep: {
    nested: {
      obj: { count: number }
    }
  }
}

  normalInc: () =>
    set((state) => ({
      deep: {
        ...state.deep,
        nested: {
          ...state.deep.nested,
          obj: {
            ...state.deep.nested.obj,
            count: state.deep.nested.obj.count + 1
          }
        }
      }
    })),

```

- Use [Immer](https://github.com/immerjs/immer)
- Use [Optics-ts](https://github.com/akheron/optics-ts/)
- Use [Ramda](https://ramdajs.com/)

Both Optics and Ramda work with types

## [You can auto generate selectors](https://zustand.docs.pmnd.rs/guides/auto-generating-selectors)


## Difference between `create` and `createStore`

### `create`

- **Purpose**: The `create` function is the primary and most commonly used way to create a Zustand store
    
- **Usage**: It is a higher-level API that simplifies the process of creating a store
    
- **Return Value**: It returns a hook that you can use in your React components to access the store
- **Example:**
```JS
import create from 'zustand';

const useStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
}));

// Usage in a component
function Counter() {
  const { count, increment } = useStore();
  return (
    <div>
      <p>{count}</p>
      <button onClick={increment}>Increment</button>
    </div>
  );
}
```


### `createStore`

- **Purpose**: The `createStore` function is a lower-level API that directly creates a store without automatically generating a React hook
    
- **Usage**: It is useful when you need more control over the store or when you want to use the store outside of React components
    
- **Return Value**: It returns a store object that you can interact with directly
- **Example:**
```JS
import { createStore } from 'zustand';

const store = createStore((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
}));

// Usage outside of React
store.getState().increment();
console.log(store.getState().count); // 1

// If you want to use it in React, you need to create a hook manually
import { useStore } from 'zustand';

const useStoreHook = () => useStore(store);
```


## Typescript

For Zustand's `create` function, the state type `T` is **invariant**, meaning TypeScript requires an explicit type annotation to ensure type safety.

It can't infer type from the initial state because
- state updates might add new fields that are not in the initial state
- partial updates may change the allowed type of certain fields

The zustand `create` function is a generic function that also takes a function as input. 
```ts
const useStore = create((set) => ({
  // initial state and actions
}));
```

When the type parameter is explicitly defined

```TS
const useStore = create<MyStateType>((set) => ({
  // initial state and actions
}));
```

TypeScript will get confused because it doesn’t know whether `MyStateType` is the type parameter or part of the function argument. 
So zustand uses **currying** to separate the generic type parameter from the function argument. The curried syntax looks like this:

```TS
const useStore = create<MyStateType>()((set) => ({
  // initial state and actions
}));
```


Zustand suggest using `combine` to avoid explicitly defining the type during state initialization which infers the state:

```TS
import { create } from 'zustand'
import { combine } from 'zustand/middleware'

const useBearStore = create(
  combine({ bears: 0 }, (set) => ({
    increase: (by: number) => set((state) => ({ bears: state.bears + by })),
  })),
)


```

Curried syntax is not used when using middleware like `combine` and `redux` that create the state i.e., define the structure/shape of the state in a way that typescript can infer it automatically.

For more information refer to the [zustand Typscript guide](https://zustand.docs.pmnd.rs/guides/typescript).
