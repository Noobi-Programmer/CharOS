# CODING_PRINCIPLES.md

> **Purpose:** Define the engineering standards for CharOS.
>
> This document is the coding constitution for every human contributor
> and AI coding model.

------------------------------------------------------------------------

# Guiding Philosophy

Code is written for **future contributors**, not just the current
author.

Prioritize:

-   readability
-   maintainability
-   modularity
-   documentation
-   testability

Avoid clever code that is difficult to understand.

------------------------------------------------------------------------

# Golden Rules

1.  Documentation before architecture changes.
2.  Architecture before implementation.
3.  Small, focused modules.
4.  Explicit interfaces.
5.  Composition over inheritance.
6.  Configuration over hardcoded values.
7.  Stable public APIs.
8.  Every subsystem has a single responsibility.

------------------------------------------------------------------------

# Project Structure

Each directory should have a clearly defined purpose.

Do not place unrelated functionality in the same module.

Prefer:

``` text
planner/
memory/
character/
skills/
plugins/
models/
ui/
```

Avoid:

``` text
utils/
helpers/
misc/
common/
```

unless the contents are genuinely generic.

------------------------------------------------------------------------

# File Size

Guidelines:

-   Target: 100--300 lines
-   500+ lines should trigger refactoring
-   1000+ lines require architectural justification

Large files usually indicate multiple responsibilities.

------------------------------------------------------------------------

# Naming

Prefer descriptive names.

Good:

-   CharacterState
-   TaskPlanner
-   MemoryProvider
-   PluginRegistry

Avoid:

-   Manager
-   Handler
-   Helper
-   Util
-   Data

unless their purpose is well defined.

------------------------------------------------------------------------

# Interfaces First

Depend on abstractions, not implementations.

Example:

``` text
Planner
    ↓
ReasoningProvider
```

instead of:

``` text
Planner
    ↓
SpecificModel
```

This enables testing and future replacement.

------------------------------------------------------------------------

# Dependency Direction

Dependencies should flow inward.

UI → Planner → Skills → Tools

The reverse should never occur.

Business logic must not depend on the UI.

------------------------------------------------------------------------

# Error Handling

Do not silently ignore failures.

Every error should:

-   contain context
-   be logged appropriately
-   expose actionable information

Avoid generic:

``` text
Something went wrong.
```

Prefer:

``` text
Failed to open memory provider: database unavailable.
```

------------------------------------------------------------------------

# Logging

Logs should help debugging.

Levels:

-   TRACE
-   DEBUG
-   INFO
-   WARN
-   ERROR

Do not log sensitive user data.

------------------------------------------------------------------------

# Configuration

Everything likely to change belongs in configuration:

-   model names
-   ports
-   themes
-   colors
-   shortcuts
-   providers
-   feature flags

Do not hardcode them.

------------------------------------------------------------------------

# Comments

Write comments explaining **why**, not **what**.

Bad:

``` text
Increment counter.
```

Good:

``` text
The counter prevents duplicate execution after reconnects.
```

------------------------------------------------------------------------

# Documentation

Whenever architecture changes:

Update:

-   roadmap
-   architecture docs
-   relevant subsystem documentation

Code and documentation should evolve together.

------------------------------------------------------------------------

# Testing Philosophy

Prefer:

-   unit tests
-   integration tests
-   deterministic behavior

Avoid relying solely on manual testing.

------------------------------------------------------------------------

# AI Coding Rules

Every AI coding model should:

1.  Read the project documentation first.
2.  Preserve existing architecture.
3.  Avoid introducing duplicate abstractions.
4.  Keep changes focused.
5.  Explain architectural trade-offs.
6.  Update documentation when required.

Never rewrite unrelated code.

------------------------------------------------------------------------

# Pull Request Checklist

Before merging:

-   Does the change follow the architecture?
-   Is it documented?
-   Are names consistent?
-   Is the module testable?
-   Does it introduce unnecessary coupling?
-   Can another contributor understand it?

------------------------------------------------------------------------

# Technical Debt

When shortcuts are necessary:

-   document them
-   isolate them
-   create follow-up tasks

Never hide technical debt.

------------------------------------------------------------------------

# Security

Assume automation can affect user systems.

Protect:

-   files
-   credentials
-   browser sessions
-   memory
-   plugins

Require explicit confirmation for destructive actions.

------------------------------------------------------------------------

# Performance

Optimize only after correctness.

Measure before optimizing.

Avoid premature optimization.

------------------------------------------------------------------------

# Contributor Mindset

Every commit should leave the project:

-   easier to understand
-   easier to extend
-   better documented
-   more maintainable

------------------------------------------------------------------------

# Final Principle

> **Good architecture enables good code. Good code preserves good
> architecture.**
