---
title: Enums in Java 21+ and TypeScript - Slides
description: Slide deck comparing enums, exhaustive matching, type-safe lookup tables, and why TypeScript unions often go further.
layout: slides
---

# Enums in Java 21+ and TypeScript

Exhaustive Matching, Lookups, and Unions

---

## Why Enums Exist

Enums are useful when a value must be one of a fixed set of named options.

- status values
- modes
- categories
- permissions
- workflow steps

They are good for modelling closed sets.

---

## Java Enums

In Java, enums are full classes with fixed instances.

```java
public enum PaymentStatus {
    PENDING,
    PAID,
    FAILED
}
```

They can also carry fields and methods.

---

## Java Enums with Extra Data 

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

Java enums are more than constants. They are objects.

---

## TypeScript Enums

```ts
enum PaymentStatus {
  PENDING = "PENDING",
  PAID = "PAID",
  FAILED = "FAILED",
}
```

This gives you a runtime object plus a type.

It works well when you want a named value namespace at runtime.

---

## TypeScript Enum Alternative 

Many TypeScript codebases prefer unions instead:

```ts
type PaymentStatus = "PENDING" | "PAID" | "FAILED"
```

This is often simpler and composes better with the type system.

Unions are way more powerful than enums in Typescript (discussed later).

---

## Exhaustive Matching in Java 21+

`switch` expressions make enum branching explicit and exhaustive.

```java
static String render(PaymentStatus status) {
    return switch (status) {
        case PENDING -> "Waiting for payment";
        case PAID -> "Paid";
        case FAILED -> "Payment failed";

        // normally we have a default value
        // default -> "Unknown status";
    };
}
```

If a new enum value is added to PaymentStatus, missing case 
in `switch` statement would cause a COMPILE error.

---

## Enum Exhaustive Matching in TypeScript

Can achieve what Java 21 has, but with a trick.

Use `switch` with `never` checking.

```ts
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


function assertNever(value: never): never {
  throw new Error(`Unexpected value: ${String(value)}`)
}
```

The compiler helps prove that all cases are handled.

---

## Type-Safe Lookup Tables with `Record`

When the logic is just a lookup, `Record` is often cleaner than `switch`.

```ts
const paymentStatusLabel: Record<PaymentStatus, string> = {
  [PaymentStatus.PENDING]: "Awaiting payment",
  [PaymentStatus.PAID]: "Payment received",
  [PaymentStatus.FAILED]: "Payment failed",
}
```

Add a new enum member and TypeScript forces the table to be updated.

---

## Type-Safe Lookup Tables with `Record` - Cont

Remember, the mapped values don't have to be primitive values.

They can be functions.

```ts
type DispatchAction = (amount: number) => string

const pendingPaymentAction: (amount: number) => string = {/*code*/}
const paidPaymentAction: (amount: number) => string = {/*code*/}
const failedPaymentAction: () => string = {/*code*/}

const paymentStatusActions: Record<PaymentStatus, DispatchAction> = {
  [PaymentStatus.PENDING]: pendingPaymentAction,
  [PaymentStatus.PAID]: paidPaymentAction,
  [PaymentStatus.FAILED]: failedPaymentAction,
}
```

Again, compiler will ensure type safety by exhaustively matching enum values.

---

## [Typescript] `Record` Also Works with Unions

```ts
type PaymentStatus = "PENDING" | "PAID" | "FAILED"

const paymentStatusLabel: Record<PaymentStatus, string> = {
  PENDING: "Awaiting payment",
  PAID: "Payment received",
  FAILED: "Payment failed",
}
```

This is great for labels, icons, colors, and route mappings.

---

## Where Enums Start to Strain

This enum only names the state:

```ts
enum RequestState {
  IDLE = "IDLE",
  LOADING = "LOADING",
  SUCCESS = "SUCCESS",
  ERROR = "ERROR",
}
```

It does not say what data belongs to each state.

---

## Solution: More Advanced Unions

This union definition
```ts
type PaymentStatus =
  | { displayName: string, colour: string, icon: string, type: "PENDING" }
  | { displayName: string, colour: string, icon: string, type: "PAID" }
  | { displayName: string, colour: string, icon: string, type: "FAILED" }
```

is the same as

```ts
type WithSharedAttrs = {
  displayName: string,
  colour: string,
  icon: string,
}

type PaymentStatus =
  | (WithSharedAttrs & { type: "PENDING" })
  | (WithSharedAttrs & { type: "PAID" })
  | (WithSharedAttrs & { type: "FAILED" })
```

Now every union value will include the 3 shared attributes.

---

## Data Modeling (bad but often done)

The Problem with Enum Plus Optional Fields

```ts
enum RequestState {
  IDLE, 
  LOADING,
  SUCCESS,
  ERROR
}

class RequestModel {
  state: RequestState
  data: User | null
  error: string | null

  constructor() {}

  // relevant methods 
}
```

This allows impossible combinations:

- success with no data
- loading with an error
- error with stale success data

---

## Data Modeling (proper way)

We can make illegal states unrepresentable.

```ts
type RequestState =
  | { type: "idle" }
  | { type: "loading" }
  | { type: "success"; user: User }
  | { type: "error"; message: string }
```

Now each state carries only the data it is allowed to have.

---

## Why Unions Are Stronger

Unions give you:

- explicit state shapes
- better narrowing
- fewer impossible states
- clearer transitions
- stronger compiler guarantees

This is often the better TypeScript abstraction for UI state.

---

## Unions Are Strong, What About Java - Sealed Interface

Java can model the same idea with a sealed interface and records:

```java
sealed interface RequestState
    permits Idle, Loading, Success, Error {}

record Idle() implements RequestState {}
record Loading() implements RequestState {}
record Success(User user) implements RequestState {}
record Error(String message) implements RequestState {}
```

This gives Java a closed set of variants with data attached to each case.

---

## Choose Between Enums and Sealed Interface

Use enum for (simple shapes):
- status codes 
- modes 
- categories
- fixed labels

Use sealed interface for (more complex shapes):
- request states 
- command/result types 
- workflows 
- domain events

---

## Practical Guidance

In Java:

- use enums for closed sets
- use `switch` expressions for exhaustive pattern matching 
- sealed interfaces is a great alternative for data/state modeling

In TypeScript:

- use `enum` when you need a runtime enum object
- use `Record<EnumType, T>` for lookup tables
- use unions when the model needs associated data
- use unions to make illegal states literally unrepresentable

---

## Extra Takeaway

Compiler is our friend, make MORE use of it!

Type safety analogy:

- compile time - security check at international airport 
  - detects drugs, other illegal items and criminals from boarding etc.
- runtime - illegal stuff/criminals onboard the plane
  - still catchable, but harder, foreign country/legislation etc

If you can catch errors at compile time, try not to leave it to runtime 

--- 

## How to catch more errors at compile time
- proper modeling & usage of types
  - avoid `any` type 
  - avoid primitive type everywhere
  - Branded types (Typescript)
  - phantom types (both Java and Typescript) 
  - more advanced typing techniques

--- 

## Branded Types
  
Branded types are mostly used for key properties in domain models, compile time type safety without runtime object wrapping cost, e.g. 

Instead of:

```ts
type User = {id: string, name: string, email:string} 
```

you can have:

```ts
/**
  Branded type CHEAP!
  Userid at compile time, but string at runtime

  Object wrapping primitive type EXPENSIVE!
  type UserId = {value: string}
**/

type UserId = Brand<string, "UserId">
type EmailAddress = Brand<string, "EmailAddress">

type User = {id: UserId, name: string, email: EmailAddress} 
```

---

## Branded Types - Cont

Can use a session to talk about Branded types and how to use
them to improve:
* code readability 
* most robust application

---

## Phantom Types

Can have a separate sesion to dicuss this and how to use
it to 
* improve our modeling 
* prevent certain illegal code/state from passing compiler check
