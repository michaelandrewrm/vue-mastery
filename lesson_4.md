# 🎯 Lesson 4 — Props, Emits, and Component Communication

Vue's component system is designed around one-way data flow:

- **Parent → child:** data flows down via props
- **Child → parent:** signals flow up via emits

This creates predictable, maintainable component hierarchies.

## 1. 🔍 Concept Overview

### Props

Props allow a parent component to pass data down to a child.

### Emits

Events allow a child component to send signals up to a parent.

### Why both?

- Props handle state flow downward.
- Emits handle behavior flow upward.

Vue enforces this to prevent chaotic data mutation and tangled dependencies.

## 2. 💡 Mental Model / Analogy

### 📦 Props = "packages delivered to the component"

- The parent gives the child a box.
- The child can open it (read it), but should NOT replace it.

### 📢 Emits = "the child ringing a doorbell"

When something happens inside the child, it doesn't directly change the parent's state. Instead it emits a message:

> "Hey parent, something happened — you decide what to do with this."

This separation keeps the data flow clear and predictable. No surprises. No side effects.

## 3. 🧱 Code Example — Props + Emits in Action

### Parent Component

```vue
<!-- Parent.vue -->
<script setup>
import Child from "./Child.vue";
import { ref } from "vue";

const count = ref(0);

function handleIncrement() {
  count.value++;
}
</script>

<template>
  <h2>Parent count: {{ count }}</h2>

  <Child :value="count" @increment="handleIncrement" />
</template>
```

### Child Component

```vue
<!-- Child.vue -->
<script setup>
const props = defineProps({
  value: {
    type: Number,
    required: true,
  },
});

const emit = defineEmits(["increment"]);

function notifyParent() {
  emit("increment");
}
</script>

<template>
  <button @click="notifyParent">Child shows: {{ props.value }}</button>
</template>
```

### What happens?

1. Parent passes `count` down as a prop.
2. Child displays it.
3. Child emits `increment` when clicked.
4. Parent hears the event and updates `count`.
5. Vue re-renders parent and propagates new props to child.

## 4. ⚙️ Internal Insight — What Vue Actually Does

### Props

- Props are made **read-only reactive bindings** inside the child.
- Child accessing `props.value` tracks that property.
- When the parent updates `count`, Vue pushes that update down.
- The child re-renders only if needed.

### Emits

- Child component instances have an internal event emitter.
- `emit('increment')` does:
  - call the parent's listener (`@increment="..."`)
  - pass any arguments you include
- Parent-side listeners compile into function props:
  ```js
  onIncrement: handleIncrement;
  ```
- No magic — just function passing.

### One-way data flow guarantees

Vue prevents:

```js
props.value = 10; // ❌ warning
```

**Why?** Because props come from the parent. The child mutating them breaks the mental model of predictability.

## 5. 🧩 Props: Types, Defaults, and Validation

### Basic types

```js
defineProps({
  title: String,
  count: Number,
  isActive: Boolean,
});
```

### Detailed prop definitions

```js
defineProps({
  user: {
    type: Object,
    required: true,
  },
  theme: {
    type: String,
    default: "light",
  },
  onChange: {
    type: Function,
  },
});
```

Vue validates these in development mode only.

### Why define types?

- Documentation
- Validation
- Better DX
- Catching mistakes early

## 6. 🧩 Common Pitfalls / Misconceptions

### ❌ Mutating a prop inside the child

Vue will warn:

```
[Vue warn]: Attempting to mutate prop 'value'
```

**Solution:** use a local copy:

```js
const local = ref(props.value);
```

### ❌ Forgetting to define emits

Vue validates emits declarations the same way it validates props.

### ❌ Not using the key when passing lists of props

This causes wrong component instance reuse.

### ❌ Assuming emits update parent state automatically

No — emits only notify; the parent must handle the update.

## 7. ⚙️ Advanced Insight / Performance Angle

### Props establish unidirectional reactive bindings

This means:

- **Parent updates → child re-renders**
- **Child updates → parent does NOT re-render** (unless parent listens to emitted events)

This prevents accidental cascading re-renders.

### Emits are fire-and-forget

- The child doesn't care what the parent does with the event.
- This separation reduces coupling and keeps components composable.

### Static props are optimized

If a prop is constant (e.g., `:label="'OK'"`), Vue knows it never changes:

- No tracking
- No reactivity
- No computation cost

Vue's compiler is very smart about this.

## 8. 🧠 Key Takeaways

- Props carry **data downward**; emits send **messages upward**.
- Props are **read-only** inside the child.
- Emits trigger parent-defined behavior but do not mutate data themselves.
- One-way data flow makes state reasoning **predictable and debuggable**.
- Prop definitions (types, defaults) improve reliability.
- Vue optimizes prop and event handling at compile time.
