# 03_TERMINOLOGY.md

> **Purpose:** Establish a single, consistent vocabulary that all future documentation and implementation must follow.
>
> This document defines canonical terms that both humans and AI coding models must use. Deviations without justification are not allowed.

---

## 1. Core Vocabulary (Used Throughout CharOS)

### 1.1 Foundation Terms

| Term | Category | Definition | Restrictions |
|------|----------|------------|--------------|
| **CharOS** | Platform | The CharOS platform/framework itself | Never use as a character name |
| **Nila** | Character | The default CharOS companion | Do not prefix with "Nila-" |
| **Companion** | Running Instance | A live CharOS instance interacting with a user | Do not call "instance" |
| **Platform** | Architecture | CharOS + default character + extensions | Not an app, not a service |
| **Assistant** | Alternative | Deprecated term, use CharOS/Companion instead | Do not use in code/api |

### 1.2 System Terms (NEVER invent synonyms)

#### Core Architecture

| Canonical Term | Abbreviation | Description | Must Use In |
|----------------|--------------|-------------|------------|
| **Orchestrator** | Orchestrator | Central coordinating component | Architecture docs, code comments |
| **Event Bus** | EventBus | Communication layer between components | Code, API references |
| **Character Provider** | CharacterProvider | Interface for character implementations | Plugin system, docs |
| **Memory Provider** | MemoryProvider | Interface for memory implementations | Plugin system, docs |
| **Reasoning Provider** | ReasoningProvider | Interface for AI reasoning models | Model routing, docs |
| **Skill Provider** | SkillProvider | Interface for skill implementations | Plugin system, docs |
| **Tool Provider** | ToolProvider | Interface for tool implementations | Plugin system, docs |
| **Model Router** | ModelRouter | Component selecting models based on task | Model routing docs |
| **Context Engine** | ContextEngine | Builds model context from multiple sources | Memory docs |
| **Task Graph** | TaskGraph | Dependency graph of tasks to execute | Planner docs |

#### Character System

| Canonical Term | Description | Related Files |
|----------------|-------------|---------------|
| **VRM Runtime** | VRM character rendering engine | character/VRM_RUNTIME.md |
| **Animation State** | Named character animation states | animations/ANIMATIONS.md |
| **Expression** | Facial blendshape/emotion state | character/CHARACTER_SPEC.md |
| **Theme** | Visual customization set | themes/THEMES.md |
| **Voice Pack** | Speech synthesis configuration | voice/VOICE.md |
| **Character Pack** | Portable bundle of character assets | CHARACTER_PACKS.md |

#### Memory System

| Canonical Term | Layer | Retention Time | Storage |
|----------------|-------|---------------|----------|
| **Working Memory** | Active | Session | In-memory cache |
| **Episodic Memory** | Events | Indefinite (until consolidation) | Provider backend |
| **Semantic Memory** | Knowledge | Long-term | Obsidian/Graph |
| **Consolidation** | Process | Daily | Memory provider |
| **Dreaming** | User-facing | Process name | Alias for consolidation |

#### UI System

| Canonical Term | Description | Related Files |
|----------------|-------------|---------------|
| **Overlay** | Transparent desktop character host | ui/OVERLAY.md |
| **Speech Bubble** | Floating dialogue UI | SPEECH_BUBBLES.md |
| **Dock** | Side-docked companion location | ui/WINDOWS.md |
| **Widget** | Small reusable UI component | N/A |
| **Shortcut** | Keyboard activation | ui/SHORTCUTS.md |

#### Plugin System

| Canonical Term | Category | Description |
|----------------|----------|-------------|
| **Plugin** | Extension | Adds new skills or integrations |
| **Provider** | Interface | Implements a capability behind an interface |
| **Adapter** | Bridge | Connects CharOS with external implementations |

---

## 2. Term Usage Rules

### 2.1 Do's

✅ Use exactly these terms in code comments, docs, filenames:

- `Orchestrator` instead of "Manager", "Controller", "Brain"
- `EventBus` instead of "MessageBus", "PubSub"
- `CharacterProvider` instead of "CharacterManager"
- `MemoryProvider` instead of "MemorySystem"
- `TaskGraph` instead of "Graph", "Plan"
- `SpeechBubble` instead of "Dialog", "MessageBox"

✅ Use terms consistently within files:

```typescript
// BAD - mixing terms
const taskPlanner = new TaskPlanner();
const taskGraph = new TaskGraph("MyGraph");

// GOOD - consistent
table orchestrator = new Orchestrator();
const taskGraph = new TaskGraph("MyGraph");
```

### 2.2 Don'ts

❌ Never use these forbidden terms:

- "Assistant", "AI Assistant", "Chatbot"
- "Character Manager", "Character Controller"
- "Skill Handler", "Skill Executor"
- "Tool Manager", "Operation Executor"
- "Memory System", "Storage Engine"
- "Config Manager", "Settings Controller"
- "Window Manager", "UI Controller"
- "Plugin Manager", "Extension Controller"

❌ Never invent new synonyms for existing terms:

- Don't create "Buddy" for "Character"
- Don't create "Guide" for "Companion"
- Don't create "Helper" for "Skill"

❌ Never use typos or abbreviations inconsistently:

- `OVRLAY` in one place, `overlay` in another
- `SPCH_BUBBLE` in filenames, `speech-bubble` in comments

---

## 3. Glossary (Reference)

### 3.1 Platform Vocabulary

| Term | Alias | When to Use |
|------|-------|-------------|
| **CharOS** | N/A | Platform, framework, project name |
| **Character** | N/A | Persona (Nila, etc.) |
| **Companion** | N/A | Running instance |
| **Skill** | N/A | High-level capability |
| **Tool** | N/A | Low-level system operation |
| **Plugin** | N/A | Extension mechanism |

### 3.2 AI/ML Vocabulary

| Term | Alias | Note |
|------|-------|------|
| **Model Provider** | ReasoningProvider | Synonym for LLM providers |
| **Vision Provider** | N/A | Computer vision |
| **Speech Provider** | N/A | Text-to-speech/speech-to-text |
| **Capability** | N/A | High-level skill capability |

### 3.3 State Machine Vocabulary

| Term | State | Event |
|------|-------|------|
| **Idle** | [`CharacterState`](#CharacterState) | - |
| **Listening** | [`CharacterState`](#CharacterState) | UserInputReceived |
| **Thinking** | [`CharacterState`](#CharacterState) | TaskDequeued |
| **Working** | [`CharacterState`](#CharacterState) | SkillExecuting |
| **Speaking** | [`CharacterState`](#CharacterState) | VoiceOutputStarted |
| **Planning** | [`CharacterState`](#CharacterState) | GoalDecomposed |

### 3.4 Communication Vocabulary

| Term | Event Type | Handler |
|------|-----------|----------|
| **TaskStarted** | Task Started | - |
| **TaskCompleted** | Task Finished | - |
| **MemoryUpdated** | Data Changed | MemoryEvents |
| **CharacterChanged** | Persona Changed | CharacterEvents |
| **SkillFinished** | Execution Complete | SkillEvents |

---

## 4. Naming Conventions (File & Directory)

### 4.1 Directory Structure

Each concept has ONE canonical file location:

```
plugins/{plugin-name}/                                  # Plugin root
plugins/{plugin-name}/manifest.json                    # Single plugin definition
plugins/{plugin-name}/src/                              # Implementation

plugins/{plugin-name}/dist/                              # Optional built output

character/                                              # Character system root
character/CHARACTER_SPEC.md                              # Default character spec
character/{character-pack}/                             # Character pack root
character/{character-pack}/{character-pack}.vrm          # VRM model
character/{character-pack}/personality.json              # Personality definition

memory/                                                # Memory system root
memory/{provider-name}/                                # Memory provider root
memory/{provider-name}/index.ts                         # Provider implementation

models/                                                # Model adapters root
models/{provider-name}/                                # Model provider root
models/{provider-name}/{model-name}/                    # Specific model config
```

### 4.2 File Naming Rules

| Concept | Filename Pattern | Examples |
|---------|------------------|----------|
| **Character** | `{character-name}.vrm` | `nila.vrm` |
| **Animation** | `{animation-name}.vrm` OR `.glb` | `idle_breathing.glb` |
| **Theme** | `{theme-name}.json` | `default.json` |
| **Plugin** | `manifest.json` | `skill-docker/manifest.json` |
| **Provider** | `index.ts` (JavaScript), `mod.rs` (Rust) | `providers/obsidian/index.ts` |
| **Skill** | `{skill-id}.json` | `docker.build.json` |

### 4.3 Reserved Words

These words have fixed meaning and CANNOT be reused:

- **Character**
- **Companion**
- **Planner**
- **Skill**
- **Tool**
- **Plugin**
- **Provider**
- **Adapter**
- **Memory**
- **Task**
- **Context**
- **Skill**
- **Tool**
- **Plugin**
- **Provider**
- **Adapter**
- **Memory**
- **Task**
- **Context**
- **Skill**

**Exception:** Words can be used as variable names or local variables, but not as class/interface names or public APIs.

---

## 5. Contract Vocabulary (Interface Names)

### 5.1 Interface Naming Pattern

```typescript
interface CharacterProvider {
    // Character capability
}

interface MemoryProvider {
    // Memory capability
}

interface SkillProvider {
    // Skill capability
}

interface ToolProvider {
    // Tool capability
}

interface PluginProvider {
    // Plugin capability
}
```

**Rule:** All interfaces must start with the capability they provide.

### 5.2 Function Naming Pattern

```typescript
// Verbs (action-oriented)
executeTask()
buildContext()
retrieveMemory()
registerPlugin()

// Not:
helperFunction()
utilFunction()
managerFunction()
processorFunction()
```

### 5.3 Event Naming Pattern

```typescript
type Event = 
    | TaskStarted
    | TaskCompleted
    | MemoryUpdated
    | CharacterChanged
    | SkillFinished
    // Prefix with verb + past tense
```

---

## 6. Cross-References

| Document | Field | Relationship |
|----------|-------|-------------|
| `docs/01_ARCHITECTURE.md` | 4.2 Subsystem Map | Full interface mappings |
| `docs/02_DESIGN_PHILOSOPHY.md` | Section 3 | Visual and interaction design |
| `docs/04_PROJECT_STRUCTURE.md` | Section 3 | Directory structure alignment |
| `docs/05_TECH_STACK.md` | All sections | Technology mappings |
| `docs/06_UI_GUIDELINES.md` | All UI terms | Implementation details |
| `docs/08_AI_GUIDELINES.md` | Model terms | Integration patterns |
| `docs/09_ROADMAP_DETAILS.md` | System terms | Timeline alignment |
| `character/CHARACTER_SPEC.md` | All character terms | Specification |
| `ai/PLANNER.md` | All task terms | Planner implementation |
| `memory/MEMORY.md` | All memory terms | Memory implementation |
| `plugins/PLUGIN_API.md` | All plugin terms | Plugin integration |
| `tools/TOOL_ROUTING.md` | All tool terms | Tool usage |
| `ai/AGENTS.md` | Related terms | Agent system |

---

## 7. Verification Script

### 7.1 grep -r "(Assistant\|Chatbot\|CharacterManager\|ToolHandler)" .

Should return **zero results**.

### 7.2 grep -r "CharacterProvider\|MemoryProvider\|SkillProvider\|ToolProvider"

Should return **all provider usages**.

### 7.3 grep -r "\.(overlay|bubble|animation|expression)\." ./src

Should return **consistent naming**.

### 7.4 File check for duplicates

```bash
# Find files with similar names
find . -type f -name "*.md" | grep -i "character\|companion" | sort

# Should see only legitimate variations
./docs/00_VISION.md
./docs/01_ARCHITECTURE.md
./docs/02_DESIGN_PHILOSOPHY.md
./docs/03_TERMINOLOGY.md
./character/CHARACTER_SPEC.md
./character/PERSONALITY.md
```

---

## 8. Open Design Questions

### 8.1 Language Ambiguity

| Option | Pros | Cons |
|--------|------|------|
| **English only** | Consistency, less complexity | Non-English users |
| **Localized UI + English code** | Better UX, maintainable code | Mixed interfaces |
| **Supporting multiple locales in code** | Accessibility | Complexity, maintenance |

**Status:** English only for code and internal documentation.

### 8.2 Pluralization

| Word | Plural Form |
|------|-------------|
| Character | Characters |
| Companion | Companions |
| Skill | Skills |
| Tool | Tools |
| Plugin | Plugins |
| Provider | Providers |
| Task | Tasks |
| Context | Contexts |
| Memory | Memories |
| SpeechBubble | SpeechBubbles |

**Rule:** No automatic pluralization; use explicit plurals in interfaces.

### 8.3 Acronym Usage

| Acronym | Full Form | Usage |
|---------|-----------|-------|
| **VCS** | Version Control System | Forbidden |
| **API** | Application Programming Interface | Allowed |
| **UI** | User Interface | Allowed |
| **HTTP** | Hypertext Transfer Protocol | Allowed |
| **JSON** | JavaScript Object Notation | Allowed |

**Rule:** Spell out acronyms in first use; use abbreviation in third use with comment.

---

## 9. Checklist for Contributors

### Before Adding New Code:

1. **Use canonical terms** from this file
2. **Check existing implementations** for duplication
3. **Update this file** if new concepts are introduced
4. **Verify naming consistency** across files
5. **Run verification script** to confirm no forbidden terms

### Before Submitting PR:

1. **Search for forbidden terms** (Assistant, Chatbot, Manager, Handler, Helper, Util, Data)
2. **Check naming uniformity** across the commit
3. **Verify file naming patterns**
4. **Update cross-references** if new files added
5. **Document new terms** if they become permanent

---

## 10. Continuous Maintenance

**Keep terminology current:**
- Review quarterly for drift
- Update documentation when architecture changes
- Clarify ambiguities when they arise
- Encourage contributors to report term changes

> **Term is more durable than code. Fixing a term is cheaper than fixing culture.**

---

> **This terminology document evolves with the project. Every new term must be reviewed for necessity and permanence.**

---

> **Exception:** Localization terms may be deferred until user interface design is complete.