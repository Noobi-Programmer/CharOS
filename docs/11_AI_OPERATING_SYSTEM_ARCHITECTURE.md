# CharOS AI Operating System Architecture

**Status:** Proposed  
**Audience:** CharOS maintainers and implementation agents  
**Decision horizon:** Milestones 1–4; review experimental dependencies quarterly.

CharOS is a local-first, character-driven AI operating companion. Its architecture must not be an autonomous shell with an avatar, or a chat product with plugins. The product is a dependable execution platform that lets the character experience evolve while data, policy, tools, and models remain replaceable.

## Open-source and zero-cost policy

CharOS is designed first for broke students, independent engineers, and contributors who cannot rely on recurring API or hosting spend. A complete useful installation must work with **free, local, open-source components**. Paid services are optional capability upgrades, never a product requirement or the only implementation path.

The project evaluates every dependency in this order:

1. **Already available:** use the user's existing hardware, local tools, and accounts.
2. **Free and local:** prefer Ollama, SQLite, local files, local embeddings, and local execution.
3. **Free and open source:** prefer self-hostable components with portable data and public source code.
4. **Free hosted:** allow rate-limited/free services only as optional adapters, never as the only route.
5. **Paid:** add only after a documented local/free limitation, an explicit user choice, and a configuration switch that preserves the free path.

No feature may require a credit card, vendor account, proprietary model, cloud database, or paid hosting to be considered part of the baseline CharOS experience. The project should document hardware requirements honestly: zero-cost software does not make GPU compute, storage, electricity, or maintenance free.

### Baseline free stack

| Capability | Baseline | Paid/hosted upgrade only when justified |
| --- | --- | --- |
| Reasoning and coding | Ollama with user-selected local models | Cloud adapter when local quality/latency is insufficient and user consents |
| Persistence | SQLite | PostgreSQL for shared, concurrent deployments |
| Knowledge vault | Markdown / Obsidian-compatible vault | Optional sync service chosen by user |
| Semantic search | FastEmbed or local Ollama embeddings with a local index | Hosted embeddings only with explicit policy approval |
| Graph extraction | Optional local Cognee configuration | Managed graph/database only at demonstrated scale |
| Worker isolation | Local processes and Docker | Hosted runners only when local capacity is inadequate |
| Voice | Windows-hosted Handy STT and KittenTTS through a local host adapter | Paid voice providers as opt-in alternatives |
| Collaboration | Git + GitHub/GitLab-compatible remotes | Hosted automation/integrations only where their free plan or budget permits |

This policy is a product constraint: every new paid or cloud dependency must state its free/local replacement, data exposure, cost trigger, and migration path in an ADR.

## Contents

1. [Executive decisions](#executive-decisions)
2. [System architecture](#system-architecture)
3. [Runtime](#runtime)
4. [Memory](#memory)
5. [Multi-agent execution](#multi-agent-execution)
6. [Model routing](#model-routing)
7. [Technology evaluation](#technology-evaluation)
8. [Voice, permissions, and safety](#voice-permissions-and-safety)
9. [Environment plan](#environment-plan)
10. [Installation roadmap](#installation-roadmap)
11. [Risks and future vision](#risks-and-future-vision)

## Executive decisions

| Decision | Recommendation | Rationale |
| --- | --- | --- |
| Runtime | Small event-driven supervisor with isolated workers | A stable core outlives any agent framework. |
| Primary inference | Ollama on user-controlled hardware | Local, private, offline-capable, configurable. |
| Cloud inference | Consent-gated provider adapters | Quality fallback without a mandatory cloud control plane. |
| Agent collaboration | Planner-mediated typed artifacts | More observable and safer than agent chat swarms. |
| Initial persistence | SQLite plus an Obsidian-compatible vault | Lowest operational burden and human-readable knowledge. |
| Graph/retrieval | Derived local index; Cognee as an adapter | A graph must never be the sole source of truth. |
| Tool protection | Capability grants, approval receipts, audit trail | Models request power; they never grant it to themselves. |
| Desktop | Tauri host, Rust boundary, TypeScript UI | Good desktop integration with a narrow native boundary. |
| Sandboxing | Containers for high-risk workers | Evaluate NemoClaw/OpenShell; do not make alpha software core. |
| NotebookLM | User-operated research import/export only | No stable public API should be a runtime dependency. |

## System architecture

```text
User / Nila UI / Voice
          |
          | typed intent and streamed state
          v
+------------------------+
| Executive Planner      |
| policy-aware planning  |
+-----+-------------+----+
      |             |
      v             v
Context Engine   Task Supervisor <---- Approval Engine
      |             |
      |             +----------+
      v                        v
+-------------+         +-------------+
| Memory      |         | Event Bus   |
| Service     |         | task events |
+-------------+         +------+------+ 
                                |
        +-----------------------+------------------------+
        |                       |                        |
        v                       v                        v
 Model Router              Worker Pool              Tool Registry
 local/cloud              isolated workers          schemas + risk
        |                       |                        |
 Ollama/adapters          task artifacts         OS / MCP / SSH / Git
```

UI, character packs, model adapters, tool adapters, and plugins depend on stable core contracts. The planner must not import a provider SDK; providers must not directly write the UI or durable memory.

## Runtime

The CharOS Supervisor owns task lifecycle, policy, recovery, and observability. It borrows versioned policies and workload isolation from modern sandboxed-agent systems, but is independent from Hermes and NemoClaw.

```text
intent.received
    |
    v
validate -> classify risk -> create TaskRecord
    |
    v
Planner emits immutable TaskPlan
    |
    +-- approval required --> wait for ApprovalReceipt --> resume plan revision
    |
    v
lease worker --> execute step --> validate result --> write artifact
    |                  |                 |
    |                  v                 v
    |           capability check      task event
    v
complete --> reflection proposal --> reviewed memory promotion
```

| Component | Responsibilities | Cannot do |
| --- | --- | --- |
| Executive Planner | Produce bounded plans and conflict decisions | Execute tools or approve itself |
| Task Supervisor | Durable state, retries, leases, cancellation | Infer user permission |
| Context Engine | Retrieve minimum source-attributed context | Keep unbounded raw history |
| Worker Pool | Run cancellable, isolated steps | Escalate privileges |
| Tool Registry | Version schemas, risks, target scopes | Hide side effects |
| Permission Engine | Validate grants and record approvals | Generate plans |
| Event Bus | Deliver typed, idempotent domain events | Become a second database |
| Reflection Service | Propose summaries and preferences | Silently commit sensitive memories |
| Model Router | Select eligible model/provider | Override data policy |

Start in-process with typed events. Separate workers only when a task is long-running, untrusted, or resource-heavy. Distributed infrastructure before proven workload is complexity, not scalability.

### Event bus contract

```text
UI -> intent.received -> planner.plan.created -> task.queued
                                                |
                      +-------------------------+---------------------+
                      |                                               |
              approval.required                               worker.leased
                      |                                               |
              approval.granted                               tool.requested
                      |                                               |
                      +------------> task.running <------------------+
                                                   |
                  artifact.created / task.progressed / tool.failed
                                                   |
                                      retry, halt, or task.completed
                                                   |
                                    reflection.proposed -> memory.reviewed
```

Task-affecting events are persisted before delivery. Each carries event ID, task ID, causation ID, correlation ID, actor, timestamp, schema version, and sensitivity label. Handlers must be idempotent.

## Memory

Memory is a set of stores with different owner, provenance, and expiration rules.

```text
Input, tools, conversation
          |
          v
Working Context ----expires----> Short-term Session Summary
          |                              |
          +------------> Episode Records-+
                                      |
                         candidate with provenance
                                      v
                           Reflection / Review Queue
                                      |
                              approved promotion
                +---------------------+--------------------+
                v                                          v
      Obsidian-compatible Markdown vault          Semantic retrieval index
      human-readable authority                    vector + citations
                                                         |
                                                         v
                                              Derived relationship graph
```

| Layer | Initial storage | Retention | Write rule | Retrieval use |
| --- | --- | --- | --- | --- |
| Working | Task-local memory | Task only | Runtime only | Current action |
| Short-term | SQLite | Hours/days | Automatic, expiry required | Turn continuity |
| Episodes | SQLite append-only | Months, reviewable | Runtime/tool provenance | What happened |
| Semantic | Markdown vault plus index | User-controlled | Human/reviewed promotion | What is known |
| Graph | Cognee or local projection | Rebuildable | Derived only | Relationship discovery |
| Preferences | SQLite settings | Until changed | Explicit confirmation or review | Personalization |
| Reflection | SQLite proposal queue | Short retention | Reflection worker | Consolidation |

Obsidian is a vault format and human-facing authority, not an app dependency. CharOS indexes an opted-in vault but preserves Markdown, front matter, links, ownership, and exportability.

Cognee is suitable for optional graph extraction and retrieval, particularly in a fully local Ollama/FastEmbed configuration. It must remain behind a memory-provider adapter. Configure its LLM and embedding providers explicitly; otherwise a missing configuration can fall back to a cloud default.

NotebookLM is not a memory backend. It may be used through an explicit user-mediated research import/export workflow, but must not be automated through unofficial browser flows or become a required dependency.

### Consolidation and expiration

1. Finish every task with an evidence bundle: request, tool outputs, artifacts, and source links.
2. Propose episode/preference updates with confidence and provenance.
3. Expire short-term logs and caches by policy.
4. Summarize completed non-sensitive task artifacts only.
5. Require review for identity, health, finance, relationships, or external-account information.
6. Rebuild indexes and graphs from durable sources when schemas change.

Context assembly is ordered: task state, explicit user input, relevant episodes, semantic sources, then graph expansion only if it improves the answer. Every retrieved item includes a source, timestamp, sensitivity label, and confidence.

## Multi-agent execution

Do not begin with a society of autonomous agents. Begin with specialized workers executing planner-owned plans.

```text
                      Executive Planner
                  /          |           \
                 v           v            v
          Research worker  Code worker  Automation worker
                 \           |           /
                  +---- typed artifacts--+
                                |
                                v
                         Result validator
                                |
                                v
                         Planner synthesis
```

- Agents receive task-scoped context; they do not get independent durable memories by default.
- Agents communicate state through typed artifacts and events, not free-form negotiation.
- A shared read-only context pack is versioned. Each worker receives only the minimum subset.
- Independent plan branches may run in parallel. Plans declare read/write artifacts; conflicts serialize.
- Long-running work uses leases, heartbeats, checkpoints, cancellation tokens, and resumable step IDs.
- Background jobs are budgeted and cannot use high-risk tools without fresh approval.
- Retry only idempotent operations. Otherwise pause at the last checkpoint and request direction.

Start with four roles: research, code, filesystem, and summarization. Add browser, communications, and desktop automation only after audit and permissions are verified.

## Model routing

```text
Request -> capability + risk + privacy classifier
              |
              +-> private / offline / routine -> Ollama
              |
              +-> local quality insufficient AND cloud approved -> provider adapter
              |
              +-> provider failure -> eligible policy-safe fallback
              |
              +-> all fail -> explain degradation and request user choice
```

| Route | Prefer it for | Do not use it for |
| --- | --- | --- |
| Ollama | Private data, offline work, routine routing, local extraction | Tasks beyond measured model/hardware capability |
| Cloud adapter | User-approved complex reasoning, coding, vision, large context | Sensitive data without explicit approval |
| OpenRouter | Optional broker for experimentation/fallback | Core control plane or sensitive default |
| Direct OpenAI/Claude/Gemini | User-selected independent adapters | Hardcoded product behavior |
| NVIDIA NIM | Evaluation/development when current policy permits | Privacy-sensitive work or production SLA needs |

Routing considers modality, sensitivity, allowed providers, estimated tokens, latency and cost budget, local queue/VRAM, model health, and structured-output reliability. A fallback never crosses a data-policy boundary. Multiple workers can share Ollama only through a concurrency governor that accounts for model residency, VRAM, queue depth, and cancellation.

## Technology evaluation

| Technology | Place in CharOS | Recommendation / maintenance risk |
| --- | --- | --- |
| Ollama | Inference adapter | Adopt early; endpoint and models remain config, never core constants. |
| Cognee | Optional graph/retrieval projection | Pilot after SQLite/Markdown memory; APIs evolve quickly, so isolate it. |
| Obsidian | Semantic-vault provider | Adopt as a file-format integration, not an Obsidian app dependency. |
| NotebookLM API | External research | Do not depend on it without a stable official API. |
| NVIDIA NemoClaw | Security/sandbox reference | Evaluate in a lab only; it is alpha software. |
| MCP | Interoperability boundary | Adopt as adapter protocol; wrap tools in CharOS schemas/risk policy. |
| KittenTTS | TTS candidate | Evaluate behind SpeechSynthesisProvider: licence, quality, interruption, latency. |
| Handy STT / Whisper | STT candidates | Support via SpeechRecognitionProvider and benchmark locally. |
| OpenRouter | Optional cloud broker | User-configured; introduces data and availability dependency. |
| OpenAI / Claude / Gemini | Optional cloud adapters | Consent-gated, independent provider policies. |
| SQLite | Durable local record | Adopt first for tasks, audit, episodes, settings. |
| PostgreSQL | Shared/server scale path | Defer until multi-user/device concurrency requires it. |
| Redis | Distributed ephemeral coordination | Defer; task runtime plus SQLite is enough at first. |
| Qdrant | Dedicated vector service | Defer until retrieval scale proves a need. |
| Chroma | Prototype vector option | Avoid as a second durable source of truth. |
| LanceDB | Local derived vector index | Prefer for an optional laptop-local projection. |
| Docker | Isolation and optional services | Use for high-risk workers/services, not every process. |
| Tauri | Desktop shell | Preferred baseline. |
| Electron | Alternative shell | Do not adopt initially without a decisive capability need. |
| Rust | OS-facing host and policy boundary | Use for Tauri host, runtime core, permissions. |
| Python | AI/data service adapters | Use behind IPC for Cognee, voice, experimentation. |
| TypeScript | UI and shared contracts | Use for UI, schemas, plugin SDK. |

## Voice, permissions, and safety

### Voice

```text
Microphone -> VAD / wake word -> streaming STT -> intent stream
                  ^                 |                |
                  |                 v                v
              barge-in <--- conversation state <--- planner
                                                   |
                                                   v
                                         response segments -> TTS -> speakers
```

Voice must support incremental transcripts, interruption propagation, and correlation IDs. Wake-word processing is local and opt-in. Stop TTS immediately on barge-in. Raw audio retention requires revocable user consent; transcripts follow memory policy.

### Current host capability

Handy STT and KittenTTS are already installed on the Windows host. CharOS should expose them through a local `VoiceHostProvider` boundary rather than installing duplicate model runtimes in WSL. The Ubuntu runtime calls a loopback-only host service with typed streaming contracts; the service owns microphone/audio-device access and the model files. This keeps GPU/audio-device integration on Windows, allows WSL workers to remain lightweight, and makes the providers replaceable. The adapter must require an explicit loopback endpoint, health check, stream cancellation, and no raw-audio retention by default.

### Permission engine

| Risk tier | Examples | Default |
| --- | --- | --- |
| Read-only | Inspect selected repo files, local status | Session/policy grant |
| External reversible | Browser navigation, draft email/calendar | Per-task approval |
| Local mutation | Write files, install packages, change settings | Exact-scope approval |
| High impact | Delete files, Git push, SSH to new host, send message, desktop automation | Just-in-time approval |
| Critical | Spend money, share secrets, credential changes, security remediation | Explicit approval plus confirmation summary |

A capability grant contains subject, task, tool, action, target patterns, constraints, issue time, expiry, and approval receipt. Tools validate it before execution. The planner requests a grant; it never mints one.

Cybersecurity is defensive: health checks, log analysis, vulnerability/configuration reports, malware scans, and repair suggestions. Scans run in constrained workers. Repairs always include evidence and rollback plan, then require explicit approval. “Self-healing” must never authorize persistence, credential access, or unapproved remote access.

### Communication and personal AI

Telegram, WhatsApp, Discord, Gmail, Calendar, Meet, Zoom, OBS, VS Code, and browser support are adapters with read, draft, and send/modify scopes; rate limits; audit records; and revocation. Prefer drafts and confirmations for all outbound communication. Use official APIs and terms; do not treat consumer-UI scraping as a product dependency.

The executive-function assistant schedules reminders, routines, focus sessions, and check-ins. It is not a diagnostic or medical system. Users own quiet hours, schedules, retention, and exports; wellbeing support includes safe escalation guidance rather than diagnosis.

## Environment plan

```text
CharOS/
├── apps/
│   ├── desktop/                 # Tauri + TypeScript UI
│   └── settings/
├── crates/
│   ├── runtime-core/
│   ├── permission-engine/
│   ├── tool-runtime/
│   └── platform-host/
├── services/
│   ├── ai-adapters/             # Python providers
│   ├── memory-service/
│   ├── voice-service/
│   └── worker-runner/
├── packages/
│   ├── contracts/
│   ├── ui-kit/
│   └── plugin-sdk/
├── tools/
│   ├── builtin/
│   └── mcp-bridge/
├── plugins/
├── config/
│   ├── defaults/
│   └── schemas/
├── infra/
│   ├── docker/
│   ├── devcontainer/
│   └── compose/
├── docs/
│   ├── decisions/
│   ├── setup/
│   └── specs/
└── tests/
    ├── contracts/
    ├── integration/
    ├── security/
    └── evals/
```

Use `.env.example` files with names but never values. Secrets live in an OS-backed store and are injected at runtime. Begin Docker Compose with only optional services; the core must run with SQLite and Ollama. Structured logs redact before persistence. CI validates schemas, lint/types, unit/integration/security tests, and a deterministic evaluation set. Cloud evaluations require explicit secrets and budget controls.

## Installation roadmap

| Phase | Build | Why this order | Exit criteria |
| --- | --- | --- | --- |
| 1. Foundations | Git hygiene, Tauri shell, Rust/TS/Python, SQLite, config/secrets, logging | Establish contracts and observability first | App launches; config and audit work |
| 2. Safe runtime | Task supervisor, events, permissions, tool registry, read-only tools | All later abilities need policy/audit boundary | Safe end-to-end task succeeds |
| 3. Local intelligence | Ollama adapter, router, structured-output validation, planner | First useful agent stays private and replaceable | Local plan and streaming pass evals |
| 4. Memory | Session/episode records, vault adapter, retrieval, Cognee pilot | Prevents premature graph coupling | Source-cited retrieval works |
| 5. Coding tools | Git, terminal, Docker plus approval UI | High-value changes arrive after safety | Patch/test/approval loop works |
| 6. Voice and character | STT/TTS, interruption, character state mappings | UI reflects real state | Safe interruptible voice turn |
| 7. Isolated agents | Workers, containers, concurrency, MCP bridge | Parallelism after recovery/observability | Jobs resume after restart |
| 8. External adapters | Cloud, communication, browser/desktop | External effects remain opt-in | Draft/send/audit/revoke pass |
| 9. Personal AI | Scheduler, routines, preferences | Depends on identity, memory, permissions | Quiet hours and exports work |

## Risks and future vision

| Risk | Consequence | Mitigation |
| --- | --- | --- |
| Multi-agent first | Untraceable coordination failures | One planner plus bounded workers first |
| Memory equals chat history | Privacy leaks and noisy context | Lifetime-based stores, provenance, expiry |
| Provider SDKs in core | Lock-in | Capability-based adapters |
| Experimental runtime dependency | Upgrade and security breaks | Evaluate NemoClaw; keep supervisor independent |
| Browser automation as API | Terms and credential risk | Official APIs/manual workflow only |
| Many databases | Operational burden | SQLite + Markdown; derived indexes only |
| Automatic external action | Reputational/security harm | Exact grants, drafts, approvals |
| Autonomous remediation | Damage from incorrect repair | Detect/report/propose; user approves |

Over three to five years, CharOS should be a portable companion runtime where a user selects a character, vault, voice, models, and tools without surrendering ownership. Local devices remain useful offline; cloud or remote workers improve capability but do not become the system of record. Plugins and MCP tools are signed, permissioned, attributable, interruptible, and revocable.

**Primary simplification:** do not build a distributed AI swarm. Build a trustworthy local task runtime with evidence, permissions, recovery, and durable contracts. Characters, memory, plugins, and agents can then evolve without destabilizing the platform.
