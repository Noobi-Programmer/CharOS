# NAMING.md

> **Purpose:** Establish a single, consistent vocabulary for CharOS.
>
> Humans and AI coding agents should use these terms consistently
> throughout the repository. Avoid inventing new names for existing
> concepts.

------------------------------------------------------------------------

# Core Principle

**One concept → One canonical name.**

Do not introduce synonyms unless there is a genuine semantic difference.

------------------------------------------------------------------------

# Platform Terminology

  -----------------------------------------------------------------------
  Canonical Name                                Meaning
  --------------------------------------------- -------------------------
  **CharOS**                                    The platform/framework

  **Character**                                 A companion persona
                                                (e.g. Nila)

  **Nila**                                      The default character
                                                shipped with CharOS

  **Companion**                                 A running Character
                                                instance interacting with
                                                the user
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# AI & Planning

  Name                     Meaning
  ------------------------ ------------------------------------------------
  **Planner**              Breaks goals into executable tasks
  **Orchestrator**         Coordinates planners, models, memory and tools
  **Task**                 A unit of work requested by the user
  **Task Graph**           Ordered/dependent collection of tasks
  **Reasoning Provider**   LLM used for planning/reasoning
  **Vision Provider**      Image understanding backend
  **Speech Provider**      Speech recognition/synthesis backend
  **Model Router**         Chooses the appropriate provider/model

Never call the Planner a "manager", "brain", or "controller" in code.

------------------------------------------------------------------------

# Skills & Tools

  Name           Meaning
  -------------- -----------------------------------------------------------
  **Skill**      High-level capability (Summarize, Backup, Refactor)
  **Tool**       Low-level executable operation (filesystem, browser, git)
  **Plugin**     Extension that adds new skills or integrations
  **Provider**   Implementation of a capability behind an interface
  **Adapter**    Bridges CharOS with an external implementation

Skill ≠ Tool.

A Skill may use multiple Tools.

------------------------------------------------------------------------

# Memory

  -----------------------------------------------------------------------
  Name                         Meaning
  ---------------------------- ------------------------------------------
  **Working Memory**           Active context for the current task

  **Episodic Memory**          Important events and experiences

  **Semantic Memory**          Long-term factual knowledge

  **Knowledge Graph**          Relationships between memories

  **Consolidation**            Daily process that summarizes and links
                               memories

  **Dreaming**                 User-facing name for consolidation (not a
                               separate system)

  **Context Engine**           Builds model context from relevant sources
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# Character System

  -----------------------------------------------------------------------
  Name                         Meaning
  ---------------------------- ------------------------------------------
  **Avatar**                   Rendered VRM model

  **Expression**               Facial blendshape/emotion

  **Animation State**          Idle, Thinking, Coding, Speaking, etc.

  **Theme**                    Visual customization

  **Voice Pack**               Speech synthesis configuration

  **Character Pack**           Bundle containing avatar, personality,
                               voice, animations and metadata
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# UI

  Name                Meaning
  ------------------- ---------------------------------------------
  **Overlay**         Transparent desktop UI
  **Speech Bubble**   Floating dialogue UI
  **Dock**            Default resting location on the screen edge
  **Widget**          Small reusable UI component
  **Shortcut**        Keyboard activation

Avoid calling the Overlay a "window" unless referring to the native OS
window.

------------------------------------------------------------------------

# Repository Conventions

## Classes

Prefer nouns.

Examples:

-   TaskPlanner
-   MemoryStore
-   PluginRegistry
-   CharacterState
-   ModelRouter

Avoid:

-   Helper
-   Utils
-   Stuff
-   Misc
-   DataManager

------------------------------------------------------------------------

## Interfaces

Prefix with capability.

Examples:

-   MemoryProvider
-   VisionProvider
-   SpeechProvider
-   PluginProvider

------------------------------------------------------------------------

## Events

Use past tense.

Examples:

-   TaskStarted
-   TaskCompleted
-   MemoryUpdated
-   CharacterChanged
-   SkillFinished

------------------------------------------------------------------------

## Functions

Use verbs.

Examples:

-   executeTask()
-   retrieveMemory()
-   buildContext()
-   renderCharacter()
-   registerPlugin()

------------------------------------------------------------------------

## Files

Use descriptive names.

Good:

-   MODEL_ROUTING.md
-   MEMORY.md
-   CHARACTER_SPEC.md

Avoid:

-   notes.md
-   temp.md
-   final2.md
-   misc.md

------------------------------------------------------------------------

# Reserved Words

These names have specific meanings and should not be reused for
unrelated concepts:

-   Character
-   Planner
-   Skill
-   Tool
-   Plugin
-   Provider
-   Adapter
-   Memory
-   Context
-   Overlay
-   Theme
-   Task

------------------------------------------------------------------------

# Glossary

**Character** → Personality + appearance presented to the user.

**Companion** → Running instance of a Character.

**Planner** → Decides *what* to do.

**Skill** → Knows *how* to solve a domain problem.

**Tool** → Performs a concrete system action.

**Provider** → Supplies an interchangeable capability.

**Plugin** → Extends CharOS without modifying the core.

------------------------------------------------------------------------

# Naming Checklist

Before introducing a new name, ask:

1.  Does this concept already exist?
2.  Is there a canonical term in this file?
3.  Is the name descriptive?
4.  Will humans and AI interpret it consistently?
5.  Does it avoid unnecessary abbreviations?

If the answer to any of these is "no", reconsider the name.

------------------------------------------------------------------------

# Guiding Principle

> **Consistency is more valuable than clever naming.**

A contributor should be able to understand the project by reading names
alone.
