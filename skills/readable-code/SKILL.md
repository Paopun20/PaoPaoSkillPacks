---
name: readable-code
description: >
  Apply this skill when the user asks to write, review, refactor, or critique code for clarity and maintainability.
  Triggers include: "make this readable", "refactor this", "clean up my code", "code review", "is this clean code",
  "improve naming", "simplify this function", "review for best practices", "add documentation", "write docstrings",
  "this code smells", "hard to test", "improve testability", or any request to write new code where quality and
  clarity are implied. Also apply when the user pastes code and asks "what do you think?" or "how can I improve this?".
  Do NOT use for pure performance optimization, security audits, architecture design, or tasks where readability
  is explicitly out of scope.
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

# Compliant
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

## Code Smell Detection

Smells are symptoms of deeper problems. Always name the smell, explain the risk, and show the fix.

### Smell Catalogue

| Smell | Signal | Refactor |
|-------|--------|----------|
| **Long Method** | > 30 lines, scrolling required | Extract sub-functions |
| **God Class** | > 400 lines, does everything | Split by responsibility |
| **Feature Envy** | Method uses another class's data more than its own | Move method closer to the data |
| **Data Clump** | Same 3+ params appear together repeatedly | Introduce a parameter object |
| **Primitive Obsession** | `int userId`, `string status` everywhere | Wrap in domain types (`UserId`, `OrderStatus`) |
| **Shotgun Surgery** | One change requires edits in 10 files | Consolidate into one module |
| **Divergent Change** | One class changes for unrelated reasons | Split into focused classes |
| **Duplicate Code** | Copy-paste with minor variations | Extract shared abstraction |
| **Switch Statement** | Large `switch`/`if-elif` on a type field | Replace with polymorphism or a strategy map |
| **Temporary Field** | Instance variable only set in one path | Move to the method or a value object |
| **Message Chains** | `a.getB().getC().getD()` | Apply Law of Demeter; add a delegation method |
| **Middle Man** | Class delegates everything, adds nothing | Inline it or remove the layer |

### Detection Workflow
1. Scan for the signal patterns above before reading logic.
2. Name every smell found with the label from the table.
3. Prioritize by blast radius: smells in core domain > utility helpers.
4. Provide a concrete before/after refactor for each.

---

## Documentation Standards

Good documentation lives at three levels: **module**, **function**, and **inline**.

### Module / File Header
Every non-trivial file should open with a one-paragraph summary:
- What this module is responsible for
- What it is NOT responsible for (boundaries)
- Key dependencies or entry points

```python
"""
order_processor.py
------------------
Handles the lifecycle of an order from validation through fulfilment.
Does NOT handle payment processing (see payment_gateway.py) or
inventory management (see inventory_service.py).

Entry point: process_order(order: Order) -> Receipt
"""
```

### Function / Method Docstrings

Use the language's idiomatic format. Include:
- **One-line summary** (imperative mood: "Calculate", not "Calculates")
- **Args** — name, type, meaning; note which are optional
- **Returns** — type and meaning
- **Raises** — which exceptions and under what conditions
- **Example** — for non-obvious usage

**Python (Google style):**
```python
def calculate_discount(price: float, tier: str) -> float:
    """Calculate the discounted price for a customer tier.

    Args:
        price: Original price in USD. Must be non-negative.
        tier: Customer tier. One of 'standard', 'premium', 'vip'.

    Returns:
        Discounted price in USD.

    Raises:
        ValueError: If tier is unrecognized or price is negative.

    Example:
        >>> calculate_discount(100.0, 'premium')
        85.0
    """
```

**JavaScript / TypeScript (JSDoc):**
```javascript
/**
 * Calculates the discounted price for a customer tier.
 *
 * @param {number} price - Original price in USD. Must be non-negative.
 * @param {'standard'|'premium'|'vip'} tier - Customer tier.
 * @returns {number} Discounted price in USD.
 * @throws {Error} If tier is unrecognized or price is negative.
 *
 * @example
 * calculateDiscount(100, 'premium'); // 85
 */
```

### What NOT to Document
- Re-stating the function name (`// Gets the user` above `getUser()`)
- Type information already in the signature (if types are enforced)
- Implementation details that will change — document the contract, not the mechanism

### README / Module-Level Docs
For any exported module or public API, include:
1. Purpose in one sentence
2. Installation / import instructions
3. Quickstart example (runnable code)
4. Link to full API reference

---

## Testing & Testability

Readable code and testable code are the same thing. If code is hard to test, it is hard to read.

### Testability Laws

**T1 — Pure Functions Are Free Tests**
A function with no side effects can be tested with a single assertion. Maximize pure functions.

```python
# Hard to test — depends on global state and I/O
def get_active_users():
    return [u for u in db.query_all() if u.active]

# Easy to test — pure transformation
def filter_active(users: list[User]) -> list[User]:
    return [u for u in users if u.active]
```

**T2 — Inject Dependencies, Don't Hardcode Them**
Any external resource (DB, HTTP client, clock, filesystem) must be injectable so tests can substitute fakes.

```python
# Hard to test — hardcoded dependency
class ReportService:
    def generate(self):
        data = Database().query("SELECT ...")  # can't swap in test

# Testable — injected dependency
class ReportService:
    def __init__(self, db: Database):
        self.db = db

    def generate(self):
        data = self.db.query("SELECT ...")
```

**T3 — One Assertion Per Test Concept**
Each test should verify one behaviour. Multiple assertions testing the same concept are fine; testing
unrelated concepts in one test hides failures.

```python
# Violates — tests two unrelated things
def test_order():
    order = create_order()
    assert order.total == 100      # price calculation
    assert order.status == "open"  # initial state

# Compliant — split by concept
def test_order_total_is_sum_of_items():
    order = create_order(items=[Item(50), Item(50)])
    assert order.total == 100

def test_new_order_status_is_open():
    order = create_order()
    assert order.status == "open"
```

**T4 — Test Names Are Specifications**
A failing test name should tell you exactly what broke without reading the body.

Format: `test_<unit>_<condition>_<expected_outcome>`

```python
# Bad — tells you nothing
def test_discount():

# Good — reads as a specification
def test_calculate_discount_for_vip_tier_applies_30_percent():
```

**T5 — Arrange / Act / Assert (AAA)**
Every test has three clearly separated phases. Add blank lines between them.

```python
def test_apply_coupon_reduces_order_total():
    # Arrange
    order = Order(total=100.0)
    coupon = Coupon(discount=0.10)

    # Act
    order.apply_coupon(coupon)

    # Assert
    assert order.total == 90.0
```

### Code Review for Testability
When reviewing, flag:
- Classes that instantiate their own dependencies (violates T2)
- Functions > 30 lines (hard to unit test in isolation)
- Global state mutations (hidden test ordering dependencies)
- Non-deterministic code (calls to `datetime.now()`, `random`, `uuid` without injection)

---

## Language-Specific Rules

Apply the universal laws above, then layer on these language idioms.

### Python
- Use type hints on all public function signatures (`def fn(x: int) -> str`)
- Prefer dataclasses or Pydantic models over raw dicts for structured data
- Use `pathlib.Path` over `os.path` string manipulation
- Prefer f-strings over `.format()` or `%` formatting
- Use `with` statements for any resource that must be closed
- Raise specific exceptions (`ValueError`, `TypeError`) not bare `Exception`
- Keep `__init__.py` files minimal — no business logic
- Use `_private` prefix (single underscore) for internal helpers; `__dunder__` only for Python protocols

### JavaScript / TypeScript
- **Always use TypeScript** for any project > 1 file; avoid `any`
- Prefer `const` over `let`; never use `var`
- Use named exports over default exports for easier refactoring
- Prefer `async/await` over raw `.then()` chains
- Destructure objects and arrays at the top of functions to name values
- Use optional chaining (`?.`) and nullish coalescing (`??`) over manual null checks
- Avoid mutation: use spread (`{...obj}`) or `Array.from()` rather than `.push()` / direct assignment
- File names: `kebab-case.ts` for modules, `PascalCase.tsx` for React components

### Go
- Error handling: always check and return errors; never `_` discard them
- Keep interfaces small — prefer single-method interfaces
- Use `context.Context` as the first parameter of any I/O function
- Avoid naked returns
- Group related constants in `iota` blocks with a named type
- Table-driven tests are idiomatic; use them for any function with multiple input cases

### General (all languages)
- Avoid deeply nested ternaries; use early returns or named variables instead
- Do not abbreviate just to save keystrokes — `calculateMonthlyRevenue` beats `calcMonRev`
- Prefer explicit over implicit: spell out what the code does even if a clever one-liner exists

---

## Review Workflow

When asked to **review code**, follow this sequence:

1. **Read entirely** before commenting — understand the full intent first.
2. **Detect smells** — scan the Smell Catalogue before diving into logic.
3. **Identify law violations** — cite the law number and line(s).
4. **Check documentation** — are public functions and modules documented to the standard above?
5. **Check testability** — flag T1–T5 violations.
6. **Apply language rules** — add any language-specific findings.
7. **Prioritize** — group feedback as Critical (bugs/confusion) → Major (clarity) → Minor (polish).
8. **Provide fixes** — never flag without showing the improved version.
9. **Acknowledge strengths** — note what is already done well.

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
4. **Document as you go** — add docstrings to all public functions before the body.
5. **Design for testability** — inject dependencies; write pure functions where possible.
6. **No placeholders** — never emit `TODO`, magic numbers, or single-letter variables in final output.
7. **Self-review** — before returning, run the full Review Workflow above against the generated code.

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

| Smell | Quick Signal |
|-------|-------------|
| Long Method | > 30 lines |
| God Class | > 400 lines, mixed concerns |
| Feature Envy | Reaches into another class's data |
| Data Clump | Same params travel together always |
| Primitive Obsession | Raw `int`/`string` for domain concepts |
| Duplicate Code | Copy-paste with variations |
| Switch on Type | Big if/switch on a type field |
| Message Chain | `a.b().c().d()` |

| Testability | Quick Signal |
|-------------|-------------|
| T1 Pure Functions | Side effects in logic functions |
| T2 Inject Deps | `new Database()` inside a class |
| T3 One Concept | Multiple unrelated asserts in one test |
| T4 Test Names | `test_thing()` with no context |
| T5 AAA Structure | No separation of arrange/act/assert |
