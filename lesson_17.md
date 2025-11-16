# ⚛️ Lesson 17 — Inside Vue's Reactivity System

Vue's reactivity engine is a fine-grained signal system built on:

- **effects**
- **dependency tracking**
- **proxy traps**
- **dependency graphs**
- **scheduler queues**

Understanding this system unlocks mastery of:

- computed properties
- watchers
- rendering updates
- dependency cycles
- performance behavior

Let's go deep.

## 1. 🔍 Concept Overview

Vue reactivity allows you to:

- Declare reactive state (`ref`, `reactive`)
- Auto-track reads (dependency collection)
- Auto-trigger updates on writes
- Re-run only the exact computations that depend on changed state

The core of Vue's reactivity consists of three internal operations:

### `effect(fn)`

Registers a reactive effect (a function that depends on reactive state).

### `track(target, key)`

When reactive state is read, Vue records that the current effect depends on this property.

### `trigger(target, key)`

When reactive state is written, Vue re-runs the effects that depend on that property.

**This triad is the beating heart of Vue.**

## 2. 💡 Mental Model / Analogy

🧠 **Think of Vue's reactivity as a giant "notification graph."**

- Each reactive property is a **newsletter channel**.
- Every effect that reads that property is a **subscriber**.
- When that property changes, Vue **sends a newsletter** to subscribers.
- Subscribers re-run their logic (render functions, computed getters, watchers).

The "dependency graph" is literally:

```
state property → set of effects that depend on it
```

Vue dynamically builds and updates this graph at runtime.

## 3. 🧱 Code Example (Visible Behavior)

```vue
<script setup>
import { ref, computed } from "vue";

const count = ref(0);

const doubled = computed(() => {
  console.log("computed run");
  return count.value * 2;
});

function increment() {
  count.value++;
}
</script>

<template>
  <p>{{ doubled }}</p>
  <button @click="increment">+1</button>
</template>
```

When you click:

1. `count.value++` triggers watchers for `count`
2. `doubled` becomes dirty → recomputed on next read
3. render effect re-runs and updates DOM

This is powered internally by `effect`, `track`, and `trigger`.

## 4. ⚙️ Internal Insight — The `effect()` system

A simplified form of Vue's internal code:

```js
let activeEffect = null;

function effect(fn) {
  activeEffect = fn;
  fn(); // run for dependency collection
  activeEffect = null;
}
```

Whenever Vue runs an effect:

- It sets `activeEffect` to the currently running effect.
- Any reactive read will call `track()` to link the effect to the state.

### Rendering a component is an effect

Vue treats each component's render function as an effect.

### Computed is a special "lazy effect"

Computed uses an effect with a dirty flag.

### Watchers use an effect that re-runs when triggered.

## 5. ⚙️ `track()`: Dependency Collection

Whenever a reactive property is read, Vue does:

```js
function track(target, key) {
  if (!activeEffect) return;

  let depsMap = targetMap.get(target);
  if (!depsMap) targetMap.set(target, (depsMap = new Map()));

  let dep = depsMap.get(key);
  if (!dep) depsMap.set(key, (dep = new Set()));

  dep.add(activeEffect);
}
```

**Data structure:**

```
WeakMap → Map → Set
target → (key → (effects))
```

**Visualized:**

```
user.age > [ renderEffect, computedEffect ]
count    > [ renderEffect ]
```

This is the reactivity graph.

## 6. ⚙️ `trigger()`: Running Effects on Change

Whenever reactive state is written, Vue does:

```js
function trigger(target, key) {
  const depsMap = targetMap.get(target);
  const dep = depsMap.get(key);

  for (const effect of dep) {
    scheduler.queue(effect);
  }
}
```

Instead of running effects immediately, Vue batches them in a scheduler for performance.

### Why batching?

If you change data 10 times quickly:

- Vue runs **one render**
- not 10 renders

DOM updates are flushed after the microtask queue.

## 7. ⚙️ Deep Dive: How Vue Uses ES6 Proxies

`reactive()` returns a proxy:

```js
const state = reactive({ count: 0 });
```

Internally:

```js
function reactive(target) {
  return new Proxy(target, {
    get(target, key, receiver) {
      const res = Reflect.get(target, key, receiver);
      track(target, key);
      return isObject(res) ? reactive(res) : res;
    },

    set(target, key, value, receiver) {
      const oldValue = target[key];
      const result = Reflect.set(target, key, value, receiver);
      if (value !== oldValue) {
        trigger(target, key);
      }
      return result;
    },
  });
}
```

**Key insight:**

- `get` → `track`
- `set` → `trigger`

This makes reactivity automatic.

**Also note:**

Vue performs deep reactive wrapping lazily (on property access). This makes initial reactive creation fast and lightweight.

## 8. 🧠 The Dependency Graph Mental Model

Imagine your entire app's dependencies as a graph:

```
            render effect
            /       |     \
           /        |      \
    count.age   user.name   items[0]
        |           |
    computed A  computed B
        |
    watcher C
```

State changes propagate downstream through the graph.

Updating `count.age` triggers:

- `computed A` recompute
- `render effect` re-run
- `watcher C` run

Nothing else runs.

**This is extremely fine-grained and efficient.**

## 9. 🧩 Common Gotchas

### ❌ Destructuring reactive objects breaks dependency collection

Because destructuring bypasses the proxy.

```js
const { age } = reactiveUser; // age is now a raw value
```

**Fix with `toRefs()`:**

```js
const { age } = toRefs(reactiveUser);
```

### ❌ Mutating nested objects without access still works

Because proxies track property access, not objects themselves:

```js
user.profile.address.city = "x"; // triggers reactivity
```

### ❌ Confusing watchers with computed

Computed is tracked lazily; watchers run eagerly.

### ❌ Circular effects

Vue prevents infinite loops by tracking dependency cycles and offering warning behavior.

## 10. ⚙️ Advanced Insight — The Scheduler

Vue's scheduler:

- Batches effects
- Orders them (computed before render)
- Avoids duplicate runs
- Uses microtask queue for async flushing

When lots of changes happen:

```js
count++;
count++;
count++;
```

→ only **1 render re-run**

Effects are collected and processed intelligently for performance.

## 11. 🧠 Key Takeaways

- Vue's reactivity is built on **Proxies + effects + dependency maps**.
- Every reactive read calls `track()`, linking the effect to the property.
- Every reactive write calls `trigger()`, re-running relevant effects.
- The **dependency graph** enables fine-grained updates.
- **Computed** is a lazy effect; **watchers** are eager effects.
- Proxy interceptors (`get`/`set`) provide automatic tracking.
- Vue batches and schedules effects for optimal performance.

**Understanding this system lets you:**

- Predict exactly when re-renders happen
- Design optimal reactivity patterns
- Debug complex reactive flows
- Build your own composables from first principles
