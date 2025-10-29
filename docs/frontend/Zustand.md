---
title: Zustand
tags: [React, Typescript, Frontend, Store, State Management]

---


## Zustand design principles

Some basic principles we use to organize our stores. With sources where possible.

### Actions have semantic names

Store actions should reflect a real action that a user takes (e.g. `userSelectedConfig`)
or an event that has happened (e.g. `describedColumn`). The action should not just be a wrapper
for `set()` of a specific variable (i.e. not `"setDataTable"`).

Example:

```js
tbd
```

Sources:

- [https://redux.js.org/style-guide/#model-actions-as-events-not-setters](https://redux.js.org/style-guide/#model-actions-as-events-not-setters)
- [https://tkdodo.eu/blog/working-with-zustand#model-actions-as-events-not-setters](https://tkdodo.eu/blog/working-with-zustand#model-actions-as-events-not-setters)

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
