---
name: design-software
description: Apply complexity-minimizing software design principles inspired by John Ousterhout when planning a nontrivial feature, refactor, architecture, API or module boundary, data contract, workflow, or PR stack. Use when deciding whether to split or combine code, comparing implementation shapes, or reducing caller and maintainer cognitive load through deep modules and information hiding.
---

# Design Software

Design for the lowest total cognitive load rather than the fewest lines, files,
functions, or classes. Treat these principles as heuristics, not mechanical
rules.

## Run the design loop

1. Establish the concrete user intent, current behavior, constraints, and facts.
   Inspect the relevant code and callers before proposing boundaries.
2. State the smallest public API that expresses the caller's intent. Keep
   internal phases, provider details, formats, and sequencing out of that API
   unless callers genuinely need to control them.
3. Identify the important knowledge and design decisions in the feature. Assign
   each invariant, policy, format, storage layout, vendor integration, and error
   rule a clear owner.
4. Produce at least two plausible designs. Include a cohesive larger-module
   design when appropriate; do not assume more files or shorter functions are
   inherently cleaner.
5. Compare the designs using the evaluation criteria below.
6. Recommend one design and explain the decisive tradeoffs. Record what remains
   intentionally combined, what is hidden, and what would justify a later split.
7. Define a helper budget for the recommended design. List the new private
   functions or methods it needs and why each earns a separate interface.
8. Turn the selection into concrete APIs, ownership boundaries, workflow
   ordering, error behavior, tests, and implementation or PR steps.

For a small change, perform this loop briefly. Do not manufacture architecture
or delay straightforward implementation when the design is already obvious.

## Evaluate module depth

Prefer deep modules: simple, stable interfaces that hide substantial useful
behavior.

- Judge depth by implementation complexity hidden per unit of interface
  complexity, not by line count.
- Accept a large cohesive implementation when splitting would expose more
  concepts or dependencies.
- Avoid shallow wrappers, pass-through methods, and files that merely rename or
  forward another layer's interface.
- Distinguish the public product API from private implementation interfaces. One
  public operation may use several private modules without increasing caller
  complexity.
- Extract a module only when it owns meaningful knowledge, can change relatively
  independently, and offers an interface substantially simpler than its
  implementation.

## Set a helper budget

For a nontrivial implementation plan, state a helper budget for the recommended
design before listing implementation steps. Scope it to new private functions
and methods introduced by the change; public operations and existing helpers
are outside the budget.

- Give a numeric ceiling. Zero is valid. Treat the ceiling as a reviewable
  prediction, not a quota or a hard architectural limit.
- List every expected helper by name or responsibility, its owner and callers,
  the knowledge or complexity it hides, and why keeping that logic inline would
  be worse.
- Justify helpers through information hiding, invariant ownership, a distinct
  algorithm or mechanism, or removal of duplicated knowledge. Reuse alone is
  neither required nor sufficient.
- Reject helpers that only rename an expression, forward arguments, split
  chronological steps, satisfy a size metric, or create new parameter and state
  plumbing.
- Exceed the budget only when implementation reveals a distinct responsibility
  that the plan missed. Revise the budget and record the new justification
  before adding the helper.
- At handoff, reconcile planned and actual helpers. Merge or inline any helper
  that no longer earns its interface.

Present the budget compactly:

| Helper | Owner and callers | Knowledge hidden | Why not inline |
| --- | --- | --- | --- |
| Name or responsibility | Owning module and direct callers | Invariant, policy, algorithm, or mechanism | Concrete cognitive-load benefit |

## Hide information

Organize code around knowledge ownership.

- Give formats, schemas, policies, storage layouts, invariants, and vendor APIs
  one authoritative owner.
- Avoid making producers and consumers independently understand the same raw
  representation.
- Return validated domain values instead of raw dictionaries, provider objects,
  or bags of loosely related options and paths.
- Prefer a change to affect only the module owning the changed decision.
- Treat duplicated tiny helpers as a possible symptom of a missing higher-level
  owner, not an automatic reason to create a utility module.

## Avoid temporal decomposition

Do not expose implementation phases merely because they occur in sequence.

- Express the caller's complete intent in one operation when intermediate phases
  are not independently useful capabilities.
- Keep required ordering in one workflow owner.
- Separate phases only when callers need them independently or they hide
  genuinely different knowledge behind deep interfaces.
- Watch for APIs that require callers to invoke `prepare`, then `execute`, then
  `validate`, or to carry an internal artifact between those calls.

## Pull complexity downward

Let the module with the most information absorb complexity for its callers.

- Choose safe defaults and derive values internally when the module can do so
  reliably.
- Enforce invariants at the boundary that owns them.
- Define invalid states and avoidable errors out of existence where practical.
- Group errors by how callers handle them rather than by the internal phase that
  produced them.
- Keep the common case simple and make uncommon configuration unobtrusive.
- Avoid punting decisions to users when the implementation has better context.

## Keep layers distinct

Require each layer to provide a meaningfully different abstraction.

- Let higher layers express user or domain intent.
- Let lower layers hide mechanisms such as serialization, persistence, process
  execution, security policy, or provider SDKs.
- Avoid leaking lower-level types and configuration through higher-level APIs.
- Combine adjacent layers when the upper layer is only a pass-through.

## Choose useful generality

Design the simplest interface that covers current concrete needs and removes
special cases.

- Prefer a somewhat general mechanism when it makes the interface deeper and
  simpler.
- Do not add speculative extension points, configuration, or abstractions.
- Push specialization upward and reusable mechanism downward when that reduces
  dependencies.
- Revisit the design when a new concrete need appears instead of prebuilding an
  imagined framework.

## Decide whether to combine or separate

Bring code together when it:

- Shares important knowledge or invariants.
- Performs one complete task.
- Repeats the same policy or error handling.
- Must change consistently.
- Would otherwise require callers to coordinate ordering or state.

Separate code when it:

- Owns genuinely independent knowledge.
- Changes for a different reason and can do so behind a small interface.
- Hides a complex mechanism such as a provider integration or persisted
  protocol.
- Lets maintainers reason locally without creating a web of shallow interfaces.

Do not use "single responsibility" as a synonym for "small." Describe the
responsibility in terms of the knowledge the module owns.

## Design errors and invariants

- List the invariants and ordering guarantees explicitly.
- Assign exactly one owner to each invariant.
- Preserve prior valid state when a prerequisite fails when feasible.
- Catch and translate implementation-specific failures at the highest layer that
  can add actionable context.
- Expose separate public error categories only when callers respond differently.
- Test contracts, security boundaries, destructive transitions, and subtle
  ordering rather than only helper implementations.

## Separate code structure from delivery structure

Do not require PR boundaries to mirror runtime module boundaries.

- Organize code for information hiding and maintainability.
- Organize stacked PRs for dependency order and focused review.
- Allow a foundational private contract to land before its public facade when it
  is independently testable and immediately consumed upstack.
- Avoid temporary public APIs created only to make an intermediate PR appear
  independently user-facing.

## Present the design

For a nontrivial design task, provide:

1. The recommended outcome first.
2. The caller intent and proposed public API.
3. At least two credible alternatives.
4. A compact mapping of each proposed module to the knowledge it owns.
5. Workflow ordering, invariants, and error semantics.
6. Caller and maintainer cognitive-load tradeoffs.
7. The helper budget for the recommended design.
8. The implementation and review sequence.
9. Open questions that materially change the design.

Use a table or small flow only when it makes ownership, alternatives, or
sequence materially easier to compare. Avoid using design terminology as a
substitute for concrete reasoning about the code at hand.
