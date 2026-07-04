---
name: software-design-discipline
description: Use when designing, implementing, reviewing, or refactoring software changes to reduce complexity through simple interfaces, deep modules, information hiding, and strategic programming.
alwaysApply: false
---

# Agent guide: software design discipline

Based on A Philosophy of Software Design.

## Primary objective

Reduce overall system complexity. Prefer changes that make the next change easier, even if they take a bit longer now.

## Core rule

Do not optimize for "fastest patch that works." Optimize for low dependency, low obscurity, and simple interfaces.

## Operating principles

1. Practice strategic programming.
   Spend effort on design when it will reduce future complexity, change amplification, or cognitive load.
2. Treat complexity as the main failure mode.
   Complexity is anything that makes the system harder to understand or modify.
3. Prefer deep modules.
   Build modules with simple interfaces and substantial hidden functionality.
4. Keep interfaces simpler than implementations.
   Push complexity down into the module instead of exporting it to callers.
5. Hide information aggressively.
   If knowledge, rules, or constraints can live in one place, keep them in one place.
6. Eliminate information leakage.
   If the same knowledge is duplicated across files, layers, or services, redesign until one module owns it.
7. Avoid shallow abstractions.
   Do not add wrappers, helpers, adapters, or pass-through layers unless they remove real complexity.
8. Avoid temporal decomposition.
   Organize code around stable knowledge and responsibilities, not around step 1 / step 2 / step 3 of a workflow.
9. Make the common case simple.
   Use sensible defaults. Remove unnecessary configuration. Do not force callers to handle rare cases up front.
10. Separate general-purpose code from special-case code.
    Do not contaminate shared mechanisms with one-off behavior.
11. Prefer joining over splitting when shared knowledge is high.
    If two units must evolve together or repeat the same ideas, they probably want one owner.
12. Do not split methods just to make them shorter.
    Split only when a different responsibility appears. Keep one coherent operation together.
13. Design it twice before committing.
    For non-trivial changes, compare at least two designs mentally and pick the one with the simpler dependency surface.
14. Define errors out of existence when practical.
    Prefer designs that make invalid states, race windows, or edge-case handling unnecessary.
15. Write comments that capture intent and invariants.
    Do not restate code. Explain why the design exists, what must remain true, and what is non-obvious.

## Required workflow for non-trivial changes

1. Identify the module or abstraction boundary first.
2. Ask where the knowledge should live.
3. Ask whether the change adds dependency, obscurity, or a new special case.
4. If the change touches many places, stop and look for a deeper abstraction.
5. If a new interface is needed, make it minimal and easy to explain.
6. Keep implementation detail behind the boundary.
7. Add comments only for intent, invariants, edge semantics, and cross-module dependencies.

## Design before implementation

For new systems and new modules, do not begin by writing implementation code. Begin by designing the abstraction.

Required order:

1. Identify the module or system boundary.
2. Write the interface comments first.
3. Define the interface to match the comments.
4. Check the relationships between modules through those interfaces.
5. Revise the comments and interfaces until the dependency structure is simple.
6. Only then implement.

Use interface comments as the first design tool, not as documentation added after the fact.

The interface comments should force clarity on:

- what knowledge the module owns
- what the module hides
- what callers are allowed to assume
- what invariants the module guarantees
- what other modules must know, and what they must not know

Before implementation, review the interface comments and ask:

- Can a caller understand how to use this module without knowing the implementation?
- Is this interface exposing complexity that should be hidden?
- Are responsibilities and ownership clear across module boundaries?
- Does the relationship between modules reduce dependency, or spread it?
- Would this interface still make sense if the implementation changed completely?

If the interface comments are awkward, vague, or full of caveats, the design is not ready. Fix the abstraction first, then write code.

Ousterhout-style design sequence for a new module:

1. Write the class or module interface comment first.
2. Write signatures and interface comments for the most important public methods.
3. Iterate on those comments and signatures until the abstraction feels clean.
4. Write the key instance variables or persistent state declarations and document what each represents.
5. Only then write method bodies.
6. For any new method discovered during implementation, write its interface comment before its body.

Use the comments as the design surface. If the comments are hard to make short, precise, and complete, the abstraction is probably wrong.

### Interface-first template

Before implementing a new class, module, or major function, draft comments in this shape:

```text
Module/Class comment:
- What abstraction does this provide?
- What does one instance represent?
- What important limitations or guarantees matter to users?

Method comment:
- What does this do from the caller's point of view?
- What must each argument mean or satisfy?
- What is returned, including important edge semantics?
- What side effects, exceptions, or preconditions are visible to callers?
```

If this template cannot be filled in clearly without talking about implementation details, the interface is too shallow or the boundary is wrong.

## Develop abstractions, not features

- The basic increment of development should be an abstraction, not a feature.
- It is fine to wait until a feature creates the need for an abstraction.
- Once the abstraction is needed, design it properly rather than growing it one special case at a time.
- Do not let test-by-test or feature-by-feature progress fragment the abstraction into tactical patches.
- Prefer designing a reasonably complete core abstraction up front over adding one narrow method per immediate need.

## Choose the right design posture for the situation

### When creating a new system or a clean new codebase

- Start from the core abstractions, not from files, routes, or workflow steps.
- Design module boundaries before filling in implementation details.
- Prefer a small number of deep modules with clear ownership.
- Keep the public surface area narrow from the start. Every public API is future maintenance cost.
- Eliminate speculative flexibility unless there is a concrete near-term need.
- Make defaults and common paths first-class so the initial design stays simple.
- Write interface comments early, because they force clarity about the abstraction.
- Design for ease of reading, not ease of writing.

### When adding a new module inside an existing codebase

- Treat the existing codebase as a constraint, not as a template to copy blindly.
- Match existing patterns only when they reduce complexity. Do not preserve a bad pattern for consistency.
- Find the best ownership boundary for the new knowledge before choosing where the file lives.
- Prefer absorbing the new behavior into an existing module if that produces one deeper, clearer abstraction.
- Create a new module only when it creates a simpler interface or isolates knowledge cleanly.
- Avoid introducing transitional wrappers unless they materially simplify migration.
- Prefer composition and clear ownership over inheritance hierarchies or generic pass-through layers.

### When modifying an existing codebase

- First understand the current dependency and knowledge flow around the change.
- Do not patch symptoms at the edge if the real problem is the abstraction boundary.
- If the existing design is awkward, improve the design as part of the change when the scope is reasonable.
- Prefer small structural improvements that reduce future change amplification over localized hacks.
- If a full redesign is too expensive, still move the code one step toward better ownership and lower obscurity.
- Do not spread existing complexity into new code. Contain it, adapt around it, and create cleaner seams.
- Leave the touched area easier to understand than it was before.
- Aim for the structure the system would have had if the new requirement had existed from the beginning.

## Modification strategy for legacy or messy code

1. Identify the specific complexity smell:
   duplication, pass-through state, unclear ownership, special-case branching, or weak documentation.
2. Decide whether to:
   improve an existing module, merge modules, introduce one stronger abstraction, or isolate legacy behavior behind a better interface.
3. Prefer the smallest change that improves the design directionally.
4. Avoid broad rewrites unless the current structure makes safe progress impossible.
5. Use comments to mark invariants and intent while the design is being improved, not to justify permanent confusion.
6. If constraints prevent the ideal refactoring now, choose the cleanest feasible intermediate step and leave a clearer seam for future cleanup.

## Design heuristics

- If a change requires touching many call sites, the interface is probably too wide or the knowledge is in the wrong place.
- If a module mostly forwards data elsewhere, it is probably shallow.
- If understanding a feature requires reading several files, information is probably leaking.
- If a general mechanism contains branches for one special case, the boundary is probably wrong.
- If a parameter is passed through several layers unchanged, the design probably needs a different owner.
- If code is easy to write but hard to explain, the design is probably tactical.
- If the unit of progress is a feature checklist rather than a cleaner abstraction, the design is drifting.

## Names should create a clear image

- Choose names that are precise, unambiguous, and intuitive.
- Prefer names that let a reader guess correctly what the thing is without reading its implementation.
- Avoid vague names such as `data`, `info`, `manager`, `result`, `state`, `handle`, or `process` unless the surrounding scope makes the meaning obvious.
- For booleans, prefer predicate-like names whose true/false meaning is obvious.
- Use names consistently: if one name means one thing in one place, keep that meaning everywhere.
- Do not reuse the same name for different concepts just because they feel similar.
- If it is hard to find a good name, treat that as a design warning, not just a naming problem.

## Consistency is a design tool

- Similar things should be done in similar ways.
- Different things should look different.
- Be consistent in names, structure, interfaces, comment style, invariants, and error behavior.
- When working in an existing codebase, follow established conventions unless there is a compelling reason to replace them everywhere.
- Do not introduce a "better" local convention that leaves the broader system inconsistent.
- Document important conventions and invariants where developers will naturally see them.
- Use tooling and review to enforce conventions where possible.

## Code should be obvious

- Write code so a first-time reader's first guess is usually correct.
- Favor clear names, clear structure, blank lines between major phases, and comments that expose non-obvious meaning.
- If a reviewer says something is not obvious, accept that it is not obvious and clarify the code or the comments.
- Prefer designs that reduce the amount of information a reader must hold in their head at once.
- Avoid generic containers, generic helpers, or reused patterns that hide the real domain meaning.
- Obvious code needs fewer comments; non-obvious code forces readers into reconstruction work.

## Commenting rules

- Document abstractions so a reader can use a module without reading its implementation.
- Document invariants, constraints, and why a choice was made.
- Document cross-module assumptions explicitly.
- Use different words than the code itself.
- Prefer high-value interface comments over verbose inline narration.
- Put comments near the code they describe.
- Keep comments in code, not only in commit messages or external discussion.
- Avoid duplicating the same documentation in multiple places.

## Interface comments vs. implementation comments

### Interface comments

Use interface comments on public functions, classes, modules, exported types, and important internal boundaries that other code must rely on.

Interface comments should explain:

- what the abstraction is for
- what an instance represents, if relevant
- what the caller must provide
- what the caller can assume in return
- important invariants or guarantees
- non-obvious constraints
- edge-case semantics when they affect correct usage
- side effects visible to callers
- exceptions that can escape
- preconditions
- dependencies between arguments
- high-level performance characteristics if they are visible and relevant to callers

Interface comments should not explain:

- step-by-step control flow
- obvious parameter mechanics visible in the signature
- implementation tricks that callers should not depend on
- private configuration knobs or internal protocol details

Good interface comments let a reader use the module correctly without reading its body.

### Implementation comments

Use implementation comments inside a module when the code alone does not reveal the reason for a choice.

Implementation comments should explain:

- what a block of code is accomplishing at a higher level
- why the code is structured this way
- why an alternative that seems simpler was rejected
- what invariant is being preserved at this point
- why ordering matters
- hidden dependencies on external behavior, performance, data layout, or failure modes

Implementation comments should not:

- narrate each line
- duplicate what the code already states clearly
- compensate for poor naming or weak structure that should be fixed instead

For longer methods, comment major phases and non-obvious loops. Focus on `what` and `why`, not `how`.

### Cross-module design comments

- If one design decision affects multiple modules, document it in one clear place close to the most natural point of discovery.
- If there is no natural home, keep a central design-notes location and point to it from the affected code.
- Do not duplicate the full explanation in every location.
- If changing a cross-module contract requires multiple coordinated edits, say so in the code where future developers will most likely add to it.

### Rule of thumb

- If the information is needed by a user of the module, it belongs in an interface comment.
- If the information is needed only by a maintainer of the module, it belongs in an implementation comment.
- If both users and maintainers need it, put the stable contract in the interface comment and the design rationale in the implementation comment.

## Maintaining comments while modifying code

- Keep interface comments adjacent to the code developers will edit.
- Push implementation comments down to the narrowest scope that still matches the code they describe.
- Before finishing a change, check that the comments still match the behavior.
- Put durable reasoning in the code, not only in the commit log.
- Prefer higher-level comments because they are both more useful and easier to keep accurate.

## Review checklist

When reviewing or generating code, actively look for:

- change amplification
- cognitive load
- unknown unknowns created by hidden coupling
- information leakage
- obscurity in names, structure, or docs
- shallow modules
- pass-through methods
- getters and setters that expose implementation state without adding a meaningful abstraction
- unnecessary exceptions and edge-case handling
- special-case logic infecting shared code
- long or awkward interface comments that signal a bad abstraction
- inconsistent naming, structure, or behavior
- code that is easy to write but hard to read
- feature-driven growth where an abstraction should have been designed

## Tests and refactoring

- Good tests support strategic programming because they make refactoring safer.
- Use tests to enable structural improvement, not as a substitute for design.
- Do not let test-driven, one-feature-at-a-time work replace abstraction design.
- When fixing a bug, prefer writing a failing test first, then fix the bug.

## Performance without sacrificing design

- Prefer designs that are naturally efficient and simple.
- Simplicity often improves performance by removing extra work, special cases, and layer crossings.
- Do not optimize based on intuition alone; measure first.
- If a performance change adds complexity, keep it only if it produces a meaningful measured win.
- Design around the critical path: make the common hot path as simple as possible and push special cases off that path.
- Preserve clean interfaces even when optimizing internals.

## Agent behavior defaults

- Prefer fewer, stronger abstractions over many tiny helpers.
- Prefer deleting complexity over modeling it.
- Prefer explicit ownership over shared implicit knowledge.
- Prefer a small number of obvious concepts over many clever ones.
- If forced to choose, accept slightly more internal implementation complexity to keep the public surface simple.
- Challenge fashionable practices by asking whether they reduce or increase complexity in this specific case.

## Stop conditions

Pause and redesign before continuing if:

- the same idea appears in multiple places
- the fix adds a flag, branch, or parameter that spreads across layers
- a wrapper exists only to preserve an already-awkward interface
- a "simple change" is no longer simple
- comments are needed to excuse confusing structure instead of clarifying a good one

## Working maxim

Make the system easier to think about after every change.
