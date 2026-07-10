# SECURITY.md

> **Purpose:** Define security requirements, threat model, and protective measures for CharOS.
> This document establishes security boundaries, risk mitigation, and compliance requirements.

---

## 1. Security Philosophy

### 1.1 Security Principles

> **Privacy by design, security by default.**

CharOS is built with the following security foundations:

| Principle | Implementation | Rationale |
|-----------|---------------|-----------|
| **Local-first** | Default execution mode; cloud only with explicit consent | Minimize external dependencies |
| **Zero trust** | Verify every request, least privilege access | Reduce attack surface |
| **Sandboxed execution** | Isolate plugins, tools, and models | Contain potential breaches |
| **Transparency** | All data handling logged and auditable | Build user trust |
| **Minimal attack surface** | Remove unnecessary features | Reduce vulnerabilities |
| **Defense in depth** | Multiple security layers | Mitigate single-point failures |

### 1.2 Threat Model

#### Key Assets at Risk

| Asset | Threat | Impact | Likelihood |
|-------|--------|--------|----------|
| **User credentials** | Keylogging, credential harvesting | Account compromise | Medium |
| **System access** | Privilege escalation | Full system control | High |
| **User data** | Data exfiltration, unauthorized access | Privacy violation | High |
| **Model weights** | Theft, reverse engineering | IP theft | Medium |
| **Plugin ecosystem** | Malicious plugins | System compromise | High |
| **Communication channels** | Eavesdropping, MITM | Data breach | Medium |
| **Configuration** | Unauthorized access | Misconfiguration, abuse | Medium |

#### Attack Vectors

1. **Malware through plugins**
2. **Escalation via tool abuse**
3. **Data exfiltration through memory**
4. **Social engineering through UI**
5. **Supply chain attacks (malicious models/files)**
6. **Network-based attacks (if cloud fallback enabled)**

---

## 2. Security Architecture

### 2.1 Security Layers

```
┌─────────────────────────────────────────────────────────┐
│                       USER DESKTOP                       │
├─────────────────────────────────────────────────────────┤
│  ┌─────────────────────┐ ┌─────────────────────────────┐ │
│  │   SANDBOX          │ │   SECURITY                │ │
│  │   CONTROLLER      │ │   MONITORING              │ │
│  │                   │ │                           │ │
│  └──────┬─────────────┘ └──────┬─────────────────────┘ │
│         │                       │                     │
│ ┌───────▼───────┐      ┌───────▼───────┐         │
│ │  ENCRYPTION    │      │  AUTHENTICATION  │         │
│ │  (local only) │      │   SYSTEM         │         │
│ └───────────────┘      └───────────────┘         │
│                       │                       │
│ ┌─────────────────────────────────────────────┐    │
│ │                 APPLICATION LAYER                   │    │
│ │  ┌─────────────────┐ ┌──────────────────┐     │    │
│ │  │  PLUGINS        │ │  SKILLS/TOOLS    │     │    │
│ │  │  (sandboxed)   │ │  (permissioned)  │     │    │
│ │  └───────────────┘ │ └──────────────────┘     │    │
│ │                   │                        │    │
│ │ ┌──────────────┐ │ ┌───────────────┐    │ ┌─────────┐ │
│ │ │   CHARACTER  │ │ │   MEMORY      │    │ │  MODELS  │ │
│ │ │   SYSTEM     │ │ │  (encrypted)   │    │ │  (safe)  │ │
│ │ └──────────────┘ │ └───────────────┘    │ └─────────┘ │
│ │                   │                        │    │
│ └─────────────────────────────────────────────┘    │
│                       │                       │
│ ┌─────────────────────────────────────────────┐    │
│ │                    OS INTERFACE                  │    │
│ │  ┌─────────────────┐ ┌──────────────────┐     │    │
│ │  │  FILE SYSTEM    │ │  NETWORK         │     │    │
│ │  │  (audited)      │ │  (monitored)     │     │    │
│ │  └───────────────┘ └───────────────┘     │    │
│ │                       │                        │    │
│ └─────────────────────────────────────────────┘    │
```

### 2.2 Security Boundaries

**Critical boundaries that must never be crossed:**

```text
┌─────────────────────────────┐
│        USER HOME           │
│  ┌───────────────────────┐ │
│  │  CHAROS SANDBOX       │ │
│  │  ┌─────────────────┐ │ │
│  │  │  PLUGIN EXEC    │ │ │
│  │  │  (syscalls)    │ │ │
│  │  └───────────────┘ │ │
│  │                    │ │
│  │  ┌─────────────────┐ │ │
│  │  │  TOOL EXECUTION │ │ │
│  │  │  (permissioned) │ │ │
│  │  └───────────────┘ │ │
│  │                    │ │
│  └───────────────────────┘ │
│              │           │
│        ┌─────▼─────┐   ┌─▼─────┐
│        │  MEMORY  │   │  UI   │
│        │ (encrypted)│   │ (safe)│
│        └───────────┘   └───────┘
```

### 2.3 Encryption Strategy

**Data protection:**

| Data Type | Storage Location | Encryption | Access Control |
|-----------|------------------|------------|----------------|
| **Memory** | Local SQLite + Obsidian | AES-256-GCM | Character tokens |
| **Configuration** | ~/.config/charos/ | OS keyring | File permissions |
| **Logs** | Rotated, compressed | AES-256 | Read-only |
| **Communication** | TLS 1.3+ | Mutual TLS | Certificate pinning |
| **Plugin code** | Sandboxed | Runtime decrypt | Memory isolation |

---

## 3. Permission System

### 3.1 Permission Architecture

**Universal permission system for all tool execution:**

```
type Permission = {
  id: string;
  name: string;
  description: string;
  category: 'filesystem' | 'network' | 'shell' | 'browser' | 'system';
  riskLevel: 'low' | 'medium' | 'high' | 'critical';
  default: 'deny' | 'request' | 'allow';
  pattern?: string; // Regex for path/file matching
};

type PermissionGrant = {
  permission: Permission;
  grantedBy: string;
  grantedAt: number;
  expiresAt?: number;
  conditions?: PermissionCondition[];
};

interface PermissionEngine {
  // Check if action is allowed
  check(permission: Permission, context: PermissionContext): Promise<PermissionResult>;
  
  // Grant permission
  grant(permission: PermissionGrant): Promise<void>;
  
  // Revoke permission
  revoke(grantId: string): Promise<void>;
  
  // List active permissions
  getActivePermissions(): Promise<PermissionGrant[]>;
  
  // Permission templates
  createTemplate(template: PermissionTemplate): Promise<void>;
  getTemplates(): Promise<PermissionTemplate[]>;
}
```

### 3.2 Permission Categories

| Category | Description | Examples | Risk |
|----------|-------------|----------|------|
| **Filesystem** | Read/write access to files and directories | `/home/user/docs`, `/tmp`, `~/.ssh` | Medium |
| **Network** | Network connections and HTTP/SOCKET access | `https://api.example.com`, `localhost:8080` | High |
| **Shell** | Command execution, process management | `bash`, `python`, `git` | Critical |
| **Browser** | Browser automation and web access | `playwright`, `selenium` | High |
| **System** | System-level operations | `systemctl`, `usermod`, `sudo` | Critical |
| **Memory** | Access to memory stores | Obsidian vault, memory queries | Low |
| **Calendar** | Schedule access | Google Calendar, Outlook | Medium |

### 3.3 Permission Workflow

**Execution flow with permission checks:**

```
┌─────────────────────────────────────────────────────────┐
│                 TOOL EXECUTION FLOW                     │
├─────────────────────────────────────────────────────────┤
│  Task Plan                                              │
│  ┌───────────────────────┐                              │
│  │  Skill: FileBackup    │                              │
│  │  ┌─────────────────┐ │                              │
│  │  │  Tool: cp src/   │ │   Permission Requested    │
│  │  │  dest/backup    │ │   ┌─────────────────────┐ │
│  │  └─────────────────┘ │   │ PermissionEngine   │ │
│  └───────────────────────┘ │   ┌─────────────────────┐ │
│                           │   │ Check permission │ │
│ ┌─────────────────────────┐ │   │ (filesystem.read) │ │
│ │  Permission Engine     │ │──▶│                   │ │
│ │  ┌─────────────────┐ │   │ ┌─────────────────┐ │
│ │  │  Database       │ │   │ Deny: No matching │ │
│ │  │  (memory)       │ │   │ path permissions  │ │
│ │  └─────────────────┘ │   └─────────────────┘ │
│ │                           │                     │
│ │ ┌─────────────────────────┐ │   ┌─────────────────┐ │
│ │ │  User Prompt /       │ │   │ Request approval │ │
│ │ │  Admin Approval       │ │──▶│ from owner         │ │
│ │ │                     │ │   ┌─────────────────┐ │
│ │ │ ┌─────────────────┐ │   │ Approve / Deny   │ │
│ │ │ │  UI Confirmation │ │   └─────────────────┘ │
│ │ │ └─────────────────┘ │                     │
│ │                           │                     │
│ │ ┌─────────────────────────┐ │   ┌─────────────────┐ │
│ │ │  Permission Granted   │ │   │ Permission stored │ │
│ │ │  (5 minutes)         │ │──▶│ (expires after   │ │
│ │ │                     │ │   │ 5 minutes)       │ │
│ │ │ ┌─────────────────┐ │   └─────────────────┘ │
│ │ │ │ Execute tool     │ │                     │
│ │ │ └─────────────────┘ │                     │
│ │                           │                     │
│ │ ┌─────────────────────────┐ │   ┌─────────────────┐ │
│ │ │  Audit Log          │ │   │ Audit record     │ │
│ │ │  (create entry)    │ │──▶│ stored          │ │
│ │ │                     │ │   └─────────────────┘ │
│ │ └─────────────────────────┘ │                     │
│ └─────────────────────────────────────────────────────────┘
```

### 3.4 Permission Templates

**Pre-defined permission sets:**

```json
{
  "id": "backup-workflow",
  "name": "Backup Workflow",
  "description": "Safe permissions for automated backups",
  "riskLevel": "low",
  "permissions": [
    {
      "category": "filesystem",
      "operations": ["read", "write"],
      "paths": ["~/Documents", "~/Desktop"],
      "restrictions": {
        "deny_write_to_admin": true,
        "max_files_per_operation": 100
      }
    },
    {
      "category": "network",
      "operations": ["https"],
      "hosts": ["backup-server.example.com"],
      "ports": [443, 22]
    }
  ],
  "approvalRequired": true,
  "autoApproveDuration": 300000,
  "subject": "backup-job-123"
}
```

---

## 4. Plugin Security

### 4.1 Plugin Signing and Verification

**Plugin authenticity and integrity:**

```typescript
interface PluginSecurity {
  // Plugin signing
  signPlugin(plugin: Plugin): Promise<SignedPlugin>;
  verifyPluginSignature(plugin: Plugin): Promise<boolean>;
  
  // Integrity verification
  verifyPluginIntegrity(plugin: Plugin): Promise<boolean>;
  checkPluginMalwareSignatures(plugin: Plugin): Promise<MalwareScanResult>;
  
  // Repository security
  validateRepositoryAccess(repo: Repository): Promise<boolean>;
  checkDependencyTrust(repo: Repository): Promise<DependencyTrust>;
  
  // Runtime security
  loadPluginWithSecurity(plugin: Plugin): Promise<SecurePlugin>;
}
```

### 4.2 Sandbox Environment

**Plugin isolation mechanisms:**

```yaml
plugin-security:
  sandbox-type: "gvisor"
  # Alternative: "container", "wsl", "network-namespace"
  
  network:
    allowed-ips: ["10.0.0.0/8", "172.16.0.0/12"]
    blocked-ports: ["22", "23", "25"]
    allowed-hostnames: ["api.github.com", "ollama.example.com"]
  
  filesystem:
    allowed-paths: ["~/data", "~/plugins"]
    blocked-paths: ["/etc", "/sys", "/proc"]
    read-only: ["/usr"]
  
  capabilities:
    require-approval: ["filesystem", "network"]
    auto-approve-duration: 300000
  
  monitoring:
    log-all-syscalls: true
    resource-limits:
      max-memory: "512m"
      max-cpu: "50%"
      max-disk: "1GB"
```

### 4.3 Supply Chain Security

**Plugin distribution security:**

- **Code signing**: Every plugin must be signed with a valid certificate
- **Security scanning**: Automated malware detection before distribution
- **Repository validation**: Trusted repositories only
- **Dependency verification**: Check all external dependencies
- **Package integrity**: Manifest-based verification

---

## 5. Runtime Security

### 5.1 Tool Execution Security

**Safe tool execution framework:**

```typescript
interface SafeToolExecutor {
  readonly engine: ExecutionEngine;
  readonly permissionCheck: PermissionEngine;
  readonly auditLogger: AuditLogger;
  readonly securityMonitor: SecurityMonitor;
  
  // Execute tool with security checks
  executeTool(toolName: string, args: string[], context: ToolExecutionContext): Promise<ToolResult>;
  
  // Pre-execution validation
  validateTool(tool: ToolDefinition): Promise<ValidationResult>;
  
  // Monitoring during execution
  getExecutionLogs(executionId: string): Promise<ExecutionLog[]>;
  monitorForAnomalies(): Promise<SecurityAlert>;
}
```

### 5.2 Anomaly Detection

**Behavior-based security monitoring:**

```typescript
interface AnomalyDetector {
  // Pattern learning
  learnNormalBehavior(userId: string, patterns: BehaviorPattern[]): Promise<void>;
  
  // Anomaly detection
  detectAnomalies(userId: string, events: SecurityEvent[]): Promise<Anomaly[]>`
  
  // Alert generation
  generateAlert(anomaly: Anomaly): Promise<SecurityAlert>;
  
  // Response actions
  takeMitigationAction(alert: SecurityAlert): Promise<void>;
}
```

### 5.3 Incident Response

**Security incident handling:**

```typescript
interface IncidentResponse {
  // Incident detection and classification
  detectIncident(event: SecurityEvent): Promise<Incident>;
  
  // Containment
  containIncident(incident: Incident): Promise<void>;
  
  // Eradication
  eradicateThreat(incident: Incident): Promise<void>;
  
  // Recovery
  recoverSystems(incident: Incident): Promise<void>;
  
  // Post-incident investigation
  investigateIncident(incident: Incident): Promise<InvestigationReport>;
}
```

---

## 6. Communication Security

### 6.1 Network Security

**Secure communication channels:**

```typescript
interface SecureChannel {
  readonly channelId: string;
  readonly protocol: 'tls' | 'wireguard' | 'ipsec' | 'custom';
  readonly encryption: 'aes256gcm' | 'chacha20poly1305' | 'aes256gcm-siv';
  readonly authentication: 'cert' | 'psk' | 'oauth' | 'none';
  
  // Connection management
  connect(peer: Peer): Promise<void>;
  disconnect(): Promise<void>;
  
  // Data transfer
  send(data: ArrayBuffer): Promise<void>;
  receive(): Promise<ArrayBuffer>;
  
  // Security monitoring
  getSecurityStatus(): Promise<SecurityStatus>;
  monitorForLeaks(): Promise<LeakAlert[]>;
}
```

### 6.2 Browser Fallback Security

**Secure browser automation:**

```typescript
interface SecureBrowserAutomation {
  readonly sandbox: BrowserSandbox;
  readonly auth: BrowserAuth;
  readonly monitoring: BrowserMonitor;
  
  // Session management
  createSession(userId: string): Promise<BrowserSession>;
  closeSession(sessionId: string): Promise<void>;
  
  // Page navigation
  navigate(sessionId: string, url: string, options?: NavigationOptions): Promise<Page>;
  
  // Action execution
  executeAction(sessionId: string, action: UserAction): Promise<ActionResult>;
  
  // Security checks
  validateAuthentication(sessionId: string): Promise<AuthenticationResult>;
  monitorForSuspiciousActivity(sessionId: string): Promise<SuspiciousActivity[]>;
}
```

---

## 7. Auditing and Compliance

### 7.1 Audit Logging

**Comprehensive audit trails:**

```typescript
interface AuditLogger {
  // Log events
  logSecurityEvent(event: SecurityEvent): Promise<AuditEntry>;
  logPermissionChange(change: PermissionChange): Promise<AuditEntry>;
  logToolExecution(execution: ToolExecution): Promise<AuditEntry>;
  logPluginLoad(plugin: PluginLoad): Promise<AuditEntry>;
  
  // Report generation
  generateAuditReport(period: TimeRange): Promise<AuditReport>;
  
  // Compliance checking
  checkCompliance(standard: ComplianceStandard): Promise<ComplianceResult>;
  
  // Privacy preservation
  anonymizePersonalData(entry: AuditEntry): AuditEntry;
}
```

### 7.2 Compliance Standards

**Adherence to security standards:**

- **GDPR**: Personal data protection, consent management
- **SOC 2**: Security controls, availability, confidentiality
- **ISO 27001**: Information security management
- **NIST CSF**: Cybersecurity framework
- **HIPAA**: Health information protection (if applicable)
- **PCI DSS**: Payment data security (if applicable)

---

## 8. Incident Response Plan

### 8.1 Severity Tiers

| Severity | Description | Response Time | Action |
|----------|-------------|---------------|--------|
| **Critical** | System compromise, data breach | 15 minutes | Immediate containment |
| **High** | Privilege escalation, ransomware | 30 minutes | Rapid response |
| **Medium** | Security policy violation | 2 hours | Investigation |
| **Low** | Configuration error, policy advisory | 4 hours | Documentation |

### 8.2 Response Procedures

**Step-by-step incident response:**

1. **Identification**: Detect and classify incident
2. **Containment**: Isolate affected systems
3. **Eradication**: Remove malicious components
4. **Recovery**: Restore affected systems
5. **Lessons learned**: Document and improve

### 8.3 Communication Protocols

**Stakeholder notification:**

- **Users**: Transparent communication about impacts
- **Admins**: Detailed technical information
- **Regulators**: Required reporting per jurisdiction
- **Partners**: Coordinated response

---

## 9. Cross-References

| Document | Field | Relationship |
|----------|-------|-------------|
| `docs/00_VISION.md` | Section 6.3 | Privacy-aware design |
| `docs/01_ARCHITECTURE.md` | All sections | Security constraints |
| `docs/02_DESIGN_PHILOSOPHY.md` | Section 10 | Security by design |
| `docs/03_TERMINOLOGY.md` | Security section | Canonical security terms |
| `docs/05_TECH_STACK.md` | Section 5 | Security implications |
| `docs/06_UI_GUIDELINES.md` | Section 13 | Security UI/UX |
| `roadmap.md` | Section 20.4 | Security roadmap |
| `PROJECT_BIBLE.md` | All sections | Constitutional security principles |

---

## 10. Open Design Questions

### 10.1. Encryption Key Management

| Option | Pros | Cons |
|--------|------|------|
| **Hardware security module** | Fastest, most secure | Single point of failure |
| **Cloud KMS** | Scalable, backup | Network dependency |
| **Split knowledge** | No single point | Complex management |
| **Blockchain key storage** | Immutable, transparent | Performance concerns |

**Status:** Hardware security module for local keys, cloud KMS for backups.

### 10.2 Plugin Signing Infrastructure

| Option | Pros | Cons |
|--------|------|------|
| **Central certificate authority** | Simple, trusted | Single point of failure |
| **Distributed trust** | Decentralized, resilient | Complex verification |
| **Self-signed certificates** | No infrastructure | Lower trust |

**Status:** Central CA with fallback to distributed trust.

### 10.3 Anomaly Detection ML

| Option | Pros | Cons |
|--------|------|------|
| **On-device ML** | Privacy, offline | Limited data |
| **Cloud ML** | Richer models | Privacy concerns |
| **Hybrid approach** | Balanced approach | Complex architecture |

**Status:** Hybrid with on-device for privacy, cloud for enhancement.

---

## 11. TODOs for Implementation

- [ ] Create PermissionEngine with runtime security
- [ ] Implement Plugin Signing and Verification system
- [ ] Build Sandbox Environment with isolation
- [ ] Set up Security Monitoring and Anomaly Detection
- [ ] Create Audit Logging and Compliance reporting
- [ ] Implement Incident Response Procedures
- [ ] Setup Security Testing and Validation
- [ ] Create Security Documentation and Training
- [ ] Record ADR for Security Architecture
- [ ] Record ADR for Permission System
- [ ] Record ADR for Plugin Security

---

> **Security is not a feature you add — it is a foundation you build upon.**
>
> *If you think security will slow you down, you will definitely be exploited.*

---

## 12. Emergency Guidelines

### If Security is Compromised

1. **Immediate Actions**
   - Disconnect from network
   - Isolate affected plugins
   - Preserve evidence
   - Notify administrators

2. ** Investigation**
   - Review logs
   - Identify affected components
   - Determine scope of damage
   - Document timeline

3. **Recovery**
   - Restore from clean backup
   - Patch vulnerabilities
   - Update all components
   - Verify integrity

4. **Communication**
   - Inform users if data exposed
   - Report to security teams
   - Document lessons learned

---

> **Better to ask for forgiveness than permission when it comes to security.**

> **But build permission systems anyway — they make security easier and sustainable.**