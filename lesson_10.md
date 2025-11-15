# 🔄 Lesson 10 — Lifecycle, Effects, and Cleanup

Vue's reactivity system is powerful, but with power comes responsibility: Whenever you introduce side effects, you must manage them carefully to avoid leaks, stale listeners, or unexpected behavior.

This lesson shows:

- How Vue's reactivity loop works
- When effects run
- How lifecycle hooks control effect timing
- How to attach and clean up side effects properly

## 1. 🔍 Concept Overview

A **side effect** is anything that touches the outside world:

- Adding event listeners
- Starting timers or intervals
- Fetching data
- Subscribing to APIs / sockets
- Interacting with DOM elements or libraries

Vue components mount, update, and unmount. Side effects must align with these phases + clean up correctly.

The key tools:

- `onMounted()` → runs effects when DOM is ready
- `onBeforeUnmount()` / `onUnmounted()` → clean up effects

## 2. 💡 Mental Model / Analogy

🧠 **Think of side effects like setting up a temporary campsite.**

When your component mounts, you:

- Set up your tent
- Light the fire
- Lay out your equipment

When your component unmounts, you must:

- Put out the fire
- Pack up your tent
- Leave no trace

If you don't clean up, fires keep burning, more tents keep piling up, and soon you have a forest of abandoned gear (memory leaks and dangling listeners).

## 3. 🧱 Code Example — Side Effect With Cleanup

```vue
<script setup>
import { onMounted, onUnmounted, ref } from "vue";

const x = ref(0);

function handleMouseMove(e) {
  x.value = e.clientX;
}

onMounted(() => {
  window.addEventListener("mousemove", handleMouseMove);
});

onUnmounted(() => {
  window.removeEventListener("mousemove", handleMouseMove);
});
</script>

<template>
  <p>Mouse X: {{ x }}</p>
</template>
```

**What happens:**

- When the component mounts → event listener attaches
- When it unmounts → listener removed
- No leaks, no leftover subscriptions, perfect cleanup

This pattern applies to every side effect.

## 4. ⚙️ Internal Insight — Vue's Reactivity Loop

Vue's reactivity engine works in phases:

1. Change detected →
2. Reactively re-run the effect (component render or computed) →
3. Patch DOM →
4. Run post-render lifecycle hooks →
5. Flush watchers →
6. Flush pending cleanup tasks

The render effect for a component is an internal watcher:

- It tracks dependencies during render
- When dependencies change, it re-schedules itself
- `onUpdated()` fires after the render patch completes

Effects like watchers or computed have their own scheduling:

- batched
- ordered
- post-render for watchers

### 🔹 Lifecycle Hook Timing Deep Dive

| Hook              | When it Runs                      | Use Case                            |
| ----------------- | --------------------------------- | ----------------------------------- |
| `onMounted`       | After DOM is created and inserted | Access DOM, start subscriptions     |
| `onUpdated`       | After each reactive update        | React to DOM changes                |
| `onBeforeUnmount` | Right before teardown             | Cleanup with state still accessible |
| `onUnmounted`     | After teardown                    | Final cleanup, release resources    |

**Why side effects must be tied to lifecycle:**

If you attach effects in `setup()`:

- DOM isn't ready
- The component might not mount
- You won't have a chance to clean up
- Leaks become extremely likely

Lifecycle hooks give you deterministic timing.

## 5. 🧩 Properly Handling Side Effects

Let's look at common patterns and best practices.

### Pattern 1: Timers and Intervals

```vue
<script setup>
import { onMounted, onUnmounted, ref } from "vue";

const count = ref(0);
let intervalId;

onMounted(() => {
  intervalId = setInterval(() => {
    count.value++;
  }, 1000);
});

onUnmounted(() => {
  clearInterval(intervalId);
});
</script>
```

**Why cleanup matters:**

If you don't call `clearInterval`, the timer keeps running → the component's reactive graph still updates → but the component no longer exists → wasted CPU and memory.

### Pattern 2: Event Listeners

Always pair:

- `addEventListener` → `removeEventListener`

### Pattern 3: Fetching and Abort Controllers

```vue
<script setup>
import { onMounted, onUnmounted, ref } from "vue";

const data = ref(null);
let abortController;

onMounted(async () => {
  abortController = new AbortController();

  const res = await fetch("/api/data", {
    signal: abortController.signal,
  });
  data.value = await res.json();
});

onUnmounted(() => {
  abortController.abort();
});
</script>
```

This avoids requests completing after a component unmounts.

### Pattern 4: External Libraries (map, chart, 3rd-party UI)

```js
let instance

onMounted(() => {
    instance = new Library(...)
})

onUnmounted(() => {
    instance.destroy()
})
```

If the library attaches DOM, events, intervals, etc., it must be cleaned up.

## 6. 🧩 Common Gotchas

❌ **Attaching event listeners in `setup()`**

DOM isn't available → use `onMounted`.

❌ **Forgetting cleanup → memory leaks**

Intervals, listeners, websockets must be cleaned up always.

❌ **Running side effects in `computed()`**

Computed must be pure. Side effects → use `watch`.

❌ **Using `onUpdated` for cleanup**

`onUpdated` runs after every reactive update, not unmount. Use `onUnmounted`.

❌ **Expecting watchers to auto-clean**

Watchers auto-clean their internal dependencies, but not your own side effects.

If your watcher triggers a side effect, you must clean it up manually.

## 7. ⚙️ Advanced Insight — Watch Cleanup Functions

Watchers can return cleanup functions:

```js
watch(source, (newVal, oldVal, onCleanup) => {
  const connection = startConnection(newVal);

  onCleanup(() => {
    connection.close();
  });
});
```

**Why useful?**

When the watcher re-runs:

1. Vue first runs cleanup
2. Then executes the new watcher body
3. Prevents stale subscriptions

This is incredibly powerful.

## 8. 🧠 Key Takeaways

- Vue's reactivity loop controls when effects run and update the DOM.
- Side effects belong in lifecycle hooks — not in pure reactive logic.
- Always clean up:
  - listeners
  - intervals
  - subscriptions
  - external library instances
  - pending network requests
- `onMounted` + `onUnmounted` are the cornerstone of safe side effect management.
- Watchers can include cleanup logic for complex reactive side effects.
- Proper cleanup prevents memory leaks, stale callbacks, and hidden bugs.
