# 🎨 Lesson 22 — Transitions and Animations

Vue's transition system provides:

- Declarative animations
- CSS or JS-based transitions
- Lifecycle hooks for entering & leaving
- Transition groups for lists
- Automatic detection of transition types
- Built-in optimizations for smooth animations

This lesson explains:

- How transitions work internally
- How Vue controls the DOM through enter/leave hooks
- How motion interacts with reactivity
- How to optimize animation performance

## 1. 🔍 Concept Overview

A transition in Vue wraps an element or component:

```vue
<transition name="fade">
    <div v-if="visible">Hello</div>
</transition>
```

Vue handles:

- adding/removing classes
- coordinating enter/leave animations
- waiting for CSS transitions or animations to finish
- removing elements only after animation completes

You can use:

- CSS transitions
- CSS animations
- JavaScript animation callbacks
- Velocity-like libraries or Web Animations API

For lists, there's `<transition-group>`.

All transitions respect Vue's reactivity + update pipeline.

## 2. 💡 Mental Model / Analogy

🧠 **Think of Vue's transition system as a "choreographer."**

When a component appears/disappears:

1. Vue instructs the DOM: "Add this class first."
2. Then: "Now switch to this class."
3. Then: "Wait until the animation finishes."
4. Then: "Remove the element."

**Vue isn't animating — CSS is.**  
Vue is orchestrating the dance.

## 3. 🧱 Basic Transition Example

```vue
<script setup>
import { ref } from "vue";
const show = ref(false);
</script>

<template>
  <button @click="show = !show">Toggle</button>

  <transition name="fade">
    <p v-if="show">Hello!</p>
  </transition>
</template>

<style>
.fade-enter-from {
  opacity: 0;
}
.fade-enter-to {
  opacity: 1;
}
.fade-enter-active {
  transition: opacity 0.3s ease;
}

.fade-leave-from {
  opacity: 1;
}
.fade-leave-to {
  opacity: 0;
}
.fade-leave-active {
  transition: opacity 0.3s ease;
}
</style>
```

Vue automatically adds:

- `fade-enter-from`
- `fade-enter-active`
- `fade-enter-to`

Then removes them in sequence.

## 4. ⚙️ Internal Insight — Transition Class Lifecycle

Vue's transition system applies classes in the exact order needed for CSS to animate.

**For enter:**

1. `v-enter-from` added
2. One animation frame later:
   - `v-enter-from` removed
   - `v-enter-to` added
3. After transition duration ends:
   - `v-enter-to` removed

**For leave:**

1. `v-leave-from` added
2. Next frame:
   - `v-leave-from` removed
   - `v-leave-to` added
3. When animation finishes:
   - element is removed from DOM

Vue uses:

- `requestAnimationFrame`
- CSS transition end events
- Promise timeouts as fallback

This ensures extremely reliable animations.

## 5. 🧱 Transition Hooks (JS-based transitions)

You can use JS hooks instead of CSS:

```vue
<transition @before-enter="beforeEnter" @enter="enter" @leave="leave">
    <div v-if="show">Slide</div>
</transition>
```

Example JS animations:

```js
function beforeEnter(el) {
  el.style.opacity = 0;
}

function enter(el, done) {
  el.animate([{ opacity: 0 }, { opacity: 1 }], {
    duration: 300,
  }).onfinish = done;
}
```

Vue waits for `done()` before moving to the next phase.

## 6. 🧱 Transition Groups

For lists, use:

```vue
<transition-group name="list" tag="ul">
    <li v-for="item in items" :key="item.id">{{ item.text }}</li>
</transition-group>
```

Features:

- Moves elements with FLIP animations
- Supports enter/leave/move
- Requires keys for tracking

Vue animates position changes automatically using:

- `transform: translate(...)`
- GPU acceleration
- `requestAnimationFrame`

This enables:

- sortable lists
- draggable interfaces
- reordering animations

## 7. 🧬 Enter/Leave Modes

### `mode="out-in"`

Wait for leave to finish before enter:

```vue
<transition mode="out-in">
    <component :is="view" />
</transition>
```

Useful for:

- page transitions
- replacing dynamic components cleanly

### `mode="in-out"`

Enter first → then leave.

Useful when preparing the next view ahead of removing the old one.

## 8. ⚙️ How Transitions Fit into the Render Pipeline

When a node is wrapped in `<transition>`, Vue:

1. Detects `v-if` or `v-show` changes
2. Blocks unmounting
3. Applies leave classes
4. Waits for transition events
5. Removes node when finished
6. Applies enter classes to the new node
7. Resumes normal patching

Vue's patch algorithm cooperates with the transition component to delay unmounts or delay mounts.

This is handled through the `Transition` vnode type in the core renderer.

## 9. 🎯 Animation Performance Best Practices

### ✔ 1. Prefer CSS transforms over layout properties

**Use:**

- `transform: translate3d(...)`
- `opacity`

**Avoid:**

- `top`, `left`, `height`, `width`

These cause layout reflow.

### ✔ 2. Use GPU acceleration whenever possible

Adding:

```css
transform: translateZ(0);
```

Forces GPU acceleration — smoother animations.

### ✔ 3. Keep animated elements out of "heavy" containers

Animating inside a parent that triggers layout (e.g., big grids) can cause jank.

### ✔ 4. Use `will-change`

```css
.element {
  will-change: transform, opacity;
}
```

Tells the browser to optimize ahead of time.

### ✔ 5. Avoid animating too many nodes

Even GPU-backed transitions can lag if many elements change at once.

### ✔ 6. Avoid causing component re-renders during transitions

Example of a bad idea:

```vue
<div>{{ expensiveReactiveProperty }}</div>
```

Updating during transitions may cause:

- jank
- skipped frames
- delayed transitions

Use `v-memo` or separate reactive islands if needed.

## 10. 🧩 Common Gotchas

❌ **Forgetting key in transition groups**  
Without keys, Vue cannot animate moves.

❌ **Using `display: none` with transitions**  
Use `v-show` instead of `display:none` if you need transitions.

❌ **Not matching enter/leave class names**  
A typo in CSS class name → animations silently fail.

❌ **Animating layout properties**  
Triggers full layout recalculation.

❌ **hydrating SSR transitions incorrectly**  
Initial transitions should be disabled in SSR-hydrated apps (Nuxt does this automatically).

## 11. 🧠 Key Takeaways

- Vue transitions **orchestrate** CSS/JS animations, not perform them.
- Vue coordinates class application precisely for enter/leave animations.
- `<transition-group>` supports move transitions and FLIP (First, Last, Invert, Play) animations.
- Modes `out-in` and `in-out` provide timing control.
- Vue's renderer cooperates with transitions by delaying mount/unmount.
- Performance hinges on using GPU-accelerated properties and minimizing reflows.
- Smooth animations depend on stable reactive boundaries.
