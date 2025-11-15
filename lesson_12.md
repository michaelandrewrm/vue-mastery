# 🧭 Lesson 12 — State Management: Pinia & Vuex 4

State management is about shared state — data that multiple components must access, update, and stay in sync with.

**This lesson explains:**

- When global state is truly needed
- Pinia's API and design
- How getters, actions, and persistence work
- The migration mindset from Vuex → Pinia

## 1. 🔍 Concept Overview

**Local state:**

- Lives inside a component or composable.

**Global state:**

- Lives outside any component and is shared across many parts of an app.

**Examples of global state:**

- authenticated user
- theme preferences
- shopping cart
- notifications
- large data caches
- persistent app-wide settings

**A global store:**

- Centralizes state
- Avoids prop-drilling
- Automatically integrates with Vue reactivity
- Ensures predictable behavior across components

**Pinia is the next-generation state management library for Vue 3:**

- Type-safe
- Devtools integrated
- Intuitive
- Modular
- Composition-API-powered

Pinia replaces Vuex for most applications.

---

## 2. 💡 Mental Model / Analogy

🧠 **Think of a global store as a "public library."**

- **Local state** = the books you privately keep on your desk
- **Global state** = the central library everyone can borrow from

**A store:**

- Holds shared data
- Provides consistent access
- Makes mutations predictable
- Ensures updates propagate everywhere
- Offers "services" like actions and computed getters

Pinia is like a modern, organized library with better tools, while Vuex is the older library with strict rules and more paperwork.

## 3. 🧱 Pinia Basics — A Minimal Store

**store/counter.js**

```js
import { defineStore } from "pinia";

export const useCounterStore = defineStore("counter", {
  state: () => ({
    count: 0,
  }),

  getters: {
    doubled: (state) => state.count * 2,
  },

  actions: {
    increment() {
      this.count++;
    },
  },
});
```

**Using it in a component**

```vue
<script setup>
import { useCounterStore } from "@/store/counter";

const counter = useCounterStore();
</script>

<template>
  <button @click="counter.increment">
    {{ counter.count }} → {{ counter.doubled }}
  </button>
</template>
```

**Key observations:**

- `state` is reactive
- `getters` are computed
- `actions` are methods with `this` bound to the store

## 4. ⚙️ Pinia Internals — Why This Is Powerful

**Under the hood:**

- Pinia stores are reactive objects powered by Vue's reactivity system.
- `state()` returns a reactive object (using `reactive()`).
- `getters` compile into computed properties.
- `actions` can mutate state directly (unlike Vuex).

**The mental model:**

- **state** → reactive source
- **getters** → derived state
- **actions** → side effects + mutations

This aligns perfectly with Composition API patterns.

## 5. 🧱 Computed Getters

```js
getters: {
    fullName: (state) => `${state.first} ${state.last}`,

    isAdult() {
        // `this` works too!
        return this.age >= 18
    }
}
```

**What Pinia does:**

- Wraps getters in `computed()`
- Tracks dependencies automatically
- Memoizes results (cached until state changes)

## 6. 🧱 Actions — Business Logic & Async

**Actions are perfect for:**

- Async operations
- API calls
- Complex logic
- Batch updates
- Cross-store coordinating

**Example:**

```js
actions: {
    async fetchUser(id) {
        const res = await fetch(`/api/users/${id}`)
        this.user = await res.json()
    }
}
```

Pinia allows direct state mutation, unlike Vuex's strict mutation rules.

## 7. 🧱 State Persistence

Pinia doesn't include persistence built-in, but it integrates easily:

**Example using a plugin:**

```js
import { defineStore } from "pinia";

export const useSettingsStore = defineStore("settings", {
  state: () => ({
    theme: "light",
  }),
  persist: true,
});
```

**With a persistence plugin like:**

- `pinia-plugin-persistedstate`
- or your own custom plugin

Pinia's architecture is plugin-friendly and simple.

## 8. 🧱 Provide/Inject vs Pinia

**When should you use each?**

**provide/inject:**

- Localized state sharing
- Good for tree-specific context (forms, modals, nested components)
- Scoped to component hierarchy

**Pinia:**

- App-wide state
- Needs to persist or survive navigation
- Used across many non-related components

Pinia is essentially `provide/inject` done globally with devtools and reactivity guarantees.

## 9. 🧩 Migrating from Vuex to Pinia — Mental Model Shift

### 💥 Vuex Mental Model (Old)

- State is global
- Getters derive state
- Mutations must be synchronous
- Actions commit mutations
- Mutations update state
- Namespaced modules

This can feel bureaucratic.

### 🌿 Pinia Mental Model (New)

- State is reactive
- Getters are computed
- Actions can mutate directly
- Logic is centralized but simple
- No mutations needed
- Typescript-friendly
- Extensible with plugins

**Migration intuition:**

| Vuex                   | Pinia              |
| ---------------------- | ------------------ |
| `commit('increment')`  | `store.count++`    |
| `dispatch('loadUser')` | `store.loadUser()` |

**Pinia preserves:**

- predictability
- devtools integration
- time-travel debugging

But removes ceremony.

## 10. 🧩 Organizing Stores in Large Projects

**Folder structure:**

```
src/
    stores/
        user.js
        cart.js
        settings.js
        notifications.js
```

**Best practices:**

- One store per domain ("feature store")
- Avoid mega-stores (too much state)
- Encapsulate logic in actions — not components
- Use getters to avoid duplicating derived logic
- Split stores when they start to feel crowded
- Use composables for local logic; stores for global logic

## 11. 🧩 Common Gotchas

**❌ Forgetting to install Pinia**

```js
const app = createApp(App);
app.use(createPinia());
```

**❌ Mutating store state outside actions (in strict mode only)**

- Pinia allows this by default, but strict mode plugins may require actions.

**❌ Using watchers inside components instead of getters**

- Getters are cached and integrated with devtools.

**❌ Trying to use `setup()` stores without calling the function**

```js
const store = useCounterStore(); // must call it!
```

## 12. 🧠 Key Takeaways

- Global state is needed when many components share the same data.
- Pinia is the modern, flexible, and simpler successor to Vuex.
- **State** = raw reactive data
- **Getters** = computed values
- **Actions** = async logic + state mutations
- Pinia works seamlessly with the Composition API.
- Migration from Vuex to Pinia removes ceremony while keeping control.
- You should organize stores by domain and keep them tight, focused modules.
