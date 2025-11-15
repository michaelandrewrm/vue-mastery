# 🎁 Lesson 9 — Slots and Component Composition

Slots allow a parent to pass template content into a child component.
They are the foundation of reusable UI components — buttons, cards, layouts, modals, dropdowns, tables, renderless components, and more.

They turn components into render portals: places where the parent injects custom DOM into a child's structure.

## 1. 🔍 Concept Overview

Slots let you:

- Insert content from the parent into the child's template
- Customize component internals without rewriting them
- Build flexible and expressive UI building blocks

There are three main types:

- Default slots
- Named slots
- Scoped slots (slots with data passed from child → parent)

Slots let you decouple structure (child) from content (parent).

## 2. 💡 Mental Model / Analogy

🧠 Slots are "cut-out windows" in a component.

A child component defines:

```vue
<slot></slot>
```

This is a window.

The parent places content into that window:

```vue
<MyCard>
    <h2>Hello!</h2>
</MyCard>
```

Named slots are specific windows, like:

- "header window"
- "footer window"

Scoped slots are windows with props, like:

- "Here's a window and here's some data to render inside it."

The child offers placeholders, the parent fills in the exact DOM.

## 3. 🧱 Code Example — Default Slot

**Child.vue**

```vue
<template>
  <div class="card">
    <slot></slot>
  </div>
</template>
```

**Parent.vue**

```vue
<template>
  <Child>
    <h2>Hello from parent!</h2>
  </Child>
</template>
```

The `<slot>` acts as a render portal: Parent content is rendered inside the child's structure.

## 4. 🧱 Named Slots

Named slots allow multiple insertion points.

**Child.vue**

```vue
<template>
  <div class="card">
    <header>
      <slot name="header"></slot>
    </header>

    <section>
      <slot></slot>
      <!-- default slot -->
    </section>

    <footer>
      <slot name="footer"></slot>
    </footer>
  </div>
</template>
```

**Parent.vue**

```vue
<template>
  <Child>
    <template #header>
      <h1>Title from Parent</h1>
    </template>

    <p>Main content!</p>

    <template #footer>
      <button>OK</button>
    </template>
  </Child>
</template>
```

This allows arbitrary structure to be composed.

## 5. 🧱 Scoped Slots — passing data from child → parent

Scoped slots allow the child to provide data for the parent to render.

**Child.vue**

```vue
<script setup>
const items = ["A", "B", "C"];
</script>

<template>
  <ul>
    <slot :item-list="items"></slot>
  </ul>
</template>
```

**Parent.vue**

```vue
<template>
  <Child>
    <template #default="{ itemList }">
      <li v-for="item in itemList" :key="item">
        {{ item }}
      </li>
    </template>
  </Child>
</template>
```

What's happening?

- Child emits data into slot props: `:item-list="items"`
- Parent receives them through destructuring: `{ itemList }`
- Parent decides how to render the list

This is wildly powerful: Children provide data, parents choose how to render it.

Render flexibility without tightly coupling components.

## 6. ⚙️ Internal Insight — How Vue Implements Slots

Vue transforms:

```vue
<slot></slot>
```

into something like:

```js
slots.default ? slots.default() : [];
```

And:

```vue
<slot :foo="bar"></slot>
```

into:

```js
slots.default ? slots.default({ foo: bar }) : [];
```

Parent-side:

```vue
<template #default="props">...</template>
```

compiles into a function:

```js
default: (props) => renderContentWithProps(props)
```

So:

- Slots are functions
- Slot props are function arguments
- Vue resolves slots at render time

This makes slots extremely efficient + flexible.

## 7. 🧩 Building Flexible, Reusable UIs with Slots

Slots are the core of reusable UI APIs.

### Example: A Reusable Modal Component

**Child (Modal.vue):**

```vue
<template>
  <div class="overlay">
    <div class="modal">
      <slot name="header"></slot>

      <slot></slot>
      <!-- default content -->

      <slot name="footer"></slot>
    </div>
  </div>
</template>
```

**Parent:**

```vue
<Modal>
    <template #header>
        <h2>Delete item?</h2>
    </template>

    <p>This action is irreversible.</p>

    <template #footer>
        <button>Cancel</button>
        <button>Delete</button>
    </template>
</Modal>
```

The modal becomes a reusable shell, while parents inject content.

### Example: Renderless Components

Children define logic only, no UI:

```vue
<!-- MouseTracker.vue -->
<script setup>
import { ref, onMounted, onUnmounted } from "vue";

const x = ref(0);
const y = ref(0);

const onMouseMove = (e) => {
  x.value = e.clientX;
  y.value = e.clientY;
};

onMounted(() => {
  window.addEventListener("mousemove", onMouseMove);
});

onUnmounted(() => {
  window.removeEventListener("mousemove", onMouseMove);
});
</script>

<template>
  <slot :x="x" :y="y"></slot>
</template>
```

Parent defines UI:

```vue
<MouseTracker>
    <template #default="{ x, y }">
        <p>Mouse: {{ x }}, {{ y }}</p>
    </template>
</MouseTracker>
```

Now you have a fully reusable logic provider.

## 8. 🧩 Common Gotchas

### ❌ Forgetting to use `<template>` for named slots

**Incorrect:**

```vue
<Child>
    #header
    <h1>Oops</h1>
</Child>
```

**Correct:**

```vue
<Child>
    <template #header>
        <h1>OK</h1>
    </template>
</Child>
```

### ❌ Expecting scoped slot props to work like regular props

Slot props are render-time bindings, not reactive data going down the tree.

### ❌ Putting `v-for` directly on `<slot>`

The slot itself isn't rendered; its content is.

Instead use `v-for` inside the slot definition in the parent.

### ❌ Forgetting keys in scoped slot loops

Just like normal `v-for` loops.

## 9. ⚙️ Advanced Insight / Performance Angle

### Slots are resolved at render-time, not mount-time

This means:

- Slot content reacts to parent state changes.
- Slot props react to child state changes.
- Updates are efficient because slots are just functions.

### Vue hoists static slot content

Static pieces of slot content are compiled once and reused.

### Scoped slot props connect reactive graphs across components

This allows extremely flexible renderless component patterns.

## 10. 🧠 Key Takeaways

- Slots are "render portals" where parents inject content into children.
- Default slots cover the primary content area.
- Named slots create multiple insertion points.
- Scoped slots send data from child → parent for rendering.
- Slots enable powerful reusable UI structures (cards, modals, lists).
- Renderless components emerge naturally using scoped slots.
- Internally, slots compile into functions — extremely flexible and performant.
