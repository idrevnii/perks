---
name: simple-typescript
description: Apply direct, functional TypeScript and JavaScript coding style preferences. Use when writing, editing, reviewing, or refactoring TS/JS code; deciding whether to introduce classes, factories, helpers, wrappers, utilities, shared modules, types vs interfaces, validation boundaries, or recoverable error handling; or when the user asks for code that matches these personal style preferences.
---

# Code Style Preferences

Apply these preferences as an overlay on top of local project conventions. If repository instructions or nearby code clearly require a different pattern, follow the local convention and keep the change consistent.

## Core Style

Prefer direct, functional TypeScript.

- Prefer named functions, small modules, and exported plain objects for grouping related operations.
- Do not introduce classes unless there is a strong reason, such as required framework integration, meaningful stateful instances, or an existing class-based local convention.
- Avoid class hierarchies, factory functions, wrapper layers, and ceremony that do not materially improve clarity.
- Optimize for local readability and ease of modification.
- Keep code readable top-to-bottom without forcing readers to jump through many tiny helpers.

## Abstractions And Helpers

Avoid abstractions that only rename obvious operations.

- Do not create helper functions that trivially wrap another function, restate syntax, or add a "pretty name" without encoding meaningful domain knowledge — inline those one-liners instead.
- Extract a helper when it removes real duplication, isolates non-trivial behavior, or names a real domain concept such as deriving a status, normalizing an input payload, selecting a canonical item, or computing a baseline value.
- Prefer duplication over a bad abstraction when extraction would hide important logic.

**Avoid trivial wrappers — inline instead:**

```ts
// Don't wrap: const label = parseStringValue(raw.label)
// Do inline:  const label = String(raw.label)
```

**Prefer — extract when the helper names a real domain concept:**

```ts
// Worth extracting: encapsulates non-trivial business logic
function resolveSubscriptionStatus(account: Account): SubscriptionStatus {
  if (account.trialEndsAt > Date.now()) return "trial";
  if (account.cancelledAt) return "cancelled";
  return account.planId ? "active" : "free";
}
```

## Module Boundaries

Keep ownership clear and put reusable infrastructure in shared modules.

- Keep feature-specific files focused on feature-specific logic.
- Put cross-cutting utilities such as HTTP clients, validated request wrappers, object-freezing helpers, and error formatting in shared modules.
- Extract generic helpers only when the same helper appears in multiple places and the abstraction remains obvious.
- Expose concrete named functions.
- Optionally group related operations at the end of a module with a plain exported object.
- Avoid factory functions unless runtime dependency injection is genuinely needed.
- Prefer optional parameters with sensible defaults for simple configuration.

**Avoid — factory function for simple configuration:**

```ts
function createUserService(db: Database) {
  return {
    getUser: (id: string) => db.find("users", id),
    saveUser: (user: User) => db.insert("users", user),
  };
}
```

**Prefer — concrete named exports, with an optional grouped object:**

```ts
export function getUser(db: Database, id: string) {
  return db.find("users", id);
}

export function saveUser(db: Database, user: User) {
  return db.insert("users", user);
}

// Optional grouping at the end of the module
export const userService = { getUser, saveUser };
```

## Types

Use `type` by default.

- Use `type` for object shapes, unions, function inputs, and local module types.
- Use `interface` only for an intentional public contract meant to be extended, implemented, augmented, or consumed across module boundaries.
- Keep types explicit enough to communicate domain meaning without adding clever type machinery.

## Schemas And Type Files

Separate reusable schemas and types into obvious local files.

- Put reusable validation schemas in `schemas.ts`.
- Put reusable TypeScript types in `types.ts`.
- Do not split prematurely: keep a schema or type next to the code that uses it when it is a single local definition and/or is not used anywhere else.
- Prefer feature-local `schemas.ts` and `types.ts` files over broad shared files unless the definitions are genuinely shared across features.

## Errors And Recoverable Failures

Prefer explicit tagged errors for recoverable async operations.

- Use result-style return values for I/O, validation, and boundary failures when callers are expected to handle failure.
- Model recoverable failures as discriminated unions with a `tag` field.
- Avoid throwing from client or boundary code when the caller can reasonably branch on the failure.
- Throw for programmer errors, impossible states, and failures the current layer cannot handle meaningfully.
- Do not return `null` or `undefined` as vague failure signals.

Example shape:

```ts
type FetchUserResult =
  | { tag: "success"; user: User }
  | { tag: "not_found" }
  | { tag: "request_failed"; message: string };
```

## Validation And Boundaries

Preserve validation and explicitness.

- Validate untrusted data at external boundaries.
- Fail clearly when external data is invalid.
- Keep validation straightforward; do not fragment schemas or parsing into excessive micro-functions.
- Parse once at the edge, then pass trusted typed values inward.
- Normalize awkward external shapes before they leak into domain code.

## Review Checklist

Before finishing a change, scan for style drift:

- Did this add a class, factory, wrapper, or helper without a strong reason? Can any trivial helper be inlined?
- Is reusable infrastructure in a shared module rather than copied into feature code?
- Are module exports concrete named functions, with grouping only where useful?
- Are local shapes modeled with `type` unless an extensible public contract is intended?
- Are reusable validation schemas in `schemas.ts` and reusable TypeScript types in `types.ts`, without splitting single/local-only definitions prematurely?
- Are recoverable I/O and validation failures represented with explicit tagged outcomes, and is validation clear at external boundaries?
- Is runtime validation clear at external boundaries?
