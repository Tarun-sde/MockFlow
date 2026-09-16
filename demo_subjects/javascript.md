### Q1. What is the difference between var, let, and const?

**A.** 
`var` is function-scoped (or globally scoped if declared outside a function) and is hoisted to the top of its scope with an initial value of `undefined`. It allows re-declaration and reassignment within the same scope.

`let` and `const` are block-scoped (scoped to the nearest enclosing pair of curly braces `{}`). While they are hoisted, they remain uninitialized in a Temporal Dead Zone (TDZ) from the start of the block until the declaration line is evaluated, throwing a `ReferenceError` if accessed beforehand. 

`let` allows reassignment but prevents re-declaration in the same scope. `const` prevents reassignment and requires an immediate initializer upon declaration. For composite data types (objects and arrays), `const` prevents reassigning the variable reference itself, but properties or elements can still be mutated unless frozen with `Object.freeze()`.

### Q2. How does the JavaScript Event Loop work with Call Stack, Microtask Queue, and Macrotask Queue?

**A.** 
JavaScript runs in a single-threaded runtime environment. Synchronous code executes immediately on the Call Stack. When asynchronous operations occur, their associated callbacks are scheduled into task queues managed by the Event Loop.

1. **Call Stack**: Executes frames synchronously in Last-In, First-Out (LIFO) order until empty.
2. **Microtask Queue**: Holds callbacks from `Promise.then()`, `catch()`, `finally()`, `queueMicrotask()`, and `MutationObserver` (or `process.nextTick` in Node.js).
3. **Macrotask Queue (Task Queue)**: Holds callbacks from `setTimeout()`, `setInterval()`, `setImmediate()`, and UI event listeners.

The Event Loop continuously checks if the Call Stack is empty. Once empty, it executes **all** pending microtasks in the Microtask Queue until it is completely drained (including any microtasks scheduled during microtask execution) before taking a single macrotask from the Macrotask Queue. After executing that macrotask and rendering UI updates (if needed), the cycle repeats.

### Q3. What are closures in JavaScript and what is a practical use case?

**A.** 
A closure is the combination of a function bundled together with references to its surrounding lexical environment. In JavaScript, functions maintain an internal reference to the lexical scope in which they were declared, allowing them to access variables outside their own immediate scope even after the outer function has completed execution and returned.

Practical use cases include:
1. **Data Encapsulation and Private Variables**: Creating factory functions that expose public methods while keeping internal state inaccessible from the outside.
2. **Function Factories and Currying**: Generating specialized utility functions pre-configured with specific arguments.
3. **Event Handlers and Callbacks**: Retaining access to parameters passed to the registering function when the asynchronous event fires later.

### Q4. Explain prototypal inheritance and how the prototype chain works.

**A.** 
Unlike class-based inheritance in languages like Java or C++, JavaScript uses prototypal inheritance, where objects inherit directly from other objects.

Every JavaScript object has an internal slot called `[[Prototype]]` (accessible via `Object.getPrototypeOf()` or the legacy `__proto__` accessor). When a property or method is accessed on an object, the JavaScript engine first checks the object's own properties. If not found, it traverses up the prototype chain to `[[Prototype]]`, continuing upward until it finds the property or reaches `Object.prototype.[[Prototype]]`, which is `null`.

The `class` syntax introduced in ES6 is syntactic sugar over this prototypal mechanism; `class Child extends Parent` sets the prototype of `Child.prototype` to `Parent.prototype`.

### Q5. How does the 'this' keyword work in JavaScript across different invocation contexts?

**A.** 
In JavaScript, `this` is not bound statically at definition time (except for arrow functions); rather, its value is determined by how a function is called:

1. **Default Binding**: In a standard function call `foo()`, `this` points to the global object (`window` or `globalThis`) in non-strict mode, and `undefined` in strict mode (`"use strict"`).
2. **Implicit Binding**: When called as an object method `obj.foo()`, `this` points to the object preceding the dot (`obj`).
3. **Explicit Binding**: Using `.call()`, `.apply()`, or `.bind()`, `this` is explicitly set to the passed object.
4. **New Binding**: When invoked as a constructor with `new Foo()`, a fresh empty object is created, linked to `Foo.prototype`, and `this` is bound to that new instance.
5. **Arrow Functions**: Arrow functions do not define their own `this` binding; they lexically capture `this` from the enclosing execution context at definition time, and cannot be rebound using `call` or `apply`.

### Q6. What is the difference between Promise.all(), Promise.allSettled(), Promise.race(), and Promise.any()?

**A.** 
These four static Promise combinators coordinate multiple concurrent asynchronous operations:

1. `Promise.all(iterable)`: Resolves when **all** promises resolve (yielding an array of values), or rejects immediately upon the **first** rejected promise (short-circuiting fail-fast behavior).
2. `Promise.allSettled(iterable)`: Waits for **all** promises to settle regardless of outcome. Never short-circuits. Returns an array of descriptor objects containing `{ status: "fulfilled", value }` or `{ status: "rejected", reason }`.
3. `Promise.race(iterable)`: Settles (resolves or rejects) as soon as the **first** promise in the iterable settles with its value or error.
4. `Promise.any(iterable)`: Resolves as soon as the **first** promise **fulfills** successfully. If all promises reject, it rejects with an `AggregateError` containing all rejection reasons.

### Q7. What is event delegation and why is it beneficial for performance?

**A.** 
Event delegation is a design pattern where an event listener is attached to a single common parent element rather than attaching individual event listeners to multiple child elements.

It works because of **Event Bubbling**: when an event (such as `click`) occurs on a target element, it first captures down from the root, targets the clicked node, and then bubbles up through its ancestors in the DOM tree. The parent listener inspects `event.target` (the actual element clicked) or uses `event.target.closest("selector")` to determine which child triggered the action.

Benefits include:
1. **Memory Conservation**: Eliminates the overhead of hundreds or thousands of listener function references in memory.
2. **Dynamic Elements**: Automatically handles new child elements added dynamically to the DOM without having to manually register new event listeners.
3. **Simplified Cleanup**: Only one listener needs to be detached upon component unmount, preventing memory leaks.

### Q8. What is the difference between debounce and throttle?

**A.** 
Both techniques limit the execution rate of an event-driven function, but address different timing requirements:

- **Debounce**: Delays function execution until a specified delay period has elapsed since the **last** time the function was invoked. If the event fires again before the timer expires, the timer resets. Ideal for search typeaheads, window resize adjustments, and autosave keystroke pauses.
- **Throttle**: Enforces a maximum frequency limit, guaranteeing that the function executes at most once every specified time interval, even if the underlying event continues firing uninterrupted. Ideal for scroll position monitoring, infinite scrolling calculations, and mousemove canvas tracking.

### Q9. How does JavaScript handle equality comparison (== vs === vs Object.is)?

**A.** 
JavaScript provides three primary mechanisms for equality checking:

1. **Abstract / Loose Equality (`==`)**: Compares values with implicit type coercion. If types differ, JavaScript converts operands according to complex type conversion rules (e.g., `false == 0` is `true`, `null == undefined` is `true`, `"" == 0` is `true`).
2. **Strict Equality (`===`)**: Compares both value and type without coercion. If the operands have different types, it immediately returns `false`. However, it treats `+0` and `-0` as equal, and `NaN === NaN` evaluates to `false`.
3. **Same-Value Equality (`Object.is`)**: Similar to strict equality, but handles the IEEE 754 edge cases correctly: `Object.is(NaN, NaN)` is `true`, and `Object.is(+0, -0)` is `false`.

### Q10. What causes memory leaks in JavaScript applications and how can they be prevented?

**A.** 
JavaScript relies on mark-and-sweep garbage collection. A memory leak occurs when an object is no longer needed by application logic, but retains an active reference in the root graph, preventing the garbage collector from reclaiming its memory.

Common causes and remedies:
1. **Accidental Global Variables**: Assigning to an undeclared variable attaches it to `window`. Fixed by using `"use strict"` and modern lexical bindings (`const`/`let`).
2. **Forgotten Timers and Callbacks**: `setInterval` holding references to outer variables in its callback. Fixed by explicitly calling `clearInterval` when the component or lifecycle ends.
3. **Detached DOM Nodes**: Keeping references to DOM elements in a JavaScript array or object after they have been removed from the document tree. Fixed by nullifying the variable reference upon removal.
4. **Uncleared Event Listeners**: Attaching listeners to global targets (`window`, `document`) without detaching them in teardown hooks.
5. **Closures Retaining Large Scopes**: Long-lived closures retaining references to heavy structures from their outer lexical environments.
