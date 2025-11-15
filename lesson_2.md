# 🎉 Lesson 2 — Creating Your First Vue Component

## 1. 🔍 Concept Overview

A Vue component is a self-contained unit of UI, combining:

- **State** → data the component holds
- **Logic** → how it transforms or reacts to that state
- **Template** → how the state appears on the screen

In Vue 3, components are usually written using:

- `<script setup>` → logic + state
- `<template>` → declarative UI output

Each component is reactive, encapsulated, and can be composed into larger interfaces.

## 2. 💡 Mental Model / Analogy

🎁 **Think of a component as a "smart UI box."**

Each box:

- Has its own internal state (count)
- Knows how to update itself when state changes
- Has a template describing how it should look

You don't manually open the box and update its visuals — you simply change its internal state, and the box redraws itself.

## 3. 🧱 Code Example — Your First Component

A simple counter component:

```vue
<!-- Counter.vue -->
<script setup>
import { ref } from "vue";

// 1. Reactive state
const count = ref(0);

// 2. Methods (just functions)
function increment() {
  count.value++;
}
</script>

<template>
  <!-- 3. Declarative template -->
  <button @click="increment">Count: {{ count }}</button>
</template>
```

**What's happening?**

- The `count` ref is reactive.
- The `increment` method updates the state.
- Vue automatically re-renders the component when `count` changes.

## 4. ⚙️ Internal Insight — What Vue Is Really Doing

### The `<script setup>` block

- Runs once when the component is created.
- Everything defined here is compiled into the component's `setup()` function.
- Variables returned implicitly from `setup()` become available in the `<template>`.

Internally Vue transforms:

```vue
<script setup>
const count = ref(0);
</script>
```

into something like:

```js
export default {
  setup() {
    const count = ref(0);
    return { count };
  },
};
```

### The `<template>` block

Is compiled into a render function, something like:

```js
function render() {
  return h("button", { onClick: increment }, `Count: ${count.value}`);
}
```

### Reactive connection

When the template accesses `count.value`:

1. Vue registers a dependency.
2. When `count` changes, Vue re-runs the render function.
3. The virtual DOM is compared with the old one.
4. Vue patches only the changed text node.

**This is the essence of declarative rendering:**  
You change the data, Vue changes the DOM.

## 5. 🧩 Component Registration & Organization

### Local Registration (most common in SFCs)

In your parent component:

```vue
<script setup>
import Counter from "./Counter.vue";
</script>

<template>
  <Counter />
</template>
```

No extra registration step — `<script setup>` automatically exposes imported components to the template.

### Global Registration (optional)

In `main.js`:

```js
import { createApp } from "vue";
import App from "./App.vue";
import Counter from "./Counter.vue";

createApp(App).component("Counter", Counter).mount("#app");
```

Now `<Counter />` is available everywhere.

## 6. 🧩 Common Pitfalls / Misconceptions

❌ **"I need to return variables from `<script setup>`."**

- No — Vue auto-exposes all top-level variables to the template.

❌ **"Methods must use `this`."**

- Not with `<script setup>`.
- Functions are just functions — no component instance or `this`-binding.

❌ **"Why do I need `.value`?"**

- In the script section, `ref` values are wrapped in an object.
- Inside templates, Vue unwraps them automatically.

So:

- **Script:** `count.value`
- **Template:** `{{ count }}`

❌ **"Templates run once."**

- Templates are re-evaluated reactively each time dependencies change.

## 7. ⚙️ Advanced Insight / Performance Angle

### `<script setup>` is compiled, not executed literally

Vue performs aggressive **compile-time optimization**:

- Dead code elimination
- Hoisting of constant expressions
- Static tree extraction (only rerender dynamic parts)
- Compile-time detection of reactive dependencies
- Inlining render context access

This makes Single-File Components with `<script setup>` extremely fast and tree-shakable.

### Templates generate optimal virtual DOM operations

Vue's compiler:

- Identifies dynamic bindings
- Marks static parts of the DOM tree
- Minimizes diffing during updates
- Produces highly optimized render functions

## 8. 🧠 Key Takeaways

- A component = state + logic + template.
- `<script setup>` is the recommended, modern way to write Vue components.
- Templates are declarative: you describe what the UI should look like.
- Vue's reactivity updates the DOM efficiently and automatically.
- Components are composable, isolated units of functionality.
