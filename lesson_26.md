# 🎓 Lesson 26 — watchEffect() vs watch(): Execution Order, Cleanup, and Flush Timing

This is one of the most misunderstood parts of Vue's reactivity system — even for intermediate developers.
At this level of depth, you will understand precisely how watchers run, when they run, and how to control them.

## 1. 🔍 Concept Overview

Vue provides two main watchers:

### `watchEffect()`

- Runs immediately
- Tracks dependencies automatically ("effect-first")
- Re-runs whenever anything it uses changes
- Useful for side effects that depend on reactive reads

### `watch()`

- Runs lazily (not immediate by default)
- Tracks a specific source (ref, getter, array of sources)
- Precise control over dependencies
- Useful for comparing old/new values
- Allows fine control over flush timing

## 2. 💡 Mental Model / Analogy

### 🧠 Think of `watchEffect()` as "always watching what I use."

Like a security camera that auto-detects what's in frame.

**`watchEffect()`:**

- Auto-discovers dependencies
- Runs immediately
- Re-runs whenever any accessed state changes
- Great for reactive computations that produce side effects

### 🧠 Think of `watch()` as "watch only what I told you."

Like putting a sensor only on the front door.

**`watch()`:**

- Monitors exactly one or several sources
- Doesn't care what happens inside the callback
- Gives you precise control over what triggers updates

## 3. 🧱 Examples

### ✔ `watchEffect` — automatic dependency tracking

```js
watchEffect(() => {
  console.log(user.value.name); // tracked automatically
});
```

If any of these change:

- `user.value`
- `user.value.name`

…the effect re-runs.

### ✔ `watch` — explicit dependency

```js
watch(
  () => user.value.name,
  (newName, oldName) => {
    console.log("Name changed:", newName);
  }
);
```

Only changes to `user.value.name` trigger the callback.

## 4. ⚙️ Deep Internal Insight — Execution Order

🚨 **This is critical to understand:**

Vue's update pipeline ensures watchers run in a predictable order.

**Runner order:**

1. Synchronous code finishes
2. "pre" flush watchers (default for `watchEffect`)
3. Component render + DOM patching
4. "post" flush watchers (default for `watch`)
5. `nextTick()` resolves

**Default behavior:**

- `watchEffect` → pre-flush
- `watch` → post-flush

**This means:**

- `watchEffect` runs **before** DOM updates
- `watch` runs **after** DOM updates

### ✔ Why this matters

```js
watchEffect(() => {
  console.log("Before DOM update");
});

watch(
  () => count.value,
  () => {
    console.log("After DOM update");
  }
);
```

This ordering gives you predictable control over:

- when to read reactive state
- when to read DOM state
- when to avoid mismatches

## 5. 🧱 Flush Timing: `flush: 'pre' | 'post' | 'sync'`

You can control when watchers run.

### ✔ `flush: 'pre'` (default for `watchEffect`)

Runs **before** DOM updates.

**Useful for:**

- scheduling work that must happen before rendering
- preparing data before the UI updates

### ✔ `flush: 'post'` (default for `watch`)

Runs **after** DOM updates.

**Useful for:**

- reading updated DOM state
- working with layout
- measuring elements

**Example:**

```js
watch(
  () => count.value,
  () => {
    console.log("DOM has updated");
  },
  { flush: "post" }
);
```

### ✔ `flush: 'sync'` — runs immediately

⚠️ **Use with caution.**

```js
watchEffect(() => console.log(count.value), { flush: "sync" });
```

Runs synchronously every time the dependency changes.

**Can be useful for:**

- debugging
- implementing custom render engines
- certain third-party integrations

**But dangerous because:**

- Might run many times inside the same tick
- Can cause cascading updates
- Can trigger infinite loops

## 6. 🧬 Cleanup Behavior

Both `watchEffect` and `watch` allow cleanup via `onCleanup`.

**Example:**

```js
watchEffect((onCleanup) => {
  const id = setInterval(() => console.log("tick"), 1000);

  onCleanup(() => clearInterval(id));
});
```

**Cleanup runs:**

- before the watcher re-runs
- when the watcher stops (component unmount)

This ensures no leaks.

## 7. 🧩 When to use which?

### ✔ Use `watchEffect` when:

- You don't want to specify dependencies
- You want auto-tracked side effects
- The effect depends on many reactive sources
- The effect doesn't care about old/new values
- You want a "computed-with-side-effects"

### ✔ Use `watch` when:

- You need `newValue` / `oldValue`
- You want targeted tracking
- You want to prevent unnecessary triggers
- You need `flush: 'post'` for DOM reads
- You want to watch arrays or multiple sources

## 8. 🧠 Key Takeaways

- `watchEffect` = auto-tracked, pre-flush, eager
- `watch` = source-specified, post-flush, controlled
- Use `flush` to precisely control timing
- Cleanup ensures effects remain safe and leak-free
- Understanding watcher ordering prevents infinite loops
- DOM measurements must be done with `flush: 'post'`
