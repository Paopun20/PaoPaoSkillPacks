---
name: clean-code-gamedev
description: >
  Apply Hugo's (Code Monkey) 10 clean-code principles from "How to Write High Quality Code that doesn't fall apart".
  Use this skill whenever the user asks to write, review, or refactor game code (Unity, Godot, Unreal, or any game
  engine), OR when they mention any of these topics: decoupling, information hiding, composition vs inheritance,
  one source of truth, magic numbers, single responsibility, naming conventions, or code architecture.
  Also trigger when the user asks "is this clean?", "how do I structure this system?", "my code is hard to change",
  "refactor this game script", "how do I avoid bugs in my game?", "Unity code review", or pastes a game script and
  asks for feedback. Complements the readable-code skill — use this one when game architecture, Unity patterns,
  or Hugo's specific principles are relevant.
---

# Clean Code for Game Dev

_Based on Hugo's "How to Write High Quality Code that doesn't fall apart" (25 years of game dev principles)_

> "Programmers spend roughly 10× more time reading code than writing it — readability is the ultimate priority."

Code is clean when it is **simple, straightforward, easy to understand, and easy to modify**.
Apply these 10 principles both when generating new game code and when reviewing existing scripts.

---

## Principle 1 — Naming Is Architecture, Not Cosmetics

Names communicate intent. A reader should understand what a variable holds or what a function does _without_ reading the body.

**Rules:**

- Be verbose. `playerCurrentHealthPoints` beats `hp` or `phcp`
- No abbreviations, no acronyms, no single-letter names (except `i/j/k` loop iterators)
- Name length ≥ meaning required. 20-character names are fine and expected

**Anti-patterns to flag:**

```csharp
// Bad — reader must guess
float h;
void UptHP(float v) { }

// Good — intent is obvious
float currentHealthPoints;
void UpdateHealthPoints(float damageAmount) { }
```

**The "And" test:** if you need "and" to describe a function, split it.

```csharp
// Violates — two responsibilities
void MovePlayerAndPlayFootsteps() { }

// Compliant
void MovePlayer() { }
void PlayFootstepSound() { }
```

---

## Principle 2 — Decoupling

Each system should be able to change without forcing changes in other systems.
The canonical game-dev split: **core game logic ↔ visuals ↔ audio**.

```csharp
// Tightly coupled — logic knows about visuals
public class PlayerHealth : MonoBehaviour
{
    public void TakeDamage(float amount)
    {
        currentHealth -= amount;
        GetComponent<Animator>().SetTrigger("HitReaction"); // ← logic reaching into visuals
        AudioSource.PlayClipAtPoint(hurtSound, transform.position); // ← and audio
    }
}

// Decoupled — game logic fires events; visuals/audio listen
public class PlayerHealth : MonoBehaviour
{
    public event Action<float> OnDamageTaken;

    public void TakeDamage(float amount)
    {
        currentHealth -= amount;
        OnDamageTaken?.Invoke(currentHealth);
    }
}

// Separate visual responder — can change independently
public class PlayerAnimator : MonoBehaviour
{
    [SerializeField] private PlayerHealth health;
    private void OnEnable() => health.OnDamageTaken += PlayHitAnimation;
    private void PlayHitAnimation(float remaining) { /* ... */ }
}
```

**Decoupling checklist:**

- [ ] Does this class import or directly reference another system it shouldn't know about?
- [ ] Would changing the renderer or audio engine require touching game logic?
- [ ] Could I swap this system for a mock in tests without changing callers?

---

## Principle 3 — Single Responsibility Principle (SRP)

Every class and function has **exactly one reason to change**.

**Signal:** if describing a class requires the word "and", it has too many responsibilities.

```csharp
// Violates — Enemy does AI, rendering, and saves
public class Enemy : MonoBehaviour
{
    public void UpdateAI() { }
    public void DrawHealthBar() { }
    public EnemyData ToSaveData() { }
}

// Compliant — each class owns one concern
public class EnemyAI : MonoBehaviour { void Update() { } }
public class EnemyHealthUI : MonoBehaviour { void Update() { } }
public class EnemySerializer : MonoBehaviour { EnemyData Serialize() { } }
```

**Function SRP — the "and" test:**

| Name contains       | Action                                                      |
| ------------------- | ----------------------------------------------------------- |
| `MoveAndShoot()`    | Split into `Move()` + `Shoot()`                             |
| `ValidateAndSave()` | Split into `Validate()` + `Save()`                          |
| `LoadOrCreate()`    | Acceptable — it's branching logic, not two responsibilities |

---

## Principle 4 — Information Hiding

Expose the **minimum surface area** possible. Every public field is a potential bug entry point.

**Rules:**

- Default to `private` for all fields
- Only promote to `public` or `[SerializeField]` when a deliberate design decision has been made
- Properties (getters/setters) let you add validation, logging, or access control later without changing callers

```csharp
// Violates — entire state is public, anyone can corrupt it
public class PlayerInventory : MonoBehaviour
{
    public int coins;
    public List<Item> items;
}

// Compliant — controlled access
public class PlayerInventory : MonoBehaviour
{
    [SerializeField] private int coins;
    private List<Item> items = new();

    public int Coins => coins; // read-only outside

    public bool TryAddCoins(int amount)
    {
        if (amount <= 0) return false;
        coins += amount;
        return true;
    }
}
```

**Why this matters:** fewer public fields = fewer places a bug can originate = faster debugging.

---

## Principle 5 — One Source of Truth

**Never store the same piece of state in more than one place.**

The classic game-dev trap: `PlayerHealth` tracks HP, _and_ `UIManager` caches it, _and_ `SaveSystem` stores a copy.
When they diverge → synchronisation bugs.

```csharp
// Violates — three copies of the same truth
public class PlayerHealth : MonoBehaviour { public float hp; }
public class UIManager : MonoBehaviour    { private float cachedHp; } // <- duplicate!
public class SaveSystem : MonoBehaviour   { private float lastSavedHp; } // <- duplicate!

// Compliant — one owner, everyone else reads from it
public class PlayerHealth : MonoBehaviour
{
    [SerializeField] private float currentHp;
    public float CurrentHp => currentHp;
}

public class UIManager : MonoBehaviour
{
    [SerializeField] private PlayerHealth playerHealth;
    void Update() => hpText.text = playerHealth.CurrentHp.ToString(); // always fresh
}
```

**Rule:** if two variables would need to stay in sync, that's one variable too many.

---

## Principle 6 — Self-Documenting Code; Comments Explain _Why_

Good architecture should make _what_ the code does obvious. Comments explain the _why_ behind a non-obvious decision.

**Valid comment types:**

1. Design rationale: `// Using object pool here — Instantiate causes GC spikes on mobile`
2. Known gotcha: `// Physics.Raycast returns false during FixedUpdate on the first frame`
3. External reference: `// Formula from GDC talk "Math for Game Programmers" (2019)`

**Anti-patterns to delete:**

```csharp
health -= damage;   // subtract damage from health   ← restates code
// TODO: fix this                                     ← undated, unowned
/* old movement code */                               ← dead code, delete it
```

**Rule:** if removing a comment makes the code harder to understand, the code needs better names — not more comments.

---

## Principle 7 — No Magic Numbers or Strings

Naked literals hide intent and create invisible coupling.

```csharp
// Violates — what is 0.85? Why "S button"?
if (difficulty == 3) speed *= 0.85f;
Transform btn = transform.Find("S button");

// Compliant — intent is explicit, typos are compile errors
private const int HardDifficultyLevel = 3;
private const float HardDifficultySpeedMultiplier = 0.85f;
[SerializeField] private Transform submitButton; // Unity reference, not string lookup

if (difficulty == HardDifficultyLevel)
    speed *= HardDifficultySpeedMultiplier;
```

**String identifiers are especially dangerous** — `transform.Find("S button")` will silently return `null` if the name changes. Prefer serialized references, enums, or ScriptableObject references.

**Extract when:** a literal appears more than once, OR requires thought to understand its meaning.

---

## Principle 8 — Composition over Inheritance

Deep inheritance hierarchies force subclasses to carry logic they don't need. Prefer **interfaces and small components**.

```csharp
// Inheritance trap — Dragon needs Fly but not Swim; Shark needs Swim but not Fly
// Both inherit from Animal which tries to serve everyone
class Animal { void Move() { } }
class Dragon : Animal { void Fly() { } void Breathe Fire() { } }
class FlyingShark : Animal, Dragon ??? // C# won't allow this, and it makes no sense

// Composition — mix in exactly what's needed
public interface IFlyable { void Fly(); }
public interface ISwimmable { void Swim(); }

public class Dragon : MonoBehaviour, IFlyable { public void Fly() { } }
public class Shark  : MonoBehaviour, ISwimmable { public void Swim() { } }
public class FlyingShark : MonoBehaviour, IFlyable, ISwimmable
{
    public void Fly() { }
    public void Swim() { }
}
```

**Unity native pattern:** Unity's own `MonoBehaviour` + `Component` system _is_ composition. Lean into it.

- One `MonoBehaviour` per concern (health, movement, input, inventory)
- `GetComponent<T>()` at `Awake`, cached to a field — never in `Update`
- Avoid deep `MonoBehaviour` inheritance chains

---

## Principle 9 — Refactoring Is a Constant Process

Clean code is not written — it is **rewritten**. The first pass is a rough draft.

**The refactoring loop:**

1. Make it work (prototype / first draft)
2. Extract functions — if a block deserves a comment, it deserves a name
3. Rename until names are unambiguous
4. Apply SRP — split any class or function with more than one reason to change
5. Remove duplication — two similar blocks → one abstraction
6. Verify decoupling — could each system be tested in isolation?

**Mindset:** do not compare your first draft to a polished, production repository. Clean architecture is always reached over multiple passes. Schedule refactoring time the same way you schedule feature work.

---

## Principle 10 — Comments Are a Last Resort

If a piece of code requires a comment to explain _what_ it does, that code should be refactored instead.

**Refactor-first ladder:**

1. Extract to a named function → `CalculateFallDamage(velocity, groundHardness)`
2. Rename variables to reflect intent
3. Use intermediate named variables to document a complex expression
4. _Only if all else fails:_ add a comment explaining the remaining non-obvious part

```csharp
// Before — comment compensating for bad code
// Calculate fall damage based on velocity and surface
float d = v * v * gh * 0.5f;

// After — code explains itself
float fallDamage = CalculateFallDamage(verticalVelocity, groundHardness);

float CalculateFallDamage(float velocity, float groundHardness) =>
    velocity * velocity * groundHardness * FallDamageCoefficient;
```

---

## Review Workflow

When asked to review game code, follow this sequence:

1. **Read entirely** before commenting — understand the full intent first
2. **Check naming** (P1) — any abbreviations, acronyms, single letters?
3. **Check coupling** (P2) — does game logic touch visuals or audio directly?
4. **Check SRP** (P3) — does any class or function do more than one thing?
5. **Check visibility** (P4) — are fields public when they could be private?
6. **Check state duplication** (P5) — is the same value stored in multiple places?
7. **Check literals** (P7) — any magic numbers or string identifiers?
8. **Check inheritance** (P8) — deep hierarchy where composition would serve better?
9. **Prioritize** — group findings as Critical (bugs/confusion) → Major (architecture) → Minor (polish)
10. **Provide fixes** — never flag without showing the improved version
11. **Acknowledge strengths** — note what is already done well

**Review comment format:**

```
[P{N} — Principle Name] Short description of the issue.
  Before: <original code>
  After:  <improved code>
  Reason: <why this matters in practice>
```

---

## Writing Workflow

When asked to write new game code, follow this sequence:

1. **Name first** — choose all identifiers before writing logic; if a name is hard to find, the abstraction is wrong
2. **Single purpose** — verify each class and function passes the "and" test
3. **Default private** — start all fields as `private`; promote only with justification
4. **One truth** — for each piece of state, decide exactly one owner
5. **No literals** — extract all non-trivial numbers and strings to named constants
6. **Decouple** — wire systems through events or interfaces, not direct references across layers
7. **Self-review** — run the Review Workflow above before returning the code

---

## Quick Reference

| Principle       | Signal it's violated                                        |
| --------------- | ----------------------------------------------------------- |
| P1 Naming       | Abbreviations, single letters, vague names (`data`, `temp`) |
| P2 Decoupling   | Logic calls `GetComponent<Animator>()` or audio directly    |
| P3 SRP          | "and" in class/function description; file > 300 lines       |
| P4 Info Hiding  | Public fields that could be private                         |
| P5 One Truth    | Same value tracked in 2+ scripts                            |
| P6 Self-Doc     | Comment restating what the code does                        |
| P7 No Literals  | Inline numbers or `transform.Find("string")`                |
| P8 Composition  | Deep inheritance; subclass carries unused logic             |
| P9 Refactoring  | "I'll clean this up later" shipped to main                  |
| P10 Last Resort | Comment compensating for a bad abstraction                  |
