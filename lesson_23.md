# 🏗️ Lesson 23 — Large-Scale Architecture & Code Organization

As Vue applications grow:

- Components multiply
- Logic spreads
- State becomes interdependent
- Features evolve over time
- Files balloon

A small, clean codebase can turn into a tangled mess without architectural discipline.

**This lesson teaches:**

- How to design scalable, modular Vue apps
- How to structure code by feature, not by type
- How to colocate logic with the components that need it
- How to use composables and stores to manage complexity
- Proven large-scale patterns used in real production apps

## 1. 🔍 Concept Overview

Vue's strengths — composition API, reactivity, modular components — naturally support scalable architectures.

**Key architectural tools:**

- ✔ **Feature-based folder structure** — Group files by feature, not file type.
- ✔ **Colocation of logic** — Keep component-specific logic close to the component.
- ✔ **Composables** — Extract reusable logic into dedicated functions.
- ✔ **Domain-specific stores** — Centralized state per feature domain.
- ✔ **Module boundaries** — Avoid leaking internal implementations.

Ultimately, good architecture is about limiting coupling and localizing complexity.

## 2. 💡 Mental Model / Analogy

🧠 **Think of a large app like a city.**

- Each feature is a **neighborhood**
- Each component is a **building**
- Each composable is **utility infrastructure** (power, water)
- Each store is a **public service** (police, library, post office)
- The router is the **road system**
- The API layer is your **connection to the outside world**

**Good architecture means:**

- Neighborhoods are distinct
- Buildings are self-contained
- Utilities don't bleed into the wrong areas
- Services are organized, reliable, and predictable

## 3. 🧱 Feature-Based File Structuring

### ❌ Not scalable (type-based structure):

```
components/
views/
store/
composables/
utils/
```

This seems neat at first…  
…but when an app grows, everything becomes mixed and hard to navigate.

### ✔ Scalable (feature-based structure):

```
src/
    features/
        auth/
            components/
            pages/
            store/
            composables/
            services/
        cart/
            components/
            composables/
            store/
        products/
            pages/
            services/
            store/
    shared/
        ui/
        composables/
        utils/
        store/
    app/
        router/
        layout/
    App.vue
    main.js
```

**Benefits:**

- Features become isolated "modules"
- Easier maintenance and deletion of outdated features
- Teams can own entire feature folders independently
- Infrastructure (router, stores, composables) is scoped to feature domains

This mirrors patterns in React (Next.js), backend services, and microfrontends.

## 4. 🧱 Colocation: Keep Logic Close to Where It's Used

**Colocation means:**  
"Put code where people expect to find it."

**Examples:**

- A component-specific composable goes into the component's feature folder
- Business logic (fetching, validation) lives next to the feature that owns it
- A page-level component declares its own async data logic

**Example directory:**

```
products/
    pages/
        ProductDetail.vue
        useProductDetail.js  <-- colocated
    components/
        ProductCard.vue
```

**Why colocate?**

- Developers don't have to hunt around the codebase
- Easy to delete code when features die
- Encourages domain ownership

## 5. 🧱 Managing Complexity with Composables

Composables allow you to break complex logic into reusable chunks.

**Example:**

```js
// useCart.js
import { ref } from "vue";

export function useCart() {
  const items = ref([]);
  const total = computed(() => items.value.reduce((a, b) => a + b.price, 0));

  function add(product) {
    items.value.push(product);
  }

  return { items, total, add };
}
```

**Where to place it?**

- ✔ If only used by Cart feature: `features/cart/composables/useCart.js`
- ✔ If used across app: `shared/composables/useCart.js`

**Composables handle:**

- Reusable logic
- Side effects
- Data fetching
- Cross-component coordination
- Utility behavior

They are the backbone of modular systems.

## 6. 🧱 Stores for Domain-Level State

Stores (Pinia) should represent:

- Global state
- Domain-level shared state
- Cross-page state
- Feature-level services

**Example folder:**

```
features/cart/store/cartStore.js
```

**Example store:**

```js
import { defineStore } from "pinia";

export const useCartStore = defineStore("cart", {
  state: () => ({
    items: [],
  }),

  getters: {
    total: (state) => state.items.reduce((sum, i) => sum + i.price, 0),
  },

  actions: {
    add(item) {
      this.items.push(item);
    },
  },
});
```

**Rules for store design:**

- Do NOT make stores massive — split by domain
- Avoid putting everything in one "global" store
- Avoid business logic in components → colocate in composables or actions
- Think of stores as services, not just "global variables"

## 7. 🧬 Organizing Asynchronous Logic

Every large-scale app needs consistent data-fetching patterns.

**Use:**

- `useFetch` or `useAsyncData` (Nuxt)
- Feature-specific `services/` folder
- Composables for request caching and error handling

**Example:**

```
products/
    services/
        api.js
```

```js
// products/services/api.js
export function fetchProduct(id) {
  return fetch(`/api/products/${id}`).then((r) => r.json());
}
```

This isolates API concerns from components and stores.

## 8. 🔍 Large-Scale Component Patterns

### Smart / Controller Components

- Own logic
- Fetch data
- Coordinate children

### Dumb / Presentational Components

- Accept data as props
- Emphasize UI
- Reusable
- Stateless or local state only

**Example:**

- `ProductPage.vue` → smart (controller)
- `ProductLayout.vue` → smart
- `ProductCard.vue` → dumb (UI-only)
- `StarRating.vue` → dumb

This prevents bloated UI components and makes logic testable.

## 9. 🧩 Preventing Architecture Decay

**Large Vue apps fail when:**

- Every component fetches its own data
- Stores become monolithic
- Too many watchers & effects create reactive explosions
- No boundaries between features
- Shared code becomes "global dumping ground"
- Logic is spread randomly instead of grouped by domain

**Architecture antidotes:**

- Feature modules
- Composables
- Domain stores
- Smart/dumb component distinction
- Clear folder structure
- SSR-aware async design (Nuxt apps)

## 10. 🧠 Key Takeaways

- **Organize your app by feature, not by file type.**
- **Colocate logic** (composables, services) with the feature that uses them.
- **Use stores per domain**, not one giant store.
- **Use composables** to encapsulate logic and limit component responsibilities.
- **Separate smart/controller components** from dumb/presentational components.
- **Use consistent patterns** for async data, caching, and side effects.
- **Architecture should make the codebase easy to extend and safe to delete.**

If you master these principles, you can build Vue/Nuxt applications that stay clean, scalable, and maintainable for years.
