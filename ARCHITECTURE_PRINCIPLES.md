# ARCHITECTURE_PRINCIPLES.md

> **Purpose:** Define the non-negotiable architectural rules for CharOS.
>
> These principles should outlive frameworks, models, and implementation
> details. Any proposal that violates these principles should be
> questioned before implementation.

------------------------------------------------------------------------

# 1. Architecture Before Code

Documentation defines the architecture.

Implementation follows documentation.

Never allow code to become the source of truth.

------------------------------------------------------------------------

# 2. Separation of Concerns

Every subsystem has a single responsibility.

  Subsystem   Responsibility
  ----------- -------------------------------------------------
  Character   Appearance, animation, personality, expressions
  Planner     Task decomposition and execution planning
  Memory      Storage, retrieval and consolidation
  Models      Reasoning, coding, speech, vision
  Skills      Domain-specific capabilities
  Tools       Operating-system interactions
  UI          Rendering and interaction
  Plugins     Extending the platform

------------------------------------------------------------------------

# 3. Character ≠ Intelligence

The character is a presentation layer.

It must never contain business logic.

Changing the character should not require changing:

-   planner
-   memory
-   model routing
-   skills
-   plugins

------------------------------------------------------------------------

# 4. Models are Replaceable

Every AI capability must be accessed through an abstraction.

Never couple the system to a specific model.

Instead of:

    Planner -> Gemma

Prefer:

    Planner -> ReasoningProvider

------------------------------------------------------------------------

# 5. Tool-Driven Architecture

The planner decides **what** to do.

Skills decide **how** to accomplish a domain task.

Tools interact with the operating system.

Models provide reasoning---not direct control of the system.

------------------------------------------------------------------------

# 6. Interface First

Core modules communicate through interfaces and contracts.

Benefits:

-   testing
-   replacement
-   mocking
-   future compatibility

------------------------------------------------------------------------

# 7. Local First

Prefer local execution whenever practical.

Cloud providers are optional enhancements.

The system must remain functional without cloud services.

------------------------------------------------------------------------

# 8. Event-Driven Communication

Prefer events over tightly coupled calls.

Examples:

-   TaskStarted
-   TaskCompleted
-   MemoryUpdated
-   CharacterStateChanged

This improves extensibility and observability.

------------------------------------------------------------------------

# 9. Layered Memory

Memory should be separated into:

1.  Working memory
2.  Episodic memory
3.  Semantic knowledge
4.  Consolidated knowledge

Each layer has different retention and retrieval rules.

------------------------------------------------------------------------

# 10. Context Assembly

Context is constructed from multiple sources:

-   current task
-   recent interactions
-   relevant memories
-   project knowledge
-   retrieved notes

Never dump all history into a prompt.

------------------------------------------------------------------------

# 11. Explicit State Machines

Long-running systems should use explicit states.

Example character states:

-   Idle
-   Listening
-   Thinking
-   Planning
-   Working
-   Speaking
-   Error
-   Sleeping

Avoid hidden state transitions.

------------------------------------------------------------------------

# 12. Configuration Over Constants

Names, themes, models, voices, and providers belong in configuration.

Business logic should not depend on hardcoded values.

------------------------------------------------------------------------

# 13. Plugin-Centric Design

Core should remain small.

Additional functionality belongs in plugins or skills.

Core should define APIs---not every feature.

------------------------------------------------------------------------

# 14. Documentation is Part of the Product

Architecture changes require documentation updates.

A pull request that changes architecture without documentation is
incomplete.

------------------------------------------------------------------------

# 15. Testability

Every subsystem should be independently testable.

Design for:

-   dependency injection
-   mocks
-   deterministic behavior

------------------------------------------------------------------------

# 16. Security by Design

Assume tools may perform sensitive operations.

Principles:

-   least privilege
-   explicit permissions
-   auditability
-   confirmation for destructive actions

------------------------------------------------------------------------

# 17. Human-Centered UX

The user should think:

"I asked Nila."

Not:

"I selected model X."

Routing, orchestration, and providers are implementation details.

# 18. NemoClaw Foundation

CharOS builds on top of NemoClaw.

CharOS must never reimplement what NemoClaw provides:

-   sandboxed agent runtime (OpenShell)
-   inference routing and provider management
-   network policy enforcement
-   blueprint orchestration

CharOS communicates with NemoClaw through its CLI or API---not directly with model providers.

------------------------------------------------------------------------

# 19. Backward Compatibility

Public interfaces should evolve carefully.

Deprecate before removing.

Document breaking changes.

-----------------------------------------------------------------------

# 20. Open-Source Friendly

Optimize for readability over cleverness.

Future contributors should understand the project without needing its
original authors.

-----------------------------------------------------------------------

# 21. Architectural Decision Records (ADR)

Every major decision should produce an ADR.

Examples:

-   framework selection
-   memory backend
-   plugin architecture
-   rendering engine
-   IPC strategy

------------------------------------------------------------------------

# Decision Checklist

Before introducing a new subsystem, ask:

-   Does it violate separation of concerns?
-   Can it be replaced independently?
-   Is there a clear interface?
-   Is it documented?
-   Does it preserve local-first philosophy?
-   Does it improve the user experience?
-   Can another contributor understand it?

If the answer to any critical question is "no", reconsider the design
before implementation.

------------------------------------------------------------------------

# Guiding Principle

> **CharOS is a platform, not a collection of AI models.**

Models, frameworks, and providers will change.

A clean architecture should not.
