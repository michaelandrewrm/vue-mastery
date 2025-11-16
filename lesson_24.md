# 🧩 Lesson 24 — Vue + TypeScript for Scalability

Vue 3 was designed from the ground up to work beautifully with TypeScript.

**TypeScript gives you:**

- Safety (no more "undefined is not a function")
- Autocomplete for props, emits, stores, and composables
- Self-documenting components
- Refactoring confidence
- Reusable typed models and DTOs

In large applications, TS pays back tenfold.

Let's break down how to type props, emits, refs, components, and composables.

## 1. 🔍 Concept Overview

TypeScript integrates with Vue in three core surfaces:

**✔ Component API**

`defineProps()`, `defineEmits()`, slots, and component return types.

**✔ Reactivity**

Typing `ref()`, `reactive()`, and derived values.

**✔ Composables**

Typing function parameters, return objects, and reactive dependencies.

Vue's "script setup" + TS gives you:

- compile-time checked props
- typed emits with argument validation
- typed component events
- typed injection through provide/inject
- fully typed composables that enforce input/output contracts

## 2. 💡 Mental Model / Analogy

🧠 **Think of TypeScript as Vue's "schema" and "contract lawyer."**

- Vue provides the runtime behavior.
- TypeScript guarantees you never break the contract.

**Props** are function parameters — typed  
**Emits** are function calls — typed  
**Refs** are reactive boxes — typed  
**Composables** are APIs — typed

The result is a Vue codebase that's safe, refactorable, and discoverable.

## 3. 🧱 Typing Props and Emits

### ✔ Typing Props with `defineProps`

```vue
<script setup lang="ts">
const props = defineProps<{
  id: number;
  title?: string;
  tags: string[];
}>();
</script>
```

**Benefits:**

- Autocomplete in template & script
- Compiler enforces types
- Optional vs required props are distinguished

### ✔ Typing Emits with `defineEmits`

```ts
const emit = defineEmits<{
  (e: "select", id: number): void;
  (e: "delete", id: number): void;
}>();
```

Now emits are validated:

```ts
emit("select", 123); // ✔
emit("select"); // ❌ missing argument
emit("remove", 1); // ❌ no such event
```

Perfect type safety.

## 4. 🧱 Typing Refs and Reactivity

### ✔ Basic typed ref

```ts
const count = ref<number>(0);
```

### ✔ Typing more complex refs

```ts
interface Product {
  id: number;
  name: string;
  price: number;
}

const product = ref<Product | null>(null);
```

### ✔ Reactive objects

```ts
const form = reactive<{
  name: string;
  age: number;
}>({
  name: "",
  age: 0,
});
```

## 5. 🧱 Strongly-Typed Components

Typing component props + emits in reusables:

**Child.vue**

```vue
<script setup lang="ts">
export interface ItemData {
  id: number;
  name: string;
}

const props = defineProps<{
  item: ItemData;
}>();

const emit = defineEmits<{
  (e: "update", item: ItemData): void;
}>();
</script>
```

**Parent.vue**

```vue
<Item :item="myItem" @update="handleUpdate" />
```

You now get:

- Autocomplete for props
- Type enforcement for event payloads
- No ambiguity about what a component expects

## 6. 🧱 Typing Composables

Composables must be typed, or else you lose all benefit at scale.

**Example: a reusable pagination composable.**

```ts
// usePagination.ts
export function usePagination<T>(items: Ref<T[]>, pageSize = 10) {
  const page = ref(1);

  const paginated = computed(() => {
    const start = (page.value - 1) * pageSize;
    return items.value.slice(start, start + pageSize);
  });

  function next() {
    page.value++;
  }

  return { page, paginated, next };
}
```

**Usage:**

```ts
const items = ref<User[]>([]);
const { page, paginated } = usePagination(items);
```

TypeScript infers `paginated` as `Ref<User[]>` — powerful.

## 7. 🧱 Typing Dependency Injection (provide/inject)

### Providing typed values

```ts
// provider
provide<Ref<number>>("count", count);
```

### Injecting with type safety

```ts
const count = inject<Ref<number>>("count");
```

No more guessing what's injected.

## 8. ⚙️ Internal Insight — How Vue + TS Work Together

Vue's compiler analyzes `defineProps()` and `defineEmits()` types:

- It constructs runtime validators (optional)
- It infers the component's "signature"
- It generates correct types for template expressions
- Templates gain autocomplete based on prop types

This means:

- Templates inherit actual TypeScript types
- Error messages appear before runtime
- Templates behave like callable TS functions

Vue's type inference is one of the strongest in any modern frontend framework.

## 9. 🧩 Common Gotchas

### ❌ Using both runtime props and TS types

```ts
defineProps({ id: Number }); // runtime-only
defineProps<{ id: number }>(); // TS-only
```

Use one or the other, not both.

### ❌ Using `reactive()` for primitives

**Bad:**

```ts
const count = reactive(0); // ❌ breaks typing
```

Use `ref()`.

### ❌ Forgetting null or undefined in types

If a prop can be missing:

```ts
title?: string
```

If data loads async:

```ts
const user = ref<User | null>(null);
```

### ❌ Typing emits incorrectly

**Wrong:**

```ts
const emit = defineEmits<{ select: (id: number) => void }>();
```

**Correct format:**

```ts
const emit = defineEmits<{
  (e: "select", id: number): void;
}>();
```

## 10. 🧠 Key Takeaways

- Vue 3 integrates deeply with TypeScript — `defineProps`/`defineEmits` are first-class TS constructs.
- Props and emits become fully typed APIs.
- Refs and reactive objects accept type parameters.
- Composables become strongly typed reusable logic units.
- Injection can be made type-safe with generic keys.
- Feature-based architecture + TypeScript leads to predictable, scalable systems.
- TS ensures components are self-documenting and refactorable.
- Mastering Vue + TS is essential for scaling to enterprise-level applications.
