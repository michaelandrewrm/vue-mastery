# 🌿 Lesson 11 — Custom Composables

The Composition API introduces a new way to organize, reuse, and share logic across components.
Custom composables are the heart of scalable Vue apps.

These are functions that:

- Use reactive primitives (`ref`, `reactive`, `computed`, etc.)
- Encapsulate logic (state + effects + watchers)
- Return reusable "logic modules"
- Can be shared across any component

They allow us to think of Vue as a "logic-first" framework.

## 1. 🔍 Concept Overview

Composables encapsulate 4 things:

- State (refs / reactives)
- Derived state (computed)
- Reactions (watch)
- Side effects (lifecycle hooks)

Instead of scattering logic across components, composables let you extract entire features into standalone, reusable functions.

This yields:

- Cleaner components
- Centralized logic
- Easier testing
- Better separation of concerns
- Reusable architecture

## 2. 💡 Mental Model / Analogy

🧠 Think of composables as "mini-components without templates."

They are:

- "Component logic in a bag"
- Feature modules
- Like React Hooks, but built on Vue's fine-grained reactivity

A composable gives you Lego blocks of state + methods + effects, and you plug them into components.

## 3. 🧱 Code Example — A Simple Counter Composable

**useCounter.js**

```js
import { ref } from "vue";

export function useCounter(initial = 0) {
  const count = ref(initial);

  const increment = () => count.value++;
  const decrement = () => count.value--;
  const reset = () => (count.value = initial);

  return { count, increment, decrement, reset };
}
```

**Using it in a component**

```vue
<script setup>
import { useCounter } from "./useCounter.js";

const { count, increment } = useCounter(10);
</script>

<template>
  <button @click="increment">Count: {{ count }}</button>
</template>
```

## 4. ⚙️ Using Composables for State & Effects

Composables aren't just for state — they can include side effects and lifecycle hooks too.

**useMouse.js**

```js
import { ref, onMounted, onUnmounted } from "vue";

export function useMouse() {
  const x = ref(0);
  const y = ref(0);

  function update(e) {
    x.value = e.clientX;
    y.value = e.clientY;
  }

  onMounted(() => {
    window.addEventListener("mousemove", update);
  });

  onUnmounted(() => {
    window.removeEventListener("mousemove", update);
  });

  return { x, y };
}
```

**Use in any component**

```vue
<script setup>
import { useMouse } from "./useMouse.js";
const { x, y } = useMouse();
</script>

<template>
  <p>Mouse: {{ x }}, {{ y }}</p>
</template>
```

The composable:

- Owns its side effects
- Cleans up after itself
- Returns state ready to be used anywhere

## 5. 🧬 Dependency Injection with `provide`/`inject`

Some composables need to expose shared context across the component tree.

Vue's solution: **`provide`/`inject`**.

### `provide`/`inject` mental model

Think of `provide`/`inject` as a reactive context system:

- Parent **provides** a value
- Any descendant component (not just child) can **inject** it

This is great for:

- Global services
- Theming
- Internationalization
- Shared state modules
- Form context
- Parent-child communication without prop drilling

### 🧱 Example — Simple Theme Composable

**theme.js**

```js
import { ref, provide, inject } from "vue";

const ThemeSymbol = Symbol("theme");

export function provideTheme() {
  const theme = ref("light");
  const toggleTheme = () =>
    (theme.value = theme.value === "light" ? "dark" : "light");

  provide(ThemeSymbol, { theme, toggleTheme });

  return { theme, toggleTheme };
}

export function useTheme() {
  const themeContext = inject(ThemeSymbol);
  if (!themeContext)
    throw new Error("useTheme() must be used after provideTheme()");
  return themeContext;
}
```

**RootComponent.vue**

```vue
<script setup>
import { provideTheme } from "./theme.js";
provideTheme();
</script>

<template>
  <slot />
</template>
```

**Somewhere deep in the tree…**

```vue
<script setup>
import { useTheme } from "./theme.js";
const { theme, toggleTheme } = useTheme();
</script>

<template>
  <button @click="toggleTheme">Theme: {{ theme }}</button>
</template>
```

This avoids passing theme through 20 levels of props.

## 6. 🧱 Organizing Composables for Large Projects

### Recommended folder structure

```
src/
    composables/
        useAuth.js
        useForm.js
        useFetch.js
        useMouse.js
        useLocalStorage.js
        useModal.js
    features/
        user/
            useUserProfile.js
            useUserSettings.js
        cart/
            useCart.js
            useCheckout.js
```

### Best practices

- Prefix with `use` (convention-based discovery)
- Group by feature, not by type
- Extract all complex logic from components
- Keep composables pure except for lifecycle hooks
- If composable uses provide/inject → it provides a "context service"
- Export plain functions — no classes

### When composables grow too large

Split them into:

- state composables
- effects composables
- feature composables

This keeps your logic tidy and composable.

## 7. 🧩 Common Gotchas

**❌ Mistake: Trying to share reactive state unintentionally**

Each call to a composable creates new state.

If you want shared state → create a store (Pinia).

**❌ Mistake: Using provide/inject as a replacement for props**

- Props → component API contracts
- Provide/inject → global or contextual services

**❌ Mistake: Forgetting lifecycle hooks work inside composables**

Composables run inside `setup()`, so:

```js
onMounted(...)
onUnmounted(...)
```

work flawlessly.

**❌ Mistake: Returning undeclared or unrelated state**

Composables should return a clean, purposeful API.

## 8. ⚙️ Advanced Insight — Composables + Re-render Optimization

Composables allow Vue to:

- Build independent reactive branches
- Reduce component complexity
- Hoist reusable logic outside render

Vue's compiler ensures:

- No extra renders
- No performance overhead for composables
- Pure logic stays cached and reused

This is one reason Composition API scales far better than Options API.

## 9. 🧠 Key Takeaways

- Composables extract reusable logic (state + effects + derived state).
- Reusable, testable features are built with composables.
- Lifecycle hooks work inside composables just like in components.
- Provide/inject allows building shared contextual services.
- Organize composables by feature for large codebases.
- Composables improve scalability, code reuse, and maintainability.
