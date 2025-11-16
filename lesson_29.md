# 🎨 Lesson 29 — Headless UI Patterns (Controlled/Uncontrolled Components & State Machines)

A headless component provides:

- All logic
- All state
- All accessibility behavior

…but NO default UI, leaving rendering entirely up to the consumer.

This pattern powers:

- Headless UI (Vue)
- Radix Vue
- Vuetify's low-level primitives
- Internal enterprise component libraries
- Accessible dropdowns, modals, tabs, accordions
- Large-scale design systems

It builds directly on renderless components plus some advanced patterns.

## 1. 🔍 Concept Overview

Headless UI patterns rely on four pillars:

### 1. Controlled / Uncontrolled Components

Components can be controlled externally or internally.

### 2. State Machines

Explicitly model component behavior, reducing "UI spaghetti."

### 3. Context Providers (provide/inject)

Share state deeply across components.

### 4. Renderless Component Architecture

Separate logic from UI.

Together, these patterns produce reliable, reusable, theme-agnostic UI logic.

## 2. 💡 Mental Model / Analogy

🧠 Think of a headless UI component like a **"robot skeleton."**

- It can move
- It can sense interactions
- It has rules
- It coordinates multiple parts

**You attach the skin and style.**

In contrast:

- **Renderless components** expose raw state
- **Headless components** expose a complete behavior model

## 3. 🧱 Controlled vs Uncontrolled Components

### ✔ Uncontrolled component

Internal state is managed by the component.

**Example:**

```vue
<Dialog>
    <DialogTrigger>Open</DialogTrigger>
    <DialogContent>...</DialogContent>
</Dialog>
```

The dialog controls its own open state.

### ✔ Controlled component

Parent controls state via props + emits.

```vue
<Dialog :open="isOpen" @update:open="isOpen = $event">
    <DialogTrigger>Open</DialogTrigger>
    <DialogContent>...</DialogContent>
</Dialog>
```

The parent owns state; the component simply follows.

### ✨ Best practice

Headless UI components should support **both**.

This gives flexibility:

- Simple usage → uncontrolled
- Complex usage → controlled

Vue 3.3's `defineModel` makes this very easy.

## 4. 🧱 State Machine Driven UI (The Secret Weapon)

Most serious headless UI libraries use **state machines** under the hood.

### What is a state machine?

A set of discrete states with defined transitions.

**Example: Dropdown**

```
closed
    └── click => open

open
    └── click outside => closed
    └── escape => closed
    └── select => closed
```

This avoids:

- nested if/else
- unpredictable side effects
- tangled event handling logic
- race conditions in async transitions

### Vue + XState or tiny custom state machines

You can use:

- `xstate`
- `@zag-js/core`
- or write small custom state machines

## 5. 🧱 Example: Headless Dropdown Using Provide/Inject + State Machine

### DropdownRoot.vue

```vue
<script setup lang="ts">
import { reactive, provide } from "vue";

const state = reactive({
  open: false,
});

function toggle() {
  state.open = !state.open;
}

function close() {
  state.open = false;
}

provide("dropdown", { state, toggle, close });
</script>

<template>
  <slot />
</template>
```

### DropdownTrigger.vue

```vue
<script setup>
import { inject } from "vue";
const ctx = inject("dropdown");
</script>

<template>
  <button @click="ctx.toggle">
    <slot />
  </button>
</template>
```

### DropdownContent.vue

```vue
<script setup>
import { inject } from "vue";
const ctx = inject("dropdown");
</script>

<template>
  <div v-if="ctx.state.open">
    <slot />
  </div>
</template>
```

### Usage

```vue
<DropdownRoot>
    <DropdownTrigger>Menu</DropdownTrigger>
    <DropdownContent>
        <button>Item 1</button>
        <button>Item 2</button>
    </DropdownContent>
</DropdownRoot>
```

This architecture:

- Encapsulates logic
- Shares state using provide/inject
- Allows any UI shape
- Provides full composability

This is exactly how **Radix Vue** and **Headless UI** work.

## 6. 🧬 Accessibility Patterns (Critical!)

Headless UI must handle accessibility behavior:

- Keyboard navigation
- Focus trapping (modals)
- Roving tabindex (menus, lists, tabs)
- ARIA attributes (`aria-haspopup`, `role="dialog"`, etc.)
- Escape key handling
- Focus return on close

Logic must be built in; UI is injected.

**Example:** roving tabindex for menus → maintain:

```js
const activeIndex = ref(-1);
```

Expose it via provide/inject.

Use arrow keys to update it.

Slots handle:

```vue
<button :tabindex="index === activeIndex ? 0 : -1">Item</button>
```

## 7. 🧱 Combining everything — Headless Tabs Example

### Root: manages selected tab

```js
const selected = ref(0);
provide("tabs", { selected });
```

### Tab List: roving focus

Manages arrow navigation.

### Tab:

- sets `selected.value = index`
- applies `aria-selected`

### Panel:

- shows based on selected tab

Each part is separated for maximum flexibility.

## 8. 🧩 Common Gotchas

### ❌ Using only renderless components without context providers

Headless UI almost always requires provide/inject.

### ❌ Mixing state ownership

Ensure controlled/uncontrolled logic is consistent.

### ❌ Forgetting accessibility rules

Headless UI must maintain:

- ARIA attributes
- focus sequencing
- keyboard interactions

### ❌ Over-complex state machines

Start small; grow only when behavior becomes tricky.

## 9. 🧠 Key Takeaways

- Headless UI ≠ renderless components
- Headless = renderless + controlled/uncontrolled + state machines + accessibility.
- Good headless components expose:
  - state
  - event handlers
  - context API
  - accessibility hints
- Controlled/uncontrolled patterns make components flexible.
- Provide/inject is the backbone of multi-part components.
- State machines prevent UI chaos and ensure predictable behavior.
- This architecture is what enables library-quality components.
