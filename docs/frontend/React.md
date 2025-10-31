## What triggers re-renders in React

A re-render happens when a component runs again after its initial render to update its output.

Below are the main scenarios that cause a component to re-render:

1. State updates: changes to component state (created using useState, useReducer, or any other state hook)

2. Parent re-rendering: when a parent component re-renders, it triggers a re-render in its child components by default

3. Context updates: changes in a context provider’s value cause all components that consume that context to re-render

4. Hook updates: updates from custom hooks or built-in hooks (like useReducer or useSyncExternalStore) cause the component using them to re-render


## Error boundary

Used to catch rendering errors and display a fallback UI. See relevant React documentation page [here](https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary) for more information.

According to [this stack overflow post](https://stackoverflow.com/questions/48482619/how-can-i-make-use-of-error-boundaries-in-functional-react-components), there are no native functional implementation for the error boundary component in react. See also the following pages of the legacy react docs:
- [Do hooks cover all cases for classes?](https://legacy.reactjs.org/docs/hooks-faq.html#do-hooks-cover-all-use-cases-for-classes)
- [Introducing Error Boundaries](https://legacy.reactjs.org/docs/error-boundaries.html#introducing-error-boundaries)


## One file per custom hook

For now, we will have each hook in a separate file, with a test of the same name next to it.

```bash
useMyHook.ts
UseMyHook.test.ts
```

Reason:

- easier to import
- clearer to manage / write tests for
