---
name: never-nester
description: >
  Refactor deeply nested code by applying "Never Nester" principles: early returns,
  guard clauses, function extraction, and condition inversion. Use this skill whenever
  the user asks to reduce nesting, flatten code, fix arrow-shaped code, apply guard
  clauses, use early returns, refactor deeply nested if/else or try/catch blocks, or
  when the user pastes code that has 3+ levels of indentation and asks to clean it up.
  Also trigger when users mention the "Never Nester" concept by name, or ask to make
  code more readable by reducing indentation. Works across all languages.
---

# Never Nester

Eliminate deep nesting to produce flat, readable, linear code. Inspired by
CodeAesthetic's "Why You Shouldn't Nest Your Code" principle.

## The Core Problem

Deeply nested code (the "arrow anti-pattern") is hard to read because:
- The happy path is buried inside layers of conditions
- Each level requires holding more context in your head
- Error cases and edge cases are scattered throughout
- The shape of the code obscures intent

## The Four Techniques

### 1. Early Return / Guard Clause
Invert the condition and return (or throw) immediately. Preconditions are checked
up front; the happy path runs at the base indentation level.

**Before:**
```python
def process(user, data):
    if user:
        if user.is_active:
            return save(data)
```
**After:**
```python
def process(user, data):
    if not user: return None
    if not user.is_active: return None
    return save(data)
```

### 2. Invert the Condition
When the main logic lives in the if-branch and the else is short, flip them.

**Before:**
```python
if is_valid(x):
    # 20 lines of logic
    return result
else:
    return error
```
**After:**
```python
if not is_valid(x):
    return error
# 20 lines of logic — now at top level
return result
```

### 3. Extract into a Function
When a nested block has a clear responsibility, pull it into a named function.
The name documents intent; the function body can be read independently.

**Before:**
```python
def handle_request(req):
    if req.method == "POST":
        if req.user.is_authenticated:
            # 20 lines of processing...
```
**After:**
```python
def handle_request(req):
    if req.method != "POST": return reject()
    if not req.user.is_authenticated: return unauthorized()
    return process_post(req)

def process_post(req):
    # 20 lines — flat and focused
```

### 4. Merge Conditions
Collapse consecutive nested conditions into a single compound check.

**Before:**
```python
if a:
    if b:
        do_thing()
```
**After:**
```python
if a and b:
    do_thing()
```

## How to Apply This Skill

When refactoring user code:

1. **Scan for hotspots** — locate the deepest indentation first.
2. **Pick the right technique** — early return, invert, extract, or merge.
3. **Annotate briefly** — tell the user which technique you're applying and why.
4. **Preserve behavior exactly** — never change logic while flattening structure.
5. **Show before and after** — so the user can compare and learn.
6. **Name extracted functions well** — describe *what* the block does, not *how*.

## Language Notes

- **JS/TS, Java, C#**: Use bare `return;` for early exit in void functions.
- **Exceptions**: Throwing is fine instead of returning when that fits the codebase.
- **Ternaries**: Never *nest* ternaries — that trades one readability problem for another.
- **Switch/match**: Suggest these when they eliminate long if-else chains cleanly.
- **Loops**: `continue` (skip iteration) and `break` (exit loop) are the loop
  equivalents of early return — use them too.

## Output Format

```
**Before** (issue: X levels of nesting)
[original code]

**After** (techniques: guard clauses, extraction)
[refactored code]
```

List every technique used. Keep explanations to one sentence each.
