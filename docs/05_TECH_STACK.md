# 05_TECH_STACK.md

> **Purpose:** Define the technology stack for CharOS — the specific frameworks, libraries, and tools that implement the architecture.
> This document guides implementation choices and ensures consistency across the codebase.

---

## 0. Development Environment

### 0.1 Operating System & Runtime

| Component | Details |
|-----------|---------|
| **Host OS** | Windows 11 |
| **Development Environment** | WSL2 Ubuntu |
| **Primary Shell** | zsh (Oh My Zsh) |
| **Package Manager** | apt (WSL), winget/scoop (Windows) |

### 0.2 Existing AI Infrastructure

**Ollama is ALREADY installed on Windows.**

- **Do NOT reinstall Ollama.**
- **Do NOT create another Ollama instance in WSL.**
- Reuse the existing Windows Ollama server from WSL.
- WSL connects to Windows Ollama via `http://host.docker.internal:11434` (or `http://172.17.0.1:11434` / `http://$(cat /etc/resolv.conf | grep nameserver | awk '{print $2}'):11434`).

**Existing Models (already installed):**
- `qwen3:14b` (code/reasoning)
- `gemma-4-heretic` / `gemma-4-abliterated` (general reasoning)
- Additional models available — detect dynamically at runtime.

**Provider Detection Logic:**
```typescript
async function detectOllamaEndpoint(): Promise<string> {
  const candidates = [
    'http://host.docker.internal:11434',
    'http://172.17.0.1:11434',
    `http://${getWindowsHostIP()}:11434`
  ];
  for (const url of candidates) {
    if (await checkHealth(url)) return url;
  }
  throw new Error('Ollama not reachable from WSL');
}
```

---

## 1. Technology Philosophy

### 1.1 Technology Selection Principles

> **Choose the right tool for the job, not the newest tool.**

**Selection Criteria:**

| Criterion | What It Means | Examples |
|-----------|---------------|----------|
| **Maturity** | Production-ready, well-documented | React, TypeScript, Three.js |
| **Performance** | Optimized for local execution | Tauri, Rust, WebGPU |
| **Privacy** | Local-first, minimal data collection | Ollama, WASM, native APIs |
| **Community** | Active, well-supported ecosystems | npm, Rust Cargo, GitHub |
| **Maintainability** | Readable, well-documented code | TypeScript over JavaScript |
| **Extensibility** | Plugin architecture, clear interfaces | Provider pattern, Event Bus |

### 2.1 Current Development Environment

| Component | Configuration |
|-----------|---------------|
| **Host OS** | Windows 11 |
| **Development Runtime** | WSL2 (Ubuntu) |
| **Ollama** | **Pre-installed on Windows host** — reuse existing instance |
| **WSL → Windows Ollama** | `http://host.docker.internal:11434` or `http://172.17.0.1:11434` |
| **Existing Models** | `qwen3:14b`, `gemma-4-heretic` (abliterated), auto-discover others |
| **NemoClaw Foundation** | NVIDIA open-source sandbox agent runtime |

> **Do NOT install Ollama in WSL. Do NOT create another Ollama instance.**
>
> **Reuse the Windows Ollama daemon. Auto-detect models at runtime.**

### 2.2 The CharOS Stack Metaphor

```
┌─────────────────────────────────────────────────────────────────┐
│                    CHAROS TECHNOLOGY STACK                     │
├─────────────────────────────────────────────────────────────────┤
│                       OUTER LAYER                              │
│  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────────┐ │
│  │  DESKTOP SHELL  │ │   WEB VIEWPORT   │ │  SYSTEM SERVICES  │ │
│  │   (Tauri)      │ │   (React + Vite) │ │  (IPC, Native)   │ │
│  └─────────┬───────┘ └─────────┬───────┘ └─────────────┬───────┘ │
│            │                  │                  │
│    ┌───────┼───────┐    ┌───────┼───────┐    ┌───────┼───────┐ │
│    ▼       ▼       ▼    ▼       ▼    ▼       ▼       ▼      │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌─────────┐ │
│  │  UI      │ │  RUNTIME   │ │  SKILLS   │ │  MEMORY  │ │ PLUGINS │ │
│  │  LAYER   │ │  ENGINE    │ │  SYSTEM   │ │  SYSTEM  │ │        │ │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └─────────┘ │
│                               ▼                               │
│              ┌────────────────────────────────────────────────┐ │
│              │       NEMOCLAW FOUNDATION (NVIDIA)            │ │
│              │  ┌──────────┐ ┌──────────┐ ┌────────────────┐ │ │
│              │  │OpenShell │ │  Routed  │ │Blueprint +     │ │ │
│              │  │ Sandbox  │ │Inference │ │Network Policies│ │ │
│              │  └──────────┘ └──────────┘ └────────────────┘ │ │
│              └────────────────────────────────────────────────┘ │
│                               ▼                               │
│      ┌─────────────────────────────────────────────────────┐   │
│      │     INFRASTRUCTURE (Docker + Rust/TypeScript)       │   │
│      └─────────────────────────────────────────────────────┘   │
```

---

## 2. Primary Technology Stack

### 2.1 Desktop Application Framework

| Technology | Version | Platform | Rationale |
|-------------|---------|----------|-----------|
| **Tauri** | `2.0+` | Windows, macOS, Linux | Native packaging, small binary, Rust safety |
| **React** | `18.x` | Web platform | Component-based, ecosystem maturity |
| **Vite** | `4.x` | Build tool | Fast HMR, optimized bundles |
| **TypeScript** | `5.x` | All | Type safety, better developer experience |
| **NemoClaw** | latest | Linux (Docker) | NVIDIA open-source sandbox agent runtime — OpenShell, routed inference, network policies |

### 2.2 NemoClaw Foundation Integration

> **CharOS is built on top of NemoClaw, which provides sandboxed agent execution via OpenShell.**
>
> **Do NOT install Ollama inside WSL. NemoClaw routes inference to the Windows Ollama host.**

| NemoClaw Component | Purpose | CharOS Usage |
|-------------------|---------|--------------|
| **OpenShell Sandbox** | Containerized agent runtime | CharOS agent runs inside OpenShell sandbox |
| **Routed Inference** | Multi-provider model routing | Routes to Ollama (Windows), OpenRouter, NIM, OpenAI, Anthropic, Gemini |
| **Blueprint Orchestration** | Sandbox lifecycle management | Declare CharOS blueprint, sandbox create/start/stop |
| **Network Policies** | Egress control and security | Govern what the agent can access |
| **CLI** | Agent management commands | `nemoclaw` CLI invoked from CharOS orchestrator |

**Tauri Implementation:**

```typescript
// tauri.conf.json
{
  "build": {
    "package-type": "isolated",
    "beforeBuildCommands": ["cargo build --release"]
  },
  "window": {
    "title": "CharOS",
    "width": 360,
    "height": 600,
    "resizable": false,
    "decorations": false,
    "transparent": true
  },
  "systemTray": {
    "icon": "./icon.png",
    "title": "CharOS Companion"
  },
  "plugins": {
    "shell": {"advice": "dangerous"},
    "dialog": {"advice": "dangerous"}
  }
}
```

### 2.2 Core Runtime (Rust + TypeScript)

```
major Runtime Code Structure:

├── core/orchestrator/              # Orchestrator (Rust)
│   ├── src/lib.rs                 # Core orchestration logic
│   ├── src/plugin.rs            # Plugin system
│   ├── src/services.rs          # External service integration
│   └── Cargo.toml               # Rust dependencies
│
├── runtime/                         # Runtime interface (TypeScript)
│   ├── src/types.ts              # Type definitions
│   ├── src/orchestrator.ts      # Runtime exports
│   └── package.json            # Runtime dependencies
│
└── interfaces/                      # Shared interfaces
    ├── src/core.ts              # Core interfaces
    ├── src/plugin.ts           # Plugin interfaces
    └── src/events.ts           # Event system
```

### 2.3 UI Framework

**React Component Architecture:**

```typescript
// packages/ui/desktop/src/components/
├── ui/                          # Core UI layer
│   ├── CharacterPreview/         # Character rendering
│   ├── SpeechBubble/           # Dialogue UI
│   ├── CharacterOverlay/        # Main overlay
│   └── HotkeyHandler/          # Keyboard handling
│
├── runtime/                     # Runtime integration
│   ├── orchestrator/            # Orchestrator hooks
│   ├── memory-provider/         # Memory state
│   └── skills-provider/         # Skills state
│
├── features/                    # Feature components
│   ├── voice-input/             # Speech-to-text UI
│   ├── vision-display/          # Vision results UI
│   └── skill-execution/         # Tool execution UI
│
└── hooks/                       # Custom hooks
    ├── useOrchestrator/         # Orchestrator state
    ├── useCharacter/            # Character state
    └── useSkills/                # Skills state
```

### 2.4 Build System

**Modern Build Architecture:**

```yaml
# .github/workflows/ci.yml
name: CharOS CI
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macOS-latest]
    
    steps:
    - uses: actions/checkout@v3
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'
        
    - name: Install Rust
      uses: actions-rs/toolchain@v1
      with:
        toolchain: stable
        
    - name: Install dependencies
      run: |
        npm ci
        cargo install cargo-watch
        
    - name: Type checking
      run: npm run typecheck
      
    - name: Unit tests
      run: npm run test:unit
      
    - name: Integration tests
      run: npm run test:integration
      
    - name: E2E tests
      run: npm run test:e2e
      
    - name: Validation
      run: |
        npm run validate:structure
        npm run validate:terminology
```

### 2.5 Development Tools

| Tool | Purpose | Configuration |
|------|---------|---------------|
| **ESLint + Prettier** | Code quality | Standard CharOS style |
| **TypeScript Compiler** | Type safety | Strict mode |
| **Jest** | Unit testing | Coverage requirements |
| **Playwright** | E2E testing | Chromatic testing |
| **Cypress** | Component testing | Visual regression |
| **Lighthouse** | Performance | Accessibility audit |

### 2.6 Key Technology Decisions

> **Technology is the means to an end, not the end itself.**

#### Frontend Tech Stack

**Why Tauri + React?**
- **Tauri**: Rust performance, small binary (~10MB), native system integration
- **React**: Virtual DOM efficiency, component reusability, mature ecosystem
- **No Electron**: Electron apps (~300MB), WebView limitations, higher memory usage

#### Runtime Architecture

**Why Rust + TypeScript?**
- **Rust**: Memory safety, performance-critical components
- **TypeScript**: Developer productivity, clear interfaces
- **Separation of concerns**: Each language for its sweet spot

#### Model Framework

**Why NemoClaw + Ollama + Cloud adapters?**
- **NemoClaw**: Provides sandboxed agent runtime and routed inference to all providers
- **Ollama (Windows host)**: Local-first, privacy-preserving — no install in WSL
- **NemoClaw Inference Router**: Manages provider priority: Ollama → OpenRouter → NVIDIA NIM → OpenAI → Anthropic → Gemini
- **Adapter pattern**: Unified `ModelProvider` interface — CharOS never knows where inference runs

---

## 3. Secondary Technology Stack

### 3.1 AI/Machine Learning

| Component | Technology | Version | Rationale |
|-----------|------------|---------|-----------|
| **Agent Runtime & Sandbox** | **NemoClaw (NVIDIA OpenShell)** | latest | Foundation layer — sandboxed agent execution, blueprint orchestration, security |
| **Inference Routing** | **NemoClaw Routed Inference** | latest | Manages provider priority, health checks, fallback chains |
| **Speech Recognition** | Handy Parakeet | v3 | Local-first, optimized for voice commands |
| **Local Models** | **Ollama (Windows host, via NemoClaw)** | latest | **Already installed — reuse existing daemon. Do NOT install in WSL** |
| **Auto-Discovered Models** | Runtime detection | runtime | `qwen3:14b`, `gemma-4-heretic`, others via `/api/tags` |
| **Vision** | Qwen2-VL | latest | High accuracy, local execution |
| **Cloud Fallback (Priority 2)** | OpenRouter | via NemoClaw | Multi-model gateway for complex reasoning |
| **Cloud Fallback (Priority 3)** | NVIDIA NIM | via NemoClaw | Enterprise/local GPU accelerated |
| **Cloud Fallback (Priority 4)** | OpenAI | via NemoClaw | GPT-4o, o1 for complex tasks |
| **Cloud Fallback (Priority 5)** | Anthropic | via NemoClaw | Claude for analysis/writing |
| **Cloud Fallback (Priority 6)** | Gemini | via NemoClaw | Google models for specific capabilities |

> **All inference goes through NemoClaw's Routed Inference.**
>
> **CharOS submits tasks to NemoClaw — it never calls model providers directly.**
>
> All providers implement unified `ModelProvider` interface.

### 3.2 Data Management

| Layer | Technology | Rationale |
|-------|------------|-----------|
| **Working Memory** | In-memory cache | Speed, ephemeral data |
| **Episodic Memory** | SQLite + provider abstraction | Portability, query efficiency |
| **Semantic Memory** | Obsidian API + GraphQL | Rich formatting, linking |
| **Consolidation** | cron + custom jobs | Scheduled processing |
| **Retrieval** | Tantivy/Elasticsearch | Fast search, relevance |

### 3.3 Model Provider Interface

> **CharOS never assumes where inference runs.**
>
> All providers implement the same interface. NemoClaw's Routed Inference handles the actual routing:

```typescript
interface ModelProvider {
  readonly id: string;
  readonly name: string;
  readonly capabilities: ModelCapability[];
  readonly priority: number;
  
  initialize(config: ProviderConfig): Promise<void>;
  isHealthy(): Promise<boolean>;
  dispose(): Promise<void>;
  
  listModels(): Promise<ModelInfo[]>;
  
  complete(request: CompletionRequest): Promise<CompletionResponse>;
  stream(request: CompletionRequest): AsyncIterable<CompletionChunk>;
  
  getStatus(): ProviderStatus;
}
```

**Priority Chain (managed by NemoClaw):** Ollama (Windows) → OpenRouter → NVIDIA NIM → OpenAI → Anthropic → Gemini

**CharOS communicates with NemoClaw CLI (subprocess JSON-RPC)** — not directly with model providers.

### 3.4 Infrastructure

### 3.4 Infrastructure

**Development Environment:**
- **Version Control**: Git with conventional commits
- **Containerization**: Docker + Docker Compose for dev environments
- **Monitoring**: Lighthouse + custom telemetry
- **Deployment**: GitHub Actions + custom scripts

---

## 4. Implementation Guidelines

### 4.1 Rust Development

**Rust Code Quality:**

```rust
// packages/core/orchestrator/src/lib.rs
use std::sync::Arc;
use tokio::sync::RwLock;

struct Orchestrator {
    plugins: PluginRegistry,
    services: Arc<Services>,
    state: RuntimeState,
}

impl Orchestrator {
    fn new(config: Config) -> Self {
        // Error handling
        // Resource management
        // Initialize subsystems
    }
    
    async fn initialize(&mut self) -> Result<(), OrchestratorError> {
        // Safe initialization
        // Resource allocation
        // Plugin loading
    }
}
```

**Rust Coding Standards:**
- **Error handling**: Use Result types, explicit error types
- **Resource management**: RAII, async operations
- **Documentation**: doc comments, crate-level documentation
- **Testing**: Unit tests + integration tests

### 4.2 TypeScript Development

**TypeScript Architecture:**

```typescript
// packages/ui/desktop/src
interface RuntimeClient {
  initialize(): Promise<void>;
  handleUserInput(input: UserInput): Promise<TaskResult>;
  registerPlugin(plugin: PluginDefinition): Promise<void>;
  getCharacterState(): CharacterState;
}

const runtimeClient = new RuntimeClient({
  baseUrl: process.env.RUNTIME_URL,
  apiKey: process.env.RUNTIME_API_KEY
});
```

**React Component Patterns:**

```typescript
// Custom hooks
function useCharacter() {
  const [character, setCharacter] = useState<CharacterState>();
  const { runtime } = useRuntime();
  
  useEffect(() => {
    runtime.getCharacterState().then(setCharacter);
  }, [runtime]);
  
  return { character, setCharacter };
}

// Component with typed events
interface CharacterDisplayProps {
  character: CharacterState;
  onStateChange?: (state: CharacterState) => void;
}

function CharacterDisplay({ character, onStateChange }: CharacterDisplayProps) {
  const handleStateChange = useCallback((newState: CharacterState) => {
    onStateChange?.(newState);
    runtime.updateCharacterState(newState);
  }, [onStateChange, runtime]);
  
  return (
    <div className="character-container">
      <CharacterRenderer character={character} />
      <StateController state={character} onChange={handleStateChange} />
    </div>
  );
}
```

### 4.3 Platform-Specific Code

**Cross-platform considerations:**

```typescript
// platform-adapters/
├── electron/                    # Browser-based fallback
├── tauri/                      # Native Tauri APIs
├── mobile/                     # Phone/Tablet specific
└── desktop/                    # OS-specific implementations
```

**Polyfill Strategy:**
- **Browser**: Use modern ES features with polyfills
- **Desktop**: Native APIs where available
- **Mobile**: Touch-optimized interfaces

---

## 5. Technology Evolution Plan

### 5.1 Immediate Needs (Phase 1-2)

> **Focus on stability, not exploration**

1. **Core Runtime** (Weeks 1-8)
   - Rust orchestrator foundation
   - Tauri + React UI skeleton
   - Basic plugin system

2. **Input Handling** (Weeks 9-12)
   - Speech-to-text integration
   - Hotkey system
   - Audio processing

3. **Character System** (Weeks 13-16)
   - VRM character runtime
   - Animation state machine
   - Basic UI integration

### 5.2 Mid-term Growth (Phase 3-4)

1. **AI Integration** (Weeks 17-20)
   - Model provider system
   - Context assembly
   - Skill router implementation

2. **Memory System** (Weeks 21-26)
   - Obsidian integration
   - Episodic storage
   - Consolidation jobs

### 5.3 Long-term Enhancements (Phase 5-10)

1. **Advanced AI** (Weeks 27-38)
   - Vision processing
   - Cloud fallback
   - Advanced reasoning

2. **Platform polish** (Weeks 39-50)
   - Advanced animations
   - Theme system
   - Plugin marketplace

---

## 6. Cross-References

| Document | Field | Relationship |
|----------|-------|-------------|
| `docs/00_VISION.md` | Section 5 | Technology choices align with vision |
| `docs/01_ARCHITECTURE.md` | All sections | Implementation of architectural choices |
| `docs/02_DESIGN_PHILOSOPHY.md` | Section 7 | Design principles drive tech choices |
| `docs/03_TERMINOLOGY.md` | Technology section | Technical terminology |
| `docs/04_PROJECT_STRUCTURE.md` | Section 4.1 | Technology-specific directories |
| `docs/05_TECH_STACK.md` | Current | Self-reference |
| `docs/06_UI_GUIDELINES.md` | All sections | UI technology alignment |
| `docs/08_AI_GUIDELINES.md` | All sections | AI technology patterns |
| `roadmap.md` | All sections | Technology roadmap alignment |

---

## 7. Open Design Questions

### 7.1 Frontend Framework Evolution

| Option | Pros | Cons |
|--------|------|------|
| **Web-First + Tauri** | Progressive enhancement, easier development | Depends on web runtime |
| **Rust-only UI** | Maximum control, performance | Steeper learning curve |
| **Cross-platform native** | Best UX on each platform | Triangulation complexity |

**Status:** Tauri + React for MVP; evaluate Rust UI later.

### 7.2 Model Integration Strategy

| Option | Pros | Cons |
|--------|------|------|
| **Ollama + adapters** | Local-first, privacy | GPU dependencies |
| **Multiple backends** | Maximum flexibility | API inconsistency |
| **Containerized models** | Portable, isolated | Performance overhead |

**Status:** Ollama-first, cloud fallback, plugin architecture for expansion.

### 7.3 Development Workflow

| Option | Pros | Cons |
|--------|------|------|
| **Monorepo** | Shared dependencies, workspaces | Complex tooling |
| **Multi-repo** | Independent lifecycles | Cross-repository complexity |
| **Mixed approach** | Flexibility | Orchestration overhead |

**Status:** Monorepo with clear package boundaries.

---

## 8. TODOs for Implementation

- [ ] Setup Rust + TypeScript monorepo structure
- [ ] Configure Tauri project with modern settings
- [ ] Setup React + Vite build configuration
- [ ] Implement core runtime interfaces
- [ ] Configure ESLint + Prettier + TypeScript strict
- [ ] Setup CI/CD pipeline with validation
- [ ] Create model provider abstraction layer
- [ ] Implement speech recognition integration
- [ ] Setup VRM runtime integration
- [ ] Create plugin system skeleton
- [ ] Write component library
- [ ] Setup testing infrastructure (Jest + Playwright)

---

> **Technology is the means to an end, not the end itself.**
>
> *Choose tools that serve the vision of a living, local AI companion.*

---

## 9. Technical Appendix

### 9.1 React Component Guidelines

**React Component Architecture:**

```typescript
// Component composition
export interface CharacterComponent {
  // Core components
  CharacterDisplay: React.ComponentType<CharacterProps>;
  SpeechBubble: React.ComponentType<SpeechProps>;
  CharacterController: React.ComponentType<CharacterControllerProps>;
  
  // Feature components
  SpeechBubble: React.ComponentType<SpeechBubbleProps>;
  CharacterRenderer: React.ComponentType<CharacterRendererProps>;
  AnimationController: React.ComponentType<AnimationControllerProps>;
}

// Component composition pattern
<CharacterDisplay>
  <CharacterRenderer />
  <AnimationController />
  <SpeechBubble/>
</CharacterDisplay>
```

**Component naming convention:**

```typescript
// Good
interface CharacterDisplayProps {
  character: CharacterState;
  onStateChange?: (state: CharacterState) => void;
}

// Good  
interface CharacterPreviewProps {
  character: CharacterState;
  isVisible: boolean;
  onClick?: () => void;
}

// Bad
interface CharacterWidgetProps {
  // Unclear what this widget does
}

// Bad
interface CharacterComponent {
  // Could be anything
  prop1: any;
  prop2: any;
}
```

### 9.2 Hook Architecture

**Custom hook patterns:**

```typescript
// State management hook
function useCharacterState(): [CharacterState, (state: CharacterState) => void] {
  const [character, setCharacter] = useState<CharacterState>(initialState);
  return [character, setCharacter];
}

// Async data fetching hook
function useAsyncData<T>(
  fetcher: () => Promise<T>,
  dependencies?: any[]
): [T | null, boolean, Error | null] {
  // Implementation
}

// Event subscription hook
function useEventSubscription(
  eventType: string,
  filter?: EventFilter
): [AppEvent[], () => void] {
  // Implementation
}
```

### 9.3 Module Structure

**React module organization:**

```
ui/
├── components/           # Core UI components
│   ├── layout/           # Layout components
│   ├── display/           # Character display components
│   ├── feedback/          # Feedback components
│   │   ├── speech-bubble/ # Speech bubble component
│   │   ├── character-state/ # Character state display
│   │   └── animation/     # Animation components
│   └── interfaces/        # Interface components
│       ├── button/        # Button components
│       ├── input/         # Input components
│       └── panel/         # Panel components
│
├── hooks/                 # Custom hooks
│   ├── useCharacter/      # Character state hooks
│   ├── useOrchestrator/   # Orchestrator hooks
│   └── useSpeech/        # Speech-related hooks
│
├── services/              # Business logic hooks
│   ├── characterService/    # Character service
│   ├── speechService/       # Speech service
│   └── overlayService/      # Overlay service
│
└── utils/                  # Utility functions
    ├── theme/               # Theme utilities
    ├── animation/          # Animation utilities
    │   └── performance/   # Performance utilities
```

### 9.4 React Component Patterns

**Higher-order component patterns:**

```typescript
// Container component with composition
<CharacterDisplay>
  <CharacterRenderer />
  <AnimationController />
  <SpeechBubble />
</CharacterDisplay>

// Composed component with render props
interface SpeechBubbleProps {
  content: string;
  type: 'say' | 'think' | 'render';
  renderProps?: RenderProps;
}

function SpeechBubble({ content, render, ...props }: SpeechBubbleProps) {
  // Conditional rendering based on props
  if (type === 'say') {
    return <SayImplementation {...props} />;
  }
  if (type === 'think') {
    return <ThinkImplementation {...props} />;
  }
  if (type === 'render') {
    return <RenderComponent {...props} />;
  }
}
```

**Render prop pattern:**

```typescript
// Child component with render prop
interface ChildProps {
  content: string;
}

function ChildComponent({ content, ...props }: ChildProps): JSX.Element {
  return props.render({ content, ...props });
}

// Usage
<ChildComponent 
  content="Hello, world!"
  render={(content, props) => <div>{content}</div>}
/>
```

### 9.5 Error Handling in React Components

**Error boundary patterns:**

```typescript
// Error boundary component
interface ErrorBoundaryProps {
  readonly error?: Error | null;
  readonly errorInfo?: ErrorInfo | null;
}

function ErrorBoundary({ error, errorInfo }: ErrorBoundaryProps): JSX.Element {
  try {
    if (error.error instanceof SyntaxError) {
      return <ErrorFallback {...error} />;
    }
    if (error.error?.constructor?.render) {
      return <ErrorFallback error=error />;
    }
    return <ErrorBoundaryComponent {...error} />;
  }
}
}

// Custom error fallback
function ErrorBoundary({ error, errorInfo }: ErrorBoundaryProps): JSX.Element {
  return <ErrorFallback error=error />;
ErrorFallback(errorInfo, errorInfo) => <ErrorFallbackComponent {...errorInfo} />;
}
```

// Error recovery component
function ErrorBoundary({ error, errorInfo }: ErrorBoundaryProps): JSX.Element {
  if (error.error?.constructor?.name === 'network') {
    return <NetworkErrorFallback error={error} />;
  }
    if (error.error?.constructor?.name === 'timeout') {
      return <TimeoutErrorFallback error={error} />;
    }
  }
}
```

### Loading state component
function LoadingState({ isLoading }: LoadingStateProps): JSX.Element {
  if (!isLoading) return null;
  return (
    <div className="loading-state">
      <LoadingSpinner />
      <LoadingText>Loading...</LoadingText>
    </div>
  );
}
```

### Suspense pattern (React 18+)
```typescript
// Suspense component
interface SuspenseBoundary {
  readonly data?: any;
  readonly error?: Error | null;
}

function Suspense({ data, error, children }: SuspenseProps): JSX.Element {
  if (data !== undefined) {
    return <SuspenseLoaded data={data} />;
  }
  if (error !== undefined) {
    return <SuspenseError error={error} />;
  
  // Fallback for Suspense boundary
  return <SuspenseFallback { ...props } />;
}
}
```

// Transition group for animations
function TransitionGroup({ children, options }: TransitionGroupProps): JSX.Element {
  return (
    <Transition
      in={children.length > 0}
      className={options.className}
      {...options}
    >
      {children}
    </Transition>
  );
}
```

// Focus manager pattern
function FocusManager({ children, options }: FocusManagerProps): JSX.Element {
  const ref = useRef<HTMLDivElement>(children);
  
  if (options.autoFocus) {
    ref.current?.focus();
  }
  
  return (
    <div ref={ref} className={options.className}>
      {children}
    </div>
  );
}
```

### 9.6 Performance Optimization

**Component optimization patterns:**

```typescript
// Memoized components for stable props
const MemoizedCharacterDisplay = memoize(CharacterDisplayProps);

// Conditional rendering for performance
function OptimizedCharacterDisplay({ character, isActive }: CharacterDisplayProps) {
  const [renderCount, setRenderCount] = useState(0);
  
  // Only re-render on prop changes
  const prevPropsRef = useRef<CharacterDisplayProps>(character);
  useEffect(() => {
    if (prevPropsRef.current !== character) {
      setRenderCount(count => count + 1);
      prevPropsRef.current = character;
    }
  }, [character]);
  
  return (
    <div className={"character-display"} data-render-count={renderCount}>
      {character}
    </div>
  );
}
```

**Virtualized lists for large datasets:**

```typescript
// Virtual scrolling component
interface VirtualListProps {
  readonly items: any[];
  readonly renderItem: (item: any, index: number) => JSX.Element;
  readonly itemHeight: number;
  readonly containerHeight: number;
  readonly overscan?: number;
}

function VirtualList({
  items,
  renderItem,
  itemHeight,
  containerHeight,
  overscan = 5
}: VirtualListProps) {
  const [startIndex, setStartIndex] = useState(0);
  const [endIndex, setEndIndex] = useState(Math.ceil(containerHeight / itemHeight) + overscan);
  
  const visibleItems = items.slice(startIndex, endIndex + 1);
  
  return (
    <div className={"virtual-list"} style={{ height: containerHeight }}>
      {visibleItems.map((item, index) => (
        <div
          key={items.indexOf(item)}
          style={{ 
            position: 'absolute',
            top: index * itemHeight,
            height: itemHeight,
            width: '100%'
          } }
        >
          {renderItem(item, index)}
        </div>
      ))}
    </div>
  );
}
```

### 9.7 Custom Hooks for Common Patterns

**Common hook patterns:**

```typescript
// Use window size hook
function useWindowSize() {
  const [windowSize, setWindowSize] = useState({ width: window.innerWidth, height: window.innerHeight });
  
  useEffect(() => {
    const handleResize = () => {
      setWindowSize({
        width: window.innerWidth,
        height: window.innerHeight
      });
    };
    
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);
  
  return windowSize;
}

// Use local storage hook
function useLocalStorage<T>(key: string, initialValue: T): [T, (setValue: (value: T) => void] {
  const storageKey = `localStorage-${key}`;
  
  // Try localStorage API first
  try {
    const storedValue = localStorage.getItem(storageKey);
    if (storedValue !== null) {
      return JSON.parse(storedValue) as T;
    }
  } catch {
    console.error(`Error parsing localStorage for key: ${key}`, error);
  }
  
  // Update localStorage on change
  useEffect(() => {
    localStorage.setItem(storageKey, JSON.stringify(value));
  }, [key, value]);
  
  return [value, (newValue) => {
    if (newValue !== value) {
      localStorage.setItem(storageKey, JSON.stringify(newValue));
      // Trigger re-render
      setValue(newValue);
    }
  }];
}

// API fetch hook with cache
function useApiFetch<T>(
  url: string,
  options?: ApiFetchOptions
): [T | null, loading: boolean, error: Error | null] {
  const [state, setState] = useState<{ data: T | null; loading: false; error: Error | null }>([null, false, null]);
  
  const fetchData = async () => {
    setState([{ ...state }, loading: true, error: null]);
    
    try {
      const response = await fetch(url, {
        method: 'GET',
        headers: {
          'Content-Type': 'application/json',
          ...options.headers
        },
        ...options.fetchOptions
      });
      
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText`);
      }
      
      const data = await response.json();
      setState([{ ...state }, loading: false, error: null]);
      return data as T;
    } catch (error) {
      setState([{ ...state }, loading: false, error]);
    }
  };
  
  return [state, () => null];
}
```

### 9.8 Communication Hooks

**Custom hooks for common communication patterns:**

```typescript
// Debounced input hook
function useDebouncedInput<T>(
  input: T,
  dependency: UseReducer<T>,
  options?: DebounceOptions<T>,
  delay?: number
): [T | null, loading: boolean, error: Error | null] {
  const [debouncedValue, setDebouncedValue] = useReducer<T>(
    (state: T, action: DebounceAction<T>) => {
      const timeout = setTimeout(action, delay);
      
      // Apply debounce
      const debounced = debounce(state, action, timeout);
      
      setDebouncedValue(debounced);
      
      return [debounced, false, null];
    },
    { dependency: dependency, initialValue, debounceOptions }
  ];
  
  // Trigger on mount
  useEffect(() => {
    setDebouncedValue(debouncedValue);
  }, [debouncedValue]);
  
  return [debouncedValue, false, null];
}

// Throttle hook
function useThrottle<T>(
  callback: () => Promise<T>,
  delay?: number,
  leading?: boolean
): [T | null, loading: boolean, error: Error | null] {
  const [throttledValue, setThrottledValue] = useReducer<T>(
    (state: T | null, action: ThrottleAction<T>) => {
      if (action.type === 'throttle') {
        clearTimeout();
        const timeout = setTimeout(delay || 0, () => {
          setThrottledValue(action.value);
        });
        return;
      }
      return state;
    },
    { dependency: throttleDependency, initialValue: null }
  );
  
  // Setup timeout
  useEffect(() => {
    if (lead && delay) {
      const timeout = setTimeout(delay, () => {
        setThrottledValue(callback());
      });
    }
  }, [lead, delay]);
  
  return [throttledValue, false, null];
}

// Previous value hook
function usePreviousValue<T>(initialValue: T): [T, (nextValue: T) => void] {
  const [previousValue, setPreviousValue] = useState(initialValue);
  
  return [previousValue, (nextValue) => {
    setPreviousValue(nextValue);
  }];
}
```

This document provides comprehensive guidance for building high-quality React components with performance optimization, error handling, and maintainability best practices.

---

## 10. Cross-References

| Document | Field | Relationship |
|----------|-------|-------------|
| `docs/00_VISION.md` | Section 5 | Technology vision alignment |
| `docs/01_ARCHITECTURE.md` | All sections | Technical architecture implementation |
| `docs/02_DESIGN_PHILOSOPHY.md` | Section 6 | Design philosophy applied |
| `docs/03_TERMINOLOGY.md` | Technology section | Technical terminology |
| `docs/04_PROJECT_STRUCTURE.md` | Section 2 | Technology directory structure |
| `docs/05_TECH_STACK.md` | Current | Self-reference |
| `docs/06_UI_GUIDELINES.md` | All sections | UI component patterns |
| `docs/08_AI_GUIDELINES.md` | All sections | AI integration patterns |
| `roadmap.md` | All sections | Technology roadmap alignment |

---

## 11. Open Design Questions

### 11.1. React Version Strategy

| Option | Pros | Cons |
|--------|------|------|
| **React 18 (current stable)** | Mature ecosystem, stable APIs | Limited new features |
| **React 19 (future)** | New server components, improved hooks | Not yet stable |
| **React 17 with new features** | Backwards compatibility, gradual migration | Complexity |

**Status:** React 18 for MVP, evaluate React 19 for future features.

### 11.2 Component Architecture Evolution

| Option | Pros | Cons |
|--------|------|------|
| **Current pattern (props + hooks)** | Familiar, widely understood | Scaling complexity |
| **Atomic design** | Design system, consistency | Additional complexity |
| **Design tokens + components** | Maintainability, theming | Learning curve |

**Status:** Current prop/hook pattern with evolution to token-based theming planned.

### 11.3 Performance Optimization Strategy

| Option | Pros | Cons |
|--------|------|------|
| **Current optimization** | Immediate improvements | Limited gains |
| **Advanced techniques** | Significant performance | Implementation complexity |

**Status:** Continue with current optimizations, add advanced techniques as needed.

---

## 12. TODOs for Technology Implementation

- [ ] Configure React + Vite project with TypeScript
- [ ] Setup Tauri desktop application
- [ ] Create core component library
- [ ] Implement build system with bundling
- [ ] Setup testing infrastructure (Jest + Playwright)
- [ ] Configure ESLint + Prettier + TypeScript strict
- [ ] Setup performance monitoring and optimization
- [ ] Create development environment setup
- [ ] Record ADR for technology stack
- [ ] Record ADR for build system
- [ ] Record ADR for testing strategy

---

> **Technology choices shape the development experience and final product quality.**
>
> *Select tools that balance immediate needs with long-term maintainability.*