# 🎓 Lesson 27 — Advanced Suspense Internals: Multi-Boundaries, Parallel Async, and SSR

Suspense is a powerful feature that controls async component rendering and error handling.

This lesson goes beyond the basics.

## 1. 🔍 Concept Overview

**Suspense:**

- Coordinates async dependencies inside components
- Provides fallback content (loading UI)
- Supports nested + parallel async components
- Works with SSR boundaries
- Integrates with Vue's async scheduler

**General structure:**

```vue
<Suspense>
    <template #default>
        <AsyncComponent />
    </template>

    <template #fallback>
        <Spinner />
    </template>
</Suspense>
```

## 2. 💡 Mental Model / Analogy

🧠 **Think of Suspense as a "traffic controller."**

It waits at the gate until:

- all async dependencies in the subtree are resolved
- then lets the subtree "enter" the UI
- fallback is shown until ready

**For nested boundaries:**

- It creates multiple gates
- The first boundary to resolve continues
- Others resolve independently

**For SSR:**

- Suspense behaves like a streaming checkpoint

## 3. 🧱 How Async Components Work Internally

**Async component:**

```js
const AsyncComponent = defineAsyncComponent(() => import("./Heavy.vue"));
```

This returns a placeholder component until:

- the module is loaded
- its setup function resolves
- its async dependencies are ready

**Suspense tracks all promises inside:**

- async setup
- async components
- composables that use async logic
- components that return promises

## 4. 🧱 Multi-Boundary Suspense

You can nest Suspense boundaries:

```js
<Suspense>
    <template #default>
        <OuterAsync>
            <Suspense>
                <InnerAsync />
            </Suspense>
        </OuterAsync>
    </template>
</Suspense>
```

Vue handles this by building a promise tree.

**Key points:**

- Each Suspense boundary waits for its own subtree
- Inner boundaries don't block outer boundaries
- Outer boundaries only wait for direct children
- Parent and child suspenses resolve independently

This allows selective, progressive hydration and rendering.

## 5. 🧬 Parallel Async Boundaries

You can render multiple async components in parallel:

```js
<Suspense>
    <template #default>
        <div>
            <Profile />
            <Notifications />
            <Feed />
        </div>
    </template>
</Suspense>
```

**Internally:**

- Vue collects all async deps in the subtree
- Waits until they all complete
- Then resolves boundary

This creates predictable loading UI for complex pages.

## 6. ⚙️ Suspense + SSR Streaming

Suspense is tightly integrated with SSR.

**When rendering SSR:**

- Server generates HTML for resolved components
- If hitting a suspended async dependency, it:
  - flushes fallback HTML immediately
  - continues rendering in background
  - streams resolved content into the stream later

**This creates:**

- Faster TTFB (Time to First Byte)
- Better interactivity
- No blocking

Nuxt 3 relies heavily on streamed Suspense boundaries.

## 7. ⚙️ Suspense State Machine (Deep Internal Behavior)

Suspense has three phases:

### 1. Pending

- Async dependencies unresolved
- Fallback rendered

### 2. Resolved

- All dependencies resolved
- Default slot rendered
- DOM patch applied

### 3. Fallback Re-activated (rare)

- Happens when deps change and become async again
- Suspense re-enters pending state

## 8. 🧩 Error Handling Inside Suspense

Suspense boundaries capture asynchronous errors.

**Two options:**

### ✔ onErrorCaptured

Catch errors inside async setup or promises.

### ✔ Error component for async component

```js
defineAsyncComponent({
  loader: () => import("./Comp.vue"),
  errorComponent: ErrorFallback,
});
```

## 9. 🧠 Key Takeaways

- Suspense coordinates async setup, async components, and async composables.
- Nested Suspense boundaries resolve independently.
- Parallel Suspense boundaries allow complex UIs to load gracefully.
- Suspense integrates deeply with streaming SSR, enabling progressive rendering.
- A Suspense boundary = a "promise aggregator" for async dependencies.
- Understanding its state machine is essential for advanced SSR apps (Nuxt, large UIs).
