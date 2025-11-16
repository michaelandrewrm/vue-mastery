# 🎨 Lesson 28 — Renderless Components (Advanced Composition Pattern)

Renderless components are one of the most powerful architectural tools in Vue for building reusable logic without UI. They are heavily used in design systems, headless UI frameworks, accessibility tooling, and complex component libraries.

## 1. 🔍 Concept Overview

A **renderless component** is a component that:

- Manages logic, state, and behavior
- But renders nothing by default
- And exposes its state via slots

This pattern allows you to:

- Separate logic from presentation
- Let consumers define their own UI
- Share reusable interactive behavior

Examples found in real ecosystems:

- Headless UI (Vue)
- Radix UI
- Date pickers
- Accordions
- Dropdowns
- Forms with validation

Renderless components are the "bridge" between UI components and composables.

## 2. 💡 Mental Model / Analogy

🧠 Think of a renderless component as the **"brain without a face."**

- It doesn't render the UI
- It only handles the logic
- You plug in the "face" (UI) via slots

You end up with reusable building blocks where you can choose how they look, while the component handles how they behave.

## 3. 🧱 Basic Renderless Component Example

A simple hover-tracker component:

**Hoverable.vue**

```vue
<script setup>
import { ref } from "vue";
const hovering = ref(false);

function onEnter() {
  hovering.value = true;
}

function onLeave() {
  hovering.value = false;
}

defineExpose({ hovering, onEnter, onLeave });
</script>

<template>
  <slot :hovering="hovering" :onEnter="onEnter" :onLeave="onLeave" />
</template>
```

**Usage**

```vue
<Hoverable v-slot="{ hovering, onEnter, onLeave }">
    <div @mouseenter="onEnter" @mouseleave="onLeave">
        Hover State: {{ hovering }}
    </div>
</Hoverable>
```

**Key takeaway:**

- Renderless component = no UI
- Consumer builds UI
- Component controls logic

## 4. ⚙️ Internal Insight — Why Renderless Components Work So Well

Renderless components use:

- **Scoped slots** to share state
- **exposed methods** to allow controlled interaction
- **Vue's reactivity system** to keep UI in sync

Under the hood:

- The component's `setup()` runs once
- Reactive state stays inside the component
- Template slot exposes state to consumer
- Consumers can ignore or override UI

This decouples UI from behavior completely.

## 5. 🧱 Renderless vs Composables

**Renderless component**

- Provides reactivity + lifecycle + slot binding
- Useful when you want reusable state + template API
- Perfect for UI frameworks and accessibility behavior

**Composable**

- Pure logic + reactivity
- No template
- No slots
- Must be used inside a component

🧠 **Rule of thumb:**

- If your reusable building block needs to "own" DOM behavior or provide slot state → **renderless**
- If it's just logic → **composable**

## 6. 🔍 Use Cases for Renderless Components

✔ **Dropdown logic**

- toggle state
- keyboard navigation
- focus trapping

✔ **Tabs logic**

- active tab tracking
- keyboard navigation
- accessibility roles

✔ **Form management**

- validation state
- input grouping
- errors + touched state

✔ **IntersectionObserver**

- shared scroll-based behavior

✔ **Virtual lists**

- scroll management
- item visibility
- optimized rendering

Renderless components are the backbone of headless UI libraries.

## 7. 🧱 Example: Renderless Dropdown Component

**Dropdown.vue**

```vue
<script setup>
import { ref, onMounted, onUnmounted } from "vue";

const open = ref(false);
const toggle = () => (open.value = !open.value);
const close = () => (open.value = false);

function onEscape(e) {
  if (e.key === "Escape") close();
}

onMounted(() => window.addEventListener("keydown", onEscape));
onUnmounted(() => window.removeEventListener("keydown", onEscape));
</script>

<template>
  <slot :open="open" :toggle="toggle" :close="close"></slot>
</template>
```

**Usage**

```vue
<Dropdown v-slot="{ open, toggle, close }">
    <button @click="toggle">Menu</button>

    <ul v-if="open">
        <li @click="close">Settings</li>
        <li @click="close">Profile</li>
    </ul>
</Dropdown>
```

**Benefits:**

- Full control over the HTML
- Reusable, encapsulated logic
- Allows building any UI styling/theme

## 8. 🧬 Accessibility Considerations

Renderless components are perfect for building accessible components.

**Use cases:**

✔ **Keyboard navigation**

- arrow keys
- tab / shift+tab

✔ **ARIA roles & attributes**

- Consumers decide which roles to apply based on the slot state.

✔ **Focus management**

- Renderless logic provides focus instructions; user defines the DOM.

Vue's reactivity + lifecycle hooks let you maintain accessibility behavior without dictating visuals.

## 9. 🧩 Common Gotchas

❌ **Storing DOM refs inside renderless components**

- Better: expose handlers through slots; let consumer bind refs.

❌ **Overusing renderless when a composable would suffice**

- Rule:
  - no template needed → composable
  - template API used → renderless

❌ **Not using key with complex slot structures**

- Slots can dynamically enter/leave → ensure stability.

❌ **Exposing too much state**

- Expose only the minimum required interface.

## 10. 🧠 Key Takeaways

- Renderless components separate behavior from presentation.
- They expose logic via scoped slots and leave UI entirely to the consumer.
- They are essential for reusable, theme-agnostic component libraries.
- Perfect for accessibility tooling and UI patterns like dropdowns, modals, tabs.
- Use composables for logic-only; renderless for slot-driven behavior.
