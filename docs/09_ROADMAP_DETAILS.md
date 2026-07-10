# 09_ROADMAP_DETAILS.md

> **Purpose:** Provide a detailed, actionable roadmap with technical specifications, milestone definitions, and implementation steps for CharOS.
> This document is the technical roadmap - it translates the high-level roadmap.md into concrete deliverables and implementation sequences.

---

## 1. Roadmap Overview

### 1.1 Milestone Mapping to Product Goals

| Milestone | Duration | Primary Goal | Success Criteria |
|-----------|----------|--------------|------------------|
| **Step 0** | Week 0-2 | Establish Foundation | Public repo, documentation, contributing guide |
| **Step 1** | Week 3-6 | Core Experience | Side-docked overlay, hotkey open/close, placeholder avatar |
| **Step 2** | Week 7-10 | Voice Input | STT integration, push-to-talk, wake word, transcript display |
| **Step 3** | Week 11-14 | Character Runtime | Idle/thinking/speaking states, VRM animations, speech bubbles |
| **Step 4** | Week 15-20 | Local Assistant Core | Prompt orchestration, tool routing, model adapters, execution |
| **Step 5** | Week 21-26 | Memory System | Obsidian integration, episodic storage, consolidation job |
| **Step 6** | Week 27-32 | Skills System | File system, terminal, git, browser, search, notes skills |
| **Step 7** | Week 33-38 | Vision | Screenshots, OCR, UI reading, image understanding |
| **Step 8** | Week 39-44 | Cloud Fallback | Logged-in browser automation, confidence-based escalation |
| **Step 9** | Week 45-50 | Polish | Animations, sounds, theme packs, character packs, plugins |
| **Step 10** | Week 51-56 | Open-Source Readiness | Docs, examples, templates, contribution guide |

---

## 2. Technical Architecture\n\n### 2.1 System Components\n\n**Implementation Phases:**\n\n| Phase | Duration | Deliverables | Key Components |
|-------|----------|--------------|----------------|\n| **Phase 1** | Weeks 1-8 | Core Application Shell | Tauri, React, VRM Runtime, Event Bus |
| **Phase 2** | Weeks 9-12 | Voice Input | Speech-to-Text, VAD, Hotword detection |
| **Phase 3** | Weeks 13-16 | Character Runtime | Animation states, Expression system, UI integration |
| **Phase 4** | Weeks 17-20 | Local Assistant Core | Context Engine, Model Router, Task Planner |
| **Phase 5** | Weeks 21-26 | Memory System | Obsidian Integration, Episodic Store, Consolidation |
| **Phase 6** | Weeks 27-32 | Skills System | Core Skills, Registry, Permission Management |
| **Phase 7** | Weeks 33-38 | Vision Integration | Qwen2-VL adapter, Screenshot capture, OCR |
| **Phase 8** | Weeks 39-44 | Cloud Fallback | Browser automation, Confidence routing |
| **Phase 9** | Weeks 45-50 | Polish & Plugins | Animations, Themes, Plugin Architecture |
| **Phase 10** | Weeks 51-56 | Release Prep | Documentation, Examples, Contribution Guide |