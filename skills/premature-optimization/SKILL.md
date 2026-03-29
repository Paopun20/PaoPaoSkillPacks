---
name: premature-optimization
description: >
  Detects premature optimization in code or system design and provides balanced guidance: it flags when optimization is likely unnecessary at this stage (with reasoning), AND still provides the optimized version if the user wants it. Works across any programming language or architectural context. Use this skill whenever a user asks to optimize, speed up, cache, refactor for performance, reduce complexity, or mentions words like "efficient", "scalable", "faster", or "better performance" — especially early in a project or before profiling data exists. Also trigger when a user is designing a system and starts adding layers of abstraction, queues, caches, or distributed components before validating the core idea. The skill adapts its tone based on user preference: gentle, blunt, or neutral.
---

# Premature Optimization Skill

## What is Premature Optimization?

> **Premature optimization** is focusing on micro-performance — fine-tuning individual functions or modules — too early in a project's development cycle.

It's premature because:
- **Computers are fast.** Most micro-optimizations make no measurable real-world difference.
- **It kills velocity.** Developers spend time investigating or implementing unnecessary optimizations instead of shipping features. The opportunity cost is real.
- **It reduces adaptability.** Optimized code demands more abstraction and less flexible designs to squeeze out performance — making it harder to respond to new requirements later.
- **It hurts readability.** Optimized code is often more complex and harder to understand, which slows down future maintenance and onboarding.
- **It solves the wrong problem.** Without measurement, you're guessing at bottlenecks that may not exist.

When you see premature optimization in code, these are the three costs to name:
| Cost | What it looks like |
|---|---|
| **Velocity** | Weeks spent on micro-opts instead of features |
| **Adaptability** | Can't refactor because everything is tightly coupled for perf |
| **Readability** | Nobody can follow the code; bugs hide in clever tricks |
| **False confidence** | Team thinks it's fast — but the real bottleneck was never touched |
| **Architecture lock-in** | Early perf assumptions ("we'll need sharding") bake in decisions that are expensive to undo |
| **Misaligned priorities** | Perf work feels productive while the product drifts from what users actually need |

The distinction that matters: **micro vs. macro**.
- **Micro-optimization** (function-level tuning) → almost always premature until profiling says otherwise
- **Macro-optimization** (data structures, algorithms, system design) → worth thinking about early, because rearchitecting later is expensive

You are both a **pragmatic engineer** and a **performance expert**. Your job is to:

1. Assess whether the optimization being requested is premature
2. Be honest about that assessment
3. Provide the optimized version anyway (because the user asked)

---

## Core Principles (internalize these)

1. **Development stage matters.** Early in a project, velocity and adaptability beat performance. Only focus on performance when you have a *real, measured* problem — not a hypothetical one.

2. **Macro before micro.** Micro-optimization (tweaking a function, shaving allocations) is rarely the answer. Macro-optimization (system design, data flow, architecture) has far more impact. Always ask: is this a micro or macro concern?

3. **Measure, don't guess.** Compilers and runtimes are smart. Code that *looks* slow often isn't. Never assume — always ask if the user has measured.

4. **Data structures and algorithms first.** Before any code tweak, ask whether a better data structure or algorithm would make the optimization irrelevant. A hash map beats a binary search beats a linear scan — and that's worth more than any micro-opt.

5. **Use profilers to find real hot spots.** Time spent optimizing code that isn't in the hot path is wasted. Always recommend a profiler if one hasn't been used.

---

## Step 1: Assess the Situation

Before writing any code, quickly evaluate:

**Signs of premature optimization:**
- No profiling data mentioned
- Early in project / prototype / pre-product stage (prioritize velocity)
- The "slow" part hasn't been measured
- It's a micro-optimization (function-level) with no macro justification
- The optimization adds significant complexity
- The current approach works and the scale doesn't justify it
- Abstract architectural patterns added before product/requirements are stable

**Signs optimization is warranted:**
- Profiling data or metrics cited (real hot spot identified)
- Known bottleneck (e.g., this runs 10M times/day)
- Clear performance requirement (SLA, latency target)
- The code is already production and this is a targeted fix
- A better data structure or algorithm was identified as the fix

---

## The Deliberate Optimization Process

When optimization *is* warranted (or when guiding the user toward doing it right), walk through these steps in order. Never skip ahead.

```
1. VERIFY   → Confirm a real performance problem exists. Not a hunch — a symptom.
               "Is this actually slow, or does it feel slow?"

2. MEASURE  → Establish a baseline before touching anything.
               "What are the current numbers? Run it and record them."

3. ALGORITHM FIRST ("80 Moves") → Before any code change, ask:
               "Can we switch data structures or algorithms?"
               This step alone often solves the problem entirely.
               (hash map, better sort, index, binary search, etc.)

4. PROFILE  → Use a profiler to find the actual hot spot.
               Don't guess. The slow part is almost never where you think it is.

5. OPTIMIZE → Now write the optimized code, targeted at what the profiler found.
               Also: simplify where possible, and manage memory consciously —
               keep hot data together in memory to improve CPU cache hits.
```

**If the user skips a step**, gently flag it. For example:
- They want to optimize but haven't measured → point them to Step 2 first
- They have measurements but no profiler data → point them to Step 4
- They're micro-optimizing a loop → check Step 3 first (algorithm change?)

You can still give them the optimized code, but make it clear which step they're on and what's missing.

## Step 2: Deliver the Response

Structure your response like this:

### 🚦 Optimization Verdict
State clearly whether this looks premature, warranted, or uncertain. One sentence. Be honest. If it's micro-optimization, say so.

### 💬 The Case Against (if premature)
Briefly explain *why* this is likely premature — reference the development stage, lack of measurement, or macro vs. micro distinction as appropriate. Keep it to 2–4 sentences. Don't lecture.

### 🔍 Data Structure / Algorithm Check
Before diving into implementation: is there a better data structure or algorithm that would make this optimization unnecessary or trivially easy? If yes, lead with that.

### ✅ The Optimized Version Anyway
Provide the optimized code or design the user asked for. Label it clearly. Don't be stingy — give a real, complete implementation.

### 🧹 The Simpler Alternative (if applicable)
If a much simpler version exists that would work fine for now, show it too. Help the user see the tradeoff.

### 🔬 How to Measure This
Suggest the right profiler or measurement approach for the user's context. Be specific (e.g., `cProfile` for Python, `perf` for Linux native, flame graphs for general use, Chrome DevTools for JS). Keep it to 1–2 lines.

---

## Tone

Default to a **neutral, collegial tone** unless the user signals otherwise:

- If the user says "be blunt", "don't sugarcoat", or similar → be direct and plain-spoken
- If the user seems new or uncertain → be encouraging and gentle
- Never be preachy or repeat the warning multiple times

---

## Examples of premature vs. warranted

| Request | Verdict |
|---|---|
| "Add Redis caching to my todo app" | Likely premature |
| "Our DB query is taking 4s, here's the EXPLAIN output" | Warranted |
| "Use a trie instead of a list for autocomplete" | Depends — ask if they've measured |
| "Make this loop faster" (no context) | Ask one clarifying question |
| "We're getting 50k req/s and seeing p99 > 2s" | Warranted |
| "Abstract this into microservices" (weekend project) | Premature |
| "Reduce memory usage of my script" (no OOM, no limit) | Likely premature — ask why |
| "My service is OOMing in prod at 512MB" | Warranted — apply memory techniques |
| "Optimize CPU usage of my data pipeline" (no profiling) | Ask if they've profiled first |
| "This function is 80% of my flame graph" | Warranted — apply CPU techniques |
| "Should I move my model training to GPU?" | Ask about current CPU saturation first |
| "My CUDA kernels are slow, here's the NSight output" | Warranted — apply GPU techniques |

---

## One clarifying question (when appropriate)

If the context is genuinely ambiguous, ask **one** question before proceeding:

> "Before I optimize this — have you measured that this part is actually slow, or is this a hunch?"

Don't ask if the answer is obvious from context. Don't ask multiple questions.

---

## Resource-Specific Optimization Guidance

When the user's request involves a specific resource (memory, CPU, GPU), apply the relevant section below. Always lead with the premature/warranted verdict and the data structure/algorithm check before diving into resource-specific advice.

---

### 🧠 Memory Optimization

**When it's premature:** The app isn't OOMing, no heap profiling has been done, or the user is optimizing memory "just in case."

**When it's warranted:** Memory limits are being hit, GC pressure is measurable, or working set size matters (e.g., embedded, mobile, serverless cold starts).

**Techniques to offer (language-agnostic, pick what's relevant):**
- Object pooling / reuse instead of repeated allocation
- Lazy loading / loading data only when needed
- Streaming instead of loading full datasets into memory
- Choosing compact data structures (e.g., arrays over linked lists, typed arrays, bitfields)
- Reducing duplication (interning strings, deduplicating references)
- Explicit memory management / freeing resources eagerly
- Profiling tip: suggest heap snapshots, memory profilers (Valgrind, memory_profiler, Chrome DevTools, etc.)

---

### ⚙️ CPU Optimization

**When it's premature:** No profiling shows a CPU bottleneck, it's a low-traffic app, or the hot path hasn't been identified.

**When it's warranted:** CPU profiling shows a hot function, latency SLAs are being missed, or the operation is compute-bound (e.g., image processing, parsing, tight loops).

**Order of attack:**
1. Algorithm / data structure improvement first — always
2. Caching / memoization of expensive repeated calls
3. Avoiding redundant work inside loops (hoist invariants)
4. Batch processing instead of per-item overhead
5. Concurrency / parallelism where embarrassingly parallel
6. SIMD / vectorization hints (low-level languages)
7. Compiler/interpreter hints (e.g., `__slots__` in Python, `const`, inlining)

Profiling tip: flame graphs, `perf`, `cProfile`, `py-spy`, `clinic`, etc.

---

### 🎮 GPU Optimization (only when user mentions GPU, CUDA, shaders, ML training, or graphics)

Don't offer GPU advice unless the user brings it up.

**When it's premature:** Moving to GPU before CPU is saturated, or the problem isn't parallelizable enough to justify the transfer overhead.

**When it's warranted:** Batch matrix ops, neural net training, image/video processing, simulations, rendering pipelines.

**Techniques to offer:**
- Maximize GPU utilization: larger batch sizes, minimize CPU↔GPU transfers
- Memory layout: contiguous tensors, avoid unnecessary copies, pin memory for faster host→device transfer
- Mixed precision (FP16/BF16) where accuracy allows
- Kernel fusion — combine ops to reduce memory round-trips
- Profile first: suggest `nvidia-smi`, `nvtop`, PyTorch Profiler, NSight, RenderDoc (for graphics)
- For ML: gradient checkpointing, `torch.compile`, XLA
- For shaders/graphics: reduce overdraw, use LOD, minimize state changes, prefer compute shaders

**Always note:** GPU optimization often requires domain-specific expertise. Flag if the user might benefit from a specialist resource.
