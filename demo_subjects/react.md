### Q1. How does React's Virtual DOM and Reconciliation algorithm (Fiber) work?

**A.** 
React maintains an in-memory tree representation of the UI known as the Virtual DOM. When component state or props change, React generates a new Virtual DOM tree and compares it to the previous tree using a diffing algorithm called Reconciliation.

Key principles:
1. **Heuristic Diffing**: Instead of an O(n³) generic tree diff, React achieves O(n) complexity by assuming elements of different types generate different trees, and by using the `key` prop to match child elements across renders.
2. **Fiber Architecture**: In React 16+, Fiber replaced the synchronous recursive stack reconciler. A Fiber is a plain JavaScript object representing a unit of work. Fiber enables cooperative multitasking by breaking the reconciliation process into interruptible units, allowing React to prioritize high-priority user interactions (typing, clicks) over lower-priority background renders.
3. **Phases**:
   - **Render / Reconciliation Phase**: Pure and without side effects. React traverses the Fiber tree, computes diffs, and marks work effects. Can be paused or aborted.
   - **Commit Phase**: Synchronous. React applies all DOM mutations and runs layout/paint effects in one batch.

### Q2. What are the rules of React Hooks and why must hooks only be called at the top level?

**A.** 
The two core rules of React Hooks are:
1. **Only Call Hooks at the Top Level**: Do not call hooks inside loops, conditional statements, or nested functions.
2. **Only Call Hooks from React Functions**: Call hooks exclusively from React functional components or custom hooks.

**Underlying Reason**:
React does not associate hook state with variable names or IDs. Instead, React tracks hook state for each component instance as an ordered singly-linked list of hook cells attached to the component's Fiber node. During every render, React increments an internal pointer through this list in exact declaration sequence. If a hook were inside a condition or loop, an unexpected branch would alter the call order, misaligning all subsequent hooks with their corresponding state cells and corrupting component state.

### Q3. What is the difference between useMemo and useCallback, and when should you avoid them?

**A.** 
Both hooks are memoization utilities designed to avoid unnecessary recomputations across renders:

- `useMemo(() => computeValue(a, b), [a, b])`: Caches the **result** of executing a computation function. Re-executes only when dependencies change.
- `useCallback(fn, [deps])`: Caches the **function definition itself** between renders, equivalent to `useMemo(() => fn, [deps])`. Preserves referential equality of callbacks passed to optimized child components.

**When to Avoid**:
1. **Premature Optimization**: Do not wrap trivial computations or inline handlers by default. The overhead of allocating dependency arrays and evaluating diffs on every render often exceeds the savings of lightweight functions.
2. **Missing `React.memo` on Children**: Passing a `useCallback` function to a child component that is not wrapped in `React.memo` provides zero performance benefit because the child re-renders whenever its parent re-renders regardless of prop identity.

### Q4. How does the useEffect cleanup function work, and when does it run?

**A.** 
When a function returned from `useEffect` is specified, React registers it as a cleanup handler.

**Execution Timing**:
1. **Before Re-running the Effect**: On subsequent renders where dependencies change, React runs the cleanup function from the *previous* render before executing the *new* effect function with the new dependency values.
2. **On Component Unmount**: When the component is removed from the DOM, React executes the final cleanup function to release resources.

**Practical Use**:
Prevents memory leaks and stale event processing by unregistering subscriptions, clearing active intervals/timeouts, disconnecting WebSockets or observers, and aborting in-flight `fetch` requests via `AbortController`.

### Q5. Why is the 'key' prop necessary in lists and what issues occur when using array indexes as keys?

**A.** 
The `key` prop provides a stable identity to list items across renders so React's reconciliation engine can identify which items were inserted, deleted, reordered, or modified without re-rendering the entire list.

**Issues with Array Indexes as Keys**:
If items in a list are reordered, prepended, or deleted:
1. An item's index changes based on its position, breaking identity stability.
2. Uncontrolled child components (such as form inputs, checkboxes, or canvas elements) retain local DOM state mapped to the index rather than the underlying data item.
3. React performs unnecessary DOM destruction and recreation instead of moving existing DOM nodes, degrading rendering performance and creating subtle UI bugs.

A key should always be a stable, unique identifier from the data (such as a database UUID or record ID).

### Q6. What is the difference between controlled and uncontrolled components in React?

**A.** 
- **Controlled Components**: The input's current value is driven by React component state. A `value` prop binds the input to state, and an `onChange` handler updates that state on every keystroke. React is the "single source of truth". Enables immediate input validation, conditional field disabling, and custom formatted inputs.
- **Uncontrolled Components**: The DOM itself maintains the input's internal value. React accesses the value on-demand when needed (e.g., during form submission) using a `ref` (`inputRef.current.value`). Useful for integrating with non-React third-party libraries, optimizing high-frequency input without re-renders, or handling file inputs (`<input type="file" />`) which are read-only in the DOM.

### Q7. How do React Error Boundaries work and what kinds of errors can they NOT catch?

**A.** 
An Error Boundary is a React component that catches JavaScript errors anywhere in its child component tree, logs the errors, and displays a fallback UI instead of crashing the entire component tree.

It is implemented as a class component defining either `static getDerivedStateFromError(error)` (to update state and render fallback UI) or `componentDidCatch(error, errorInfo)` (to log diagnostic data).

**What Error Boundaries Do NOT Catch**:
1. Event handlers (e.g., `onClick`), because event handlers run outside the render cycle (use standard `try/catch` instead).
2. Asynchronous code (e.g., `setTimeout`, `Promise`, `async/await` callbacks).
3. Server-side rendering (SSR) errors.
4. Errors thrown inside the Error Boundary component itself rather than in its children.

### Q8. What causes unnecessary re-renders in React and how do React.memo and context splitting help?

**A.** 
In React, when a parent component re-renders, all of its descendant components re-render by default recursively, regardless of whether their props changed.

**Remedies**:
1. **`React.memo`**: A higher-order component that performs a shallow comparison of current and next props. If props are referentially equal, React skips rendering the wrapped component and reuses the last rendered output.
2. **Context Splitting**: A monolithic React Context holding frequently changing state alongside static state causes every consumer component to re-render whenever *any* part of the context value changes. Splitting the context into separate Providers (e.g., `ThemeContext`, `UserContext`, `DispatchContext`) ensures components only re-render when the specific data slice they consume changes.
3. **Lifting Content Up / Component Composition**: Passing children as JSX props (`children`) allows the child element to be evaluated in the outer scope, preventing re-renders when the wrapper component updates its own state.

### Q9. What is the difference between Server-Side Rendering (SSR), Static Site Generation (SSG), and Client-Side Rendering (CSR)?

**A.** 
- **Client-Side Rendering (CSR)**: The browser downloads a minimal HTML shell with a bundled JavaScript file. The browser executes JavaScript, fetches data via client API calls, and renders the DOM. Pros: Rich interactivity, fast transitions once loaded. Cons: Slow First Contentful Paint (FCP), larger initial bundle, poor SEO for simple crawlers.
- **Server-Side Rendering (SSR)**: The web server renders HTML on every incoming HTTP request, populating data dynamically before sending the rendered page to the browser. The browser displays HTML immediately and "hydrates" it with JavaScript for interactivity. Pros: Fast FCP, always up-to-date dynamic data, excellent SEO. Cons: Higher server compute load, slower Time to First Byte (TTFB).
- **Static Site Generation (SSG)**: HTML pages are pre-rendered at build time and served via global CDN edges. Pros: Extremely fast TTFB, lowest hosting costs, high reliability. Cons: Rebuilding is required to update content (unless using Incremental Static Regeneration / ISR).

### Q10. How does React 18 Concurrent Mode and useTransition differ from standard synchronous state updates?

**A.** 
Prior to React 18, state updates were strictly synchronous and uninterruptible; once a render started, the browser main thread was blocked until completion.

**Concurrent Mode & `useTransition`**:
Concurrent React enables state updates to have distinct priority levels.
- **Urgent Updates**: Direct user interactions like typing, pressing buttons, or hovering require immediate visual feedback.
- **Transition Updates (`startTransition` / `useTransition`)**: Non-urgent UI transitions like switching tabs, filtering heavy lists, or loading search results.

Wrapping a state update in `startTransition` tells React that the update can be interrupted if an urgent event (e.g., user typing another character) occurs. React pauses the background transition work, handles the high-priority keystroke immediately, and resumes or discards stale background rendering work, maintaining high UI responsiveness.
