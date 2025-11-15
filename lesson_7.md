# 🔮 Lesson 7 — Computed & Watchers

Computed properties and watchers are both triggered by reactive state…
…but they exist for very different reasons.

Understanding this distinction is essential for predicting Vue's behavior and writing clean, maintainable code.

## 1. 🔍 Concept Overview

### Computed properties

- Represent derived state — state that can be computed from other state.
- Are cached and lazy.
- Automatically update the DOM when dependencies change.
- Should be pure functions with no side effects.

### Watchers

- Run side effects in response to reactive changes.
- Are not cached.
- Let you watch a specific reactive source and respond manually.

---

## 2. 💡 Mental Model / Analogy

### 🧠 Computed = a smart spreadsheet cell

Like Excel:

```
A1 = 10
A2 = 20
A3 = A1 + A2 // A3 automatically updates
```

Vue treats computed values the same way — if inputs change, the output recalculates.

### 🧠 Watchers = a security camera

- Watching for changes.
- When something changes, it "notifies" you and you run a custom action.

**Computed = derived value**  
**Watch = reaction**

## 3. 🧱 Code Example — Computed vs Watch

```vue
<script setup>
import { ref, computed, watch } from "vue";

const first = ref("");
const last = ref("");

// Computed (derived state)
const fullName = computed(() => {
  console.log("computed run");
  return `${first.value} ${last.value}`.trim();
});

// Watch (side effects)
watch(fullName, (newVal, oldVal) => {
  console.log(`Name changed from "${oldVal}" to "${newVal}"`);
});
</script>

<template>
  <input v-model="first" placeholder="First" />
  <input v-model="last" placeholder="Last" />

  <p>Full name: {{ fullName }}</p>
</template>
```

**What happens internally:**

1. Reading `{{ fullName }}` registers a dependency.
2. Vue runs the computed getter lazily (only when needed).
3. Changing `first` or `last` invalidates the cache → computed recomputes.
4. The watcher sees the change and runs its callback.

## 4. ⚙️ Internal Insight — Vue's Computed & Watch Mechanisms

### 🔹 Computed's Lazy Evaluation

A computed has an internal "effect" with `lazy: true`.

**Steps:**

1. Vue does not run computed on creation.
2. The first time something reads the computed:
   - Vue runs the getter
   - Tracks dependencies
   - Stores result in a cached value
3. When dependencies change:
   - Vue marks computed as dirty
4. Next time someone reads it:
   - It recalculates and re-caches the value

**This ensures:**

- Computed runs only when needed
- Never recalculates if the value wasn't read
- Avoids unnecessary work

### 🔹 Watchers Run Side Effects

When you call `watch(source, callback)`:

1. Vue evaluates `source` to track dependencies.
2. Whenever dependencies change:
   - The watcher callback runs with `(newValue, oldValue)`.

Watchers do not return anything useful.  
Their purpose is performing effects, like:

- Fetching data
- Logging
- Imperative DOM logic
- Syncing local and external state
- Updating localStorage

### 🔹 Computed vs Watch — Key Internal Differences

| Feature               | computed      | watch         |
| --------------------- | ------------- | ------------- |
| Purpose               | derived state | side effects  |
| Cached?               | Yes           | No            |
| Lazy?                 | Yes           | No            |
| Return value          | usable        | callback-only |
| Stateful?             | No            | Yes           |
| Reruns when not read? | No            | Yes           |

## 5. 🧩 Example: Reactive Form Validation

Let's build a simple reactive form:

```vue
<script setup>
import { ref, computed, watch } from "vue";

const email = ref("");
const password = ref("");

// Derived state: computed validation
const isEmailValid = computed(() =>
  /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email.value)
);

const isPasswordValid = computed(() => password.value.length >= 6);

// Derived state: form validity
const isFormValid = computed(() => isEmailValid.value && isPasswordValid.value);

// Side effect: auto-save when valid
watch(isFormValid, (valid) => {
  if (valid) {
    console.log("Auto-saving valid form...");
  }
});
</script>

<template>
  <input v-model="email" placeholder="Email" />
  <input v-model="password" type="password" placeholder="Password" />

  <p>Email valid: {{ isEmailValid }}</p>
  <p>Password valid: {{ isPasswordValid }}</p>
  <p>Form valid: {{ isFormValid }}</p>
</template>
```

**What's happening?**

- Validations are pure, derived logic → **computed**.
- Auto-save is an imperative side effect → **watch**.

This is the exact separation of responsibilities Vue encourages.

## 6. 🧩 Common Gotchas / Misconceptions

### ❌ Using watch to compute a new piece of state

```js
watch(count, () => (total.value = count.value * 2));
// Don't do this — use computed instead
```

**Use:**

```js
const total = computed(() => count.value * 2);
```

### ❌ Using computed for side effects like console.log or fetch

Computed must remain pure.  
If it has side effects → use **watch**.

### ❌ Expecting computed to run multiple times unnecessarily

Computed is cached.  
If dependencies didn't change, it won't re-run.

### ❌ Watching reactive objects without deep option

```js
watch(user, callback); // only triggers on object reference change
```

**Use:**

```js
watch(user, callback, { deep: true });
```

or watch a specific key:

```js
watch(() => user.age, callback);
```

## 7. ⚙️ Advanced Insight / Performance Angle

### Computed prevents unnecessary DOM updates

Because computed values cache their output:

- Multiple components reading the same computed don't cause re-computations.
- When dependencies haven't changed, computed returns the cached value immediately.
- This saves massive CPU time in large apps.

### Watchers are scheduled intelligently

Vue batches watchers:

- Multiple state changes → one batch execution
- DOM updates run before watcher callbacks
- Allows consistent DOM state in watchers (post-render)

## 8. 🧠 Key Takeaways

- **Computed = derived state** (pure, cached, lazy).
- **Watch = side effects** (imperative, non-cached).
- Computed runs only when dependencies change and the computed is read.
- Watch **always** runs when dependencies change.
- Use **computed** for validation, formatting, derived fields.
- Use **watch** for API calls, logging, DOM interaction, syncing state.
- Proper separation leads to predictable, performant code.
