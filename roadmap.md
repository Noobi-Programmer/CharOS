# CharOS / CompanionOS Roadmap

## What this project is

CharOS is an open-source desktop companion platform: a **character-driven AI operating layer** for Linux/Windows that feels like a living assistant instead of a chat app.

Default character: **Nila**  
Default vibe: cute, funny, helpful, side-docked companion with VRM/avatar support, speech, vision, memory, and agentic automation.

This repo should be reusable for other characters too.

---

## Core vision

The platform should do this:

1. Listen to voice or text.
2. Understand the intent.
3. Route the task to the right local model or cloud fallback.
4. Use tools safely.
5. Remember useful things.
6. Show progress through the character UI.
7. Feel lightweight, fun, and useful.

The product is not “a chatbot.”
It is a **desktop companion system**.

---

## Design principles

- Local-first by default.
- Privacy-aware.
- Character-first UX.
- Tool-driven, not prompt-dump driven.
- Modular and swappable.
- Open source friendly.
- Easy for future coding models to continue from the repo.
- Small, stable abstractions instead of one giant file.
- No model switching exposed to the user unless needed.
- Keep the assistant cute, fast, and non-intrusive.
- Build with future contributors in mind.

---

## Persona / UX theme

Nila is a side-of-screen companion.

Visual style ideas:

- transparent / invisible background
- side-docked companion
- pink speech bubbles
- cute, funny, expressive animations
- thinking pose
- coding pose with a laptop
- searching pose with magnifying glass
- idle, listening, happy, confused, error, sleep, typing states
- soft, friendly, playful, not cluttered

The UI should feel like:
- a desktop buddy
- a virtual assistant
- a helpful presence
- not a bulky chatbot window

---

## Model / capability roles

### Speech
- **Handy-Parakeet v3**: speech-to-text only.

### Local reasoning / assistant tasks
- **Gemma 4 Heretic**: general assistant, planning, summaries, everyday tasks.
- **Qwen3 Coder Heretic 30B A3B**: coding, bug fixes, repo work, tool execution, refactors, multi-step agent work.
- **Qwen2-VL**: vision, screenshots, UI understanding, image/PDF reading.

### Cloud fallback
Use logged-in browser automation only when the task is too big for local models and the user already has access.
Examples:
- ChatGPT
- Gemini
- GLM

Browser automation should be treated as a fallback, not the default.

---

## Memory stack

This project should not rely on raw chat history alone.

Use layered memory:

### Working memory
Current task, current files, current commands.

### Episodic memory
Important events and decisions:
- repo created
- model choices
- bug fixes
- user preferences
- working workflows

### Semantic memory
Curated knowledge in Obsidian markdown.

### Consolidation / “dreaming”
A daily process that:
- summarizes the day
- removes duplicates
- links related notes
- promotes useful memories to Obsidian
- forgets junk

Recommended memory tools:
- Obsidian for long-term notes
- Cognee / graph-like memory for relations
- Optional search index for retrieval

---

## Repository rules

### Step 0
Create a **public Git repo** first.

Before large implementation:
- initialize repo
- add license
- add README
- add roadmap
- add contributing guide
- add clear architecture notes
- make the repo easy for other models and humans to continue

### Important rule
Never let the project become “mystery code.”
Every major subsystem must be documented.

---

## Suggested folder structure

```text
CharOS/
├── README.md
├── roadmap.md
├── LICENSE
├── CONTRIBUTING.md
├── SECURITY.md
├── .gitignore
├── docs/
│   ├── vision.md
│   ├── architecture.md
│   ├── decisions/
│   ├── memory.md
│   ├── ui-theme.md
│   ├── model-routing.md
│   └── plugins.md
├── apps/
│   ├── desktop/
│   ├── overlay/
│   ├── settings/
│   └── character-preview/
├── core/
│   ├── orchestrator/
│   ├── planner/
│   ├── state-machine/
│   ├── permissions/
│   └── event-bus/
├── agents/
│   ├── speech/
│   ├── vision/
│   ├── coding/
│   ├── research/
│   └── memory/
├── models/
│   ├── local/
│   ├── cloud/
│   └── routing/
├── memory/
│   ├── obsidian/
│   ├── episodic/
│   ├── semantic/
│   └── consolidation/
├── character/
│   ├── vrm/
│   ├── animations/
│   ├── expressions/
│   ├── voice/
│   └── themes/
├── skills/
│   ├── browser/
│   ├── filesystem/
│   ├── terminal/
│   ├── git/
│   ├── obsidian/
│   └── mcp/
├── plugins/
│   ├── calendar/
│   ├── notes/
│   ├── web/
│   ├── media/
│   └── custom/
├── assets/
│   ├── vrm/
│   ├── icons/
│   ├── ui/
│   └── sounds/
└── tests/
```

---

## Architecture guide

### 1. Input layer
- mic
- keyboard
- hotkey
- optional screen capture

### 2. Understanding layer
- STT
- intent classification
- vision parsing
- context assembly

### 3. Orchestration layer
- decides what to do
- picks tools
- picks model
- creates plan
- runs steps

### 4. Execution layer
- browser
- terminal
- file system
- git
- MCP tools
- browser fallback for cloud assistants

### 5. Memory layer
- recent state
- important events
- Obsidian notes
- consolidation jobs

### 6. Presentation layer
- VRM avatar
- speech bubbles
- status animations
- side dock overlay

---

## UI behavior rules

- The character should appear from the side.
- The background should be invisible or transparent.
- Use compact pink speech bubbles.
- Show animation changes for thinking, coding, searching, listening, idle, happy, error, sleep.
- Keep the UI lightweight and non-blocking.
- Never cover the whole screen unless explicitly requested.
- The assistant should feel present, not annoying.

---

## State machine ideas

Recommended states:

- idle
- listening
- thinking
- planning
- coding
- searching
- speaking
- waiting
- success
- error
- sleeping

Transitions should be deterministic and logged.

---

## Skill system rules

Every skill should be a small module with:
- name
- purpose
- inputs
- outputs
- permissions
- fallback behavior
- test cases

Examples:
- summarize lecture
- search web
- fix code
- read screenshot
- organize notes
- create commit
- backup files

---

## Memory rules

Do not store everything.

Store:
- decisions
- preferences
- project milestones
- recurring workflows
- important errors
- durable facts

Do not store:
- spammy outputs
- temporary logs
- one-off noise
- trivial chatter

Daily consolidation should:
- compress
- merge duplicates
- link related notes
- upgrade useful notes into Obsidian
- discard junk

---

## Coding rules for future models

Any model working in this repo should obey:

1. Read `roadmap.md` first.
2. Read `docs/architecture.md` before changing structure.
3. Prefer small, testable changes.
4. Avoid monolithic files.
5. Keep UI, core logic, memory, and model routing separated.
6. Document every new subsystem.
7. Add tests or clear verification steps for non-trivial changes.
8. Preserve backward compatibility when possible.
9. Never hardcode model names into business logic unless routed through config.
10. Keep the character layer independent from the AI engine.
11. Use config files for names, colors, avatars, voices, and model choices.
12. Keep prompts short, explicit, and reusable.
13. Prefer interfaces over direct dependencies.
14. Never break the public repo’s clarity.

---

## Open-source rules

This project should be easy for others to fork and customize.

Support:
- alternate characters
- alternate voices
- alternate themes
- alternate model providers
- alternate memory backends
- alternate automation backends

The default character is Nila, but the platform should support other characters cleanly.

---

## Recommended tech direction

A practical starting stack:

- **TypeScript** for app orchestration and UI
- **Tauri** or another lightweight desktop shell
- **React** for UI
- **Three.js / VRM tooling** for avatar rendering
- **Ollama** for local models
- **MCP** for tools
- **Obsidian markdown** for durable memory
- **Playwright** for logged-in browser fallback tasks
- **Qwen2-VL** for vision

The exact stack can change, but the structure should stay modular.

---

## Roadmap

### Step 0 — Public repo
- Create public Git repo
- Add license
- Add README
- Add roadmap
- Add architecture docs
- Add contributing guide

### Step 1 — Skeleton app
- app shell
- side dock
- hotkey open/close
- placeholder avatar
- speech bubble UI

### Step 2 — Voice input
- STT integration
- push-to-talk
- wake hotkey
- transcript display

### Step 3 — Character runtime
- idle / listening / thinking / speaking states
- simple avatar animation hooks
- text bubble reactions

### Step 4 — Local assistant core
- prompt orchestration
- tool routing
- model adapters
- command execution

### Step 5 — Memory
- obsidian integration
- episodic store
- consolidation job
- retrieval from memory

### Step 6 — Skills
- filesystem
- terminal
- git
- browser
- search
- notes

### Step 7 — Vision
- screenshots
- OCR
- UI reading
- image understanding

### Step 8 — Cloud fallback
- logged-in browser automation
- prompt submission
- response extraction
- confidence-based escalation

### Step 9 — Polish
- animations
- sounds
- theme packs
- character packs
- plugin system

### Step 10 — Open-source readiness
- docs
- examples
- templates
- contribution guide
- plugin docs
- character pack docs

---

## Definition of done for the MVP

The MVP is complete when:

- hotkey opens Nila
- voice input works
- text input works
- local model responds
- Nila can speak or show text
- the character animates states
- memory can save and retrieve useful notes
- a simple skill can execute a real task
- repo docs are good enough for another model to continue work

---

## Prompt for coding models

Use this as the standard instruction for future coding agents:

> You are working inside the CharOS repository. Read `roadmap.md` first, then read the relevant docs before editing code. This project is a modular, open-source desktop companion platform with a side-docked VRM character, agentic orchestration, memory, skills, and optional browser fallback. Keep changes small, well-structured, and documented. Prefer clean abstractions over hacks. Do not couple the UI, memory, model routing, and automation engine together. Preserve support for alternate characters and alternate models. Update docs when architecture changes. Add verification steps or tests for any non-trivial change. The default character is Nila, but the platform must remain character-agnostic. The assistant should feel cute, fast, and useful, not like a bulky chatbot.

---

## Final note

This file is the anchor for future work.
Keep it updated whenever the architecture changes.
