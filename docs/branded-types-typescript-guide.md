---
title: Phantom Types and Branded Types in TypeScript
description: Distinguishing branded types from true phantom type parameters, with Haskell-style comparisons and practical TypeScript examples.
layout: default
---

# Phantom Types and Branded Types in TypeScript

These are not the same concept.

That distinction matters, especially if you are coming from Haskell.

In Haskell:

- a `newtype` creates a distinct wrapped type
- a phantom type uses a type parameter that does not appear in the value representation

TypeScript has analogues for both ideas, but they are simulated through the type system rather than built into the runtime model.

## The Haskell Distinction

Branded-like distinction in Haskell:

```hs
newtype UserId = UserId String
newtype OrderId = OrderId String
```

These are different types even though both wrap `String`.

Phantom type in Haskell:

```hs
data Document state = Document String
```

Here `state` does not appear in the runtime data.
It exists only at the type level.

That is the core phantom-type idea.

## The TypeScript Analogue for Branded Types

In TypeScript, the closest analogue to Haskell `newtype` is a branded or opaque type:

```ts
type Brand<Value, Tag> = Value & {
  readonly __brand: Tag
}
```

Example:

```ts
type UserId = Brand<string, "UserId">
type OrderId = Brand<string, "OrderId">

function loadUser(id: UserId) {
  return `user:${id}`
}

const userId = "user-123" as UserId
const orderId = "order-456" as OrderId

loadUser(userId)   // ok
loadUser(orderId)  // type error
```

At runtime both are still strings.
The distinction only exists for the compiler.

## Why This Is Not the Same as a Phantom Type

Branding is about making one representation behave like multiple nominally distinct types.

That is closer to:

```hs
newtype UserId = UserId String
```

than to:

```hs
data Document state = Document String
```

So if you want to be precise:

- Haskell `newtype` maps more closely to TypeScript branded types
- Haskell phantom types map more closely to TypeScript unused generic type parameters

## Safer Branded Construction

Raw `as` casts work, but they are easy to abuse.

A better pattern is to hide construction behind functions:

```ts
type EmailAddress = Brand<string, "EmailAddress">

function makeEmailAddress(value: string): EmailAddress | null {
  if (!value.includes("@")) {
    return null
  }

  return value as EmailAddress
}
```

This makes the brand mean something useful:

- this is not just any string
- it has passed validation

## Common Branded-Type Use Cases

Branded types are useful for:

- domain IDs
- validated strings
- units of measure
- values that share the same runtime representation but should not mix

Examples:

```ts
type ProductId = Brand<string, "ProductId">
type InvoiceId = Brand<string, "InvoiceId">

type Meters = Brand<number, "Meters">
type Feet = Brand<number, "Feet">
```

## True Phantom Types in TypeScript

The closest TypeScript analogue to a phantom type is an unused type parameter.

Example:

```ts
type Document<State> = {
  id: string
  content: string
}
```

The `State` parameter is not used in the value shape.
It exists only to carry compile-time information.

That is the phantom-type pattern.

Example workflow:

```ts
type Draft = { readonly _tag: "Draft" }
type Published = { readonly _tag: "Published" }

type Document<State> = {
  id: string
  content: string
}

function publish(doc: Document<Draft>): Document<Published> {
  return doc as Document<Published>
}
```

Here:

- the runtime shape of `Document<Draft>` and `Document<Published>` is the same
- the type parameter tracks state only at compile time

That is much closer to a real phantom type.

## Branded Types vs Phantom Type Parameters

They solve different problems.

Branded types:

- distinguish values with the same runtime representation
- simulate nominal identity on primitives or plain objects
- are the closer TypeScript analogue to Haskell `newtype`

Phantom type parameters:

- track compile-time state without changing runtime shape
- model stages, permissions, capabilities, or protocols
- are the closer TypeScript analogue to Haskell phantom types

## Branded Types vs Discriminated Unions

These are different again.

Use branded types when:

- the runtime representation stays the same
- you want stronger distinctions between similar values

Use phantom type parameters when:

- the value shape stays the same
- but you want to track compile-time state transitions

Use discriminated unions when:

- the value has multiple actual variants
- each variant has different fields
- you want exhaustive branching

Example union:

```ts
type RequestState =
  | { type: "idle" }
  | { type: "loading" }
  | { type: "success"; user: User }
  | { type: "error"; message: string }
```

That is not a branded-type or phantom-parameter problem.
It is a union-modelling problem.

## Important Limitations

TypeScript is structurally typed, so these techniques are compiler-level conventions, not true runtime barriers.

That means:

- unsafe `as` casts can defeat them
- values from JSON or APIs still need validation
- helper constructors are usually safer than arbitrary assertions

Also:

- branded markers are erased at runtime
- phantom type parameters are erased at runtime

If you need runtime guarantees, you still need validation and parsing logic.

## Practical Guidance

Use branded types for:

- `UserId`, `OrderId`, `EmailAddress`
- validated values
- units of measure

Use phantom type parameters for:

- workflow stages
- capability tracking
- protocol state transitions

Use discriminated unions for:

- UI state
- request lifecycle modelling
- event/state machines
- cases where each variant has different data

## Takeaway

If you want technical precision:

- branded types in TypeScript are closer to Haskell `newtype`
- phantom types in TypeScript are closer to unused generic type parameters

They are related, but they are not the same concept.
