# 🌐 Lesson 21 — SSR and Hydration (Nuxt Concepts)

Vue supports SSR (Server-Side Rendering) through its built-in SSR API and through Nuxt, the official meta-framework.

SSR involves:

- Rendering the app on the server
- Sending HTML to the client
- Reusing that HTML and attaching Vue's runtime → hydration
- Supporting dynamic data via async fetching
- Enabling progressive/streaming rendering for speed

If you truly understand this pipeline, Nuxt architecture becomes intuitive.

## 1. 🔍 Concept Overview

SSR solves problems like:

- ⚡ Faster first paint (HTML arrives ready to display)
- 🧠 SEO-friendliness (search bots get real content)
- 📶 Better perceived performance on slow networks

**SSR Flow:**

1. Server runs the Vue app
2. Produces HTML string
3. Sends HTML to browser
4. Browser displays HTML immediately
5. Vue runtime loads, parses, and hydrates the HTML into active components

**Hydration binds:**

- event listeners
- reactive state
- component instances

…onto the existing DOM instead of replacing it.

## 2. 💡 Mental Model / Analogy

🧠 **Think of SSR as "assembling a pre-built Lego model" on the server…**

- The server sends a finished Lego structure.
- On the client, Vue:
  - Walks through each Lego piece
  - Adds the "glue" and "interactive bits"
  - Turns the static model into a dynamic one

**Hydration says:**

> "Here's the HTML you already have — I'll attach the reactivity and event listeners so it becomes alive."

**Streaming SSR** is like sending the Lego model in chunks, and assembling it as it arrives.

## 3. 🧱 SSR Fundamentals

### Server-side:

```js
import { renderToString } from "vue/server-renderer";
import { createApp } from "./app.js";

const app = createApp();
const html = await renderToString(app);
```

This:

- Runs your component tree
- Executes reactive logic (without DOM)
- Produces HTML

### Client-side:

```js
import { createApp } from "./app.js";
const app = createApp();
app.mount("#app"); // triggers hydration
```

**Note:** `mount()` performs hydration if HTML already exists.

## 4. 🧱 Hydration — Making Static HTML Interactive

During hydration:

- Vue walks the DOM
- Matches it against the Virtual DOM tree
- Converts DOM nodes into managed nodes
- Attaches event listeners
- Activates reactive bindings
- Reconstructs component boundaries

**Hydration must match exactly.**

If server HTML and client VDOM differ:

- Vue logs a mismatch and forces full client-side replacement.

**Common causes:**

- ❌ Using browser-only APIs during SSR
- ❌ Global state that differs between client and server
- ❌ Date/time values
- ❌ Random numbers
- ❌ Missing keys in `v-for`

## 5. ⚙️ Async Data and SSR

SSR needs data before HTML is rendered.

Nuxt solves this with **`asyncData()`**

Runs server-side before rendering:

```js
export default {
  async asyncData({ $api }) {
    const user = await $api.user.me();
    return { user };
  },
};
```

The returned values become props/data for the component.

**On the client:**

- Nuxt serializes initial data into HTML as JSON
- Client resumes with preloaded state
- Avoids double-fetching

This is **data hydration**.

## 6. ⚙️ Streaming SSR (Vue 3 + Nuxt 3)

Streaming SSR sends HTML in chunks as it's generated.

**Use case:**

- Dynamic content that requires API calls
- Slow network conditions
- Large page content

**How it works:**

1. Server starts rendering
2. When it hits async boundaries, it yields partial HTML
3. Browser starts painting immediately
4. Remaining components stream as they resolve

This improves **First Contentful Paint** and perceived loading speed.

Nuxt supports this natively via:

- `renderToNodeStream()` (Node)
- `renderToWebStream()` (Edge/Workers)

## 7. 🧱 Nuxt's SSR Architecture

Nuxt 3 introduces a powerful architecture:

### ✔ Universal app entry points

```
app.vue
pages/
layouts/
plugins/
server/api/
```

Nuxt generates:

- server entry for SSR
- client entry for hydration

### ✔ File-based routing

Pages automatically become routes.

### ✔ Auto async data handling

`useAsyncData`, `useFetch`, and server routes integrate with SSR.

**Example:**

```js
const { data } = await useFetch("/api/products");
```

**On the server:**

- Fetch executes directly
- Data is injected into rendered HTML

**On the client:**

- Data is already available
- No refetch unless configured

### ✔ Hybrid rendering

Nuxt can do:

- Full SSR
- Partial SSR
- SPA fallback
- Static site generation (SSG)
- Edge-rendered SSR
- Island architecture (coming soon)

## 8. 🧬 How Hydration Actually Works (Deep Dive)

Hydration is a matching process:

**Server sends HTML:**

```html
<div><h1>Hello</h1></div>
```

**Client generates VNode:**

```
div
└─ h1 -> "Hello"
```

**Vue walks DOM nodes:**

1. DOM node `<div>` matches VNode `<div>`
2. DOM node `<h1>` matches VNode `<h1>`
3. Text node "Hello" matches

**Vue attaches reactive effects:**

- Props → reactive getters
- Event listeners → DOM events
- Scoped styles → transforms
- Suspense boundaries → restored

Now updates can run normally.

**Vue does not recreate DOM — only augments it.**

## 9. 🧩 Common Gotchas

### ❌ Mismatched HTML breaks hydration

```html
<!-- server -->
<p>Count: 0</p>

<!-- client initial state -->
<p>Count: 1</p>
```

**Fix:** Always sync initial state.

### ❌ Using browser-only APIs during SSR

Such as:

- `window`, `document`
- `localStorage`
- `getComputedStyle`
- Canvas APIs

**Fix:** Wrap in `if (process.client)` (Nuxt) or `onMounted()`.

### ❌ Using randomized IDs on server but not client

Avoid random behavior in SSR unless seeded consistently.

### ❌ Event listeners not firing

Hydration not completing properly → mismatch.

### ❌ Async data fetched twice

Nuxt 3 fixes this with data hydration, but custom SSR apps must serialize state manually.

## 10. 🧠 Key Takeaways

- SSR renders Vue components as HTML on the server → fast first paint.
- Hydration activates the HTML on the client by binding reactivity.
- HTML from server and VDOM from client must match 1:1.
- Streaming SSR improves perceived performance by sending partial HTML.
- Nuxt handles async data loading automatically with `useAsyncData` & serialization.
- Hydration is not rendering — it's attaching behaviors to existing DOM.
- Proper SSR requires avoiding non-deterministic or browser-only logic during server phase.
