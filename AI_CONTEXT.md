# AI_CONTEXT.md

> **Purpose:** This file is written specifically for AI coding agents
> (OpenCode, Qwen, Nemotron, GPT, Gemini, Claude, etc.). It provides the
> current state of the project and the rules every model should follow
> before making changes.

------------------------------------------------------------------------

# Read Order (Mandatory)

Before making any change, read the following in order:

1.  `README.md`
2.  `roadmap.md`
3.  `PROJECT_BIBLE.md`
4.  `AI_CONTEXT.md` (this file)
5.  `docs/AGENT_EXECUTION_BRIEF.md`
6.  Relevant documents under `docs/`

Do **not** assume undocumented behavior.

------------------------------------------------------------------------

# Project Status

**Project phase:** Documentation-first

Current objective:

-   Build a complete architectural specification.
-   Delay implementation until the architecture is stable.
-   Produce documentation that can guide both humans and AI coding
    agents.

The documentation is the current source of truth.

For active implementation priorities and contributor operating rules, read
`docs/AGENT_EXECUTION_BRIEF.md`. It establishes the free/open-source-first
baseline and the mandatory foundation-first implementation order.

------------------------------------------------------------------------

# Current Milestone

## Milestone 0

Repository initialization.

Completed:

-   README.md
-   roadmap.md
-   PROJECT_BIBLE.md
-   AI_CONTEXT.md

Next priorities:

-   Architecture documentation
-   Character specification
-   Memory architecture
-   Planner architecture
-   Plugin system
-   MCP integration design

------------------------------------------------------------------------

# What CharOS Is

CharOS is an open-source platform for AI desktop companions.

The default companion is **Nila**, but the platform itself is
character-agnostic.

The project combines:

-   local AI
-   VRM characters
-   desktop automation
-   long-term memory
-   agentic planning
-   expressive UI

------------------------------------------------------------------------

# Long-Term Vision

The user should never think about model selection.

Instead:

User → Character → Planner → Tools → Result

Model routing is an implementation detail.

------------------------------------------------------------------------

# Planned Capability Stack

Speech: - Handy-Parakeet

General reasoning: - Gemma 4 Heretic

Coding: - Qwen3 Coder Heretic 30B A3B

Vision: - Qwen2-VL

Optional browser fallback: - User-owned logged-in AI services via
browser automation

The platform must allow these to be replaced.

------------------------------------------------------------------------

# Current Constraints

-   Local-first philosophy.
-   Avoid vendor lock-in.
-   Keep modules independent.
-   Documentation before implementation.
-   Open-source friendly.
-   Easy for future contributors.

------------------------------------------------------------------------

# Repository Rules

Never:

-   hardcode provider names into core logic
-   mix UI with planner logic
-   mix memory with rendering
-   tightly couple the character with AI providers
-   create undocumented architecture

Always:

-   document architecture changes
-   keep modules small
-   use interfaces
-   keep names consistent

------------------------------------------------------------------------

# Architectural Boundaries

These systems should remain independent:

-   UI
-   Character
-   Planner
-   Memory
-   Models
-   Skills
-   Plugins
-   Tool execution

Communication should happen through explicit interfaces.

------------------------------------------------------------------------

# Coding Philosophy

Prefer:

-   readable code
-   small commits
-   composition
-   dependency injection
-   configuration over constants

Avoid:

-   giant files
-   circular dependencies
-   hidden state
-   magic values

------------------------------------------------------------------------

# Documentation Philosophy

Every subsystem should eventually have its own Markdown specification.

Documentation should answer:

-   Why?
-   What?
-   How?
-   Extension points
-   Limitations

------------------------------------------------------------------------

# AI Contributor Checklist

Before editing:

-   Read the architecture.
-   Check for existing conventions.
-   Preserve modularity.
-   Avoid duplicate functionality.
-   Update documentation if behavior changes.

Before submitting changes:

-   Can another model understand this?
-   Can another developer extend this?
-   Does it align with PROJECT_BIBLE.md?

------------------------------------------------------------------------

# Known Unknowns

The following are intentionally undecided:

-   Final desktop framework
-   Final UI toolkit
-   Memory backend implementation
-   Plugin loading mechanism
-   Character asset pipeline
-   Voice synthesis provider
-   Packaging strategy

Document decisions before implementing them.

------------------------------------------------------------------------

# Current Success Metric

The repository is successful when:

-   A new contributor can understand the project by reading
    documentation.
-   An AI coding model can continue work without external explanation.
-   Architecture remains coherent as the project grows.

------------------------------------------------------------------------

# Guiding Instruction

If there is uncertainty:

**Do not invent architecture.**

Instead:

1.  Record the ambiguity.
2.  Propose alternatives.
3.  Explain trade-offs.
4.  Wait for the project owner to choose if the decision affects the
    core architecture.
