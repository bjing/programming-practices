
---
title: Discriminated Unions and State Machines in TypeScript
---

# Discriminated Unions & State Machines in TypeScript (Frontend Robustness Guide)

## What is a Discriminated Union?

A **discriminated union** is a TypeScript pattern where multiple object types are combined into a union,
and each variant has a **common literal field (the discriminator)** that identifies which variant it is.

Example:

```ts
type Result =
  | { type: "success"; data: string }
  | { type: "error"; message: string }
```

Here:

- `Result` is a **union**
- `type` is the **discriminator**
- `"success"` and `"error"` identify the specific variant

---

# Why Discriminated Unions Matter

They allow us to model **states explicitly** and eliminate invalid combinations of data.

Bad pattern (boolean soup):

```ts
type PageState = {
  loading: boolean
  data?: User
  error?: string
}
```

Possible invalid states:

- `loading = true` and `data != null`
- `loading = true` and `error != null`
- `data` and `error` both present

Better with discriminated union:

```ts
type PageState =
  | { type: "loading" }
  | { type: "success"; user: User }
  | { type: "error"; message: string }
```

Now **illegal states cannot exist**.

---

# Type Narrowing

TypeScript automatically narrows the type when checking the discriminator.

```ts
function handle(result: Result) {
  if (result.type === "success") {
    console.log(result.data)
  } else {
    console.log(result.message)
  }
}
```

The compiler knows which fields are available.

---

# Switch Statements

Discriminated unions work very well with `switch`.

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

---

# Exhaustive Checking

You can force the compiler to ensure all cases are handled.

```ts
function assertNever(x: never): never {
  throw new Error("Unexpected state")
}

function render(state: PageState) {
  switch (state.type) {
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

If a new state is added later, TypeScript will fail compilation until it is handled.

---

# Using Discriminated Unions for React State

Instead of:

```ts
const [loading, setLoading] = useState(false)
const [error, setError] = useState<string | null>(null)
const [user, setUser] = useState<User | null>(null)
```

Use:

```ts
type UserState =
  | { type: "idle" }
  | { type: "loading" }
  | { type: "success"; user: User }
  | { type: "error"; message: string }

const [state, setState] = useState<UserState>({ type: "idle" })
```

Transitions become explicit:

```ts
setState({ type: "loading" })
setState({ type: "success", user })
setState({ type: "error", message: err.message })
```

---

# State Machines

You can extend this concept into a **state machine** by introducing **events**.

State:

```ts
type FormState =
  | { type: "editing"; form: FormData }
  | { type: "validating"; form: FormData }
  | { type: "submitting"; form: FormData }
  | { type: "success"; receiptId: string }
  | { type: "error"; form: FormData; message: string }
```

Events:

```ts
type FormEvent =
  | { type: "field_changed"; form: FormData }
  | { type: "submit_requested" }
  | { type: "validation_failed"; message: string }
  | { type: "validation_passed" }
  | { type: "submit_succeeded"; receiptId: string }
  | { type: "submit_failed"; message: string }
```

Transition function:

```ts
function transition(state: FormState, event: FormEvent): FormState {
  switch (state.type) {
    case "editing":
      if (event.type === "submit_requested") {
        return { type: "validating", form: state.form }
      }
      return state

    case "validating":
      if (event.type === "validation_passed") {
        return { type: "submitting", form: state.form }
      }
      if (event.type === "validation_failed") {
        return { type: "error", form: state.form, message: event.message }
      }
      return state

    case "submitting":
      if (event.type === "submit_succeeded") {
        return { type: "success", receiptId: event.receiptId }
      }
      if (event.type === "submit_failed") {
        return { type: "error", form: state.form, message: event.message }
      }
      return state

    case "error":
      if (event.type === "field_changed") {
        return { type: "editing", form: event.form }
      }
      return state

    case "success":
      return state
  }
}
```

---

# Why This Reduces Bugs

This approach:

- prevents impossible UI states
- centralizes transitions
- makes flows predictable
- enables exhaustive compiler checking
- improves refactoring safety

Instead of asking:

> Which flags are currently true?

You ask:

> Which **state** are we in?

---

# When to Use Discriminated Unions

They are excellent for modelling:

- API request states (`idle / loading / success / error`)
- forms
- authentication
- workflows
- permissions
- payment flows
- feature states

---

# Key Principle

> **Frontend UI should behave like a typed state machine around an unreliable world.**

Discriminated unions are one of the best tools TypeScript provides to achieve this.
