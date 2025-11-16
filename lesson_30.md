# 🎨 Lesson 30 — Advanced Slot Patterns: Dynamic Slots, Slot-State Sharing & Higher-Order Components

Slots are far more than "template placeholders."

At an advanced level, they become a powerful composition mechanism for:

- Passing reactive state down the tree
- Creating dynamic, user-defined UIs
- Encapsulating behavior behind flexible APIs
- Implementing patterns similar to React's render props or higher-order components
- Building complex and reusable design system components

This lesson explores the techniques only expert-level Vue developers use.

## 1. 🔍 Concept Overview

Slots in Vue come in three forms:

1. **Default slots** — Simple content injection.
2. **Named slots** — Inject multiple regions of a component layout.
3. **Scoped slots** — Pass reactive data from child → parent template.

Advanced slot patterns extend these basic ideas to:

- ✔ Dynamic slot names
- ✔ Slots that expose reactive logic
- ✔ Slot-driven layout patterns
- ✔ Higher-order components created through slot APIs

## 2. 💡 Mental Model / Analogy

🧠 Think of slots as **"mini-portals"** into a child component's internal logic.

But unlike props (one-way), slots let children send data up to the parent template while still letting the parent control the markup.

**Slots = data flow down + UI flow up.**

This allows extremely flexible component APIs.

## 3. 🧱 Scoped Slots — The Foundation of Advanced Slot Patterns

A scoped slot exposes internal component state:

**Child.vue**

```vue
<template>
  <slot :count="count" :increment="increment"></slot>
</template>

<script setup>
import { ref } from "vue";
const count = ref(0);
const increment = () => count.value++;
</script>
```

**Parent.vue**

```vue
<Child v-slot="{ count, increment }">
    <button @click="increment">
        Count: {{ count }}
    </button>
</Child>
```

This is the foundation for:

- renderless components
- headless UI components
- advanced reusable widgets
- internal state sharing

## 4. 🧱 Dynamic Slots (Highly Advanced)

You can bind slot names dynamically:

**Inside a child**

```vue
<slot name="header"></slot>
<slot name="content"></slot>
<slot :name="dynamicSection"></slot>
```

**Parent**

```vue
<MyComponent>
    <template #header>...</template>
    <template #content>...</template>
    <template #[someDynamicName]>...</template>
</MyComponent>
```

**Use cases:**

- CMS-driven layouts
- Dynamic dashboards
- Pluggable UI modules
- Menu + toolbar builders

This lets the component author define the regions, and the consumer dynamically inject UI.

## 5. 🧱 Slot-Based State Sharing (The Secret to Flexible UI Libraries)

Components can share state through slots without needing `provide`/`inject`.

**Example: Simple Tabs built only with slots**

**Tabs.vue**

```vue
<script setup>
import { ref, provide } from "vue";

const selected = ref(0);
const select = (i: number) => (selected.value = i);
</script>

<template>
  <slot :selected="selected" :select="select" />
</template>
```

**Parent usage**

```vue
<Tabs v-slot="{ selected, select }">
    <div class="tabs">
        <button
            v-for="(tab, i) in tabs"
            :class="{ active: selected === i }"
            @click="select(i)"
        >
            {{ tab.label }}
        </button>
    </div>

    <div class="content">
        {{ tabs[selected].content }}
    </div>
</Tabs>
```

This is **renderless behavior + slot-driven UI**.

UI libraries like Radix, Headless UI, etc. work exactly like this.

## 6. 🧱 Using Slots to Implement Higher-Order Components (HOCs)

A higher-order component is a component that wraps other components and enhances them.

Vue doesn't use HOCs as often as React, but you can replicate them easily with slots:

**Example: HOC that adds loading state**

**WithLoading.vue**

```vue
<script setup>
import { ref } from "vue";
const loading = ref(true);

setTimeout(() => (loading.value = false), 1000);
</script>

<template>
  <slot :loading="loading"></slot>
</template>
```

**Usage**

```vue
<WithLoading v-slot="{ loading }">
    <MyComponent v-if="!loading" />
    <Loader v-else />
</WithLoading>
```

This effectively enhances the wrapped component with loading logic.

## 7. 🧬 Composition API + Slots = Hyper-Flexible Patterns

Combined patterns allow you to create:

### ✔ Layout components

Page, Sidebar, Toolbar, etc.

Slots allow multiple layout regions

### ✔ Configurable widgets

- Charts
- Forms
- Timelines

### ✔ Permission-based rendering

```vue
<IfPermission role="admin" v-slot="{ allowed }">
    <AdminPanel v-if="allowed" />
</IfPermission>
```

### ✔ Reactive form shells

Slots expose:

- validation
- dirty/touched state
- submit handlers

Parents render any form they want.

## 8. 🧱 Ultra-Advanced Pattern: Slot Props as Injectable State APIs

You can expose a reactive state API via slots.

**Provider.vue**

```vue
<script setup>
import { reactive } from "vue";
const store = reactive({ count: 0 });
</script>

<template>
  <slot :store="store" />
</template>
```

**Consumer**

```vue
<Provider v-slot="{ store }">
    <button @click="store.count++">
        {{ store.count }}
    </button>
</Provider>
```

This is effectively:

- a fully reactive local store
- passed through slot-prop injection
- used in place of global state or `provide`/`inject`
- scoped only to the subtree

**One of the most powerful UI patterns in Vue.**

## 9. 🧩 Common Gotchas

- ❌ **Overusing slot-state for everything** — If your component doesn't need flexible UI, use props.
- ❌ **Exposing too much internal state** — Expose minimal API. Hide implementation details.
- ❌ **Deeply nested slot use (slot pyramids)** — Refactor into renderless or provider components.
- ❌ **Wrapping large templates in `<slot>` can hurt readability** — Use named slots to keep structure clear.

## 10. 🧠 Key Takeaways

- Slots are more than placeholders — they're a **UI composition API**.
- Scoped slots allow state to flow from child → parent elegantly.
- Dynamic slots support user-defined regions for extremely flexible UIs.
- Slot-based state sharing powers renderless and headless UI patterns.
- Slots enable patterns similar to higher-order components.
- This approach is essential for reusable design systems and libraries.

**You now understand the three most advanced component patterns in Vue:**

1. Renderless components
2. Headless UI patterns
3. Advanced slot architecture

These are the tools professionals use to create real UI frameworks.
