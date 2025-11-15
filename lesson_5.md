# 🔥 Lesson 5 — Lifecycle Hooks and Component Mounting

Vue components aren't static — they enter, update, and leave the DOM.
Lifecycle hooks let you "tap into" these moments to perform side effects, initialize logic, or clean up resources.

## 1. 🔍 Concept Overview

Every Vue component follows a **mount → update → unmount** lifecycle.

Vue exposes **lifecycle hooks** so you can run code at specific moments:

**Most commonly used:**

- **onMounted** → run after component is inserted into the DOM
- **onUpdated** → run after reactive state triggers a re-render
- **onUnmounted** → run cleanup when component is removed

**Hooks help you:**

- Access the actual DOM
- Start intervals, fetch data, observe elements
- Clean up listeners, timers, subscriptions
- Execute logic at predictable timing

## 2. 💡 Mental Model / Analogy

🧠 **Think of a component like a stage actor.**

- **Before Mount** — backstage preparation
- **Mounted** — actor walks on stage (DOM created and attached)
- **Updated** — actor reacts to cues and changes costume during the play
- **Unmounted** — actor exits the stage, and crew removes props

Lifecycle hooks let you run code exactly when the actor enters or leaves the stage.

## 3. 🧱 Code Example — Using Core Hooks

```vue
<script setup>
import { ref, onMounted, onUpdated, onUnmounted } from "vue";

const count = ref(0);

onMounted(() => {
  console.log("Component mounted!");
});

onUpdated(() => {
  console.log("Component updated! Count is now", count.value);
});

onUnmounted(() => {
  console.log("Component unmounted, cleanup here!");
});
</script>

<template>
  <button @click="count++">Count: {{ count }}</button>
</template>
```

**What happens?**

- First render: `onMounted` fires
- Every time `count` changes: `onUpdated` fires
- If the component disappears (e.g., `v-if="false"`): `onUnmounted` fires

## 4. ⚙️ Internal Insight — What Vue Actually Does

**Component lifecycle under the hood:**

```
setup()
    ↓
reactivity tracking
    ↓
first render → virtual DOM → real DOM
    ↓
mounted() hook callbacks
    ↓
state changes → effect runs → patch DOM
    ↓
updated() hook callbacks
    ↓
removed from tree
    ↓
unmounted() hook callbacks
```

### How hooks are registered

**Inside `<script setup>`:**

```js
onMounted(() => {...})
```

**Vue internally:**

- Stores the callback inside the component instance
- Associates it with the mount phase of the renderer
- Executes it after the initial DOM insertion

### Why onMounted is needed

You cannot access the DOM inside `setup()`, because:

- The component hasn't rendered yet
- No DOM nodes exist
- Vue's virtual DOM is not applied to the real DOM yet

`onMounted` is the earliest hook where DOM access is legal.

## 5. 🧩 When to Use Each Hook (Side Effects & Cleanup)

### 🔹 onMounted — Run side effects that need the DOM

**Use for:**

- Fetching data
- Subscribing to event listeners
- Setting up IntersectionObserver
- Interacting with 3rd-party libraries
- Measuring DOM elements

### 🔹 onUpdated — React to state-driven re-renders

**Use when:**

- You need to respond specifically to DOM changes
- You need to interact with updated layout
- You need to run an animation after updates

**But be careful** — updates can be frequent.
Avoid heavy logic in `onUpdated`.

### 🔹 onUnmounted — Clean up everything you created

**Crucial to avoid memory leaks.**

**Use to remove:**

- Event listeners
- Intervals/timeouts
- Subscriptions (WebSocket, Firebase, etc.)
- Observers
- 3rd-party instances

**Example: Setting up and cleaning up a timer**

```vue
<script setup>
import { onMounted, onUnmounted } from "vue";

let timer;

onMounted(() => {
  timer = setInterval(() => console.log("tick"), 1000);
});

onUnmounted(() => {
  clearInterval(timer);
});
</script>
```

Strict, predictable lifecycle = no surprises and no leaks.

## 6. 🧩 Common Pitfalls / Misconceptions

- ❌ **Accessing `document.querySelector` in `setup()`**

  - DOM isn't ready yet → use `onMounted`.

- ❌ **Forgetting to clean up intervals or listeners**

  - Leads to background timers running even after the component disappears.

- ❌ **Running expensive logic in `onUpdated`**

  - Because updates can fire many times.

- ❌ **Expecting `onMounted` to run on every re-render**
  - It fires once.

## 7. ⚙️ Advanced Insight / Performance Angle

### Lifecycle hooks map directly to renderer phases

Vue's renderer has well-defined stages:

- `beforeMount`
- `mount`
- `patch`
- `unmount`

Hooks correspond exactly to these.
This means Vue can optimize updates aggressively while still guaranteeing hooks run in the correct order.

### Hooks run after the DOM is stable

Vue schedules hooks:

- after rendering
- after patching
- in microtasks, ensuring the DOM is finalized

This gives you reliable, predictable DOM access.

## 8. 🧠 Key Takeaways

- Lifecycle hooks let you respond to DOM creation, updates, and removal.
- **onMounted** = perform setup work that needs the actual DOM.
- **onUpdated** = side effects in response to reactive re-renders.
- **onUnmounted** = clean up listeners, intervals, subscriptions.
- Vue's lifecycle is tightly aligned with its rendering pipeline.
- Properly using hooks prevents memory leaks and ensures consistent behavior.
