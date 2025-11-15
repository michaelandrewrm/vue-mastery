# 🧭 Lesson 15 — Routing with Vue Router

Routing transforms your Vue SPA into a multi-page application by mapping URLs → components.

Vue Router manages:

- Navigation state
- Nested and dynamic routes
- Guards & async validation
- Lazy-loading
- Transitions between views
- Metadata updates (titles, SEO hints, auth markers)

Let's build a deep understanding of it.

## 1. 🔍 Concept Overview

Vue Router enables:

### ✔ Single-Page App navigation

Switch between components without reloading the page.

### ✔ URL → Component mapping

The URL always reflects app state.

### ✔ Nested & dynamic routes

Great for dashboards, admin UIs, user profiles.

### ✔ Route guards

Control navigation (authentication, validation).

### ✔ Lazy-loaded route components

Improves performance with code splitting.

### ✔ Transitions between views

Smooth page animations.

Vue Router is the official routing solution for Vue 3.

## 2. 💡 Mental Model / Analogy

🧠 **Vue Router is like a "traffic control tower" for your app.**

- Every URL is a flight path.
- Routes tell Vue Router which plane (component) to land.
- Guards act like passport control.
- Nested routes act like terminal gates.
- Lazy loading is calling a plane only when needed.
- Route transitions are animations as people enter/exit gates.

Vue Router orchestrates the movement through your SPA.

## 3. 🧱 Setting Up Vue Router

**router/index.js**

```js
import { createRouter, createWebHistory } from "vue-router";
import Home from "@/views/Home.vue";
import About from "@/views/About.vue";

const routes = [
  { path: "/", name: "home", component: Home },
  { path: "/about", name: "about", component: About },
];

export const router = createRouter({
  history: createWebHistory(),
  routes,
});
```

**main.js**

```js
import { createApp } from "vue";
import App from "./App.vue";
import { router } from "./router";

createApp(App).use(router).mount("#app");
```

**Using the router in templates**

```vue
<router-link to="/about">About</router-link>
<router-view />
```

- `<router-link>` handles navigation
- `<router-view>` is where matched components render

## 4. 🧱 Nested Routes (Layout + Subviews)

Nested routes let you build layered UIs.

**router/index.js**

```js
const routes = [
  {
    path: "/dashboard",
    component: () => import("@/views/Dashboard.vue"),
    children: [
      { path: "stats", component: () => import("@/views/Stats.vue") },
      { path: "users", component: () => import("@/views/Users.vue") },
    ],
  },
];
```

**Dashboard.vue**

```vue
<template>
  <h1>Dashboard</h1>
  <router-view />
  <!-- renders Stats or Users -->
</template>
```

**URLs**

- `/dashboard/stats`
- `/dashboard/users`

**Mental model:**
`<router-view>` is a portal where child routes appear.

## 5. 🧱 Dynamic Route Parameters

**Route definition**

```js
{ path: '/user/:id', component: User }
```

**Accessing params inside setup:**

```vue
<script setup>
import { useRoute } from "vue-router";

const route = useRoute();
console.log(route.params.id);
</script>
```

**Use case examples:**

- `/product/42`
- `/blog/2024/vue-internals`

Dynamic params are reactive — changing the URL updates the component.

## 6. ⚙️ Navigation Guards

Guards run before or after navigation to validate or block the route.

**Global guard: Authentication example**

```js
router.beforeEach((to, from, next) => {
  if (to.meta.requiresAuth && !isLoggedIn()) {
    next("/login");
  } else {
    next();
  }
});
```

**Per-route guard**

```js
{
    path: '/admin',
    component: Admin,
    beforeEnter: (to, from) => {
        if (!isAdmin()) return '/not-authorized'
    }
}
```

**In-component guard (Composition API)**

```js
import { onBeforeRouteLeave } from "vue-router";

onBeforeRouteLeave((to, from) => {
  const confirmLeave = window.confirm("Leave the page?");
  return confirmLeave;
});
```

**Mental model:**
Guards are like security checkpoints that decide:

- allow
- redirect
- cancel navigation
- wait for async ops (return promises)

## 7. 🧱 Lazy Loading & Code Splitting

Large apps should load components only when needed.

```js
const Product = () => import("@/views/Product.vue");

const routes = [{ path: "/product/:id", component: Product }];
```

**Benefits:**

- Faster initial load time
- Smaller bundles
- On-demand loading

Vue Router integrates seamlessly with dynamic imports.

## 8. 🧱 Route Transitions

Wrap `<router-view>` in a transition:

```vue
<template>
  <transition name="fade" mode="out-in">
    <router-view />
  </transition>
</template>
```

**CSS**

```css
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
```

Transitions apply whenever the active route changes.

## 9. 🧱 Meta Fields & Dynamic Document Title

Routes can contain metadata:

```js
{
    path: '/profile',
    component: Profile,
    meta: { requiresAuth: true, title: 'Your Profile' }
}
```

Use a global after hook:

```js
router.afterEach((to) => {
  document.title = to.meta.title || "My App";
});
```

**Use cases:**

- Page titles
- Permissions
- Breadcrumbs
- Feature flags

## 10. ⚙️ Internal Insight — How Vue Router Works

Vue Router is built on top of:

**Reactive route state**

`useRoute()` returns a reactive object:

- params
- query
- matched routes
- fullPath

**When you navigate:**

1. Vue Router matches the new path
2. Runs guards & async loaders
3. Updates the reactive route object
4. Triggers component swaps in `<router-view>`
5. Vue re-renders affected components

`<router-view>` is essentially:

```js
resolveMatchedComponent(route);
render(component);
```

## 11. 🧩 Common Gotchas

### ❌ Forgetting to use :key when rendering dynamic routes

```vue
<router-view :key="$route.fullPath" />
```

Ensures fresh renders for param-based pages.

### ❌ Confusing nested routes with named views

Use named views when you want parallel multiple router-views, not nested ones.

### ❌ Missing trailing slash on nested paths

```js
children: [{ path: "settings" }];
```

NOT `'/settings'` — that breaks nesting.

### ❌ Using query params instead of route params for IDs

IDs belong in route params.

### ❌ Running expensive logic repeatedly in route watchers

Use `watch(() => route.params.id)` sparingly and debounce if needed.

## 12. 🧠 Key Takeaways

- Vue Router maps URLs → components with nested, dynamic, and async behaviors.
- `<router-view>` is a dynamic portal for matched components.
- Route params are fully reactive.
- Navigation guards manage authentication & navigation rules.
- Lazy loading improves performance with code splitting.
- Route transitions enable smooth page animations.
- Route meta fields allow dynamic titles, permissions, and more.
