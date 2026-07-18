# API_CONTRACTS.md

> **Purpose:** Define the core application programming interfaces (APIs) for CharOS.
> This document serves as the interface contract specification that ensures consistent implementation across all subsystems.

---

## 1. Purpose and Scope

### 1.1 Interface Contract System

> **All external interfaces must be explicitly defined before implementation.**

The API Contracts document defines:

- **Interface definitions** for all public APIs
- **Data structures** and protocols
- **Message formats** and serialization
- **Method signatures** and parameter validation
- **Error handling** and return codes
- **Security constraints** and access controls

### 1.2 Contract Categories

| Category | Interface Type | Scope | Examples |
|----------|----------------|-------|----------|
| **Core APIs** | Essential platform interfaces | Global, foundational | `Orchestrator`, `EventBus`, `API` |
| **Component APIs** | Subsystem-specific interfaces | Localized, focused | `CharacterRuntime`, `MemoryProvider` |
| **Event API** | Event system interfaces | Network-wide | `EventEmitter`, `EventHandler` |
| **Configuration API** | Configuration interfaces | Application-wide | `ConfigProvider`, `ConfigSchema` |
| **Security API** | Security interfaces | Security-controlled | `PermissionEngine`, `AuthProvider` |
| **Plugin API** | Extension interfaces | Plugin ecosystem | `PluginProvider`, `PluginRegistry` |

---

## 2. Core Interface Contract

### 2.1 Global Core API

**Primary interface for CharOS platform:**

```typescript
interface ICharOS {
  // Core orchestration
  readonly id: string;
  readonly version: string;
  readonly platform: PlatformInfo;
  
  // Event system
  readonly events: IEventBus;
  
  // Service access
  getService<T>(serviceToken: ServiceToken<T>): T;
  registerService<T>(serviceToken: ServiceToken<T>, impl: T): void;
  
  // Configuration
  config: IConfigProvider;
  
  // Plugin management
  plugins: IPluginRegistry;
  
  // Lifecycle
  initialize(config: CharOSConfig): Promise<void>;
  shutdown(): Promise<void>;
  isInitialized(): boolean;
  
  // Health and monitoring
  healthCheck(): Promise<HealthStatus>;
  getMetrics(): Promise<PerformanceMetrics>;
  
  // Error handling
  handleError(error: PlatformError): Promise<void>;
}
```

### 2.2 Core Data Types

```typescript
// Platform identification
interface PlatformInfo {
  readonly name: 'charos';
  readonly version: string;
  readonly platform: 'windows' | 'linux' | 'macos' | 'unknown';
  readonly arch: string;
  readonly runtime: 'node' | 'rust' | 'deno';
}

// Service token for dependency injection
interface ServiceToken<T> {
  readonly name: string;
  readonly version: string;
  readonly type: 'interface' | 'implementation';
  
  // Factory function
  create(): T;
  
  // Validation
  validate(service: any): boolean;
}

// Platform configuration
interface CharOSConfig {
  // Core settings
  readonly environment: 'development' | 'testing' | 'production';
  readonly dataDir: string;
  readonly logLevel: 'trace' | 'debug' | 'info' | 'warn' | 'error' | 'fatal';
  
  // Security settings
  readonly security: SecurityConfig;
  readonly permissions: PermissionConfig;
  
  // Performance settings
  readonly performance: PerformanceConfig;
  readonly limits: ResourceLimits;
  
  // UI settings
  readonly ui: UIConfig;
  
  // Plugin settings
  readonly plugins: PluginConfig;
}

// Security configuration
interface SecurityConfig {
  readonly enablePluginSigning: boolean;
  readonly requireAuthentication: boolean;
  readonly sandboxMode: 'strict' | 'permissive' | 'none';
  readonly allowedNetworkRanges: string[];
  readonly blockedPorts: number[];
}

// Permission configuration
interface PermissionConfig {
  readonly defaultPolicy: 'deny' | 'allow';
  readonly requireApproval: boolean;
  readonly approvalTimeout: number;
  readonly permissionTemplates: PermissionTemplate[];
}

// Performance configuration
interface PerformanceConfig {
  readonly maxConcurrentTasks: number;
  readonly maxMemoryUsageMB: number;
  readonly maxCpuUsagePercent: number;
  readonly workerThreads: number;
}

// Resource limits
interface ResourceLimits {
  readonly maxTasks: number;
  readonly maxCharacters: number;
  readonly maxSkills: number;
  readonly maxMemoryMB: number;
  readonly maxNetworkConnections: number;
}

// UI configuration
interface UIConfig {
  readonly theme: 'light' | 'dark' | 'auto';
  readonly animationsEnabled: boolean;
  readonly accessibility: AccessibilitySettings;
  readonly overlay: OverlayConfig;
}

// Accessibility settings
interface AccessibilitySettings {
  readonly screenReaderSupport: boolean;
  readonly highContrastMode: boolean;
  readonly reducedMotion: boolean;
  readonly keyboardNavigation: boolean;
}

// Overlay configuration
interface OverlayConfig {
  readonly position: 'left' | 'right' | 'center';
  readonly defaultWidth: number;
  readonly minWidth: number;
  readonly maxWidth: number;
  readonly autoHideDelay: number;
  readonly clickThrough: boolean;
}

// Plugin configuration
interface PluginConfig {
  readonly autoLoad: boolean;
  readonly trustedRepositories: string[];
  readonly permissionTemplates: PluginPermissionTemplate[];
}

// Permission template
interface PluginPermissionTemplate {
  readonly id: string;
  readonly name: string;
  readonly description: string;
  readonly permissions: Permission[];
  readonly conditions: PermissionCondition[];
}
```

---

##  3. Event System API

### 3.1 Event Interface

**Unified event system interface:**

```typescript
interface IEventBus {
  // Event publishing
  publish(event: AppEvent): void;
  publish<Type extends AppEvent>(type: string, payload: Type['payload']): void;
  
  // Event subscription
  subscribe<EventType extends AppEvent>(eventType: string, handler: EventHandler<EventType>): Subscription;
  subscribeOnce<EventType extends AppEvent>(eventType: string, handler: EventHandler<EventType>): Subscription;
  
  // Event filtering
  subscribeFilter<EventType extends AppEvent>(filter: EventFilter, handler: EventHandler<EventType>): Subscription;
  
  // Event hierarchy and namespacing
  createChannel(channelName: string): IEventChannel;
  getChannel(channelName: string): IEventChannel;
  
  // Event management
  unsubscribe(subscription: Subscription): void;
  unsubscribeAll(eventType?: string): void;
  getSubscriptions(eventType?: string): Subscription[];
  
  // Event querying
  getEvents(eventType: string, since?: number, limit?: number): AppEvent[];
  clearEvents(eventType?: string): void;
}
```

### 3.2 Event Types and Structure

```typescript
// Base event interface
interface AppEvent {
  readonly id: string;
  readonly type: string;
  readonly namespace: string;
  readonly timestamp: number;
  readonly source: EventSource;
  readonly correlationId: string;
  
  // Event data
  readonly payload: any;
  readonly metadata: EventMetadata;
  
  // Computed properties
  readonly eventName: string;
}

// Event source
interface EventSource {
  readonly component: string;
  readonly service: string;
  readonly instance: string;
  readonly processId: number;
}

// Event metadata
interface EventMetadata {
  readonly priority: 'low' | 'normal' | 'high' | 'critical';
  readonly persistence: 'volatile' | 'persistent' | 'archived';
  readonly compression: boolean;
  readonly encryption: boolean;
  readonly size: number;
}

// Event filter for selective subscription
interface EventFilter {
  readonly eventType?: string;
  readonly namespace?: string;
  readonly source?: Partial<EventSource>;
  readonly since?: number;
  readonly until?: number;
  readonly metadata?: Partial<EventMetadata>;
}

// Event handler type
interface EventHandler<T extends AppEvent> {
  (event: T): void;
}

// Subscription management
interface Subscription {
  readonly id: string;
  readonly eventType: string;
  readonly handler: Function;
  readonly filter?: EventFilter;
  readonly createdAt: number;
  readonly lastInvoked?: number;
  readonly invocationCount: number;
  
  // Lifecycle
  unsubscribe(): void;
}
```

### 3.3 Event Channels

**Channel-based event organization:**

```typescript
interface IEventChannel {
  readonly name: string;
  readonly bus: IEventBus;
  
  // Event publishing for this channel
  publish<EventType extends AppEvent>(event: EventType): void;
  publish<EventType extends AppEvent>(type: string, payload: EventType['payload']): void;
  
  // Subscription management
  subscribe<EventType extends AppEvent>(eventType: string, handler: EventHandler<EventType>): Subscription;
  subscribeOnce<EventType extends AppEvent>(eventType: string, handler: EventHandler<EventType>): Subscription;
  
  // Channel management
  getChannel(name: string): IEventChannel;
  createChannel(name: string): IEventChannel;
  removeChannel(): boolean;
}
```

---

## 4. Service Interface Standards

### 4.1 Service Token Definition

**Standard service token pattern:**

```typescript
// Service identification helpers
const ServiceTokens = {
  // Core services
  ORCHESTRATOR: ServiceToken<IOchestrator>,
  EVENT_BUS: ServiceToken<IEventBus>,
  CONFIG: ServiceToken<IConfigProvider>,
  MEMORY: ServiceToken<IMemoryProvider>,
  PLANNER: ServiceToken<IPlanner>,
  CHARACTER: ServiceToken<ICharacterProvider>,
  SKILLS: ServiceToken<ISkillsProvider>,
  MODELS: ServiceToken<IModelProvider>,
  PLUGINS: ServiceToken<IPluginRegistry>,
  TOOLS: ServiceToken<IToolProvider>,
  
  // Provider tokens
  MEMORY_PROVIDERS: ServiceToken<IMemoryProvider[]>,
  MODEL_PROVIDERS: ServiceToken<IModelProvider[]>,
  SKILL_PROVIDERS: ServiceToken<ISkillProvider[]>,
} as const;

// Service token factory
function createServiceToken<T>(name: string, version: string, type: 'interface' | 'implementation'): ServiceToken<T> {
  return {
    name,
    version,
    type,
    create: () => {} as T,
    validate: () => true
  };
}
```

### 4.2 Interface Naming Conventions

**Consistent interface naming:**

```typescript
// Preference order for interfaces
interface TaskPlannerInterface {
  // Good naming
  planTask(goal: string, context: Context): Promise<TaskPlan>;
  executeTask(task: Task, context: Context): Promise<TaskResult>;
  cancelTask(taskId: string): Promise<boolean>;
  
  // Avoid these patterns
  // ❌ TaskPlanner
  // ❌ PlannerManager
  // ❌ TaskManager
  // ❌ Planner
  
  // ✅ TaskPlanner
  // ✅ TaskProvider
}
```

### 4.3 Interface Validation

**Runtime interface validation:**

```typescript
class InterfaceValidator {
  private readonly typeCache: Map<string, boolean>;
  
  // Validate service implementation
  validateService<T>(token: ServiceToken<T>, service: T): ValidationResult {
    const interface = this.getInterfaceDefinition(token);
    return this.validateAgainstInterface(service, interface);
  }
  
  // Validate event handler
  validateEventHandler<EventType extends AppEvent>(
    eventType: string,
    handler: EventHandler<EventType>
  ): ValidationResult {
    // Check handler signature and behavior
  }
  
  // Validate all registered services
  validateAllServices(): ValidationReport {
    // Check all services against their interfaces
  }
}

// Interface definition structure
interface InterfaceDefinition {
  readonly name: string;
  readonly version: string;
  readonly methods: MethodDefinition[];
  readonly properties: PropertyDefinition[];
  readonly events: EventDefinition[];
  readonly dependencies: DependencySpec[];
  readonly lifetime: 'singleton' | 'transient';
}
```

---

## 5. Configuration API Contract

### 5.1 Configuration Interface

**Hierarchical configuration system:**

```typescript
interface IConfigProvider {
  readonly id: string;
  readonly name: string;
  readonly version: string;
  
  // Configuration retrieval
  get<T>(key: string): T;
  get<T>(key: string, defaultValue: T): T;
  get<T>(key: string, schema: ConfigSchema<T>): T;
  
  // Configuration modification
  set<T>(key: string, value: T): void;
  merge(config: Partial<CharOSConfig>): void;
  
  // Watch configuration changes
  watch(key: string, callback: (value: any) => void): Subscription;
  unwatch(subscription: Subscription): void;
  
  // Configuration validation
  validate(config: Partial<CharOSConfig>): ValidationResult;
  validateSegment<T>(segment: string, value: any): ValidationResult<T>;
  
  // Configuration scope
  getScoped<T>(scope: ConfigScope): IConfigProvider;
  setScoped<T>(scope: ConfigScope, key: string, value: T): void;
}
```

### 5.2 Configuration Schema

**Typed configuration schema:**

```typescript
interface ConfigSchema {
  readonly type: 'object' | 'array' | 'string' | 'number' | 'boolean' | 'null';
  readonly required: boolean;
  readonly description: string;
  readonly default: any;
  readonly validation?: ValidationRule[];
  readonly pattern?: string;
  readonly minLength?: number;
  readonly maxLength?: number;
  readonly min?: number;
  readonly max?: number;
  readonly enum?: any[];
  readonly schema?: ConfigSchema;
}

// Validation rules for configuration
interface ValidationRule {
  readonly type: 'required' | 'format' | 'range' | 'pattern' | 'enum' | 'minLength' | 'maxLength';
  readonly message: string;
  readonly params?: ValidationRuleParams;
  readonly severity: 'error' | 'warning' | 'info';
}

interface ValidationRuleParams {
  readonly value?: any;
  readonly pattern?: string;
  readonly min?: number;
  readonly max?: number;
  readonly minLength?: number;
  readonly maxLength?: number;
  readonly enum?: any[];
}

// Configuration scope for isolation
interface ConfigScope {
  readonly type: 'global' | 'user' | 'project' | 'plugin';
  readonly id: string;
  readonly parent?: ConfigScope;
  readonly inheritance: 'override' | 'merge';
}
```

---

## 6. Security API Contract

### 6.1 Permission System Interface

**Unified permission interface:**

```typescript
interface IPermissionEngine {
  readonly id: string;
  readonly name: string;
  readonly version: string;
  
  // Permission checking
  check(permission: Permission, context: PermissionContext): Promise<PermissionResult>;
  
  // Permission management
  grant(permission: PermissionGrant): Promise<GrantResult>;
  revoke(grantId: string): Promise<RevokeResult>;
  
  // Permission queries
  getPermissions(context?: PermissionQuery): Promise<PermissionGrant[]>;
  getPermission(id: string): Promise<PermissionGrant | null>;
  
  // Policy management
  createPolicy(policy: PermissionPolicy): Promise<void>;
  updatePolicy(policyId: string, updates: Partial<PermissionPolicy>): Promise<void>;
  deletePolicy(policyId: string): Promise<void>;
  
  // Template management
  createTemplate(template: PermissionTemplate): Promise<void>;
  getTemplates(): Promise<PermissionTemplate[]>;
}
```

### 6.2 Permission Types

```typescript
// Core permission structure
interface Permission {
  readonly id: string;
  readonly name: string;
  readonly description: string;
  readonly category: PermissionCategory;
  readonly riskLevel: PermissionRiskLevel;
  readonly defaultGrant: 'deny' | 'allow' | 'request';
  
  // Operation definition
  readonly operation: PermissionOperation;
  readonly resources: ResourcePattern[];
  readonly conditions: PermissionCondition[];
  
  // Metadata
  readonly version: string;
  readonly deprecated: boolean;
  readonly replacement?: string;
}

// Permission categories
enum PermissionCategory {
  FILESYSTEM = 'filesystem',
  NETWORK = 'network',
  SHELL = 'shell',
  BROWSER = 'browser',
  MEMORY = 'memory',
  CALENDAR = 'calendar',
  SYSTEM = 'system',
  AI_MODEL = 'ai_model',
  SKILL = 'skill'
}

// Permission risk levels
enum PermissionRiskLevel {
  LOW = 'low',
  MEDIUM = 'medium',
  HIGH = 'high',
  CRITICAL = 'critical'
}

// Permission operation
interface PermissionOperation {
  readonly type: 'read' | 'write' | 'execute' | 'create' | 'delete' | 'modify' | 'access';
  readonly scope: 'file' | 'directory' | 'process' | 'network' | 'system' | 'memory';
  readonly target?: string;
  readonly filters?: OperationFilter[];
}

// Resource patterns
interface ResourcePattern {
  readonly type: 'path' | 'host' | 'port' | 'pid' | 'domain';
  readonly pattern: string;
  readonly description?: string;
}

// Operation filters
interface OperationFilter {
  readonly key: string;
  readonly value: any;
  readonly type: 'exact' | 'regex' | 'range' | 'glob';
}
```

---

## 7. Character Runtime API

### 7.1 Character Interface

**Core character contract:**

```typescript
interface ICharacterRuntime {
  readonly id: string;
  readonly name: string;
  readonly metadata: CharacterMetadata;
  
  // State management
  setState(state: CharacterState): Promise<void>;
  getState(): CharacterState;
  getCurrentAnimation(): string;
  isInState(state: CharacterState): boolean;
  
  // Expression control
  setExpression(expression: string, weight?: number): Promise<void>;
  getExpression(expression: string): number;
  blendExpressions(expressions: Map<string, number>): Promise<void>;
  
  // Animation control
  playAnimation(animationName: string, loop?: boolean, speed?: number): Promise<void>;
  stopAnimation(animationName: string): Promise<void>;
  pauseAnimation(): Promise<void>;
  resumeAnimation(): Promise<void>;
  
  // Speech
  speak(text: string, options?: SpeechOptions): Promise<void>;
  stopSpeaking(): Promise<void>;
  isSpeaking(): boolean;
  
  // Interaction
  handleUserInput(input: UserInput): Promise<UserInteractionResult>;
  on(event: CharacterEvent, handler: CharacterEventHandler): Subscription;
  
  // Plugin extensibility
  registerPlugin(plugin: CharacterPlugin): Promise<void>;
  unregisterPlugin(pluginId: string): Promise<void>;
}
```

---

## 8. Memory System API

### 8.1 Memory Provider Interface

**Layered memory abstraction:**

```typescript
interface IMemoryProvider {
  readonly id: string;
  readonly name: string;
  readonly version: string;
  readonly layer: MemoryLayer;
  readonly capacity: MemoryCapacity;
  readonly performance: PerformanceProfile;
  
  // Storage operations
  store(entry: MemoryEntry): Promise<StorageResult>;
  retrieve(query: MemoryQuery): Promise<MemoryEntry[]>;
  delete(key: string): Promise<boolean>;
  update(entry: MemoryEntry): Promise<UpdateResult>;
  
  // Metadata operations
  getMetadata(key: string): Promise<MemoryMetadata>;
  listKeys(pattern: string): Promise<string[]>;
  count(query?: MemoryQuery): Promise<number>;
  
  // Consolidation
  consolidate(): Promise<ConsolidationResult>;
  
  // Cleanup
  cleanup(): Promise<void>;
  
  // Health
  healthCheck(): Promise<HealthStatus>;
}
```

### 8.2 Memory Layer Types

```typescript
enum MemoryLayer {
  WORKING = 'working',
  EPISODIC = 'episodic',
  SEMANTIC = 'semantic',
  CONSOLIDATED = 'consolidated'
}

interface MemoryEntry {
  readonly id: string;
  readonly layer: MemoryLayer;
  readonly timestamp: number;
  readonly source: MemorySource;
  readonly data: any;
  readonly metadata: MemoryMetadata;
  readonly relationships: MemoryRelationship[];
  
  // Computed properties
  readonly importance: number;
  readonly accessCount: number;
  readonly lastAccessed: number;
}

interface MemoryQuery {
  readonly layer?: MemoryLayer;
  readonly source?: MemorySource;
  readonly timeRange?: TimeRange;
  readonly importance?: number;
  readonly tags?: string[];
  readonly filter?: QueryFilter;
  readonly sort?: QuerySort;
  readonly limit?: number;
  readonly offset?: number;
}
```

---

## 9. Model Provider API

### 9.1 Model Provider Interface

**Provider abstraction for all AI models:**

```typescript
interface IModelProvider {
  readonly id: string;
  readonly name: string;
  readonly version: string;
  readonly capabilities: ModelCapabilities;
  readonly limits: ModelLimits;
  readonly health: ProviderHealth;
  
  // Reasoning operations
  complete(request: CompletionRequest): Promise<CompletionResponse>;
  stream(request: CompletionRequest): AsyncIterable<CompletionChunk>;
  
  // Model management
  listModels(): Promise<ModelInfo[]>;
  switchModel(modelName: string): Promise<void>;
  
  // Configuration
  updateConfig(config: ModelConfig): Promise<void>;
  getConfig(): ModelConfig;
  
  // Health checks
  healthCheck(): Promise<HealthStatus>;
  reload(): Promise<void>;
  
  // Batch operations
  completeBatch(requests: CompletionRequest[]): Promise<CompletionResponse[]>;
}
```

### 9.2 Model Capabilities

```typescript
interface ModelCapabilities {
  readonly reasoning: boolean;
  readonly coding: boolean;
  readonly vision: boolean;
  readonly speech: boolean;
  readonly research: boolean;
  readonly planning: boolean;
  readonly maxTokens: number;
  readonly maxContextLength: number;
  readonly supportedLanguages: string[];
  readonly maxTemperature: number;
  readonly minTemperature: number;
  readonly stopSequences: string[];
  readonly optionalParameters: string[];
}
```

---

## 10. Plugin System API

### 10.1 Plugin Registry Interface

**Central plugin management:**

```typescript
interface IPluginRegistry {
  readonly id: string;
  readonly name: string;
  readonly version: string;
  
  // Plugin loading
  loadPlugin(plugin: PluginManifest): Promise<PluginInstance>;
  loadPluginFromFile(filePath: string): Promise<PluginInstance>;
  loadPluginFromUrl(url: string): Promise<PluginInstance>;
  
  // Plugin management
  getPlugin(pluginId: string): PluginInstance | null;
  listPlugins(): PluginManifest[];
  updatePlugin(pluginId: string, update: PluginUpdate): Promise<void>;
  removePlugin(pluginId: string): Promise<void>;
  
  // Capability discovery
  getExtensions<T>(extensionPoint: string): T[];
  getExtensionsByCapability(capability: string): any[];
  
  // Lifecycle
  initialize(): Promise<void>;
  shutdown(): Promise<void>;
  isInitialized(): boolean;
  
  // Security
  validatePlugin(plugin: PluginManifest): Promise<SecurityValidation>;
  signPlugin(plugin: PluginManifest): Promise<SignedPlugin>;
}
```

---

## 11. Error Handling API

### 11.1 Error System Interface

**Centralized error management:**

```typescript
interface IErrorHandler {
  readonly id: string;
  readonly name: string;
  readonly version: string;
  
  // Error reporting
  report(error: PlatformError): Promise<ReportingResult>;
  
  // Error classification
  classify(error: PlatformError): ErrorClassification;
  categorize(error: PlatformError): ErrorCategory;
  
  // Error recovery
  handle(error: PlatformError): Promise<ErrorHandlingResult>;
  
  // Error monitoring
  getErrors(since?: number, limit?: number): Promise<PlatformError[]>;
  getErrorsByType(errorType: string): Promise<PlatformError[]>;
  
  // Configuration
  setErrorHandler(handler: ErrorHandler): void;
  getErrorHandler(): ErrorHandler;
}
```

### 11.2 Error Types

```typescript
// Core error categories
enum ErrorCategory {
  SYSTEM = 'system',
  NETWORK = 'network',
  PERMISSION = 'permission',
  MEMORY = 'memory',
  PLUGIN = 'plugin',
  MODEL = 'model',
  UI = 'ui',
  TIMEOUT = 'timeout',
  VALIDATION = 'validation',
  AUTHENTICATION = 'authentication'
}

// Error severity
enum ErrorSeverity {
  DEBUG = 'debug',
  INFO = 'info',
  WARNING = 'warn',
  ERROR = 'error',
  CRITICAL = 'critical'
}

// Platform error structure
interface PlatformError {
  readonly id: string;
  readonly timestamp: number;
  readonly category: ErrorCategory;
  readonly severity: ErrorSeverity;
  readonly message: string;
  readonly context: ErrorContext;
  readonly stack?: string;
  readonly cause?: PlatformError;
  
  // Computed properties
  readonly name: string;
  readonly isRecoverable: boolean;
  readonly isRetryable: boolean;
  readonly isUserError: boolean;
}
```

---

## 12. Observability API

### 12.1 Metrics Interface

**Performance and health metrics:**

```typescript
interface IMetricsCollector {
  readonly id: string;
  readonly name: string;
  readonly version: string;
  
  // Counter metrics
  incrementCounter(name: string, tags?: MetricTags, value?: number): void;
  decrementCounter(name: string): void;
  getCounter(name: string, tags?: MetricTags): number;
  
  // Gauge metrics
  setGauge(name: string, value: number, tags?: MetricTags): void;
  getGauge(name: string, tags?: MetricTags): number;
  
  // Histogram metrics
  observeHistogram(name: string, value: number, tags?: MetricTags): void;
  getHistogramStats(name: string, tags?: MetricTags): HistogramStats;
  
  // Timing metrics
  timing(name: string, fn: () => Promise<void>, tags?: MetricTags): Promise<void>;
  
  // Custom metrics
  recordCustomMetrics(metrics: CustomMetric[]): void;
  
  // Export
  getMetrics(forTimeRange?: TimeRange): Promise<MetricsSnapshot>;
}
```

---

## 13. Configuration Summary

### 13.1 API Contract Principals

**Core principles:**

1. **Explicit Contracts**: Every interface must have a clear, defined contract
2. **Version Management**: Interfaces must specify version for backward compatibility
3. **Error Handling**: Comprehensive error classification and recovery
4. **Security Enforcement**: All APIs must enforce security boundaries
5. **Validation**: All inputs must be validated against schemas
6. **Performance**: APIs must be efficient and low-latency
7. **Extensibility**: New interfaces can be added without breaking existing ones
8. **Documentation**: All interfaces must have complete documentation

### 13.2 Interface Naming Conventions

**Consistent naming across all APIs:**

- **Interfaces**: Start with capital letter (IExample)
- **Types**: Use descriptive names (CharacterState, MemoryQuery)
- **Enums**: ALL_CAPS
- **Constants**: UPPER_SNAKE_CASE
- **Functions**: camelCase
- **Methods**: verbNoun format

### 13.3 API Versioning Strategy

**Backward-compatible versioning:**

```typescript
// API version format: MAJOR.MINOR.PATCH
// Example: 1.2.3

// Breaking changes increment MAJOR
// Feature additions increment MINOR  
// Bug fixes and patches increment PATCH

// Deprecation policy
interface Deprecation {
  version: string;
  message: string;
  replacement?: string;
  deadline?: string;
}
```

---

## 14. Cross-References

| Document | Field | Relationship |
|----------|-------|-------------|
| `docs/00_VISION.md` | Section 7 | Platform abstraction |
| `docs/01_ARCHITECTURE.md` | All sections | Implementation contracts |
| `docs/02_DESIGN_PHILOSOPHY.md` | Section 6 | Interface design principles |
| `docs/03_TERMINOLOGY.md` | All sections | API terminology |
| `docs/05_TECH_STACK.md` | Section 2 | Technology-specific APIs |
| `docs/06_UI_GUIDELINES.md` | All sections | UI component interfaces |
| `docs/07_CHARACTER_GUIDELINES.md` | All sections | Character runtime contracts |
| `docs/08_AI_GUIDELINES.md` | All sections | Model provider interfaces |
| `docs/09_ROADMAP_DETAILS.md` | Section 2 | API implementation timeline |
| `SECURITY.md` | All sections | Security API integration |

---

## 15. TODOs for API Implementation

- [ ] Define all interface contracts with TypeScript types
- [ ] Implement validation and error handling for each interface
- [ ] Create interface documentation with examples
- [ ] Set up interface testing and validation
- [ ] Create Interface Definition Language (IDL) for cross-language support
- [ ] Implement interface documentation generation
- [ ] Setup interface compatibility testing
- [ ] Record ADR for API contract system
- [ ] Record ADR for service discovery
- [ ] Record ADR for interface versioning strategy

---

> **Great APIs are like well-designed roads: they guide traffic efficiently, provide clear destinations, and make unexpected journeys possible.**

> *In CharOS, every interface contract should be written as if it were going to be read by both human developers and AI coding agents.*