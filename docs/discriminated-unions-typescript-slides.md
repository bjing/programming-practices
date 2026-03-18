---
title: Discriminated Unions in TypeScript Slides
description: Slide deck version of the guide, focused on UI state, narrowing, exhaustive checks, and simple state machines.
layout: slides
---

# Discriminated Unions

## Safer UI State in TypeScript

From boolean soup to explicit state machines.

---

## What They Are

A discriminated union is a union of object shapes with a shared literal field.

```ts
type Result =
  | { type: "success"; data: string }
  | { type: "error"; message: string }
```

The `type` field tells TypeScript which variant you are handling.

---

## Why They Matter

Boolean flags allow invalid combinations:

```ts
type PageState = {
  loading: boolean
  data?: User
  error?: string
}
```

This allows impossible states like loading and error at the same time.

---

## Better State Model

```ts
type PageState =
  | { type: "loading" }
  | { type: "success"; user: User }
  | { type: "error"; message: string }
```

Illegal states cannot be represented.

---

## Narrowing Is Automatic

```ts
function handle(result: Result) {
  if (result.type === "success") {
    return result.data
  }

  return result.message
}
```

Checking the discriminator narrows the type.

---

## Switch Works Well

```ts
function render(state: PageState) {
  switch (state.type) {
    case "loading":
      return "Loading..."
    case "success":
      return state.user.name
    case "error":
      return state.message
  }
}
```

The code follows the state model directly.

---

## Exhaustive Checking

```ts
function assertNever(x: never): never {
  throw new Error("Unexpected state")
}
```

Use a `default` branch with `assertNever` to force all cases to be handled.

---

## React State Becomes Clearer

Instead of separate `loading`, `error`, and `user` state variables:

```ts
type UserState =
  | { type: "idle" }
  | { type: "loading" }
  | { type: "success"; user: User }
  | { type: "error"; message: string }
```

One state value makes transitions explicit.

---

## Events Lead to State Machines

Model both state and events:

```ts
type FormEvent =
  | { type: "field_changed"; form: FormData }
  | { type: "submit_requested" }
  | { type: "validation_failed"; message: string }
```

Then write transitions based on `(state, event)`.

---

## Why This Reduces Bugs

- Prevents impossible UI states
- Centralizes allowed transitions
- Improves refactoring safety
- Makes behavior easier to reason about
- Lets the compiler help

---

## Best Use Cases

- request lifecycle state
- forms
- authentication
- workflows
- permissions
- payment flows

Use discriminated unions wherever the UI has clear modes.
