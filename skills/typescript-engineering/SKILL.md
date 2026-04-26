---
name: typescript-engineering
description: Use this skill whenever the user asks you to write, edit, review, refactor, debug, or design TypeScript or TSX code. It is especially relevant for application code, backend routes, React/UI work, schemas, runtime boundaries, persistence, async workflows, API contracts, tests, lint/typecheck fixes, and code review. Apply it even when the user does not explicitly mention "TypeScript" if the files or project are TypeScript-based.
---

# TypeScript Engineering

Use these rules when writing or editing TypeScript code. The goal is senior-level application engineering: clear ownership, precise types, explicit boundaries, small reviewable changes, and behavior that remains understandable after the original author leaves the room.

## How To Use This Skill

Before changing TypeScript or TSX:

1. Read local project instructions first, including `AGENTS.md`, contributing docs, and nearby conventions.
2. Inspect the existing code shape before deciding on abstractions, dependencies, or file layout.
3. Make the smallest change that fully solves the request.
4. Keep runtime boundaries explicit: parse untrusted data at the edge, then pass trusted typed values inward.
5. Run the deterministic checks the project supports (typecheck, lint, focused tests, format checks). If any check fails, fix the failure before considering the change complete — do not leave the codebase in a broken state. If a check cannot be run in the current environment, note what was verified manually and what risk remains.

When reviewing TypeScript, lead with behavioral risks, broken contracts, weak validation, async/state bugs, missing tests, and type unsoundness. Keep style-only observations secondary unless they hide a real maintenance risk.

## Core Principle

Preserve the system's important invariants before optimizing for convenience.

For meaningful changes, make the cause and effect visible:

- What behavior changed.
- What data crosses a boundary.
- What assumptions the code relies on.
- What evidence would show the change works.

Types should make invalid states harder to express, but they should not hide simple runtime behavior behind clever type machinery.

## Change Shape

Make the smallest change that fully solves the problem. Prefer changes that are easy to review, easy to revert, and easy to explain.

- Keep edits scoped to the requested behavior.
- Avoid unrelated refactors while changing behavior.
- Let existing code style and module ownership guide the implementation.
- When a local fix is enough, do not introduce a shared abstraction.
- When the same bug appears in several places, fix the shared cause instead of patching symptoms independently.
- If a change affects a public contract, persistence format, or cross-module API, make that explicit in code and tests.

## Ownership And Boundaries

Code should live where its owner is obvious.

- Keep behavior near the feature, domain, route, component, command, or adapter that owns it.
- Import through narrow, intentional exports instead of reaching into another module's internals.
- Prefer domain-named functions and files when the code has domain meaning.
- Shared modules should exist because multiple real callers need the same stable behavior, not because code might be reusable.
- Generic helper files are acceptable only for genuinely generic, low-level operations such as formatting, small collection transforms, date math, or string normalization.
- Do not let `helpers`, `utils`, `common`, `core`, `services`, or `managers` become catch-all locations for domain behavior.
- Separate external boundaries from internal logic: parse and validate at the edge, then pass trusted typed values inward.
- Keep framework glue thin. Route handlers, controllers, UI units, CLI commands, and workers should delegate meaningful domain work to named functions.

## Types And Schemas

Use TypeScript to model domain facts, not to decorate JavaScript.

- Use `type` by default for application code.
- Use `interface` only when openness is intentional: declaration merging, module augmentation, ambient declarations, or a public extension point.
- Prefer precise domain types over broad primitives when the distinction matters.
- Use discriminated unions for states, variants, and finite workflows.
- Make impossible states unrepresentable when it stays readable.
- Avoid `any`; use `unknown` at boundaries and narrow it deliberately.
- Avoid type assertions unless they are close to a runtime check or a well-documented external guarantee.
- Do not use clever generic types when a simple explicit type is clearer.
- Let schemas define contracts at runtime boundaries, and infer TypeScript types from those schemas when the schema is the source of truth. Do not duplicate the same contract as both a hand-written type and a schema without a clear reason.

## Null And Undefined

Use `undefined` for absence inside application code.

- Prefer optional properties (`foo?: number`) over `foo: number | null`.
- Use `null` only when the external contract, database, protocol, or API explicitly distinguishes `null` from absence.
- Normalize external `null` values to internal `undefined` at the boundary when the application does not need to preserve the distinction.
- Do not pass `null` deeper into domain code just because an external API used it.
- Do not blindly erase `null` when it carries meaning, such as "explicitly cleared" or "intentionally unset".
- Prefer `exactOptionalPropertyTypes` when the codebase can support it.

## Functions And Control Flow

Write functions that make the happy path and failure paths obvious.

- Give functions one clear responsibility, but do not split code into tiny wrappers that only rename each other.
- Prefer early returns for invalid, empty, or exceptional cases.
- Keep branching explicit when business rules matter.
- Avoid boolean parameters that change a function's mode; prefer separate named functions or an options object with a discriminant.
- Avoid functions that both compute a value and cause unrelated side effects.
- Prefer pure functions for calculations, transformations, filtering, parsing, and policy decisions.
- Keep async boundaries visible: do not hide network, filesystem, database, or time-dependent work inside innocent-looking helpers.
- Do not swallow errors silently. Either handle them with domain intent or let them propagate with useful context.
- For discriminated unions, use exhaustive handling so new variants fail loudly during typecheck.
- Use concurrency deliberately. Parallelize independent async work with `Promise.all`, but do not create hidden races around shared state or ordering.

## Errors And Results

Choose the error shape by how callers can respond.

- Throw for programmer errors, violated invariants, and failures the current layer cannot reasonably recover from.
- Use a local typed union for expected domain outcomes that callers should branch on: validation failures, unavailable resources, declined actions, or recoverable external failures.
- Do not introduce a Result library or rewrite an exception-based codebase around `Result` unless the codebase already uses that style or the benefit is concrete and broad.
- Do not wrap every function in a `Result` type by default; use typed outcomes only when the caller has meaningful recovery branches.
- Do not return `null` or `undefined` as a vague failure signal. Use a named union variant or throw.
- Preserve useful context when rethrowing or wrapping errors.
- Avoid catch blocks that only log and continue unless continuing is a deliberate domain decision.
- Keep user-facing error messages separate from diagnostic details when security or UX matters.
- At process, request, and job boundaries, convert unknown thrown values into structured errors.

Example — typed domain outcome:

```ts
type CreateUserOutcome =
  | { status: "created"; user: User }
  | { status: "email_taken" }
  | { status: "invalid_invite" };

async function createUser(input: CreateUserInput): Promise<CreateUserOutcome> {
  const isEmailTaken = await isEmailExists(input.email)
  if (isEmailTaken) return { status: "email_taken" };
  if (!isValidInvite(input.inviteCode)) return { status: "invalid_invite" };
  const user = await db.users.insert(input);
  return { status: "created", user };
+}
```

## Runtime Boundaries And Validation

Static types stop at runtime boundaries. Validate data when it enters the system.

Validate: HTTP request bodies, params, query strings, and headers; external API responses; environment variables before startup; persisted JSON blobs; job, queue, and event payloads; AI/model outputs before treating them as structured data.

Guidelines:

- Use a schema library such as Zod, Valibot, ArkType, or the project's existing choice; do not mix validation libraries casually.
- Keep boundary schemas close to the boundary or domain that owns the contract.
- Parse once at the edge, then pass trusted typed values inward.
- Normalize awkward external shapes into cleaner internal types at the boundary.
- Do not repeatedly re-validate trusted internal values through every layer.
- Prefer explicit schema failures over defensive optional chaining deep in domain code.
- Preserve raw external payloads when auditability or debugging matters.

Example — Zod boundary validation:

```ts
import { z } from "zod";

const WebhookPayloadSchema = z.object({
  eventType: z.enum(["order.created", "order.cancelled"]),
  orderId: z.string().uuid(),
  occurredAt: z.string().datetime(),
});

type WebhookPayload = z.infer<typeof WebhookPayloadSchema>;

export function parseWebhookPayload(raw: unknown): WebhookPayload {
  return WebhookPayloadSchema.parse(raw); // throws ZodError with field-level detail on failure
}
```

## Async, Effects, And State

Make side effects visible and intentional.

- Keep network, filesystem, database, clock, random, and process-level effects near adapters or clearly named functions.
- Pass dependencies explicitly when it improves testability or makes effects clearer.
- Avoid hidden module-level mutable state unless it is truly process-wide configuration or a managed cache.
- Use `async`/`await` for readable control flow; use promise chains only when they are clearer for composition.
- Always handle or return promises; do not leave floating promises.
- Use cancellation, timeouts, or abort signals for external calls when the path can hang or outlive its caller.
- Make retries, backoff, and idempotency explicit around external side effects.
- Be careful with `Promise.all`: it is for independent work, not work that depends on sequencing or shared mutation.
- Keep derived state derived; do not store duplicate mutable state without a clear synchronization plan.

## Data Modeling And Persistence

Do not pretend that every layer has the same data shape.

- Distinguish external DTOs, persistence records, domain objects, and UI/view models when their meanings differ.
- Avoid one giant `User`, `Order`, or `Config` type reused across unrelated boundaries.
- Store durable data with enough version, timestamp, and source metadata to explain where it came from.
- Treat persisted historical inputs and raw external outputs as append-only unless the domain explicitly allows correction.
- Prefer explicit migrations or transformation functions when a stored shape changes.
- Keep serialization and deserialization near the boundary that owns the format.
- Do not let database nullability, API awkwardness, or UI convenience leak through every layer.
- Use stable identifiers and explicit timestamps instead of relying on array positions, object key order, or implicit runtime timing.
- Use `readonly` for data that should not be mutated after construction, especially shared config, parsed inputs, and domain snapshots.

## Naming And Readability

Names should make code review easier.

- Use domain nouns and verbs rather than generic technical verbs.
- Prefer `calculateInvoiceTotal`, `parseWebhookPayload`, or `selectVisibleItems` over `processData`, `handleResult`, or `formatPayload`.
- Name booleans as predicates: `isArchived`, `hasPermission`, `shouldRetry`.
- Name functions by the observable action they perform or value they return.
- Avoid names that describe implementation mechanics instead of intent.
- Avoid vague suffixes like `Manager`, `Service`, `Helper`, `Util`, `Processor`, and `Handler` unless the domain meaning is still clear.
- Do not use abbreviations unless they are established in the domain or codebase.
- Keep comments for why, tradeoffs, invariants, and non-obvious constraints — not for what the TypeScript already makes clear from types and names.

## Imports, Modules, And Dependencies

Make module relationships explicit and boring.

- Prefer named exports for application code. Avoid default exports except where the framework or existing convention expects them.
- Avoid broad `export *` barrel files that hide ownership or expose internals. A barrel is acceptable when it is a narrow, intentional public API for a module.
- Keep dependency direction clear: framework glue may call domain code, but domain code should not import framework glue.
- Do not introduce a package for something the standard library or existing dependency already handles well.
- Do not add dependencies for tiny helpers.
- Isolate side-effect imports, global setup, polyfills, and instrumentation in obvious entrypoint files.
- Avoid cyclic imports; split by domain meaning instead of creating a generic shared dumping ground.
- Prefer explicit relative or configured path imports that match the project convention; do not mix import styles casually.

## Testing And Checks

Use tests and tooling to protect behavior, not to decorate the repo.

- Prioritize tests for parsing, normalization, calculations, permissions, state transitions, data migrations, and bug fixes.
- Prefer focused tests with fixed examples over broad snapshot tests that are hard to review.
- Add regression tests when fixing a bug unless the cost is clearly disproportionate.
- Test public behavior and important invariants more than private implementation details.
- Mock at external boundaries; avoid mocking the function you are trying to prove.
- For UI code, test user-visible behavior and accessibility-relevant states instead of component internals.
- Keep tests deterministic: control time, randomness, network, filesystem, and concurrency.
- Do not chase coverage numbers before the core behavior is trustworthy.
- If a change cannot be tested cheaply, explain what was verified manually and what risk remains.

## Code Size And Complexity

Prefer code that fits in a reviewer's head.

- Keep functions focused enough that their responsibility, inputs, outputs, and failure modes are obvious at a glance.
- Avoid deeply nested conditionals; use guard clauses, discriminated unions, or small named steps.
- Do not extract a function unless its name adds meaning or it removes real duplication.
- Large files are acceptable for schemas, generated code, migrations, constants, or domain-owned `types.ts` files.
- When a file grows too large, split by ownership and behavior rather than creating `helpers.ts`.
- As a rough review signal, reconsider ordinary source files above 300 lines and ordinary functions above 50 lines.

## Configuration And Environment

Configuration should fail early and be typed after startup.

- Parse and validate environment variables at the application boundary.
- Export a typed config object instead of reading `process.env` throughout the codebase.
- Convert strings to domain types early: numbers, booleans, URLs, enums, durations, and feature flags.
- Keep secrets out of logs, errors, snapshots, and client bundles.
- Make defaults explicit. Avoid hidden fallback behavior for important production settings.
- Separate build-time, server-runtime, and client-exposed configuration.
- Treat feature flags as configuration with owners and expected removal paths.

Example — typed config parsing:

```ts
import { z } from "zod";

const ConfigSchema = z.object({
  databaseUrl: z.string().url(),
  port: z.coerce.number().int().min(1).max(65535).default(3000),
  logLevel: z.enum(["debug", "info", "warn", "error"]).default("info"),
  stripeWebhookSecret: z.string().min(1),
});

export type AppConfig = z.infer<typeof ConfigSchema>;

export function loadConfig(): AppConfig {
  const result = ConfigSchema.safeParse(process.env);
  if (!result.success) {
    throw new Error(`Invalid configuration:\n${result.error.toString()}`);
  }
  return result.data;
}
```

## Constants

Give important constants a domain home.

- Put domain constants in a `constants.ts` file at the level of the domain, feature, module, or component that owns them.
- This applies even when a constant currently has only one caller, if the value represents a business rule, protocol value, limit, timeout, default, storage key, or route.
- Keep purely local readability constants near the code that uses them.
- Avoid magic numbers and magic strings in executable logic.
- Name constants by their meaning, not by their value.

## UI Code

Keep UI code declarative, typed, and behavior-focused.

- UI units should express structure, rendering, and user interactions; move non-trivial domain logic, data shaping, and policy decisions into named feature-owned functions.
- Model UI state as the smallest state that cannot be derived from props, server data, URL state, or existing application state.
- Prefer discriminated unions for async and multi-mode UI states instead of several loosely related booleans.
- Keep event handlers small; delegate meaningful work to named functions.
- Do not mirror server data into local state unless the user can edit it locally or there is a clear synchronization plan.
- Keep URL state, form state, server/cache state, and local UI state conceptually separate.
- Use accessible semantic elements before custom div-based controls.

Example — discriminated union UI state:

```tsx
type SubmitState =
  | { status: "idle" }
  | { status: "submitting" }
  | { status: "success"; confirmedId: string }
  | { status: "error"; message: string };

// Renders based on state rather than multiple booleans like isLoading, hasError, isSuccess
function SubmitButton({ state }: { state: SubmitState }) {
  if (state.status === "submitting") return <Spinner />;
  if (state.status === "error") return <ErrorBanner message={state.message} />;
  if (state.status === "success") return <Confirmation id={state.confirmedId} />;
  return <button type="submit">Submit</button>;
}
```

## APIs And Public Contracts

Treat public contracts as promises.

- Be explicit when changing exported functions, HTTP APIs, event payloads, database shapes, CLI flags, environment variables, or component props used outside the file.
- Prefer additive changes when callers may exist outside the immediate edit.
- Keep compatibility code only with an owner and a removal path.
- Version contracts when historical interpretation matters.
- Do not reuse the same field name for a different meaning.
- Prefer narrow request/response types over exposing broad internal domain objects.
- Keep internal types internal until there is a real external caller.
- Document non-obvious invariants at the contract boundary.
- If a contract is generated from a schema, update the schema first and regenerate rather than editing generated artifacts by hand.

## Security And Privacy

Treat inputs, identity, and secrets with suspicion. Key TypeScript-specific checks:

- Validate and narrow untrusted input at runtime boundaries with a schema library before it reaches domain code — static types give no runtime protection.
- Keep authentication and authorization checks close to the operation they protect; do not rely on client-side checks for server-side authorization.
- Include tenant/user ownership filters in data access paths where applicable.
- Avoid unsafe HTML injection; if HTML rendering is required, sanitize with a project-approved library.
- Use parameterized database queries or the project's query builder; do not concatenate user input into queries.
- Be careful with redirects, file paths, shell commands, and URLs derived from user input.
- Fail closed when permission, identity, or tenant context is missing or ambiguous.
- Never log secrets, tokens, credentials, or private user data.

## Performance And Resource Use

Avoid obviously wasteful shapes; measure before making invasive changes.

- Prefer simple code until there is evidence of a performance problem or the inefficient shape is obvious.
- Avoid accidental N+1 queries, expensive renders, and unbounded loops over untrusted input.
- Bound untrusted or potentially large work: pagination, limits, timeouts, streaming, batching, or backpressure.
- Do not load large datasets, files, or response bodies into memory unless the size is known and acceptable.
- Avoid blocking the event loop with heavy synchronous work on request, UI, or job hot paths.
- Cache only when invalidation, ownership, and memory growth are understood.

## Review Mindset

Before finishing a meaningful change, check the engineering story.

- What behavior changed?
- What data crosses a boundary?
- What assumptions does the code now rely on?
- What contracts, schemas, migrations, or public APIs changed?
- Did this add an abstraction, dependency, or shared module? Why is it needed now?
- Are errors, loading states, empty states, and permission failures handled deliberately?
- Are important inputs, outputs, timestamps, versions, and source metadata preserved where relevant?
- What deterministic checks or focused tests prove the change?
- What remains unverified, and is that acceptable for the risk?
