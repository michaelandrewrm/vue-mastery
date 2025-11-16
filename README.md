# 🧠 Vue.js Mastery Curriculum — From Fundamentals to Expert

## 📘 Level 1: Fundamentals of Vue Thinking

Goal: Build a solid foundation in Vue’s declarative model, template syntax, and component-driven architecture.

### [Lesson 1: What Is Vue.js?](lesson_1.md)

- Declarative vs. imperative UI
- “Reactive data + declarative templates = dynamic UI”
- The Vue instance and the concept of reactivity
- How Vue efficiently updates the DOM

### [Lesson 2: Creating Your First Component](lesson_2.md)

- The role of `<script setup>` and `<template>`
- Component registration and organization
- Using reactive data and methods
- Rendering logic with data bindings ({{ }})

### [Lesson 3: Template Syntax and Directives](lesson_3.md)

- Interpolation, attribute binding (`v-bind`), event handling (`v-on`)
- Conditional rendering (`v-if`, `v-show`)
- List rendering (`v-for`) and the importance of key
- Two-way data binding with v-model

### [Lesson 4: Props, Emits, and Component Communication](lesson_4.md)

- Passing data down via props
- Emitting custom events upward
- Defining prop types and default values
- The one-way data flow mental model

### [Lesson 5: Lifecycle Hooks and Component Mounting](lesson_5.md)

- The component lifecycle in Vue 3
- Common hooks: `onMounted`, `onUpdated`, `onUnmounted`
- When to perform side effects and cleanup

## ⚡ Level 2: Core Vue Concepts

Goal: Develop deeper understanding of Vue’s reactivity, computed state, and composition patterns.

### [Lesson 6: Reactivity Fundamentals](lesson_6.md)

- `ref()` vs. `reactive()`
- Dependency tracking and reactivity graph mental model
- Proxy-based reactivity explained
- Common gotchas: destructuring reactive state

### [Lesson 7: Computed and Watchers](lesson_7.md)

- The difference between computed and watch
- Lazy evaluation in computed properties
- Side effects vs. derived state
- Example: reactive form validation

### [Lesson 8: Composition API Basics](lesson_8.md)

- Why the Composition API exists
- The role of `setup()` and script setup
- Organizing logic with composables
- Using reactive primitives together

### [Lesson 9: Slots and Component Composition](lesson_9.md)

- The concept of slots as render portals
- Named and scoped slots
- Building flexible, reusable UIs with slots

### [Lesson 10: Lifecycle, Effects, and Cleanup](lesson_10.md)

- Understanding Vue’s reactivity loop
- Using lifecycle hooks (`onMounted`, `onUnmounted`)
- Properly handling side effects
- Avoiding memory leaks with cleanups

## 🧩 Level 3: Advanced Vue Patterns & Architecture

Goal: Master composition, abstraction, and patterns for scalable and reusable component systems.

### [Lesson 11: Custom Composables](lesson_11.md)

- Extracting logic into composable functions
- Using composables for state and effects
- Dependency injection patterns (provide/inject)
- Organizing composables for large projects

### [Lesson 12: State Management — Pinia and Vuex 4](lesson_12.md)

- When to use global state
- Pinia’s API and store design
- Computed getters, actions, and persistence
- Migrating from Vuex to Pinia mental model

### [Lesson 13: Asynchronous Behavior and Suspense](lesson_13.md)

- Handling async data loading
- Vue’s built-in `<Suspense>` component
- Async components and fallbacks
- Integrating composables with async logic

### [Lesson 14: Dynamic and Recursive Components](lesson_14.md)

- component and dynamic component rendering
- Recursive components and tree structures
- Managing conditional and dynamic UI hierarchies

### [Lesson 15: Routing with Vue Router](lesson_15.md)

- Setting up routes and nested views
- Navigation guards and route parameters
- Lazy loading and code splitting routes
- Route transitions and dynamic meta updates

## ⚙️ Level 4: Vue Internals & Performance Engineering

Goal: Understand how Vue works internally — from its reactivity system to template compilation and virtual DOM.

### [Lesson 16: The Virtual DOM and Rendering Process](lesson_16.md)

- What the Virtual DOM is and why Vue uses it
- The render pipeline: render → diff → patch
- Optimized diffing and static hoisting

### [Lesson 17: Inside Vue’s Reactivity System](lesson_17.md)

- Dependency collection and effect tracking
- The effect() and track() / trigger() cycle
- Deep dive: how Vue’s reactivity uses ES6 Proxies
- The dependency graph mental model

### [Lesson 18: Scheduler and Next Tick](./lesson_18.md)

- The async update queue
- `nextTick()` and microtask scheduling
- How Vue batches updates for performance
- Visualizing reactivity updates step-by-step

### Lesson 19: Template Compilation

- From template to render function
- Vue’s compiler-inferred optimizations
- Static tree hoisting and patch flags
- Custom render functions (`h()`)

### Lesson 20: Performance Optimization Techniques

- Measuring component performance
- `v-memo`, `v-once`, and caching strategies
- Optimizing reactivity granularity
- Avoiding over-rendering and reactive explosions

## 🚀 Level 5: Expert Vue Patterns & Future Architecture

Goal: Reach expert fluency in building, scaling, and reasoning about advanced Vue systems.

### Lesson 21: SSR and Hydration (Nuxt Concepts)

- Server-Side Rendering (SSR) fundamentals
- Hydration and client activation
- Streaming SSR and async data hydration
- Using Nuxt for SSR architecture

### Lesson 22: Transitions and Animations

- Transition components and lifecycle hooks
- Enter/leave transitions and groups
- Animation performance best practices

### Lesson 23: Large-Scale Architecture & Code Organization

- Colocation and modular design
- Feature-based file structuring
- Managing complexity with composables and stores

### Lesson 24: Vue + TypeScript for Scalability

- Typing props, emits, and refs
- Using defineProps and defineEmits with TS
- Creating strongly-typed composables

### Lesson 25: Debugging and Reasoning from First Principles

- Vue DevTools deep dive
- Tracing reactivity updates and watcher triggers
- Debugging performance and render issues
- Developing “Vue intuition” — predicting behavior before testing
