# PROJECT_BIBLE.md

> **The Constitution of CharOS**
>
> This document defines the long-term vision, philosophy, and
> non-negotiable principles of the project. Every contributor---human or
> AI---should read this before making architectural decisions.

------------------------------------------------------------------------

# Mission

CharOS is an **open-source, local-first desktop companion platform**
that enables expressive AI characters to live alongside the user,
understand context, automate work, and build long-term knowledge while
respecting user privacy and control.

The default companion is **Nila**, but CharOS itself is
**character-agnostic**.

------------------------------------------------------------------------

# Vision

Most AI products are chat interfaces.

CharOS aims to become a **persistent operating companion**.

The assistant should feel like a helpful presence on the desktop---not
another website.

The user interacts with a character.

The character uses tools.

The models remain implementation details.

------------------------------------------------------------------------

# Core Goals

-   Local-first whenever practical.
-   Modular architecture.
-   Character-driven UX.
-   Extensible through plugins.
-   Open-source community.
-   Long-term memory.
-   Safe automation.
-   Beautiful desktop experience.
-   Replace repetitive computer tasks with natural interaction.

------------------------------------------------------------------------

# Non-Goals

CharOS is NOT:

-   another web chatbot
-   a prompt collection
-   tied to one LLM
-   tied to one avatar
-   dependent on one vendor
-   built around API lock-in

------------------------------------------------------------------------

# Project Philosophy

## The Character is the Product

Users remember the companion.

Not the model.

Not the framework.

Not the API.

The experience comes first.

------------------------------------------------------------------------

## Models are Replaceable

Speech, vision, reasoning and coding models will improve over time.

Every model must be replaceable through adapters.

Never tightly couple business logic to a specific model.

------------------------------------------------------------------------

## Local First

Prefer local execution whenever it provides an acceptable experience.

Cloud services are optional enhancements---not requirements.

------------------------------------------------------------------------

## Privacy by Default

Users own:

-   their memories
-   their notes
-   their conversations
-   their automation

CharOS should avoid unnecessary data collection.

------------------------------------------------------------------------

## Documentation First

Architecture is documented before implementation.

Future contributors should understand the project by reading
documentation rather than reverse-engineering code.

------------------------------------------------------------------------

# Design Principles

1.  Composition over inheritance.
2.  Interfaces over implementations.
3.  Configuration over hardcoded values.
4.  Small focused modules.
5.  Explicit data flow.
6.  Predictable behavior.
7.  Observable systems.
8.  Documentation is part of the codebase.

------------------------------------------------------------------------

# Architectural Principles

The following systems should remain independent whenever possible:

-   Character
-   Planner
-   Memory
-   UI
-   Models
-   Tool execution
-   Plugins

Communication should happen through well-defined interfaces.

------------------------------------------------------------------------

# User Experience Principles

The companion should feel:

-   friendly
-   calm
-   expressive
-   lightweight
-   responsive
-   respectful

Avoid overwhelming the user.

Animation communicates state.

Speech bubbles communicate intent.

The desktop remains the primary workspace.

------------------------------------------------------------------------

# Character System

A character defines:

-   appearance
-   voice
-   animations
-   personality
-   theme
-   expressions

A character does NOT define:

-   planner
-   memory implementation
-   model routing
-   automation engine

Those belong to the platform.

------------------------------------------------------------------------

# Memory Philosophy

Memory should improve usefulness---not accumulate noise.

Memory layers:

1.  Working Memory
2.  Episodic Memory
3.  Semantic Knowledge
4.  Consolidated Knowledge

Useful memories are promoted.

Temporary information is forgotten.

------------------------------------------------------------------------

# Agent Philosophy

The planner owns decisions.

Agents execute capabilities.

Skills perform actions.

Tools interact with the operating system.

Models provide reasoning.

Each layer has one responsibility.

------------------------------------------------------------------------

# Success Criteria

A successful CharOS release should allow a user to:

-   launch the companion with a shortcut
-   talk naturally
-   receive useful help
-   automate repetitive work
-   understand what the assistant is doing
-   trust the assistant
-   customize the experience
-   replace models without breaking workflows

------------------------------------------------------------------------

# Community Principles

Contributors should:

-   document decisions
-   discuss architecture before major changes
-   write understandable code
-   keep APIs stable when possible
-   value maintainability over cleverness

------------------------------------------------------------------------

# AI Contributor Rules

Every coding model should:

1.  Read README.md.
2.  Read roadmap.md.
3.  Read PROJECT_BIBLE.md.
4.  Read architecture documentation.
5.  Keep documentation synchronized with implementation.
6.  Avoid inventing undocumented architecture.
7.  Preserve modularity.
8.  Prefer reusable abstractions.

------------------------------------------------------------------------

# Long-Term Vision

CharOS should grow into an ecosystem where users can mix and match:

-   characters
-   voices
-   models
-   plugins
-   memory providers
-   themes
-   automation skills

without modifying the core platform.

------------------------------------------------------------------------

# Guiding Principle

> **People should remember the companion, not the model behind it.**

The goal of CharOS is not to showcase AI.

The goal is to create a trustworthy, delightful, capable desktop
companion that helps people accomplish meaningful work while remaining
transparent, extensible, and enjoyable to use.
