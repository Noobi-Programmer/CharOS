# CharOS Agent Execution Brief

**Status:** Active implementation direction  
**Audience:** Human contributors and all coding agents  
**Read with:** `AI_CONTEXT.md`, `PROJECT_BIBLE.md`, and `docs/11_AI_OPERATING_SYSTEM_ARCHITECTURE.md`

## Mission

Build CharOS as a useful, local-first AI operating companion for broke students and independent engineers. The baseline product must run using free, open-source, locally controlled components. Paid or hosted services are optional adapters, never prerequisites.

## Current objective

Build the safe foundations before capability breadth:

1. Verify the actual Ubuntu WSL repository, Git state, GitHub CLI authentication, and SSH access.
2. Establish the repository skeleton, configuration schema, secrets policy, logging, and CI baseline.
3. Define versioned contracts for tasks, events, permissions, tools, and audit records.
4. Implement the SQLite-backed task supervisor and approval flow.
5. Add Ollama and Windows-host voice adapters.
6. Add memory, coding tools, and worker isolation only after the permission/audit foundation is working.

Do not skip ahead to multi-agent swarms, browser automation, external communications, or cloud-provider dependencies.

## Non-negotiable rules

- Prefer the free/local path in every design. Document any paid/hosted option as an opt-in adapter with a free/local fallback.
- Keep the character, UI, planner, memory, model providers, workers, tools, and permissions independent.
- Never hardcode a model or provider into core logic.
- The planner requests tool actions; it does not execute them directly.
- Models request permissions; only the permission engine validates time-bound capability grants.
- Treat Obsidian-compatible Markdown as human-owned knowledge. Cognee/vector/graph data is derived and rebuildable.
- Never automate NotebookLM through unofficial browser flows or make it a required dependency.
- Treat NemoClaw as an experimental sandboxing reference, not a CharOS runtime dependency.
- Preserve user control: draft before sending, propose before persisting, approve before high-impact action.
- Do not run broad cleanup, destructive Git commands, package removal, credential changes, publishing, or network exposure without explicit scope and approval.

## Working protocol

Before changing code:

```bash
cd ~/Projects/CharOS
git status --short --branch
git remote -v
gh auth status
```

Then:

1. Read the relevant architecture and contract documents.
2. Select the smallest complete vertical slice.
3. State the assumptions and permission/risk boundary.
4. Make focused, reversible changes.
5. Run proportionate checks.
6. Report changed files, verification, remaining risk, and the next dependency.

## Definition of ready for implementation

The project is ready to build the runtime only when the following are verified in the actual Ubuntu repository:

- Git repository and remote are identified.
- GitHub CLI authentication and scopes are confirmed.
- SSH access is working for approved local agents.
- Current worktree state is known and preserved.
- Local Ollama endpoint can be health-checked from WSL.
- Windows-host Handy STT and KittenTTS have a documented loopback-only adapter boundary.

## First milestones

| Milestone | Outcome | Gate |
| --- | --- | --- |
| M1: Foundation | Repository, config, secrets policy, logging, CI | No provider/tool coupling |
| M2: Safe task runtime | Task lifecycle, events, SQLite audit, approvals | Read-only tool task is traceable |
| M3: Local intelligence | Ollama adapter and planner | Structured output and fallback tests |
| M4: Memory | Vault, episodic records, cited retrieval | Promotion and expiry are reviewable |
| M5: Coding workflow | Git/terminal/Docker tools | Exact-scoped approval before mutation |

## Escalation rule

When a decision would introduce a vendor dependency, a paid service, permanent user data, destructive action, an external side effect, or a new security boundary, stop and record options and trade-offs. Do not invent the decision.

The architecture target is trustworthiness and maintainability—not the largest possible feature list.

