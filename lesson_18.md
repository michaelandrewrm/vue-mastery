# 🕰 Lesson 18 — Scheduler and nextTick

Vue does not update the DOM immediately when you change reactive state.

Instead, it:

- Queues updates into an async job queue
- Batches multiple changes together
- Flushes them in a microtask (after the current call stack)
- Lets you wait for the flush via `nextTick()`

Understanding this flow is crucial to:

- Predict DOM updates
- Avoid race conditions
- Use `nextTick()` correctly
- Reason about performance and batching

## 1. 🔍 Concept Overview

**Key ideas:**

- **Async update queue**: Vue schedules DOM updates instead of applying them instantly.
- **Batching**: Multiple reactive changes in the same "tick" are merged into a single render.
- **Microtask scheduling**: Vue uses microtasks (like `Promise.resolve().then(...)`) to flush updates.
- **`nextTick()`**: A helper that waits until Vue has finished flushing all pending DOM updates.

So the pattern is:

**You mutate reactive state** → **Vue schedules a render** → **DOM updates later** → **`nextTick()` waits for that moment**.

## 2. 💡 Mental Model / Analogy

🧠 **Think of Vue like a "smart secretary" handling tasks.**

Every state change is a request: "Update the UI!"

But instead of doing each request immediately (chaos), the secretary says:

> "Cool, I'll note that down and process all requests at the end of this time slot."

Vue collects all these update requests, then processes them in a batch.

`nextTick()` is like saying:

> "Let me know when you're done with all the paperwork so I can look at the final result."

## 3. 🧱 Code Example — Batching and nextTick

```vue
<script setup>
import { ref, nextTick } from "vue";

const count = ref(0);

async function updateAndLog() {
  count.value++;
  count.value++;
  count.value++;

  console.log("Before nextTick, DOM still shows old count");

  await nextTick();

  console.log("After nextTick, DOM is updated with new count");
}
</script>

<template>
  <p id="count-el">Count: {{ count }}</p>
  <button @click="updateAndLog">Update</button>
</template>
```

**What happens:**

1. Click button → `updateAndLog` runs.
2. `count.value++` called 3 times.
3. **Vue does not re-render 3 times.**  
   It schedules one render for this tick.
4. `console.log` before `nextTick` happens before DOM updates.
5. `await nextTick()` waits for Vue to flush the DOM updates.
6. `console.log` after `nextTick` runs with DOM in sync with latest count.

## 4. ⚙️ Internal Insight — The Async Update Queue

Whenever a reactive change triggers an effect (like a component's render effect), Vue doesn't call it immediately.

Instead, it:

1. Adds the effect to a job queue
2. Ensures each effect appears only once in the queue (no duplicate reruns)
3. Schedules a microtask to flush the queue

**Conceptually:**

```js
const queue = new Set();
let isFlushing = false;

function queueJob(job) {
  if (!queue.has(job)) {
    queue.add(job);
    queueFlush();
  }
}

function queueFlush() {
  if (!isFlushing) {
    isFlushing = true;
    Promise.resolve().then(flushJobs);
  }
}

function flushJobs() {
  for (const job of queue) {
    job();
  }
  queue.clear();
  isFlushing = false;
}
```

**Key points:**

- Jobs are de-duplicated using a `Set`.
- All updates for the current tick are batched.
- DOM updates happen in a microtask, not synchronously.

## 5. 🧬 nextTick() and Microtask Scheduling

`nextTick()` is essentially:

```js
function nextTick(fn) {
  return Promise.resolve().then(fn);
}
```

…but chained into Vue's flush lifecycle so it runs after queued jobs.

**Timeline:**

1. Event handler starts
2. You update state multiple times
3. Vue schedules a flush in a microtask
4. Event handler finishes
5. JavaScript call stack is empty
6. Microtask queue runs:
   - Vue flushes all jobs (re-renders components, patches DOM)
7. `nextTick()` callbacks run
8. UI is now fully updated

**So:**

- Synchronous code after `count.value++` sees stale DOM.
- Code inside `nextTick` sees the final DOM.

## 6. 🧱 Visualizing Reactivity Updates Step-by-Step

Let's walk through a concrete example:

```vue
<script setup>
import { ref, nextTick } from "vue";

const count = ref(0);

async function handleClick() {
  console.log("Initial count:", count.value); // 0

  count.value++;
  count.value++;
  console.log("After increments, count:", count.value); // 2

  const el = document.getElementById("count-el");
  console.log("DOM text BEFORE nextTick:", el.textContent); // still old (Count: 0)

  await nextTick();

  console.log("DOM text AFTER nextTick:", el.textContent); // now Count: 2
}
</script>

<template>
  <p id="count-el">Count: {{ count }}</p>
  <button @click="handleClick">Click</button>
</template>
```

**Step-by-step:**

1. DOM shows `Count: 0`.
2. You click the button → `handleClick` runs.
3. `count.value` goes from `0` → `1` → `2`.
4. Vue queues one render job in the microtask queue.
5. Inside `handleClick`, before `nextTick`, DOM is still `Count: 0`.
6. `await nextTick()` suspends until Vue finishes flushing.
7. Vue re-renders the component & updates DOM to `Count: 2`.
8. The log after `nextTick` sees the updated DOM.

## 7. ⚙️ How Vue Batches Updates for Performance

**Without batching:**

- Each `count.value++` would cause a separate re-render.
- Complex UIs would crawl.

**With batching:**

```js
count.value++; // mark job as queued
count.value++; // job already queued
count.value++; // job still only once
```

→ 1 render job, not 3.

**Vue's scheduler:**

- De-duplicates jobs
- Orders jobs (parents before children, computed before watchers, etc.)
- Uses microtasks to stay responsive

**Why microtasks?**

- Run after the current call stack
- But before the next frame render
- Perfect spot to update the DOM before the browser paints
- Keeps the UI consistent per frame

## 8. 🧩 Common Pitfalls & Misconceptions

### ❌ "I changed the state, why isn't the DOM updated yet?"

Because Vue updates DOM asynchronously.

🔑 **Fix:** If you need to inspect the new DOM, use `await nextTick()`.

### ❌ "nextTick fixes all timing issues, right?"

Not quite.

`nextTick` waits for Vue's update flush, not arbitrary async operations.

You still need correct async logic for API calls, etc.

### ❌ "Each change triggers a separate re-render."

No — in a single tick (event loop run), Vue batches all changes into one render per effect.

### ❌ "I can rely on DOM being updated after any async await."

Not necessarily. Only `await nextTick()` is tied to Vue's flush cycle.  
`await somePromise` does not guarantee DOM has updated.

## 9. ⚙️ Advanced Insight — Multiple Flush Phases

Internally, Vue distinguishes phases:

1. **Pre-flush jobs** (e.g. watchers with `flush: 'pre'`)
2. **Component render jobs**
3. **Post-flush jobs** (e.g. watchers with `flush: 'post'`, `onUpdated`)

**Execution order matters, e.g.:**

1. Computed & pre-flush watchers
2. Render updates
3. DOM patching
4. Post-flush watchers
5. `onUpdated` hooks
6. `nextTick` callbacks

**This ensures:**

- Computed values are up-to-date before rendering
- DOM is in its final state when `onUpdated` and `nextTick` run

## 10. 🧠 Key Takeaways

- Vue uses an **async scheduler** with a microtask-based update queue.
- State changes are **batched**; many changes → one render.
- `nextTick()` lets you wait until DOM updates are finished.
- Synchronous code after a mutation sees old DOM; use `nextTick` to see the updated UI.
- Vue de-duplicates jobs and orders them for correctness and performance.
- Understanding this loop helps you avoid timing bugs and write smoother UIs.
