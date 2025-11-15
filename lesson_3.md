# 🎨 Lesson 3 — Template Syntax and Directives

Vue's template syntax is a declarative layer that allows you to express what the UI should look like based on your component state. Under the hood, templates compile to highly optimized JavaScript render functions.

This lesson builds the foundation for understanding how Vue maps state → DOM through declarative markup.

## 1. 🔍 Concept Overview

Vue's template system provides four major categories of directives:

- **Data Interpolation** → `{{ count }}`
- **Attribute Binding** → `v-bind` or the `:` shorthand
- **Event Handling** → `v-on` or the `@` shorthand
- **Structural Directives** → `v-if`, `v-for`, etc.
- **Two-way Binding** → `v-model`

These directives let you express dynamic behavior without manually manipulating the DOM.

## 2. 💡 Mental Model / Analogy

🧠 **Think of Vue templates like "reactive blueprints."**

You sketch "where things go" and "how they behave":

- `{{ message }}` → "Put the message text here"
- `:class="active"` → "Apply this class if the state says so"
- `@click="toggle"` → "When clicked, run this logic"
- `v-if="loggedIn"` → "Only show this part of the blueprint if a condition holds"
- `v-for="item in list"` → "Stamp this blueprint for each item"

Vue ensures that when the underlying data changes, the blueprint is re-applied in the most efficient way.

## 3. 🧱 Code Example — All Core Directives in One Small Component

```vue
<script setup>
import { ref } from "vue";

const message = ref("Hello Vue!");
const isVisible = ref(true);
const items = ref(["A", "B", "C"]);
const inputValue = ref("");
</script>

<template>
  <!-- 1. Interpolation -->
  <h1>{{ message }}</h1>

  <!-- 2. Attribute binding -->
  <p :title="`The message is: ${message}`">Hover to see title</p>

  <!-- 3. Event handling -->
  <button @click="isVisible = !isVisible">Toggle Visibility</button>

  <!-- 4. Conditional rendering -->
  <p v-if="isVisible">Now you see me 👀</p>
  <p v-show="isVisible">I'm always in the DOM, just toggled by CSS.</p>

  <!-- 5. List rendering -->
  <ul>
    <li v-for="(item, index) in items" :key="index">
      {{ index }} — {{ item }}
    </li>
  </ul>

  <!-- 6. Two-way binding -->
  <input v-model="inputValue" placeholder="Type here..." />
  <p>You typed: {{ inputValue }}</p>
</template>
```

## 4. ⚙️ Internal Insight — What Vue Is Doing Under the Hood

Let's break it down directive by directive.

### 🔹 1. Interpolation — `{{ message }}`

- Compiles to something like:

  ```js
  children: [_toDisplayString(message.value)];
  ```

- Vue tracks `message.value` as a dependency.
- When `message` changes → text node is efficiently updated.

### 🔹 2. Attribute Binding — `:title="expression"`

- Compiles to a dynamic prop:

  ```js
  title: "The message is: " + message.value;
  ```

- Vue updates only that DOM attribute when dependencies change.
- No full re-render.

### 🔹 3. Event Handling — `@click="fn"`

- Compiles to:

  ```js
  onClick: fn;
  ```

- Event listeners attach to the DOM normally.
- No reactivity required here — logic runs in response to user interaction.

### 🔹 4. Conditional Rendering

**`v-if`**

- Branch-based rendering.
- DOM nodes are created/destroyed depending on the condition.
- More expensive but fully removes elements.

**`v-show`**

- Inserts the element once, then toggles `display: none`.
- Cheaper toggle, but DOM remains in place.

Vue chooses the most efficient diffing behavior based on the directive used.

### 🔹 5. List Rendering — v-for

```vue
<li v-for="item in items" :key="item.id"></li>
```

- Compiles to a loop that generates a list of virtual DOM nodes.
- `key` guides Vue's diff algorithm:
  - avoids unnecessary DOM moves
  - preserves component instance identity
  - prevents bugs when editing data

### 🔹 6. Two-Way Binding — `v-model`

On an `<input>`, Vue generates:

- A value binding (`:value="inputValue"`)
- An event listener (`@input="inputValue = $event.target.value"`)

In other words:

```vue
<input v-model="inputValue" />
```

is syntactic sugar for manually syncing input value → state and state → input value.

## 5. 🧩 Common Pitfalls / Misconceptions

❌ **"`v-if` and `v-show` are interchangeable"**

- `v-if` = conditional rendering (add/remove DOM)
- `v-show` = CSS toggle (fast but always in DOM)

❌ **Forgetting to add a key on `v-for`**

Leads to:

- Incorrect DOM reuse
- Wrong component state
- Rendering anomalies

❌ **"v-model works on any element"**

- No — it works on form inputs by default.
- Custom components require defining `modelValue` + `@update:modelValue`.

❌ **Using index as a key**

- Fine for static lists.
- Bad for re-orderable lists → causes DOM confusion.

## 6. ⚙️ Advanced Insight / Performance Angle

**Vue Tags Dynamic Nodes and Skips Static Ones**

Vue's compiler analyzes the template:

- **Static nodes** → rendered once, skipped on updates
- **Dynamic nodes** → patched only when dependencies change
- **v-if branches** → compiled into lazy conditional blocks
- **v-for** → optimized diffing with keys

This is why Vue templates are both expressive and fast.

## 7. 🧠 Key Takeaways

- Templates are reactive blueprints for rendering UI.
- Interpolation, bindings, and events connect state to the DOM declaratively.
- `v-if` removes DOM; `v-show` hides DOM.
- `v-for` requires a `key` to help Vue track identity.
- `v-model` synchronizes input state both ways.
- Vue compiles templates into highly optimized render functions.
