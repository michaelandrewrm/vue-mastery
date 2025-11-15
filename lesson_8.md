# 🌱 Lesson 8 — Composition API Basics

The Composition API solves a fundamental problem in Vue application architecture: **How to organize, reuse, and scale component logic cleanly.**

## 1. 🔍 Concept Overview

The Composition API introduces:

- `setup()` (and its shorthand: `<script setup>`)
- **reactive primitives** (`ref`, `reactive`, `computed`, `watch`)
- **composables** (reusable logic functions)
- A consistent pattern for organizing code around **logic**, not component options

This moves Vue away from the clunky Options API (`data`, `methods`, `computed`, `watch`, etc.) toward a more flexible and scalable model.

## 2. 💡 Mental Model / Analogy

🧠 **Think of the Composition API as "Logic Lego Blocks."**

Each block is a reactive primitive:

- `ref()` → a reactive value
- `reactive()` → a reactive object
- `computed()` → a derived value
- `watch()` → a reaction
- lifecycle hooks → timing blocks

You snap these blocks together in `setup()`, and Vue builds the reactive graph for you.

**Composables** are bags of Lego blocks packaged together so you can reuse them.

## 3. 🧱 Code Example — Minimal Composition API Component

Here's how a simple component looks using `<script setup>`:

```vue
<script setup>
import { ref, computed } from "vue";

const count = ref(0);

const doubled = computed(() => count.value * 2);

function increment() {
  count.value++;
}
</script>

<template>
  <button @click="increment">{{ count }} → doubled: {{ doubled }}</button>
</template>
```

- No `data()`, no `methods`, no `computed:` section.
- Everything is written together, based on logical grouping.

## 4. ⚙️ Why the Composition API Exists

The Options API groups code by **option type**, not by **feature**.

Example:

```js
export default {
  data() {
    return { a: 1, b: 2 };
  },
  methods: {
    incrementA() {},
    logB() {},
  },
  watch: {},
  computed: {},
};
```

If you implement multiple features, related logic gets split across sections.
Reading or maintaining such components becomes difficult.

**Composition API solves this.**

It lets you write:

- all logic for feature A together
- all logic for feature B together

Like this:

```js
const a = ref(1);
function incrementA() {
  a.value++;
}

const b = ref(2);
function logB() {
  console.log(b.value);
}
```

Instead of scattering it across the component options.

## 5. 🧩 The Role of `setup()` and `<script setup>`

### `setup()`

- Runs before the component is created.
- Has no access to `this` (deliberately).
- Returns everything exposed to the template.
- Hosts the entire Composition API logic.

Internally, `<script setup>` compiles to:

```js
export default {
    setup(props, { emit }) {
        // your code
        return { ... }
    }
}
```

### `<script setup>` benefits:

- No need for explicit `return` — everything is auto-exposed
- No need for `props` or `emit` declarations in arguments
- Fewer lines of boilerplate
- Better static analysis and tree-shaking
- Faster runtime because of compile-time optimizations

## 6. 🧱 Organizing Logic with Composables

**What is a composable?**

A function that uses Composition API primitives and returns reactive state + logic.

**Example: `useCounter.js`**

```js
// useCounter.js
import { ref } from "vue";

export function useCounter() {
  const count = ref(0);
  const increment = () => count.value++;

  return { count, increment };
}
```

Use it inside any component:

```vue
<script setup>
import { useCounter } from "./useCounter.js";

const { count, increment } = useCounter();
</script>

<template>
  <button @click="increment">Count: {{ count }}</button>
</template>
```

Composables allow:

- Reusable logic
- Shared reactive state (or isolated state, depending on design)
- Clean separation of concerns
- Testing logic in isolation

### Mental Model

A composable is like a **mini-setup function** that returns reactive Lego blocks.

## 7. 🧬 Using Reactive Primitives Together

Let's combine `ref`, `reactive`, `computed`, and `watch`:

```vue
<script setup>
import { ref, reactive, computed, watch } from "vue";

const user = reactive({
  first: "",
  last: "",
});

const fullName = computed(() => {
  return `${user.first} ${user.last}`.trim();
});

watch(
  () => user.first,
  (newVal) => console.log("First name changed:", newVal)
);

const length = computed(() => fullName.value.length);
</script>

<template>
  <input v-model="user.first" placeholder="First" />
  <input v-model="user.last" placeholder="Last" />

  <p>{{ fullName }} ({{ length }} chars)</p>
</template>
```

**What the Composition API excels at:**

- All logic for this feature is grouped together
- Composables can extract this entire feature cleanly
- You can literally copy/paste this into any component

## 8. 🧩 Common Gotchas

### ❌ Mistake: Using the Options API style inside `setup()`

```js
methods: { ... } // not allowed in setup
```

Everything in `setup` is just functions and variables.

### ❌ Mistake: Thinking `setup` runs multiple times

It runs **once**, during component initialization.

Only reactive effects re-run.

### ❌ Mistake: Forgetting that destructuring breaks reactivity

Use `toRefs` when destructuring reactive objects.

### ❌ Mistake: Expecting `this` inside `setup()`

`this` is intentionally `undefined` — `setup` avoids `this`-binding issues.

## 9. ⚙️ Advanced Insight / Performance Angle

`<script setup>` enables compile-time optimizations:

- Hoisting constant expressions out of renders
- Tree-shaking unused imports
- Eliminating unused variables
- Detecting which variables are reactive
- Auto-exposing variables to the render context

Traditional Options API can't do this.

### Composables are stateless by default

Each component instance calling a composable gets its own reactive state

- avoids shared state bugs
- encourages pure logic functions

Unless intentionally using a global store like Pinia.

## 10. 🧠 Key Takeaways

- **Composition API** improves scalability, organization, and reusability.
- `setup()` is where Composition API logic lives; `<script setup>` is the modern equivalent.
- **Composables** encapsulate reusable logic like "hooks" in React.
- Reactive primitives (`ref`, `reactive`, `computed`, `watch`) combine like Lego pieces.
- Composition API groups code by **feature**, not by **options**.
- `<script setup>` is faster, cleaner, and more ergonomic.
