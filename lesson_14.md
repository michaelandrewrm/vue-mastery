# 🌳 Lesson 14 — Dynamic & Recursive Components

Vue's component system is not limited to static, predefined component structures. With dynamic components and recursive rendering, you can build extremely flexible and composable UIs.

**We'll cover:**

- Rendering components dynamically
- Using the `<component>` tag
- Building recursive components
- Handling dynamic, conditional hierarchies

## 1. 🔍 Concept Overview

### Dynamic Components

Let you render different components dynamically based on state.

Vue provides:

```vue
<component :is="currentComponent" />
```

Where `currentComponent` can be:

- A component definition
- A registered component name
- A Promise (async component)

### Recursive Components

A component that renders itself inside its own template — especially useful for:

- Tree data
- Comments with nested replies
- Category hierarchies
- Graphs / organizational charts

Recursive rendering is fully supported in Vue 3.

## 2. 💡 Mental Model / Analogy

### 🧠 Think of dynamic components like "shape-shifting containers."

The `<component :is="">` tag is an empty vessel.

You tell Vue:

- "Right now, you're a Button."
- "Now you're a Card."
- "Now you're a Modal."

Vue swaps the internal component while preserving state (if key is used correctly).

### 🧠 Recursive components are like "matryoshka dolls."

A recursive component is:

- A component that contains a smaller version of itself
- Which contains another…
- And so on

Each instance gets:

- Its own props
- Its own state
- Its own place in the reactivity graph

This makes tree-like UI structures elegant.

## 3. 🧱 Code Example — Dynamic Components

Let's dynamically switch between components:

```vue
<script setup>
import ButtonPrimary from "./ButtonPrimary.vue";
import ButtonSecondary from "./ButtonSecondary.vue";

const current = ref("primary");

const map = {
  primary: ButtonPrimary,
  secondary: ButtonSecondary,
};
</script>

<template>
  <select v-model="current">
    <option value="primary">Primary</option>
    <option value="secondary">Secondary</option>
  </select>

  <component :is="map[current]" />
</template>
```

**What happens:**

- Changing `current` changes the rendered component
- Vue efficiently swaps the internal component instance

## 4. ⚙️ Internal Insight — How Dynamic Components Work

`<component :is="X" />` compiles to:

```js
resolveDynamicComponent(X);
```

Vue then:

1. Resolves `X` to a component definition
2. Creates or updates the component instance
3. Patches it into the DOM tree
4. Tears down old instances if needed

**Component identity is managed via keys**

Use `:key` to control whether Vue:

- Preserves component state
- Rebuilds it from scratch

For example:

```vue
<component :is="view" :key="view" />
```

forces fresh mounts when switching.

## 5. 🧱 Recursive Components — Foundation for Tree UIs

**RecursiveTree.vue**

```vue
<script setup>
const props = defineProps({
  node: {
    type: Object,
    required: true,
  },
});
</script>

<template>
  <li>
    {{ node.label }}

    <ul v-if="node.children">
      <RecursiveTree
        v-for="child in node.children"
        :key="child.id"
        :node="child"
      />
    </ul>
  </li>
</template>
```

**Usage**

```vue
<RecursiveTree :node="rootNode" />
```

Each component recursively renders its children.

Vue ensures each instance is:

- Independent
- Fully reactive
- Efficiently updated

## 6. ⚙️ Internal Insight — How Vue Handles Recursion

Unlike template languages that forbid recursion, Vue's component model is truly recursive.

**Here's how Vue handles it:**

1. Each recursive invocation generates a new component instance.
2. Vue avoids infinite recursion by requiring:
   - conditional checks (`v-if`, children length)
   - data structure with termination points
3. The virtual DOM handles recursion naturally as a tree of nodes.

Every time recursion occurs:

```
RecursiveTree
├── RecursiveTree
│   ├── RecursiveTree
│   └── ...
└── RecursiveTree
```

This matches how tree data structures are naturally traversed.

## 7. 🧱 Managing Conditional & Dynamic UI Hierarchies

Arbitrary, nested, dynamic UIs often combine all these patterns.

**Example: A file explorer.**

```vue
<FileNode v-for="file in tree" :key="file.path" :node="file" :depth="0" />
```

**FileNode.vue**

```vue
<script setup>
import { ref } from "vue";

const props = defineProps({
  node: Object,
  depth: Number,
});

const open = ref(false);
const toggle = () => (open.value = !open.value);
</script>

<template>
  <div :style="{ marginLeft: depth * 10 + 'px' }">
    <span @click="toggle">
      {{ node.children ? (open ? "📂" : "📁") : "📄" }}
      {{ node.name }}
    </span>
  </div>

  <div v-if="open && node.children">
    <FileNode
      v-for="child in node.children"
      :key="child.path"
      :node="child"
      :depth="depth + 1"
    />
  </div>
</template>
```

This creates:

- Dynamic depth
- Recursive UI
- Expand/collapse logic
- Dynamic hierarchical rendering

## 8. 🧩 Common Gotchas

### ❌ Forgetting component self-registration for recursion

Vue 3 SFCs auto-register themselves by filename only in `<script setup>`.

Otherwise you must:

```js
components: {
  RecursiveTree;
}
```

### ❌ Infinite recursion

Usually caused by:

- cycles in data
- missing base case
- always rendering children without check

### ❌ Reusing keys incorrectly

Keys in recursive structures must uniquely represent the node.

### ❌ Using `<component :is="">` without keys

This may lead to unintended state preservation. Always use keys when switching component types.

### ❌ Trying to force recursion without required props

Each recursive call must supply full props for the child instance.

## 9. ⚙️ Advanced Insight — Vue's Renderer Handles Nested Structures Efficiently

Vue's runtime:

- Treats recursive trees like nested component subtrees
- Uses highly optimized diff algorithms
- Patches only changed branches of the UI
- Never re-renders the entire tree unless required

For large trees (filesystems, menus, nested comments):

- Vue's fine-grained reactivity optimizes updates at the property level
- Only the updated branch's nodes re-render

This is why Vue excels at hierarchical UIs.

## 10. 🧠 Key Takeaways

- `<component :is="">` dynamically switches components at runtime.
- Keys control component lifecycle (preserve vs rebuild).
- Recursive components allow elegant tree structure rendering.
- Each recursive call creates an independent component instance.
- Vue efficiently updates only the changed parts of nested UIs.
- Dynamic & recursive patterns power file explorers, menus, comments, and more.
