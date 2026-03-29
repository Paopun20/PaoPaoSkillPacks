---
name: readable-code
description: >
  Apply this skill when the user asks to write, review, refactor, or critique code for clarity and maintainability.
  Triggers include: "make this readable", "refactor this", "clean up my code", "code review", "is this clean code",
  "improve naming", "simplify this function", "review for best practices", or any request to write new code where
  quality and clarity are implied. Also apply when the user pastes code and asks "what do you think?" or "how can
  I improve this?". Do NOT use for pure performance optimization, security audits, architecture design, or tasks
  where readability is explicitly out of scope.
license: MIT
---

# Law of Readable Code

This skill instructs how to write, review, and refactor code so it is maximally readable and maintainable.
Apply these laws both when generating new code and when critiquing or improving existing code.

## Core Philosophy

> "Code is read far more often than it is written."

Readable code is not a style preference — it is a professional obligation. Every decision should optimize for
the human reader, not the compiler. When reviewing code, cite the specific law being violated. When writing
code, internalize these laws before generating the first line.

---

## The 12 Laws

### Law 1 — Principle of Least Surprise
Code must behave exactly as its name implies. No hidden side effects, no unexpected mutations, no silent failures.

**Checklist:**
- Does the function name fully describe what it does?
- Does it modify state the caller cannot see from the signature?
- Does it return something surprising given its name?

**Pattern — separate side effects from queries:**
```python
# Violates — hidden write inside a read
def get_user(id):
    user = db.find(id)
    db.log_access(id)   # surprise!
    return user

# Compliant — explicit names, separated concerns
def get_user(id):
    return db.find(id)

def get_user_and_log(id):
    db.log_access(id)
    return db.find(id)
```

---

### Law 2 — Names Are Documentation
Every identifier is a micro-comment. If a reader needs a comment to understand a name, the name is wrong.

**Naming conventions:**
| Kind | Convention | Examples |
|------|-----------|---------|
| Variables | Descriptive nouns | `invoiceTotal`, `retryCount` |
| Functions | Verb phrases | `calculateTax()`, `fetchProfile()` |
| Booleans | Question form | `isActive`, `hasPermission`, `canRetry` |
| Constants | SCREAMING_SNAKE_CASE | `MAX_RETRIES`, `DEFAULT_TIMEOUT_MS` |
| Classes | PascalCase nouns | `PaymentProcessor`, `UserSession` |

**Anti-patterns to flag:**
- Single-letter names outside loop counters (`i`, `j`) or math (`x`, `y`)
- Abbreviations that require domain knowledge (`usrMgr`, `cfg`, `val`)
- Misleading types in names (`userList` that is actually a `Map`)
- Generic placeholders (`data`, `info`, `result`, `temp`, `obj`)

---

### Law 3 — One Function, One Purpose
A function does one thing. If "and" is needed to describe it, split it.

**Signal words that demand a split:** `and`, `or`, `also`, `then`, `plus`, `while`.

**Metrics:**
- Lines: ≤ 20–30 lines per function
- Parameters: ≤ 3–4; beyond that, group into an object/struct
- Cyclomatic complexity: ≤ 10 branches

```python
# Violates — two responsibilities
def validate_and_save(user):
    if not user.email:
        raise ValueError("Email required")
    db.save(user)

# Compliant
def validate_user(user):
    if not user.email:
        raise ValueError("Email required")

def save_user(user):
    db.save(user)
```

---

### Law 4 — Comments Explain *Why*, Not *What*
If the code is clear, it explains *what*. Comments explain *why* a decision was made, or warn of a non-obvious danger.

**Valid comment categories:**
1. Business rule justification: `// Per SOX compliance, all writes must be logged`
2. Non-obvious algorithm: `// Using Fisher-Yates for uniform distribution`
3. Known gotcha: `// API returns 204 on success, not 200`
4. Reference: `// See RFC 2822 §3.6 for header format`

**Comment anti-patterns to remove:**
```javascript
i++;              // increment i              ← restates code
return null;      // return null              ← restates code
// TODO: fix this                             ← undated, unowned
/* commented-out block */                     ← dead code, delete it
```

---

### Law 5 — Prefer Shallow Nesting
Deep nesting (> 2–3 levels) imposes a cognitive stack on the reader. Flatten with guard clauses and extraction.

**Refactor pattern — invert conditions and return early:**
```python
# Violates — pyramid of doom
def process(order):
    if order:
        if order.is_paid():
            if order.items:
                for item in order.items:
                    if item.in_stock():
                        ship(item)

# Compliant — guard clauses + extraction
def process(order):
    if not order or not order.is_paid() or not order.items:
        return
    ship_available_items(order.items)

def ship_available_items(items):
    for item in items:
        if item.in_stock():
            ship(item)
```

---

### Law 6 — Keep Functions Short
A function should fit on one screen without scrolling. If it does not, it is doing too much.

**Hard limits to enforce:**

| Metric | Target |
|--------|--------|
| Lines per function | ≤ 30 |
| Lines per file | ≤ 400 |
| Parameters | ≤ 4 |
| Nesting depth | ≤ 3 |

When a function exceeds limits: extract cohesive sub-steps into well-named helpers.

---

### Law 7 — Symmetry and Consistency
Inconsistency is a cognitive tax on every reader. Pick one convention per project and apply it without exception.

**Consistency checklist:**
- [ ] One naming style per language idiom (camelCase, snake_case, etc.)
- [ ] One error-handling pattern (exceptions vs. result types vs. callbacks)
- [ ] One import ordering convention
- [ ] Consistent function signature style (positional vs. keyword args)
- [ ] Symmetrical paired functions: `open/close`, `start/stop`, `create/destroy`

When reviewing: flag any file that mixes styles. When writing: match the surrounding codebase.

---

### Law 8 — No Magic Numbers or Strings
Naked literals are unreadable and unmaintainable. Every literal that requires thought must be a named constant.

```java
// Violates
if (status == 3) applyDiscount(0.15);

// Compliant
static final int STATUS_PREMIUM = 3;
static final double PREMIUM_DISCOUNT = 0.15;

if (status == STATUS_PREMIUM) applyDiscount(PREMIUM_DISCOUNT);
```

**Rule:** If a literal appears more than once, OR requires thought to understand its meaning, extract it.

---

### Law 9 — Make the Happy Path Obvious
The normal, successful execution flow must read linearly from top to bottom. Errors, guards, and edge cases
belong at the edges — not inline.

```go
// Violates — happy path buried in else branches
func load(path string) (Config, error) {
    if path != "" {
        if data, err := os.ReadFile(path); err == nil {
            return parse(data)
        } else {
            return Config{}, err
        }
    } else {
        return Config{}, errors.New("path required")
    }
}

// Compliant — guards up top, happy path flows down
func load(path string) (Config, error) {
    if path == "" {
        return Config{}, errors.New("path required")
    }
    data, err := os.ReadFile(path)
    if err != nil {
        return Config{}, err
    }
    return parse(data)
}
```

---

### Law 10 — Delete Dead Code Ruthlessly
Commented-out code, unused variables, unreachable branches, and stale TODOs are noise. Delete them.

**Delete on sight:**
- Blocks of commented-out code (`// old implementation`)
- Variables declared but never read
- Functions with zero call sites
- Imports that are unused
- Conditions that are always true or always false

> Version control is the safety net. Dead code in the file is not a backup — it is a liability.

---

### Law 11 — Express Intent, Not Mechanics
Write *what* should happen, not the step-by-step *how*. Use higher-order functions, declarative APIs,
and expressive abstractions.

```javascript
// Violates — describes mechanical steps
const result = [];
for (let i = 0; i < users.length; i++) {
    if (users[i].active === true) {
        result.push(users[i].email);
    }
}

// Compliant — expresses intent
const result = users
    .filter(u => u.active)
    .map(u => u.email);
```

---

### Law 12 — The Boy Scout Rule
Leave every file you touch slightly cleaner than you found it.
No refactoring sprint needed — just small, continuous improvement.

**Micro-improvements to apply while in a file:**
- Rename one confusing variable
- Extract one repeated expression into a constant
- Remove one stale comment
- Add one missing edge-case check
- Delete one unused import

Over time these compound into a dramatically healthier codebase.

---

## Review Workflow

When asked to **review code**, follow this sequence:

1. **Read entirely** before commenting — understand the full intent first.
2. **Identify violations** — cite the law number and line(s).
3. **Prioritize** — group feedback as Critical (bugs/confusion) → Major (clarity) → Minor (polish).
4. **Provide fixes** — never flag without showing the improved version.
5. **Acknowledge strengths** — note what is already done well.

**Review comment template:**
```
[Law N — Name] Short description of the issue.
  Before: <original code>
  After:  <improved code>
  Reason: <why this matters>
```

---

## Writing Workflow

When asked to **write new code**, follow this sequence:

1. **Name first** — choose all identifiers before writing logic. If a name is hard to find, the abstraction is wrong.
2. **Single purpose** — verify each function passes the "and" test before writing it.
3. **Guard early** — write all validations and error cases first; happy path last.
4. **No placeholders** — never emit `TODO`, magic numbers, or single-letter variables in final output.
5. **Self-review** — before returning, check each of the 12 laws against the generated code.

---

## Quick Reference Cheat Sheet

| # | Law | Failure Signal |
|---|-----|----------------|
| 1 | Least Surprise | Hidden side effects, misleading names |
| 2 | Names Are Docs | Abbreviations, generic names, wrong type hints |
| 3 | One Purpose | "and" in description, long functions |
| 4 | Comments = Why | Restating code, undated TODOs |
| 5 | Shallow Nesting | > 3 levels deep, else-chains |
| 6 | Short Functions | > 30 lines, scrolling required |
| 7 | Consistency | Mixed styles in same file/project |
| 8 | No Magic Values | Inline numbers/strings without names |
| 9 | Happy Path First | Successful flow buried in else blocks |
| 10 | Delete Dead Code | Commented-out blocks, unused vars |
| 11 | Express Intent | Manual loops where filter/map fits |
| 12 | Boy Scout Rule | File left messier than found |
