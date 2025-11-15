# ⚡️ Lesson 6 — Reactivity Fundamentals

This lesson explains exactly how Vue knows when to update the DOM, why refs exist, how reactive objects work, and the mental model that lets you predict re-renders like an expert.

## 1. 🔍 Concept Overview

Vue's reactivity system turns plain JavaScript values and objects into reactive sources that:

- Track who depends on them
- Trigger updates when they change

**The main APIs:**

- **`ref(value)`** — Creates a reactive wrapper around any value — primitives or objects.
- **`reactive(object)`** — Turns an object into a deeply reactive proxy.

These work through **dependency tracking**:

- When the template or a computed property reads reactive state, Vue remembers it.
- When that state changes, Vue re-runs the appropriate update functions.

This creates a **full reactivity graph**: a network of dependencies between state and effects.

## 2. 💡 Mental Model / Analogy

### 🕸 Think of Vue's reactivity as a "web of wires."

- Each reactive variable is a **signal wire**.
- Each component render or computed function is a **light bulb**.
- When a bulb reads a wire, Vue connects them with a tiny filament.

So the dependency graph looks like:

```
count -----> render()
    |
    ---> doubled computed()
```

When `count` changes, Vue "powers" only the bulbs connected to it.

No more. No less. This is why Vue is fast.

## 3. 🧱 Code Example — `ref()` and `reactive()` in action

```vue
<script setup>
import { ref, reactive } from "vue";

const count = ref(0); // a reactive primitive
const user = reactive({
  // a reactive object
  name: "Alice",
  age: 25,
});

function increment() {
  count.value++;
  user.age++;
}
</script>

<template>
  <p>Count: {{ count }}</p>
  <p>User: {{ user.name }} ({{ user.age }})</p>
  <button @click="increment">Update</button>
</template>
```

**Here:**

- Changing `count.value` re-renders only places that depend on `count`.
- Modifying `user.age` updates only places using `user.age`.

## 4. ⚙️ Internal Insight — How Vue's Reactivity Actually Works

Let's break down the true internals.

### 🔹 1. `ref()` — the reactive wrapper

A ref is an object:

```js
{ value: <your-value> }
```

When you access `count.value`, Vue:

- Tracks `count` as a dependency of the current effect (e.g., render function).
- On mutation (`count.value++`), Vue triggers those effects.

**This is why:**

- In scripts: you must use `.value`
- In templates: `.value` is automatically unwrapped

### 🔹 2. `reactive()` — proxy-based reactivity

A reactive object is a `Proxy`:

```js
const state = reactive({
  name: "Alice",
  age: 25,
});
```

**Internally:**

- On `get`, Vue records a dependency: _"Effect X depends on state.age"_
- On `set`, Vue triggers updates: _"state.age changed → re-run effects depending on state.age"_

The proxy allows Vue to intercept every property read/write.

### 🔹 3. Vue builds a "Reactivity Graph"

Imagine:

```
user.age ---> render()
    |
    ---> computed(fullInfo)

count -------> renderCounter()
```

Vue keeps:

- A **dependency map** (source → effects)
- A **scheduler** that batches updates
- A **renderer** that efficiently patches only the changed DOM nodes

This is the beating heart of Vue.

## 5. 🧩 Common Gotchas — Especially Destructuring

The most common mistake in Vue 3:

### ❌ Destructuring `reactive()` breaks reactivity

**Example:**

```js
const user = reactive({ name: "Alice", age: 25 });

// ❌ This breaks reactivity
const { name, age } = user;
```

**Why?**

Because:

- `reactive()` returns a proxy
- Destructuring copies raw values
- Vue can no longer track access to the proxy

### ✔ Correct approaches

**1. Use the object directly**

```js
user.name;
```

**2. Use `toRefs()` to preserve reactivity**

```js
import { toRefs } from "vue";

const { name, age } = toRefs(user);
```

Now `name` and `age` are `ref()`s internally linked back to the reactive object.

### Other common pitfalls

**❌ Forgetting `.value` in script**

```js
count++; // WRONG
count.value++; // correct
```

**❌ Using `reactive()` for primitives**

```js
const n = reactive(0); // ❌ doesn't work as expected
```

Vue warns against this. Always use `ref()` for primitives.

**❌ Mutating objects inside ref without `.value`**

```js
const obj = ref({ x: 1 });
obj.x = 2; // ❌ wrong — bypasses reactivity
obj.value.x = 2; // correct
```

## 6. ⚙️ Advanced Insight / Performance Angle

### Fine-Grained Reactivity

Vue tracks dependencies **per property**, not per component.

So updating:

```js
user.age++;
```

only affects effects depending on `user.age`, not:

- the entire `user` object
- the entire component
- the entire app

**React (pre-signals) re-renders components.**  
**Vue re-runs only the exact effects tied to changed properties.**

### Dependency Tracking Uses WeakMaps

Internally:

```
WeakMap (proxy → Map)
    Map (key → Set of effects)
        Set (effects to run)
```

This is incredibly memory efficient and GC-friendly.

## 7. 🧠 Key Takeaways

- **`ref()`** = reactive wrapper around any value, great for primitives.
- **`reactive()`** = proxy that makes objects deeply reactive.
- Vue builds a **dependency graph** connecting reactive sources to effects.
- The Virtual DOM updates only the parts of the UI affected by state.
- **Destructuring reactive objects breaks reactivity** unless using `toRefs()`.
- **`.value` is required in scripts** because ref stores its value in an object.
