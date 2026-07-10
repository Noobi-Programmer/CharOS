# PERMISSIONS.md

> **Purpose:** Define the complete permission system architecture, policies, and access control mechanisms for CharOS.
> This document establishes the security foundation for all subsystem interactions.

---

## 1. Permission System Architecture

### 1.1 Permission System Philosophy

> **Security through explicit control, not concealment.**
>
> CharOS uses a deny-by-default permission model with explicit grants for all operations.
>
> Every component, plugin, and action must declare its permissions and obtain approval before execution.

### 1.2 Permission System Overview

**Complete permission architecture:**

```
┌─────────────────────────────────────────────────────────────┐
│                  PERMISSION SYSTEM                        │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────┐ ┌─────────────────┐ ┌──────────────┐ │
│  │   Policy       │ │   Decision     │ │   Enforcement│ │
│  │   Engine       │ │     Layer      │ │   Layer      │ │
│  └─────────────────┘ └─────────────────┘ └──────────────┘ │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────┐ ┌─────────────────┐ ┌──────────────┐ │
│  │   Subject       │ │   Resource     │ │   Context    │ │
│  │   (Who)         │ │   (What)       │ │   (When/Where)│ │
│  └─────────────────┘ └─────────────────┘ └──────────────┘ │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────┐ ┌─────────────────┐ ┌──────────────┐ │
│  │   Permission    │ │   Authorization│ │   Audit      │ │
│  │   Definitions   │ │   Management   │ │   Logging    │ │
│  └─────────────────┘ └─────────────────┘ └──────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### 1.3 Permission System Categories

| Category | Scope | Purpose | Examples |
|----------|-------|---------|----------|
| **Identity** | Individual/entity | Who is acting? | User roles, component IDs |
| **Resource** | Asset/target | What is being accessed? | Files, processes, memory |
| **Action** | Operation | What is being done? | read, write, execute |
| **Policy** | Rules | When/why is allowed? | RBAC, ABAC, RBAC+ABAC |
| **Decision** | Evaluation | Determining access | Allow/Deny, Conditional |
| **Enforcement** | Implementation | Executing decisions | Token-based, capability-based |
| **Audit** | Records | Tracking decisions | Log, metrics, alerts |

---

## 2. Permission System Core

### 2.1 Subject, Resource, Action

**Foundation of all permission decisions:**

```typescript
// Subject (Who is acting)
interface Subject {
  readonly id: string;
  readonly type: SubjectType;
  readonly identity: Identity;
  readonly roles: UserRole[];
  readonly attributes: SubjectAttributes;
  readonly context: SubjectContext;
}

enum SubjectType {
  USER = 'user',
  SYSTEM = 'system',
  PLUGIN = 'plugin',
  SERVICE = 'service',
  COMPONENT = 'component'
}

interface Identity {
  readonly principal: string; // username, service name, etc.
  readonly credentials: string; // hash, token, etc.
  readonly groups: string[];
  readonly attributes: IdentityAttributes;
}

// Resource (What is being accessed)
interface Resource {
  readonly id: string;
  readonly type: ResourceType;
  readonly identifier: string;
  readonly properties: ResourceProperties;
  readonly ownership: Ownership;
  readonly relationships: ResourceRelationship[];
  readonly constraints: ResourceConstraint[];
}

interface ResourceType {
  readonly category: ResourceCategory;
  readonly subtype: string;
  readonly schema: ResourceSchema;
}

enum ResourceCategory {
  FILESYSTEM = 'filesystem',
  NETWORK = 'network',
  PROCESS = 'process',
  MEMORY = 'memory',
  UI = 'ui',
  MODEL = 'model',
  SKILL = 'skill',
  TOOL = 'tool',
  PLUGIN = 'plugin',
  SYSTEM = 'system'
}

// Action (What is being done)
interface Action {
  readonly id: string;
  readonly type: ActionType;
  readonly scope: ActionScope;
  readonly parameters: ActionParameters;
  readonly requirements: ActionRequirements;
}

interface ActionType {
  readonly operation: string;
  readonly method: string;
  readonly effect: OperationEffect;
}

enum OperationEffect {
  READ = 'read',
  WRITE = 'write',
  EXECUTE = 'execute',
  CREATE = 'create',
  DELETE = 'delete',
  MODIFY = 'modify',
  ACCESS = 'access',
  CONFIGURE = 'configure',
  ADMINISTER = 'administer'
}
```

### 2.2 Context Objects

**Contextual information for permission decisions:**

```typescript
interface DecisionContext {
  readonly environment: EnvironmentContext;
  readonly situational: SituationalContext;
  readonly temporal: TemporalContext;
  readonly spatial: SpatialContext;
  readonly security: SecurityContext;
}

interface EnvironmentContext {
  readonly host: string;
  readonly port: number;
  readonly userAgent: string;
  readonly ipAddress: string;
  readonly sessionId: string;
}

interface SituationalContext {
  readonly task: Task;
  readonly purpose: string;
  readonly urgency: UrgencyLevel;
  readonly riskTolerance: RiskLevel;
  readonly expectation: ExpectationLevel;
}

interface TemporalContext {
  readonly time: number;
  readonly date: Date;
  readonly timezone: string;
  readonly schedule: Schedule;
}

interface SpatialContext {
  readonly location: Location;
  readonly networkZone: NetworkZone;
  readonly fileSystemScope: FileSystemScope;
}

interface SecurityContext {
  readonly level: SecurityClearance;
  readonly classification: SecurityClassification;
  readonly needToKnow: boolean;
  readonly specialAccess: SpecialAccess[];
}
```

---

## 3. Permission Policy Engine

### 3.1 Policy Engine Interface

**Core policy evaluation interface:**

```typescript
interface IPermissionEngine {
  readonly id: string;
  readonly name: string;
  readonly version: string;
  readonly configuration: PermissionEngineConfig;
  
  // Policy evaluation
  checkPermission(subject: Subject, resource: Resource, action: Action, context: DecisionContext): PermissionResult;
  
  // Policy management
  createPolicy(policy: PermissionPolicy): Promise<PolicyNode>;
  updatePolicy(policyId: string, updates: Partial<PermissionPolicy>): Promise<void>;
  deletePolicy(policyId: string): Promise<void>;
  
  // Policy querying
  getPolicies(subject?: Subject, resource?: Resource, action?: Action): PermissionPolicy[];
  evaluatePolicyTree(subject: Subject, resource: Resource, action: Action, context: DecisionContext): PolicyDecision;
  
  // Policy optimization
  optimizePolicies(): Promise<OptimizationResult>;
  getPolicyStats(): PolicyStatistics;
  
  // Policy lifecycle
  loadPolicies(policies: PermissionPolicy[]): Promise<void>;
  exportPolicies(): Promise<ExportedPolicies>;
  importPolicies(policies: ImportedPolicies): Promise<void>;
}
```

### 3.2 Permission Policy Types

**Different policy types and combinations:**

```typescript
enum PolicyType {
  ROLE_BASED = 'role_based',
  ATTRIBUTE_BASED = 'attribute_based',
  CONTEXTUAL = 'contextual',
  CONDITIONAL = 'conditional',
  HIERARCHICAL = 'hierarchical',
  INHERITANCE = 'inheritance',
  CONSOLIDATION = 'consolidation'
}

interface PermissionPolicy {
  readonly id: string;
  readonly name: string;
  readonly version: string;
  readonly type: PolicyType;
  readonly description: string;
  readonly rules: PolicyRule[];
  readonly combiningAlgorithm: CombiningAlgorithm;
  readonly metadata: PolicyMetadata;
}

interface PolicyRule {
  readonly id: string;
  readonly name: string;
  readonly description: string;
  readonly effect: PolicyEffect;
  readonly conditions: Condition[];
  readonly decision: PolicyDecision;
  readonly obligations: Obligation[];
  readonly prohibitions: Prohibition[];
}

interface Condition {
  readonly id: string;
  readonly type: ConditionType;
  readonly target: string;
  readonly expectation: ConditionExpectation;
  readonly operators: Operator[];
  readonly parameters: ConditionParameters;
}

interface ConditionType {
  readonly category: ConditionCategory;
  readonly subtype: string;
  readonly schema: ConditionSchema;
}

enum ConditionCategory {
  IDENTITY = 'identity',
  RELATIONSHIP = 'relationship',
  ATTRIBUTE = 'attribute',
  CONTEXT = 'context',
  TIME = 'time',
  LOCATION = 'location',
  ENVIRONMENTAL = 'environmental',
  BUSINESS = 'business'
}
```

### 3.3 Policy Decision Making

**Policy evaluation algorithms:**

```typescript
interface PolicyDecision {
  readonly granted: boolean;
  readonly effect: PolicyEffect;
  readonly reason: DecisionReason;
  readonly reasoning: DecisionReasoning;
  readonly obligations: Obligation[];
  readonly prohibitions: Prohibition[];
  readonly alternatives: AlternativeAction[];
  readonly metadata: DecisionMetadata;
}

enum PolicyEffect {
  ALLOW = 'allow',
  DENY = 'deny',
  INHERIT = 'inherit',
  CONDITIONAL = 'conditional',
  MULTIVALUED = 'multivalued'
}

class PolicyEvaluator {
  constructor(private policies: PermissionPolicy[]) {}
  
  evaluate(subject: Subject, resource: Resource, action: Action, context: DecisionContext): PolicyDecision {
    const applicablePolicies = this.filterApplicablePolicies(subject, resource, action, context);
    const decisions = applicablePolicies.map(policy => this.evaluatePolicy(policy, subject, resource, action, context));
    return this.combineDecisions(decisions, context);
  }
  
  private evaluatePolicy(policy: PermissionPolicy, subject: Subject, resource: Resource, action: Action, context: DecisionContext): PolicyDecision {
    const applicableRules = policy.rules.filter(rule => this.ruleApplies(rule, subject, resource, action, context));
    const ruleDecisions = applicableRules.map(rule => this.evaluateRule(rule, subject, resource, action, context));
    return this.combineRuleDecisions(ruleDecisions, policy.combiningAlgorithm, context);
  }
  
  private evaluateRule(rule: PolicyRule, subject: Subject, resource: Resource, action: Action, context: DecisionContext): PolicyDecision {
    const ruleApplies = this.ruleApplies(rule, subject, resource, action, context);
    const allConditionsSatisfied = this.allConditionsSatisfied(rule.conditions, subject, resource, action, context);
    
    if (!ruleApplies) {
      return { granted: false, effect: PolicyEffect.DENY, reason: 'Rule does not apply' };
    }
    
    if (!allConditionsSatisfied) {
      return { granted: false, effect: PolicyEffect.DENY, reason: 'Conditions not met' };
    }
    
    const decision = this.determineDecision(rule, subject, resource, action, context);
    
    return {
      granted: decision.granted,
      effect: decision.effect,
      reason: `Rule ${rule.name}: ${decision.reason}`,
      reasoning: this.generateReasoning(rule, subject, resource, action, context),
      obligations: rule.obligations,
      prohibitions: rule.prohibitions
    };
  }
}
```

---

## 4. Permission Definitions

### 4.1 Permission Types

**Comprehensive permission definitions:**

```typescript
// Core permission structure
interface Permission {
  readonly id: string;
  readonly name: string;
  readonly description: string;
  readonly category: PermissionCategory;
  readonly riskLevel: PermissionRiskLevel;
  readonly defaultGrant: 'deny' | 'allow' | 'conditional';
  
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

// Permission conditions
interface PermissionCondition {
  readonly id: string;
  readonly type: ConditionType;
  readonly target: string;
  readonly expectation: ConditionExpectation;
  readonly operators: Operator[];
  readonly parameters: ConditionParameters;
}

interface ConditionExpectation {
  readonly operator: Operator;
  readonly value: any;
  readonly truthiness: 'positive' | 'negative';
}

interface Operator {
  readonly type: OperatorType;
  readonly symbol: string;
  readonly description: string;
}

enum OperatorType {
  EQUALS = 'equals',
  NOT_EQUALS = 'not_equals',
  GREATER_THAN = 'greater_than',
  LESS_THAN = 'less_than',
  CONTAINS = 'contains',
  MATCHES = 'matches',
  IN = 'in',
  NOT_IN = 'not_in',
  EXISTS = 'exists',
  NOT_EXISTS = 'not_exists',
  AFTER = 'after',
  BEFORE = 'before',
  SIMILAR = 'similar'
}
```

---

## 5. Authorization Models

### 5.1 Role-Based Access Control (RBAC)

**Classic RBAC implementation:**

```typescript
interface RBACModel {
  readonly id: string;
  readonly name: string;
  readonly version: string;
  
  // Role management
  readonly roles: Role[];
  readonly permissions: Permission[];
  readonly roleHierarchy: RoleHierarchy;
  readonly userRoles: UserRoleMapping[];
  readonly sessionRoles: SessionRoleMapping[];
}

interface Role {
  readonly id: string;
  readonly name: string;
  readonly description: string;
  readonly permissions: Permission[];
  readonly inheritedRoles: string[];
  readonly constraints: RoleConstraint[];
}

interface RoleHierarchy {
  readonly parentChild: Map<string, string[]>;
  readonly depth: number;
  readonly maxDepth: number;
}
```

### 5.2 Attribute-Based Access Control (ABAC)

**Attribute-based authorization:**

```typescript
interface ABACModel {
  readonly id: string;
  readonly name: string;
  readonly version: string;
  
  // Attributes
  readonly subjectAttributes: SubjectAttribute[];
  readonly resourceAttributes: ResourceAttribute[];
  readonly actionAttributes: ActionAttribute[];
  readonly contextAttributes: ContextAttribute[];
  
  // Policies
  readonly policies: ABACPolicy[];
  readonly attributeSchemas: AttributeSchema[];
}

interface ABACPolicy {
  readonly id: string;
  readonly name: string;
  readonly description: string;
  readonly rules: ABACPolicyRule[];
  readonly combiningAlgorithm: CombiningAlgorithm;
}

interface ABACPolicyRule {
  readonly id: string;
  readonly subject: AttributeExpression;
  readonly action: AttributeExpression;
  readonly resource: AttributeExpression;
  readonly environment: AttributeExpression;
  readonly decision: PolicyDecision;
}
```

### 5.3 Attribute Expressions

**Complex attribute evaluation:**

```typescript
interface AttributeExpression {
  readonly type: ExpressionType;
  readonly value: any;
  readonly operator: ExpressionOperator;
  readonly subExpressions?: AttributeExpression[];
  readonly functionCall?: FunctionCall;
}

enum ExpressionType {
  VARIABLE,      // $user.role
  FUNCTION,      // function(args)
  LITERAL,       // "value"
  COMPARISON,    // x > 5
  BOOLEAN,       // true/false
  NULL,          // null
  ARRAY,         // [1, 2, 3]
  OBJECT,        // {"key": "value"}
  UNKNOWN        // Fallback
}

interface FunctionCall {
  readonly function: string;
  readonly arguments: AttributeExpression[];
  readonly returnType: AttributeType;
}
```

---

## 6. Contextual Authorization

### 6.1 Contextual Policies

**Context-aware authorization:**

```typescript
class ContextualAuthorization {
  constructor(private policyEngine: IPermissionEngine) {}
  
  evaluateWithContext(subject: Subject, resource: Resource, action: Action, context: DecisionContext): PolicyDecision {
    // Evaluate baseline policy
    const baselineDecision = this.policyEngine.checkPermission(subject, resource, action, context);
    
    // Apply contextual modifications
    const contextualDecision = this.applyContextualModifications(baselineDecision, context);
    
    // Apply time-based rules
    const temporalDecision = this.applyTemporalRules(baselineDecision, context);
    
    // Apply location-based rules
    const spatialDecision = this.applySpatialRules(baselineDecision, context);
    
    // Apply risk-based rules
    const riskDecision = this.applyRiskBasedRules(baselineDecision, context);
    
    return this.consolidateDecisions([
      baselineDecision,
      contextualDecision,
      temporalDecision,
      spatialDecision,
      riskDecision
    ], context);
  }
}

class TemporalAuthorization {
  evaluateTimeConstraints(subject: Subject, resource: Resource, action: Action, context: DecisionContext): TimeDecision {
    const now = context.temporal.time;
    const schedule = context.temporal.schedule;
    
    if (schedule.blockedPeriods && schedule.blockedPeriods.some(period => now >= period.start && now <= period.end)) {
      return { granted: false, reason: 'Access blocked by schedule' };
    }
    
    if (schedule.allowedPeriods && schedule.allowedPeriods.some(period => now >= period.start && now <= period.end)) {
      return { granted: true, reason: 'Access allowed by schedule' };
    }
    
    return { granted: false, reason: 'No schedule match' };
  }
}
```

### 6.2 Risk-Based Authorization

**Dynamic risk evaluation:**

```typescript
class RiskBasedAuthorization {
  evaluateRiskScore(subject: Subject, resource: Resource, action: Action, context: DecisionContext): RiskDecision {
    const riskFactors = this.evaluateRiskFactors(subject, resource, action, context);
    const riskScore = this.calculateRiskScore(riskFactors);
    const riskLevel = this.determineRiskLevel(riskScore);
    
    const decision = this.applyRiskPolicy(riskLevel, riskScore, riskFactors);
    
    return {
      granted: decision.granted,
      riskScore,
      riskLevel,
      reason: decision.reason,
      mitigations: decision.mitigations,
      alternatives: decision.alternatives
    };
  }
  
  private evaluateRiskFactors(subject: Subject, resource: Resource, action: Action, context: DecisionContext): RiskFactor[] {
    const factors = [];
    
    // User identity risk
    factors.push(...this.evaluateIdentityRisk(subject));
    
    // Resource sensitivity risk
    factors.push(...this.evaluateResourceRisk(resource));
    
    // Action risk
    factors.push(...this.evaluateActionRisk(action));
    
    // Context risk
    factors.push(...this.evaluateContextRisk(context));
    
    // Behavioral risk
    factors.push(...this.evaluateBehavioralRisk(subject, context));
    
    return factors;
  }
}
```

---

## 7. Dynamic Access Control

### 7.1 Dynamic Permission Assignment

**Runtime permission management:**

```typescript
interface DynamicPermissionManager {
  readonly id: string;
  readonly name: string;
  readonly version: string;
  
  // Temporary permissions
  grantTemporaryPermission(subject: Subject, permission: Permission, duration: Duration, reason: string): Promise<PermissionGrant>;
  revokeTemporaryPermission(grantId: string): Promise<void>;
  
  // Conditional permissions
  createConditionalPermission(definition: ConditionalPermissionDefinition): Promise<ConditionalPermission>;
  updateConditionalPermission(permissionId: string, updates: Partial<ConditionalPermissionDefinition>): Promise<void>;
  
  // Delegation
  delegatePermission(grantor: Subject, grantee: Subject, permission: Permission, constraints: DelegationConstraint[]): Promise<DelegationRecord>;
  
  // Escalation
  requestPermissionEscalation(subject: Subject, permission: Permission, justification: string): Promise<EscalationRequest>;
  approveEscalation(requestId: string, approver: Subject, reason: string): Promise<void>;
}
```

### 7.2 Entitlement Management

**Complex entitlement systems:**

```typescript
interface EntitlementSystem {
  readonly id: string;
  readonly name: string;
  readonly version: string;
  
  // Entitlement hierarchy
  readonly entitlements: Entitlement[];
  readonly entitlementHierarchy: EntitlementHierarchy;
  readonly entitlementRelationships: EntitlementRelationship[];
  
  // Entitlement evaluation
  evaluateEntitlements(subject: Subject, resource: Resource, action: Action, context: DecisionContext): EntitlementDecision;
  
  // Entitlement management
  createEntitlement(entitlement: EntitlementDefinition): Promise<Entitlement>;
  updateEntitlement(entitlementId: string, updates: Partial<EntitlementDefinition>): Promise<void>;
  deleteEntitlement(entitlementId: string): Promise<void>;
}
```

---

## 8. Permission Enforcement

### 8.1 Enforcement Mechanisms

**Various enforcement strategies:**

```typescript
enum EnforcementMechanism {
  CODE = 'code',
  POLICY = 'policy',
  TOKEN = 'token',
  CERTIFICATE = 'certificate',
  BIOMETRIC = 'biometric',
  QUANTUM = 'quantum',
  BLOCKCHAIN = 'blockchain',
  MACHINE_LEARNING = 'ml'
}

interface PermissionEnforcer {
  readonly id: string;
  readonly name: string;
  readonly version: string;
  
  // Rule-based enforcement
  enforce(rule: PermissionRule, subject: Subject, resource: Resource, action: Action, context: DecisionContext): EnforcementResult;
  
  // Programmatic enforcement
  validateAccess(subject: Subject, resource: Resource, action: Action, context: DecisionContext): ValidationResult;
  
  // Capability-based enforcement
  issueCapability(subject: Subject, capability: Capability, duration: Duration): CapabilityToken;
  validateCapability(capabilityToken: CapabilityToken, expectedCapability: Capability): boolean;
}
```

### 8.2 Capability-Based Security

**Capability-based access control:**

```typescript
interface Capability {
  readonly id: string;
  readonly name: string;
  readonly description: string;
  readonly type: CapabilityType;
  readonly target: Resource;
  readonly permissions: Permission[];
  readonly constraints: CapabilityConstraint[];
  readonly expiry: number;
}

interface CapabilityToken {
  readonly id: string;
  readonly capability: Capability;
  readonly holder: Subject;
  readonly issuedAt: number;
  readonly expiresAt: number;
  readonly signature: string;
  readonly encrypted: boolean;
}

class CapabilityManager {
  constructor(private enforcer: PermissionEnforcer) {}
  
  issueCapability(subject: Subject, capability: Capability, recipient: Subject): CapabilityToken {
    const token = new CapabilityToken(
      generateId(),
      capability,
      recipient,
      Date.now(),
      Date.now() + capability.expiry * 1000,
      this.generateSignature(capability, recipient)
    );
    
    this.enforcer.issueCapability(subject, capability, Duration.INDEFINITE);
    
    return token;
  }
  
  validateCapability(token: CapabilityToken, expectedCapability: Capability): ValidationResult {
    // Validate token signature
    if (!this.verifySignature(token)) {
      return { valid: false, reason: 'Invalid signature' };
    }
    
    // Validate token expiration
    if (Date.now() > token.expiresAt) {
      return { valid: false, reason: 'Token expired' };
    }
    
    // Validate capability permissions
    if (!this.validateCapabilityPermissions(token, expectedCapability)) {
      return { valid: false, reason: 'Capability permissions mismatch' };
    }
    
    // Validate token constraints
    if (!this.validateTokenConstraints(token)) {
      return { valid: false, reason: 'Token constraints violated' };
    }
    
    return { valid: true };
  }
}
```

---

## 9. Permission Auditing

### 9.1 Audit Logging

**Comprehensive audit trails:**

```typescript
interface PermissionAuditor {
  readonly id: string;
  readonly name: string;
  readonly version: string;
  readonly configuration: AuditConfiguration;
  
  // Event logging
  logAccessAttempt(event: AccessAttempt): Promise<void>;
  logPolicyDecision(event: PolicyDecision): Promise<void>;
  logAuthorizationResult(event: AuthorizationResult): Promise<void>;
  
  // Sensitive operations
  logPrivilegeEscalation(event: PrivilegeEscalationEvent): Promise<void>;
  logExceptionAccess(event: ExceptionAccessEvent): Promise<void>;
  logComplianceViolation(event: ComplianceViolation): Promise<void>;
  
  // Audit trail management
  getAuditEvents(query: AuditQuery): Promise<AuditEvent[]>;
  getAuditStats(timeRange: TimeRange): Promise<AuditStatistics>;
  retainAuditEvents(retentionPolicy: RetentionPolicy): Promise<void>;
}
```

### 9.2 Audit Event Types

**Complete audit event taxonomy:**

```typescript
enum AuditEventType {
  // Access control events
  ACCESS_ATTEMPT = 'access_attempt',
  ACCESS_DENIED = 'access_denied',
  ACCESS_GRANTED = 'access_granted',
  AUTHORIZATION_SUCCESS = 'authorization_success',
  AUTHORIZATION_FAILURE = 'authorization_failure',
  
  // Policy events
  POLICY_EVALUATION = 'policy_evaluation',
  POLICY_CHANGED = 'policy_changed',
  POLICY_EFFECT = 'policy_effect',
  
  // Subject events
  SUBJECT_CREATED = 'subject_created',
  SUBJECT_UPDATED = 'subject_updated',
  SUBJECT_DELETED = 'subject_deleted',
  SUBJECT_AUTHENTICATED = 'subject_authenticated',
  
  // Resource events
  RESOURCE_CREATED = 'resource_created',
  RESOURCE_UPDATED = 'resource_updated',
  RESOURCE_DELETED = 'resource_deleted',
  RESOURCE_ACCESS = 'resource_access',
  
  // Session events
  SESSION_CREATED = 'session_created',
  SESSION_UPDATED = 'session_updated',
  SESSION_TERMINATED = 'session_terminated',
  
  // Compliance events
  COMPLIANCE_CHECK = 'compliance_check',
  COMPLIANCE_VIOLATION = 'compliance_violation',
  COMPLIANCE_FIX = 'compliance_fix',
  
  // Security events
  SECURITY_ALERT = 'security_alert',
  SECURITY_INCIDENT = 'security_incident',
  SECURITY_REMEDIATION = 'security_remediation'
}
```

---

## 10. Configuration

### 10.1 Permission System Configuration

**Complete permission system configuration:**

```typescript
interface PermissionSystemConfig {
  // Core configuration
  readonly id: string;
  readonly name: string;
  readonly version: string;
  
  // Policy configuration
  readonly policyConfiguration: PolicyConfiguration;
  readonly rbacConfiguration: RBACConfiguration;
  readonly abacConfiguration: ABACConfiguration;
  readonly dynamicConfiguration: DynamicConfiguration;
  
  // Security configuration
  readonly securityConfiguration: SecurityConfiguration;
  readonly auditConfiguration: AuditConfiguration;
  readonly complianceConfiguration: ComplianceConfiguration;
  
  // Performance configuration
  readonly performanceConfiguration: PerformanceConfiguration;
  readonly scalabilityConfiguration: ScalabilityConfiguration;
}
```

### 10.2 Policy Configuration Schema

**Policy configuration specifications:**

```typescript
interface PolicyConfiguration {
  readonly defaultPolicy: PolicyEffect;
  readonly combiningAlgorithm: CombiningAlgorithm;
  readonly maxPolicyDepth: number;
  readonly policyTimeout: number;
  readonly enablePolicyCaching: boolean;
  readonly policyCacheSize: number;
  readonly enforcePolicyHierarchy: boolean;
  readonly inheritChildPolicies: boolean;
}
```

---

## 11. Cross-References

| Document | Relationship |
|----------|--------------|
| `docs/01_ARCHITECTURE.md` | Core permission system architecture |
| `docs/02_DESIGN_PHILOSOPHY.md` | Security architecture principles |
| `docs/03_TERMINOLOGY.md` | Permission terminology |
| `docs/05_TECH_STACK.md` | Technology choices |
| `API_CONTRACTS.md` | Security API contracts |
| `EVENT_SYSTEM.md` | Event-based permission updates |
| `SECURITY_MODEL.md` | Security requirements integration |

---

## 12. TODOs for Implementation

- [ ] Design `IPermissionEngine` with complete interface contract
- [ ] Implement `PolicyEvaluator` with rule-based evaluation
- [ ] Create `AttributeBasedAuthorization` with ABAC support
- [ ] Build `DynamicPermissionManager` with runtime management
- [ ] Implement `CapabilityManager` with capability-based security
- [ ] Create `PermissionAuditor` with comprehensive audit trails
- [ ] Set up `EntitlementSystem` for complex access models
- [ ] Record ADR for permission system architecture
- [ ] Record ADR for authorization models
- [ ] Record ADR for dynamic access control
- [ ] Create permission system testing framework

---

## 13. Optimization Strategies

### 13.1 Performance Optimization

**Permission system performance tuning:**

1. **Caching:** Cache policy decisions for repeated evaluations
2. **Pre-filtering:** Apply quick filters before full evaluation
3. **Parallel evaluation:** Evaluate independent policies in parallel
4. **Hierarchical optimization:** Leverage policy hierarchy for efficient evaluation
5. **Batch processing:** Process multiple permission checks in batches

### 13.2 Scalability Design

**Permission system scalability considerations:**

- **Horizontal scaling:** Permission engine can be scaled horizontally
- **Caching:** Use distributed caching for policy decisions
- **Partitioning:** Partition policies by subject, resource, or action
- **Load balancing:** Distribute permission checks across multiple instances
- **Fault tolerance:** Implement retry logic and fallback mechanisms

---

> **Permissions are the gates that protect the castle. Make them strong, precise, and easy to manage.**

> *CharOS permissions must be explicit, auditable, and enforceable across all subsystems.*

---

## 14. Technical Appendix

### 14.1 Condition Types

**Complete condition type definitions:**

```typescript
interface ConditionSchema {
  readonly type: ConditionType;
  readonly name: string;
  readonly description: string;
  readonly properties: PropertySchema[];
  readonly examples: Example[];
  readonly validation: ValidationRule[];
}

interface PropertySchema {
  readonly name: string;
  readonly type: SchemaType;
  readonly required: boolean;
  readonly description: string;
  readonly default?: any;
  readonly validation?: ValidationRule[];
}
```

### 14.2 Obligation and Prohibition Types

**Types of obligations and prohibitions:**

```typescript
enum ObligationType {
  LOGGING = 'logging',
  NOTIFICATION = 'notification',
  MACHINE_ACTION = 'machine_action',
  MANUAL_REVIEW = 'manual_review',
  FINANCIAL_PENALTY = 'financial_penalty',
  TECHNICAL_LIMITATION = 'technical_limitation'
}

enum ProhibitionType {
  ACTION = 'action',
  ACCESS = 'access',
  MODIFICATION = 'modification',
  DELETION = 'deletion',
  ESCALATION = 'escalation',
  DELEGATION = 'delegation'
}
```

### 14.3 Combining Algorithms

**Policy combination algorithms:**

```typescript
enum CombiningAlgorithm {
  FIRST_APPLICABLE,          // Apply first matching policy
  DENY_OVERRIDES_ALLOW,       // Deny overrides allow
  ALLOW_OVERRIDES_DENY,       // Allow overrides deny
  PERMISSIVE_OVERRIDES_DENY,  // Permissive overrides deny
  STRICT_OVERRIDES_PERMISSIVE, // Strict overrides permissive
  RESPECTS_DEPENDENCIES,     // Respects policy dependencies
  CONSOLIDATES_EFFECTS,      // Consolidates all effects
  SELECTS_HIGHEST_PRIORITY,    // Select highest priority decision
  CONSIDERS_CONTEXT,          // Considers contextual factors
  APPLICATION_SPECIFIC      // Application-specific logic
}
```

### 14.4 Risk Calculation

**Risk factor calculation:**

```typescript
interface RiskFactor {
  readonly type: RiskFactorType;
  readonly weight: number;
  readonly score: number;
  readonly details: RiskFactorDetails;
  readonly mitigations: Mitigation[];
}

enum RiskFactorType {
  IDENTITY_RISK,
  BEHAVIORAL_RISK,
  CONTEXTUAL_RISK,
  ENVIRONMENTAL_RISK,
  RESOURCE_RISK,
  ACTION_RISK,
  TEMPORAL_RISK,
  SPATIAL_RISK,
  HISTORICAL_RISK
}

class RiskCalculator {
  calculateRiskScore(riskFactors: RiskFactor[]): number {
    const weightedSum = riskFactors.reduce((sum, factor) => sum + factor.score * factor.weight, 0);
    const maxScore = riskFactors.reduce((sum, factor) => sum + factor.weight, 0);
    return (weightedSum / maxScore) * 100;
  }
  
  determineRiskLevel(score: number): RiskLevel {
    if (score < 30) return RiskLevel.LOW;
    if (score < 60) return RiskLevel.MEDIUM;
    if (score < 80) return RiskLevel.HIGH;
    return RiskLevel.CRITICAL;
  }
}
```

### 14.5 Delegation Management

**Delegation tracking and management:**

```typescript
interface DelegationRecord {
  readonly id: string;
  readonly grantor: Subject;
  readonly grantee: Subject;
  readonly permission: Permission;
  readonly constraints: DelegationConstraint[];
  readonly issuedAt: number;
  readonly expiresAt: number;
  readonly active: boolean;
  readonly auditTrail: DelegationAuditEntry[];
}

interface DelegationConstraint {
  readonly type: ConstraintType;
  readonly target: string;
  readonly expectation: any;
  readonly operators: Operator[];
}

enum ConstraintType {
  TIME_CONSTRAINT,
  CONTEXT_CONSTRAINT,
  USAGE_LIMIT,
  GEOGRAPHIC_CONSTRAINT,
  CONDITIONS_CONSTRAINT
}
```

### 14.6 Entitlement Validation

**Entitlement evaluation algorithm:**

```typescript
class EntitlementValidator {
  evaluateEntitlement(subject: Subject, resource: Resource, action: Action, context: DecisionContext): EntitlementDecision {
    const applicableEntitlements = this.findApplicableEntitlements(subject, resource, action, context);
    const entitlementDecisions = applicableEntitlements.map(entitlement => this.evaluateEntitlement(entitlement, subject, resource, action, context));
    return this.combineEntitlementDecisions(entitlementDecisions, context);
  }
  
  private findApplicableEntitlements(subject: Subject, resource: Resource, action: Action, context: DecisionContext): Entitlement[] {
    return subject.entitlements.filter(entitlement => 
      this.entitlementApplies(entitlement, subject, resource, action, context)
    );
  }
  
  private evaluateEntitlement(entitlement: Entitlement, subject: Subject, resource: Resource, action: Action, context: DecisionContext): EntitlementDecision {
    const applies = this.entitlementApplies(entitlement, subject, resource, action, context);
    const valid = this.entitlementValid(entitlement, subject, resource, action, context);
    
    if (!applies) {
      return { granted: false, reason: 'Entitlement does not apply' };
    }
    
    if (!valid) {
      return { granted: false, reason: 'Entitlement invalid' };
    }
    
    const decision = this.determineEntitlementDecision(entitlement, subject, resource, action, context);
    
    return {
      granted: decision.granted,
      effect: decision.effect,
      reason: `Entitlement ${entitlement.name}: ${decision.reason}`,
      conditions: entitlement.conditions,
      constraints: entitlement.constraints
    };
  }
}
```