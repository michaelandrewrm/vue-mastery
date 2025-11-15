# 🕒 Lesson 13 — Asynchronous Behavior & Suspense

Modern apps fetch and load data asynchronously. Vue provides several powerful patterns for organizing async logic and rendering UI elegantly, especially with the `<Suspense>` component introduced in Vue 3.

## We'll connect:

- Async logic in components & composables
- Async components
- Vue's built-in `<Suspense>` system
- Fallbacks and controlled loading
- How async fits into Vue's reactive update loop

## 1. 🔍 Concept Overview

Vue's async capabilities revolve around four key concepts:

### 1. Async state (Promises + reactive state)

Handle data loading, errors, and loading indicators.

### 2. Async components

Dynamically load components when needed (code splitting).

### 3. The `<Suspense>` component

Render async components with fallbacks while awaiting completion.

### 4. Async composables

Encapsulate async logic cleanly (e.g., fetching data).

**These features help build:**

- Data-driven pages
- Deferred loading
- Split bundles
- Smooth loading experiences

## 2. 💡 Mental Model / Analogy

🧠 **Think of `<Suspense>` like a "loading airlock."**

When a component (or composable) performs async work, Vue checks:

- Has everything resolved yet?
- If not, `<Suspense>` holds back the main UI and shows a fallback instead.

**Once async resolves?**

- The "airlock door" opens
- The real UI "enters the room"

This creates a smooth handoff from loading → ready.

## 3. 🧱 Basic Async Data Loading in Vue

A simple pattern:

```vue
<script setup>
import { ref, onMounted } from "vue";

const user = ref(null);
const loading = ref(true);

onMounted(async () => {
  const res = await fetch("/api/user");
  user.value = await res.json();
  loading.value = false;
});
</script>

<template>
  <p v-if="loading">Loading...</p>
  <p v-else>Hello {{ user.name }}</p>
</template>
```

**This is the foundation:**  
Async → update reactive state → UI updates automatically.

## 4. ⚙️ Async Components

You can load a component lazily:

```js
import { defineAsyncComponent } from "vue";

const AsyncProfile = defineAsyncComponent(() => import("./Profile.vue"));
```

Or with options:

```js
const AsyncProfile = defineAsyncComponent({
  loader: () => import("./Profile.vue"),
  delay: 200,
  timeout: 3000,
  loadingComponent: LoadingSpinner,
  errorComponent: ErrorFallback,
});
```

**This enables:**

- Code splitting
- Feature-level lazy loading
- Better performance

## 5. 🧱 Using `<Suspense>` for Async UI

Basic structure:

```vue
<Suspense>
    <template #default>
        <AsyncProfile />
    </template>

    <template #fallback>
        <LoadingSpinner />
    </template>
</Suspense>
```

**What happens:**

1. `<AsyncProfile />` triggers an async import
2. Suspense detects the pending promise
3. Suspense renders `<LoadingSpinner />`
4. Once async resolves → renders `<AsyncProfile />`

This also works for components that use async `setup()` — a powerful pattern.

## 6. 🧱 Async Setup Function (Core Feature)

A component's `setup()` can be async:

```vue
<script setup>
const res = await fetch("/api/user");
const user = await res.json();
</script>

<template>
  <p>Hello {{ user.name }}</p>
</template>
```

### How this interacts with `<Suspense>`

- Template waits for `setup()` to finish
- If wrapped in `<Suspense>`, fallback is shown in the meantime

This leads to very concise data-loading components.

## 7. 🧬 Composables with Async Logic

Let's build a reusable composable.

**useFetch.js**

```js
import { ref } from "vue";

export function useFetch(url) {
  const data = ref(null);
  const error = ref(null);
  const loading = ref(true);

  const load = async () => {
    loading.value = true;
    try {
      const res = await fetch(url);
      if (!res.ok) throw new Error("Fetch failed");
      data.value = await res.json();
    } catch (err) {
      error.value = err;
    } finally {
      loading.value = false;
    }
  };

  load();

  return { data, error, loading, reload: load };
}
```

Use it in a component:

```vue
<script setup>
import { useFetch } from "./useFetch.js";

const { data, loading, error, reload } = useFetch("/api/user");
</script>

<template>
  <div v-if="loading">Loading...</div>
  <pre v-else-if="error">{{ error.message }}</pre>
  <div v-else>Hello {{ data.name }}</div>
</template>
```

This separates async logic cleanly.

## 8. 🧱 Combining Composables + Suspense

A powerful pattern:

**useUser.js**

```js
export async function useUser() {
  const res = await fetch("/api/user");
  return await res.json();
}
```

**User.vue**

```vue
<script setup>
const user = await useUser();
</script>

<template>
  <p>Hello {{ user.name }}</p>
</template>
```

**Page.vue**

```vue
<Suspense>
    <template #default>
        <User />
    </template>

    <template #fallback>
        <p>Loading user info...</p>
    </template>
</Suspense>
```

This yields extremely clean data-driven components.

## 9. ⚙️ Internal Insight — How Vue Tracks Async Operations

When Vue sees an async component or async setup:

1. It tracks pending promises
2. Suspense waits for all of these promises
3. The fallback is shown until all promises resolve
4. Rendering resumes once async completes

**The reactivity system isn't blocked** —  
only the initial rendering of the async component is deferred.

### Suspense maintains rendering boundaries

It creates a controlled region of the tree:

```
Suspense boundary
├── fallback
└── async subtree
```

This lets Vue alternate between completed and pending content granularly.

## 10. 🧩 Common Gotchas

### ❌ Suspense only works in async setup or async components

Normal async logic in `mounted()` doesn't trigger Suspense fallback.

### ❌ Mixing loading UI in both Suspense and component

Choose one:

- Suspense handles loading
- Component handles loading

Avoid duplicating.

### ❌ Not catching async errors

Always wrap async in try/catch if using inside setup or composables.

### ❌ Forgetting that Suspense waits for all nested async components

Deeply nested async components → all must resolve before showing content.

### ❌ Assuming Suspense works everywhere (e.g., SSR differs)

On the server, Suspense acts differently (full hydration rules apply).

## 11. 🧠 Key Takeaways

- Async logic integrates naturally with Vue's reactivity system.
- Async components enable code splitting.
- `<Suspense>` provides clean loading states for async content.
- Async setup works seamlessly with Suspense boundaries.
- Composables allow organizing async logic cleanly and reusably.
- Suspense serves as a "loading airlock" for async rendering.
- Proper async patterns lead to clean, declarative data-driven UIs.
