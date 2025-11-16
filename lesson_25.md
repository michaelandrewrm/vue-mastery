# 🔬 Lesson 25 — Advanced Reactivity: Effect Scopes, Shallow Reactivity & Custom Refs

We will explore:

- Effect scopes (`effectScope`) and why composables use them
- `shallowRef` / `shallowReactive` / `triggerRef`
- Creating your own reactive primitives using `customRef`
- How these tools prevent reactive explosions
- When and why to use these patterns in real applications

## 1. 🔍 Concept Overview

Vue's reactivity is made of small, fine-grained effects that:

- track dependencies
- respond to changes
- update UI or recompute state

But as your code grows, a major challenge appears:

> How do you manage, isolate, or clean up effects created inside composables?

Also:

- How do you avoid unnecessary deep reactivity when working with large objects or external libraries?
- How can you create custom reactive behavior (debounced refs, persisted refs, etc.)?

Vue exposes low-level APIs for these needs:

- `effectScope()` — to group & dispose effects
- `shallowRef()`, `shallowReactive()` — for shallow tracking
- `customRef()` — for building custom reactive logic
- `triggerRef()` — to manually notify updates

## 2. 💡 Mental Model / Analogy

🧠 **Think of reactivity as "webs of wires."**

- Every reactive variable is a signal source
- Every watcher/computed/render is a node wired to those signals
- Vue automatically wires everything together

But you need tools to manage these wires:

✔ **Effect scopes = "circuit breakers"**  
Group wires so you can shut them all off at once.

✔ **Shallow refs/reactives = "don't wire inside this object"**  
Track the box, not what's inside it.

✔ **Custom refs = "build your own wire behavior"**  
Example: debounce the signal before it reaches listeners.

## 3. 🧱 Effect Scopes — Managing Reactivity at Scale

### What problem they solve

Composables often create:

- watchers
- computeds
- inner effects

But these effects keep running… even after the component using them unmounts.

**Unless you isolate them inside an effect scope.**

### Basic usage

```js
import { effectScope, ref } from "vue";

const scope = effectScope();

scope.run(() => {
  const count = ref(0);
  watch(count, () => console.log("changed"));
});
```

Later:

```js
scope.stop(); // cleans ALL watchers created inside
```

### How composables use effect scopes

Vue's recommended pattern:

```js
export function useSomething() {
  const scope = effectScope();

  scope.run(() => {
    // all refs, watchers, computeds here
  });

  onUnmounted(() => scope.stop());
}
```

This ensures:

- no leaking watchers
- composables fully clean up after themselves
- predictable lifecycle behavior

**Nuxt composables do this automatically.**

## 4. 🧱 Shallow Reactivity: shallowRef & shallowReactive

By default:

- `ref()` tracks `.value` deeply
- `reactive()` tracks all nested properties

This can cause **reactive explosions** if used with:

- large objects
- DOM nodes
- class instances
- graph or tree structures
- 3rd-party libraries (e.g., Mapbox, Three.js)

### ✔ `shallowRef()` — only tracks `.value`

```js
const chart = shallowRef(null)
chart.value = new Chart(...) // reactive
chart.value.data.datasets.push(...) // NOT reactive
```

### ✔ `shallowReactive()` — only tracks first-level props

```js
const state = shallowReactive({
  user: { name: "Alice" },
});

state.user = newUser; // tracked
state.user.name = "Bob"; // NOT tracked
```

### Use cases:

- Optimizing performance
- Preventing Vue from walking deep structures
- Storing external library objects (Three.js, Mapbox)
- Managing immutable data

### ✔ `triggerRef()` — manually trigger updates

When using shallow refs:

```js
chart.value.data.push(newItem);
triggerRef(chart); // force DOM update
```

Useful when:

- external systems mutate objects
- Vue cannot detect changes
- you want full manual control

## 5. 🧱 Custom Refs — Build Your Own Reactive Primitives

`customRef()` lets you intercept `get` and `set` of a ref.

### Example: debounced input

```js
import { customRef } from "vue";

function useDebouncedRef(value: string, delay = 300) {
  let timeout: any;

  return customRef((track, trigger) => ({
    get() {
      track(); // collect dependencies
      return value;
    },
    set(newVal) {
      clearTimeout(timeout);
      timeout = setTimeout(() => {
        value = newVal;
        trigger(); // notify Vue
      }, delay);
    },
  }));
}
```

Usage:

```js
const search = useDebouncedRef("", 300);
```

### Other powerful uses:

- throttled refs
- persisted localStorage refs
- syncing with URL query params
- refs that reject invalid input
- lazy-loaded refs

**`customRef` is one of Vue's hidden gems.**

## 6. ⚙️ Internal Insight — Why These APIs Exist

Vue's reactivity is built around **effects and dependencies**.

- Every watcher/computed/render is an effect
- Effects belong to effect scopes
- Scopes store and manage cleanup
- Deep reactivity uses recursive Proxy walkers
- Shallow reactivity avoids this cost
- `customRef` gives you direct access to track/trigger internals

### Architecturally:

- `reactive()` → deep Proxy
- `shallowReactive()` → shallow Proxy
- `ref()` → wrapper object around a value
- `shallowRef()` → only tracks wrapper
- `customRef()` → manual control of track + trigger

**Effect scopes bind all reactive effects created during function execution.**

This is why composables can be safely reused across components without leaks.

## 7. 🧩 Common Gotchas

❌ **Thinking `shallowRef` tracks nested fields**  
It only tracks `.value`, not inner fields.

❌ **Not stopping effect scopes in reusable hooks**  
This creates hidden memory leaks.

❌ **Misusing `customRef` without calling `track` or `trigger`**  
The ref will stop updating completely.

❌ **Using deep reactive structures for large datasets**  
Always evaluate if shallow reactivity is better.

## 8. 🧠 Key Takeaways

- **Effect scopes** are essential for composables — they prevent leaked watchers.
- **Shallow reactivity** improves performance and avoids heavy dependency tracking.
- `triggerRef()` lets you manually notify changes for shallow refs.
- `customRef()` enables creating your own reactive primitives (debounced, persisted, validated refs).
- These tools give you **full control** over Vue's reactivity system.

**Advanced reactivity is what enables Vue to scale to complex applications and external library integration.**
