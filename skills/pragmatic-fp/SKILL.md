---
name: pragmatic-fp
description: >
  Apply pragmatic functional programming principles when writing or reviewing code, inspired by
  the CodeAesthetic "Dear Functional Bros" philosophy. Use this skill whenever the user asks to
  write new code, refactor existing code, or review code for quality — especially if they mention
  "functional", "pure functions", "pipelines", "immutability", "side effects", or "clean code".
  Also trigger when the user pastes code and asks "how can I improve this?" or "is this clean?"
  or "refactor this" — even without explicit FP language. The skill applies language-agnostically
  to any language (JS, TS, Python, Go, Rust, Java, etc.).
---

# Pragmatic Functional Programming

Inspired by CodeAesthetic's *Dear Functional Bros*: use FP **when it makes code more concise and
easier to read**. Not as a religion — as a tool.

---

## Core Philosophy

> "Functional programming is great — use it when applicable, and when the resulting code is more
> concise and easier to read. Except if it involves recursion."

Three questions to ask before applying FP:
1. Does this make the code **shorter**?
2. Does this make the code **easier to read**?
3. Does this **eliminate a bug class** (shared state, mutation, side-effect surprises)?

If the answer to at least one is "yes" and none of the three are harmed, apply FP. Otherwise, don't.

---

## The Core Principles (in priority order)

### 1. Pure Functions First
A function is pure if:
- It only reads its inputs (no external state)
- It only affects its output (no side effects)
- Same inputs → same output, always

**Why:** Pure functions are trivially testable, composable, and safe to refactor.

**Apply when:** You can isolate a computation from I/O, state mutation, or system calls.

**Don't force it when:** The function *is* a side effect (logging, writing to disk, network calls).
Those are fine — just quarantine them at the edges.

```python
# ❌ Impure — depends on external state, hard to test
def get_discount(user):
    if datetime.now().hour < 12:
        return user.cart_total * MORNING_DISCOUNT
    return user.cart_total

# ✅ Pure — all inputs explicit, trivially testable
def get_discount(cart_total, hour, morning_discount):
    if hour < 12:
        return cart_total * morning_discount
    return cart_total
```

---

### 2. Data Pipelines Over Loops
When transforming a collection, prefer a pipeline of named operations over a mutable loop.

**Apply when:** You are filtering, mapping, reducing, or chaining transformations on data.

**Don't force it when:** You need to break early with complex conditions, or when the pipeline
becomes longer and harder to read than the loop.

```javascript
// ❌ Loop — accumulates state, harder to follow intent
const result = [];
for (const order of orders) {
  if (order.status === 'paid') {
    const total = order.items.reduce((sum, i) => sum + i.price, 0);
    if (total > 100) result.push({ ...order, total });
  }
}

// ✅ Pipeline — each step declares what it does
const result = orders
  .filter(order => order.status === 'paid')
  .map(order => ({ ...order, total: order.items.reduce((s, i) => s + i.price, 0) }))
  .filter(order => order.total > 100);
```

---

### 3. Immutability by Default
Prefer creating new values over mutating existing ones.

**Apply when:** You're working with objects/arrays that are passed around or used after mutation.

**Don't force it when:** Performance is critical and mutation is local and contained (fine inside
a pure function that returns a new value, for example).

```python
# ❌ Mutation — caller's data changed unexpectedly
def apply_tax(items):
    for item in items:
        item['price'] *= 1.1
    return items

# ✅ Immutable — original untouched
def apply_tax(items):
    return [{ **item, 'price': item['price'] * 1.1 } for item in items]
```

---

### 4. Avoid Recursion (Unless It's the Natural Shape of the Problem)
Iteration is almost always clearer, faster, and safer than recursion in most languages.

**Use recursion only when:**
- The problem is inherently recursive (tree traversal, parsers, divide-and-conquer)
- The language optimizes tail recursion (e.g., some Lisps, Erlang)
- The recursive version is genuinely simpler to read

**Prefer iteration when:**
- Processing flat or semi-flat collections
- You'd reach for recursion just because it "feels functional"

```javascript
// ❌ Recursive for no reason — tail call not guaranteed, harder to debug
function sum(arr, acc = 0) {
  if (arr.length === 0) return acc;
  return sum(arr.slice(1), acc + arr[0]);
}

// ✅ Just use reduce
const sum = arr => arr.reduce((a, b) => a + b, 0);
```

---

### 5. Push Side Effects to the Edges
Don't eliminate side effects — control where they live. Keep the core logic pure, and let the
outer layer handle I/O, mutation, and system interaction.

```
┌────────────────────────────────────────┐
│  Outer layer: I/O, DB, API, logging    │  ← side effects live here
│  ┌──────────────────────────────────┐  │
│  │  Core logic: pure functions      │  │  ← no side effects here
│  └──────────────────────────────────┘  │
└────────────────────────────────────────┘
```

---

## When Writing New Code

1. Start with the data transformation: what goes in, what comes out?
2. Write the core as pure functions.
3. Use pipelines for collection work.
4. Add I/O at the edges only.
5. Reach for recursion only if the problem has a recursive shape.

---

## When Reviewing Code

Check for each principle above and flag violations with the **pragmatic test**:
> "Would making this more functional make it shorter AND easier to read?"

If no — leave it. If yes — suggest the refactor with a brief explanation.

When reviewing, always note:
- 🟢 **Good FP use** — worth calling out and explaining why it works
- 🟡 **FP overkill** — functional style that adds complexity without clarity
- 🔴 **Missed opportunity** — a place where FP would clearly improve things

---

## Anti-Patterns to Flag

| Anti-pattern | Why it's a problem | FP fix |
|---|---|---|
| Mutating function arguments | Caller's data changes unexpectedly | Return new value |
| Mixed pure + impure logic | Untestable core | Separate pure core from side effects |
| Recursion on flat data | Stack risk, harder to debug | Use map/reduce/filter |
| 8-level nested callbacks | Pyramid of doom | Pipeline / composition |
| Shared mutable state across functions | Race conditions, hard to test | Pass state explicitly |

---

## The Pragmatic Override Rule

If a FP approach requires:
- A helper library the codebase doesn't already use
- Explaining a concept (monad, functor, applicative) to the next reader
- More lines than the imperative version

...then **don't use it**. Clarity always wins.
