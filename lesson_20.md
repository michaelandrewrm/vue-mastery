# 🚀 Lesson 20 — Performance Optimization Techniques

Vue's rendering engine is highly optimized, but poorly structured components or "reactive explosions" can slow even a small app.

This lesson covers:

- How to measure performance
- Key compiler/runtime optimization tools (v-memo, v-once, caching)
- Designing fine-grained reactivity to avoid unnecessary updates
- Preventing component over-rendering and effects from propagating too far

By the end, you'll know how to build Vue apps that scale smoothly.

## 1. 🔍 Concept Overview

Performance optimization in Vue comes down to:

**✔ Rendering performance**  
How often a component re-renders and how much DOM patching is needed.

**✔ Reactivity performance**  
How much work gets triggered by reactive changes — especially if state objects are large or watchers are deep.

**✔ Memory / lifecycle performance**  
Ensuring effects, watchers, timers, and subscriptions are cleaned up.

**✔ Preventing reactive chains from spreading unpredictably**  
This is where "granularity control" becomes important.

Vue provides both compiler-level and runtime-level tools to improve performance.

## 2. 💡 Mental Model / Analogy

**🧠 Think of Vue components as "reactive islands."**

Each island has:

- Inputs (props)
- Internal reactive state
- Effects (render, computed, watchers)

Performance problems happen when:

- A small ripple in one island sends a wave to many other islands
- or when island boundaries are too big and everything updates together

Optimizing Vue means:

- Making islands smaller
- Preventing waves from leaving the island
- Reducing how often waves occur
- Using breakwaters (v-memo, v-once, caching) to stop waves altogether

## 3. 🧱 Measuring Component Performance

Vue Devtools provides:

- **✔ Component render count** — Tells you how often a component re-renders.
- **✔ The performance timeline** — Shows slow components and the cost of their re-renders.
- **✔ Profiler** — Records full render cycles.

When measuring:

- Trigger interactions that cause updates
- Inspect which components re-render
- Look for unexpected child updates
- Check dependency chains: props → computed → render
- Examine deeply reactive objects (possible reactive explosion)

**Simple technique: add logs in render**

```js
onUpdated(() => console.log("Component updated"));
```

or using the render effect directly:

```js
const instance = getCurrentInstance();
console.log("rendering", instance.type.name);
```

This helps you see exactly when the component re-renders.

## 4. 🧱 v-once — Skip All Future Updates

`v-once` renders a node once and never updates it again.

```vue
<h1 v-once>Static Title</h1>
```

**Useful for:**

- Static banners
- Headers that don't change
- Footer sections
- Read-only labels
- Large static tables or lists

**Effect:**

- Vue hoists it
- No tracking
- No diffing
- No patching
- Super cheap.

## 5. 🧱 v-memo — Skip Re-rendering Based on Conditions

Added in Vue 3.2.

```vue
<div v-memo="[id]">
    {{ expensiveContent }}
</div>
```

Vue will skip diffing and re-rendering this block unless `id` changes.

**This is extremely useful for:**

- Large tables
- Complex SVGs or charts
- Render-heavy components
- Recursively rendered structures

**How it works:**

- Evaluates the memo condition array
- If values are unchanged → skip entire subtree

This is the closest Vue equivalent to React's `memo()`.

## 6. 🧱 Caching Event Handlers and VNodes

**Inline object props or inline handlers force re-renders**

Example:

```vue
<div :style="{ color: 'red' }"></div>
<!-- new object every render -->
```

**Fix:**

```vue
<script setup>
const redStyle = { color: "red" };
</script>

<template>
  <div :style="redStyle"></div>
</template>
```

**Inline functions behave the same way**

```vue
<button @click="() => doSomething(item)">Press</button>
```

Creates a new function on every render → cannot be optimized.

**Fix:**

```vue
<script setup>
function handleClick(item) {
  doSomething(item);
}
</script>

<template>
  <button @click="handleClick(item)">Press</button>
</template>
```

This helps Vue recognize stability and skip work.

## 7. 🔍 Optimizing Reactivity Granularity

**Reactivity granularity** refers to how precisely state changes trigger updates.

Vue aims for fine-grained updates, but you can make it even better with:

### ✔ ref() instead of reactive()

Refs avoid tracking every property of a large object.

If you mutate a large reactive object:

```js
state.bigObject.foo = 123;
```

Vue must track property-level changes for all nested proxies.

But if you store the object as a ref():

```js
const big = ref(bigObject);
big.value.foo = 123;
```

You reduce deep tracking.

### ✔ Split large objects

Instead of:

```js
const state = reactive({
    user: { ...100 props... },
    settings: { ...20 props... },
    theme: {...20 props...}
})
```

Split:

```js
const user = reactive({ ... })
const settings = reactive({ ... })
const theme = reactive({ ... })
```

Now components depend only on the piece they actually read.

### ✔ Avoid deeply nested reactive structures

Flatten state if needed — every nested access triggers tracking.

## 8. 🧨 Avoiding Over-Rendering and "Reactive Explosions"

A **reactive explosion** is when one small state change unexpectedly triggers many re-renders.

**Common causes:**

### ❌ Passing entire reactive objects via props

```vue
<MyChild :user="user" />
```

If `user` is reactive and has many properties:

- Any change to any `user.x` triggers child re-render
- Even if child only uses `user.name`

**Fix:**

```vue
<MyChild :name="user.name" />
```

### ❌ Using watchers on entire objects

```js
watch(user, () => { ... }, { deep: true })
```

This triggers on any nested change — dangerous.

**Fix:**

```js
watch(() => user.name, () => {...})
```

### ❌ Mutating arrays without keys

Leads to costly full re-diffs.

## 9. 🧰 More Optimization Tools

**✔ defineComponent({ name: ... })** improves devtools & caching  
Helps Vue identify stable component boundaries.

**✔ shallowRef() and shallowReactive()**  
Avoid deep reactivity on large objects (graphs, maps, charts).

**✔ markRaw()**  
Use for:

- third-party instances (Mapbox, Leaflet, Three.js)
- class instances
- huge immutable objects

Prevents Vue from proxying them.

**✔ computed()** for expensive derived values  
Memoizes results until dependencies change.

## 10. ⚙️ Internal Insight — Why These Optimizations Work

Vue runtime uses:

- Patch flags
- Block trees
- Hoisted static nodes
- Reactive dependency graphs
- Batched updates

When your template is stable, Vue can skip nearly all diffing.

When your reactivity is granular, Vue knows exactly which effects to re-run.

When memoization and hoisting are used together, Vue's rerendering becomes extremely cheap.

**Vue's runtime + compiler are symbiotic:**

- the compiler provides hints
- the runtime uses them to skip work

Optimizing both sides gives maximum performance.

## 11. 🧠 Key Takeaways

- **Measure performance before optimizing** — use Devtools render count.
- **Use v-once for static content** — zero runtime cost.
- **Use v-memo to skip expensive subtrees.**
- **Avoid inline objects/functions in templates.**
- **Split and flatten state for fine-grained reactivity.**
- **Avoid reactive explosions by limiting what is reactive.**
- **Use shallow refs, markRaw, and memoization strategically.**
- **Vue's performance comes from compiler + runtime working together** — optimizations amplify both.
