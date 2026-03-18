---
title: Enums in Java 21+ and TypeScript
description: Comparing enums, exhaustive pattern matching, type-safe lookup tables, and when TypeScript unions are a better model.
layout: default
---

# Enums in Java 21+ and TypeScript

Enums are useful when a value must be one of a fixed set of named options.

They work well for:

- status values
- modes
- categories
- permissions
- workflow steps

Java and TypeScript both support enums, but they push you toward different design styles.

Java enums are rich runtime objects with strong language-level support.
TypeScript enums exist at runtime too, but many TypeScript codebases prefer string literal unions because they compose better with the type system.

## Java 21+ Enums

Java enums are full classes with a fixed set of instances.

```java
public enum PaymentStatus {
    PENDING,
    PAID,
    FAILED
}
```

They can also contain fields, constructors, and methods:

```java
public enum PaymentStatus {
    PENDING("Awaiting payment"),
    PAID("Payment received"),
    FAILED("Payment failed");

    private final String label;

    PaymentStatus(String label) {
        this.label = label;
    }

    public String label() {
        return label;
    }
}
```

This makes Java enums useful when each enum constant carries behavior or metadata.

## TypeScript Enums

TypeScript supports `enum`, including string enums:

```ts
enum PaymentStatus {
  PENDING = "PENDING",
  PAID = "PAID",
  FAILED = "FAILED",
}
```

This gives you a named runtime object and a type derived from it.

That can be useful when:

- you want a real runtime value namespace
- you need compatibility with existing enum-based APIs
- you want a closed set of named constants

Still, many TypeScript teams prefer string literal unions for domain modelling:

```ts
type PaymentStatus = "PENDING" | "PAID" | "FAILED"
```

That becomes more important once you move beyond simple constant sets.

## Exhaustive Pattern Matching in Java 21+

Java 21 introduced permanent pattern matching for `switch`, which makes branching logic clearer and more type-driven.

For enums, `switch` is already a natural fit:

```java
static String render(PaymentStatus status) {
    return switch (status) {
        case PENDING -> "Waiting for payment";
        case PAID -> "Paid";
        case FAILED -> "Payment failed";
    };
}
```

Because the `switch` covers every enum constant, the compiler can treat it as exhaustive.

That matters for maintainability:

- if a new enum constant is added later
- existing `switch` expressions may stop compiling until the new case is handled

This is one of the strongest benefits of enums in Java: they work very well with compiler-enforced exhaustiveness.

## Exhaustive Matching in TypeScript

TypeScript does not have Java-style native pattern matching syntax, but you can still get exhaustive checking with `switch` and `never`.

With a TypeScript enum:

```ts
enum PaymentStatus {
  PENDING = "PENDING",
  PAID = "PAID",
  FAILED = "FAILED",
}

function assertNever(value: never): never {
  throw new Error(`Unexpected value: ${String(value)}`)
}

function render(status: PaymentStatus): string {
  switch (status) {
    case PaymentStatus.PENDING:
      return "Waiting for payment"
    case PaymentStatus.PAID:
      return "Paid"
    case PaymentStatus.FAILED:
      return "Payment failed"
    default:
      return assertNever(status)
  }
}
```

The `assertNever` pattern forces the compiler to prove that every possible case has been handled.

The same approach works with string literal unions:

```ts
type PaymentStatus = "PENDING" | "PAID" | "FAILED"

function assertNever(value: never): never {
  throw new Error(`Unexpected value: ${String(value)}`)
}

function render(status: PaymentStatus): string {
  switch (status) {
    case "PENDING":
      return "Waiting for payment"
    case "PAID":
      return "Paid"
    case "FAILED":
      return "Payment failed"
    default:
      return assertNever(status)
  }
}
```

The core idea is the same in both Java and TypeScript:

- define a closed set of states
- branch on that set explicitly
- let the compiler catch missing cases

## Type-Safe Enum Lookups in TypeScript with `Record`

A common TypeScript pattern is to use a `Record` when you want a lookup table keyed by an enum.

```ts
enum PaymentStatus {
  PENDING = "PENDING",
  PAID = "PAID",
  FAILED = "FAILED",
}

const paymentStatusLabel: Record<PaymentStatus, string> = {
  [PaymentStatus.PENDING]: "Awaiting payment",
  [PaymentStatus.PAID]: "Payment received",
  [PaymentStatus.FAILED]: "Payment failed",
}
```

This is useful because it is both:

- concise
- exhaustive

If you add a new enum member and forget to update the lookup table, TypeScript will complain.

That is often cleaner than a chain of `if` statements when the operation is just data lookup.

The same works with unions:

```ts
type PaymentStatus = "PENDING" | "PAID" | "FAILED"

const paymentStatusLabel: Record<PaymentStatus, string> = {
  PENDING: "Awaiting payment",
  PAID: "Payment received",
  FAILED: "Payment failed",
}
```

This pattern is especially useful for:

- labels
- colors
- icons
- route mappings
- permissions tables
- configuration by state

## Java Enum Metadata vs TypeScript Union Metadata

In Java, enum metadata is usually defined once on the enum itself:

```java
public enum PaymentStatus {
    PENDING("Awaiting payment"),
    PAID("Payment received"),
    FAILED("Payment failed");

    private final String displayName;

    PaymentStatus(String displayName) {
        this.displayName = displayName;
    }

    public String displayName() {
        return displayName;
    }
}
```

That is a central definition. Every enum constant always has a `displayName`.

In TypeScript, you can enforce a `displayName` on every union member by making each member an object type:

```ts
type PaymentStatus =
  | { type: "PENDING"; displayName: "Awaiting payment" }
  | { type: "PAID"; displayName: "Payment received" }
  | { type: "FAILED"; displayName: "Payment failed" }
```

This guarantees that every valid `PaymentStatus` value includes `displayName`.

If you want to avoid repeating the shared field shape, extract a base type:

```ts
type WithDisplayName = {
  displayName: string
}

type PaymentStatus =
  | (WithDisplayName & { type: "PENDING" })
  | (WithDisplayName & { type: "PAID" })
  | (WithDisplayName & { type: "FAILED" })
```

This still enforces `displayName` on every variant.

However, there is an important modelling difference:

- Java enum metadata is attached once at the enum constant definition
- union object metadata lives on each value instance

So if `displayName` is really metadata about the category itself rather than part of each runtime value, a lookup table is often the closer TypeScript equivalent:

```ts
type PaymentStatus = "PENDING" | "PAID" | "FAILED"

const displayNameByStatus: Record<PaymentStatus, string> = {
  PENDING: "Awaiting payment",
  PAID: "Payment received",
  FAILED: "Payment failed",
}
```

That gives you a single central definition with compile-time completeness, much like Java enum attributes.

## When `Record<EnumType, string>` Is Better Than a `switch`

Use a `Record` when:

- the logic is a pure lookup
- each key maps directly to data
- you want missing entries to fail at compile time

Use a `switch` when:

- each case has branching logic
- cases need multiple statements
- the behavior is more than a simple mapping

Both patterns improve type safety. They solve slightly different problems.

## Why TypeScript Unions Go Beyond Enums

Enums are good at representing one value from a fixed list.

But many real frontend states need more than a label. They need associated data.

For example, this enum:

```ts
enum RequestState {
  IDLE = "IDLE",
  LOADING = "LOADING",
  SUCCESS = "SUCCESS",
  ERROR = "ERROR",
}
```

does not tell you:

- what the successful data is
- what the error message is
- which fields are valid in each state

That usually leads to separate fields and invalid combinations:

```ts
type RequestModel = {
  state: RequestState
  data: User | null
  error: string | null
}
```

Now impossible states are representable:

- `state === Success` with `data === null`
- `state === Loading` with `error !== null`
- `state === Error` with both `data` and `error`

This is where discriminated unions are stronger.

## Going Beyond Enums with Discriminated Unions

Instead of a single enum plus optional fields:

```ts
type RequestState =
  | { type: "idle" }
  | { type: "loading" }
  | { type: "success"; user: User }
  | { type: "error"; message: string }
```

Now each state carries exactly the data it is allowed to have.

This gives you:

- better modelling of real application state
- safer narrowing in `if` and `switch`
- fewer impossible states
- clearer transitions

Example:

```ts
function render(state: RequestState): string {
  switch (state.type) {
    case "idle":
      return "Ready"
    case "loading":
      return "Loading"
    case "success":
      return state.user.name
    case "error":
      return state.message
    default:
      return assertNever(state)
  }
}
```

This is more expressive than enums because the type system understands both:

- which variant you are in
- which data belongs to that variant

## Practical Guidance

In Java 21+:

- use enums when you have a closed set of named values
- use `switch` expressions for exhaustive branching
- attach fields or methods to enums when the constants have stable behavior or metadata

In TypeScript:

- use `enum` when you need a runtime enum object or compatibility with enum-based APIs
- use `Record<EnumType, T>` for exhaustive lookup tables
- use `switch` plus `assertNever` for exhaustive branching
- prefer discriminated unions when states need associated data

## A Useful Mental Model

Use enums for:

- fixed names
- simple categories
- stable lookup keys

Use unions for:

- state modelling
- event modelling
- values with associated payloads
- situations where illegal combinations should be impossible

TypeScript supports both, but unions are often the more powerful abstraction once your model starts carrying real data.
