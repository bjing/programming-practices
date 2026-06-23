---
title: Algebraic Data Types
layout: default
---

# Algebraic Data Types

## A compiler that argues back

*Making illegal states unrepresentable*

---

## What we're NOT doing today

- Rewriting everything in Haskell
- Abandoning OOP
- Learning category theory

---

## What we ARE doing

Picking up **one new tool** that:

- Eliminates a whole class of runtime bugs
- Makes intent clearer at the call site
- You can use in Java 17+ and TypeScript **today**

---

## You already know this

`Optional<T>` is an ADT.

```java
Optional<User> user = findUser(id);
```

It's either:

- `Optional.of(user)` — has a value
- `Optional.empty()` — has no value

You can't accidentally treat an empty as a present.
**That's the idea. We're just generalising it.**

---

## The problem — spot the bug

```java
public WorkingPaper resolveWorkingPaperForComment(CommentPojo pojo) {
  if (pojo.getTargetType() == TargetType.WP_ID) {
    return workingPaperRepository
        .findById(Long.parseLong(pojo.getTargetId()))
        .orElseThrow(...);
  }
  if (pojo.getTargetType() == TargetType.ACCOUNT_ID) {
    return workingPaperDetailRepository
        .findByAccountIdAndWppId(pojo.getTargetId(), pojo.getWorkingPaperPackId())
        .map(WorkingPaperDetail::getWorkingPaper)
        .orElseThrow(...);
  }
  throw new IllegalStateException(
      "Cannot resolve WorkingPaper for targetType " + pojo.getTargetType());
}
```

---

## What happens when someone adds `DATASHEET_FIELD_ID`?

```java
public enum TargetType {
  WP_ID,
  ACCOUNT_ID,
  WPP_ID,
  DATASHEET_FIELD_ID   // ← new
}
```

- Code compiles
- All existing tests pass
- Falls through to `IllegalStateException` at runtime
- Production incident

---

## A second problem in the same code

```java
pojo.getTargetId()  // String — means different things depending on targetType
```

When `targetType == WP_ID`, it's a numeric ID.
When `targetType == ACCOUNT_ID`, it's an account string.

**The type system knows nothing about this.**
A string is a string.

---

## The concept: sum types

We have two kinds of types:

| Name             | Meaning                             | Java keyword       |
|------------------|-------------------------------------|--------------------|
| **Product type** | has *all* of several fields (`AND`) | `record`, `class`  |
| **Sum type**     | is *one of* several variants (`OR`) | `sealed interface` |

Together: **Algebraic Data Types**

---

## Sum type — the syntax

```java
public sealed interface CommentTarget
    permits CommentTarget.ForWP, CommentTarget.ForAccount {

  record ForWP(long workingPaperId) implements CommentTarget {
  }

  record ForAccount(String accountId, long wppId) implements CommentTarget {
  }
}
```

- `sealed` — only listed types can implement this
- `permits` — the complete list of variants
- `record` — lightweight, immutable, carries the right data

---

## Pattern matching — exhaustiveness enforced

```java
WorkingPaper resolve(CommentTarget target) {
  return switch (target) {
    case ForWP(var id) -> workingPaperRepository.findById(id).orElseThrow();

    case ForAccount(var accountId, var wppId) -> workingPaperDetailRepository
        .findByAccountIdAndWppId(accountId, wppId)
        .map(WorkingPaperDetail::getWorkingPaper)
        .orElseThrow();
  };
}
```

No `default`. No runtime exception.
**Add a new variant → doesn't compile until every switch handles it.**

---

## Already in our codebase — simple example

```java
// workingPaperWeb/.../datasheet/DatasheetValueKey.java

public sealed interface DatasheetValueKey
    permits DatasheetValueKey.ByFieldType, DatasheetValueKey.ByReportCode {

  record ByFieldType(DatasheetFieldType type) implements DatasheetValueKey {
  }

  record ByReportCode(String code) implements DatasheetValueKey {
  }
}
```

A key is *either* a field type *or* a report code.
Never both. Never neither. The type says so.

---

## Already in our codebase — with shared interface

```java
// workingPaperWeb/.../eto/DocumentFunctionResponseEto.java

public sealed interface DocumentFunctionResponseEto
    permits BalanceCheckSuccess, Failure {

  String requestId();   // ← available on ALL variants

  String bglGRID();

  record BalanceCheckSuccess(
      String requestId, String bglGRID,
      DocumentFunctionPayloadEto.BalanceCheckSuccess payload
  ) implements DocumentFunctionResponseEto {
  }

  record Failure(
      String requestId, String bglGRID,
      DocumentFunctionType type,
      DocumentFunctionPayloadEto.Failure payload   // ← only on Failure
  ) implements DocumentFunctionResponseEto {
  }
}
```

---

## Compare: the old way

```java
// LoanItemType.java

DIVIDENDS_DECLARED(LoanType.DIV7A, "Dividends Declared"),

CLOSING(null,"Closing Balance"),   // ← null = "common to all loan types"

private final LoanType loanType;    // null or not — you have to know

public boolean isValidForLoanType(LoanType loanType) {
  return this.loanType == null     // ← null check every time
      || this.loanType == loanType;
}
```

`loanType == null` carries meaning. That meaning isn't in the type.

---

## Compare: the new way

```java
sealed interface LoanItemType permits LoanItemType.Common, LoanItemType.TypeSpecific {

  String displayName();

  record Common(String displayName)
      implements LoanItemType {
  }

  record TypeSpecific(LoanType loanType, String displayName)
      implements LoanItemType {
  }
}
```

`TypeSpecific` *always* has a `loanType`. No null check.
`Common` *never* has one. Can't even ask.

---

## You're already doing this in TypeScript

```typescript
type DocumentFunctionResponse =
  | { kind: 'success'; requestId: string; payload: BalanceCheckPayload }
  | { kind: 'failure'; requestId: string; type: string; error: string }

function handle(response: DocumentFunctionResponse) {
  switch (response.kind) {
    case 'success':
      return processPayload(response.payload);  // payload typed here
    case 'failure':
      return handleError(response.error);
    // TypeScript errors if you add a variant and miss it here
  }
}
```

Same idea. Different syntax. You've probably written this already.

---

## Live refactor — let's do it together

**Starting point:** `TargetType` enum + `CommentService`

**Goal:**

1. Replace the enum discriminant + string data with a sealed type
2. Replace the if/else chain with a switch expression
3. Add a new variant and watch the compiler do the work

---

## "Why not just use an enum?"

Good question. Both give you exhaustiveness checking in a switch.
The difference is **what each variant can carry**.

---

## Enums: every constant shares the same fields

```java
// All constants must have the same field structure
enum LoanItemType {
  COMMON(null, "Closing Balance"),
  DIV7A_SPECIFIC(LoanType.DIV7A, "Dividends Declared");

  final LoanType loanType;  // null for COMMON — meaning hidden in nullability
  final String displayName;
}
```

You can't give `DIV7A_SPECIFIC` fields that `COMMON` doesn't have.
The only escape valve is `null`.

---

## Sealed interfaces: each variant has its own shape

```java
sealed interface LoanItemType permits LoanItemType.Common, LoanItemType.TypeSpecific {
  record Common(String displayName) implements LoanItemType {
  }

  record TypeSpecific(LoanType loanType, String displayName) implements LoanItemType {
  }
}
```

`TypeSpecific` has `loanType`. `Common` doesn't. **Can't even ask.**
No null. No convention. Enforced by the compiler.

---

## Enums: singletons. Records: instances.

```java
// Enum constants — there is exactly ONE FAILURE in the JVM
Result.FAILURE

// Sealed records — each call creates a new instance with its own data
new Result.

Failure("network timeout",504)
new Result.

Failure("not authorised",401)
```

Enums can't model "a failure *with a specific message*".
They can only model "the concept of failure".

---

## Sealed interfaces: generics work

```java
// This is impossible with enums:
enum Option<T> { ...}  // ← won't compile

// Works fine with sealed interfaces:
sealed interface Option<T> permits Option.Some, Option.None {
  record Some<T>(T value) implements Option<T> {
  }

  record None<T>() implements Option<T> {
  }
}
```

---

## Pattern matching: deconstruction

```java
// Sealed record — pull out fields inline, compiler knows their types
case Result.Failure(var message, var code) ->log.

error("{}: {}",code, message);

// Enum — no deconstruction, you call getters manually
case FAILURE ->log.

error("{}: {}",result.getCode(),result.

getMessage());
```

---

## Rule of thumb: enum vs sealed interface

| Use **enum** when                  | Use **sealed interface** when            |
|------------------------------------|------------------------------------------|
| Variants are stateless tags        | Variants carry data                      |
| All variants share the same fields | Variants have **different** field shapes |
| You need `EnumSet` / `EnumMap`     | You need generics                        |
| JPA / Jackson serialisation        | You control serialisation                |

`WorkingPaperStatus` → enum. Pure tags, no variant-specific data.
`DocumentFunctionResponseEto` → sealed. `Failure` has fields `Success` doesn't.

---

## When to reach for a sealed type

| Situation                                | Signal                                  |
|------------------------------------------|-----------------------------------------|
| Enum with nullable fields                | Different variants need different data  |
| `if/else` or `switch` on a type field    | You're manually discriminating variants |
| `instanceof` chains                      | Same thing                              |
| Fields only valid in some states         | Illegal states are representable        |
| Abstract class with 2–3 known subclasses | Closed hierarchy — seal it              |

---

## When NOT to bother

- Genuinely open hierarchies (plugins, extensibility points)
- JPA entities — ORM doesn't support sealed types well
- Simple status enums with no associated data

---

## Beyond ADTs: GADTs

### Generalised Algebraic Data Types

ADTs let each variant carry **different data**.
GADTs go further — each constructor can return a **more specific type**.

```
// Normal ADT — all variants produce the same type
sealed interface Expr permits Lit, BoolLit, Add, If {
  record Lit(int value) implements Expr { }
  record BoolLit(boolean value) implements Expr { }
  record Add(Expr left, Expr right) implements Expr { }
  record If(Expr condition, Expr thenBranch, Expr elseBranch) implements Expr { }
}

// GADT — each record pins the type parameter to something specific
sealed interface Expr<A> permits Lit, BoolLit, Add, If {}
record Lit(int value)                          implements Expr<Integer> {}
record BoolLit(boolean value)                  implements Expr<Boolean> {}
record Add(Expr<Integer> l, Expr<Integer> r)   implements Expr<Integer> {}
record If<A>(Expr<Boolean> cond,
             Expr<A> thenBranch,
             Expr<A> elseBranch)               implements Expr<A> {}
```

`If` requires a `Bool` condition. `Add` requires `Int` operands.
**These are type errors at construction time, not runtime.**

---

## GADTs: what they're useful for

The type index on each variant lets the compiler prove correctness
that would otherwise require runtime checks or casts.

**Primary use cases:**

- **Embedded languages** — writing a mini-language inside your program (e.g. a query builder, a rules engine, a formula evaluator)
  where the type system prevents you from expressing invalid programs
- **Typed syntax trees** — representing parsed code or expressions as a tree where each node's type tells the compiler what
  operations are valid on it
- **Proof-carrying data** — encoding invariants like "this list is non-empty" or "this value is in range" directly in the type, so
  you never need to check at runtime

---

## Java does not support GADTs

Java sealed types get you ADTs — per-variant data shapes.
But the compiler **does not propagate type refinements** through pattern matching.

```java
// This does NOT compile:
<A> A eval(Expr<A> expr) {
  return switch (expr) {
    case Lit(var n) -> n;   // A should be Integer here — Java doesn't know
    case Bool(var b) -> b;   // A should be Boolean here — Java doesn't know
  };
}

// You're forced to cast — losing the type safety GADTs provide:
<A> A eval(Expr<A> expr) {
  return (A) switch (expr) { ...};  // unchecked cast
}
```

Root cause: Java erases generic types at runtime and does not support
higher-kinded types. Both are required for GADTs.

---

## Three things to take away

1. **Sealed interfaces + records** are the Java tool. Available from Java 17.

2. **The compiler enforces exhaustiveness.** New variant = compile error at every unhandled switch.

3. **Make illegal states unrepresentable.**
   If a field is only valid in one variant, put it only in that variant's record.

---

## Questions?

Look for the pattern:
> *"This field is only meaningful when that other field equals X."*

That's where a sealed type pays off.

---

*Slides: `docs/slides-adts-for-oop-teams.md`*
*Full notes: `docs/presentation-adts-for-oop-teams.md`*
