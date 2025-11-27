# ISO 25010 Quality Requirements

Use when defining quality requirements. Guides through the ISO/IEC 25010 quality model to create measurable, testable quality requirements.

## Purpose

Systematically identifies and documents quality requirements (non-functional requirements) using the ISO/IEC 25010 quality model. Creates measurable quality attributes with clear acceptance criteria.

## When to Use

- Starting a new project that needs quality requirements
- Conducting a quality attribute workshop
- Preparing for architecture decisions driven by quality
- Creating quality scenarios for testing
- Documenting quality requirements for stakeholders

## Prerequisites

- Obsidian vault initialized (use init-obsidian-vault skill)
- Basic understanding of system requirements
- Stakeholder input on quality priorities

## ISO/IEC 25010 Quality Model

The ISO/IEC 25010 standard defines 8 main quality characteristics and ~30 sub-characteristics:

```
Product Quality
├── 1. Functional Suitability
│   ├── Functional Completeness
│   ├── Functional Correctness
│   └── Functional Appropriateness
├── 2. Performance Efficiency
│   ├── Time Behavior
│   ├── Resource Utilization
│   └── Capacity
├── 3. Compatibility
│   ├── Co-existence
│   └── Interoperability
├── 4. Usability
│   ├── Appropriateness Recognizability
│   ├── Learnability
│   ├── Operability
│   ├── User Error Protection
│   ├── User Interface Aesthetics
│   └── Accessibility
├── 5. Reliability
│   ├── Maturity
│   ├── Availability
│   ├── Fault Tolerance
│   └── Recoverability
├── 6. Security
│   ├── Confidentiality
│   ├── Integrity
│   ├── Non-repudiation
│   ├── Accountability
│   └── Authenticity
├── 7. Maintainability
│   ├── Modularity
│   ├── Reusability
│   ├── Analyzability
│   ├── Modifiability
│   └── Testability
└── 8. Portability
    ├── Adaptability
    ├── Installability
    └── Replaceability
```

## Execution Steps

### Step 1: Verify Vault Structure

Check vault exists:

```bash
ls -d vault/requirements/ 2>/dev/null || echo "ERROR: Run init-obsidian-vault skill first"
```

### Step 2: Interview Stakeholders

Guide stakeholder through quality characteristics to identify priorities:

#### Priority Setting Exercise

Ask: "Rate each quality characteristic by importance for your system:"

- **Critical**: System cannot succeed without this
- **High**: Very important but not critical
- **Medium**: Important but can be compromised
- **Low**: Nice to have but not required

#### Context Questions

1. **What type of system is this?**
   - Web application, mobile app, embedded system, API, etc.

2. **Who are the primary users?**
   - End users, developers, operators, etc.

3. **What is the expected usage?**
   - User count, transaction volume, data volume

4. **What are the business criticality factors?**
   - Revenue impact, regulatory requirements, brand reputation

5. **What quality issues would be most damaging?**
   - Downtime, data loss, security breach, poor performance

### Step 3: Explore Each Characteristic

For each HIGH or CRITICAL characteristic, explore sub-characteristics in detail.

#### 1. Functional Suitability

**Definition**: Degree to which product provides functions that meet stated and implied needs.

**Guide Questions**:

- **Functional Completeness**: Does it do everything users need?
  - What functions are essential?
  - What functions are nice-to-have?
  - Are there gaps in functionality?

- **Functional Correctness**: Does it produce correct results?
  - How accurate must calculations be?
  - What tolerance for errors?
  - What validation is needed?

- **Functional Appropriateness**: Does it help users achieve goals efficiently?
  - Are there simpler ways to achieve goals?
  - Are functions appropriate for the task?

**Example Requirements**:
- "System must support all CRUD operations for user accounts"
- "Calculations must be accurate to 2 decimal places"
- "Users must be able to complete checkout in ≤ 3 steps"

#### 2. Performance Efficiency

**Definition**: Performance relative to resources used under stated conditions.

**Guide Questions**:

- **Time Behavior**: How fast must it be?
  - API response time requirements?
  - UI interaction response time?
  - Batch processing time limits?
  - Real-time constraints?

- **Resource Utilization**: How efficiently must it use resources?
  - Memory limits?
  - CPU usage limits?
  - Storage requirements?
  - Network bandwidth?

- **Capacity**: How much load must it handle?
  - Concurrent users?
  - Transactions per second?
  - Data volume?
  - Growth projections?

**Example Requirements**:
- "API p95 response time < 200ms under normal load"
- "System must handle 1000 concurrent users"
- "Memory usage < 4GB per instance"
- "Database queries p95 < 50ms"

#### 3. Compatibility

**Definition**: Degree to which product can exchange information and perform functions while sharing environment.

**Guide Questions**:

- **Co-existence**: Can it coexist with other software?
  - What other software runs in the same environment?
  - Are there resource conflicts?
  - Port conflicts?

- **Interoperability**: Can it exchange data with other systems?
  - What systems must it integrate with?
  - What data formats?
  - What protocols?
  - API compatibility requirements?

**Example Requirements**:
- "Must integrate with existing LDAP authentication"
- "Must export data in CSV and JSON formats"
- "API must be backward compatible for 2 major versions"
- "Must support OAuth 2.0 for third-party integrations"

#### 4. Usability

**Definition**: Degree to which product can be used by specified users to achieve specified goals.

**Guide Questions**:

- **Appropriateness Recognizability**: Can users recognize if it's appropriate?
  - Is the purpose clear?
  - Can users quickly understand what it does?

- **Learnability**: How quickly can users learn it?
  - Time to first successful use?
  - Training requirements?
  - Documentation needs?

- **Operability**: How easy is it to operate?
  - Steps to complete tasks?
  - Error recovery?
  - Undo/redo capabilities?

- **User Error Protection**: How well does it prevent errors?
  - Input validation?
  - Confirmation for destructive actions?
  - Clear error messages?

- **User Interface Aesthetics**: Is the interface pleasant?
  - Brand consistency?
  - Professional appearance?
  - Visual design quality?

- **Accessibility**: Can people with disabilities use it?
  - WCAG compliance level?
  - Screen reader support?
  - Keyboard navigation?
  - Color contrast?

**Example Requirements**:
- "New users must complete first task within 5 minutes without help"
- "All destructive actions require confirmation"
- "UI must meet WCAG 2.1 Level AA"
- "Error messages must suggest corrective actions"
- "All features accessible via keyboard"

#### 5. Reliability

**Definition**: Degree to which system performs specified functions under specified conditions.

**Guide Questions**:

- **Maturity**: How stable and bug-free?
  - Acceptable bug rate?
  - Mean time between failures?
  - Defect density targets?

- **Availability**: How often must it be available?
  - Uptime percentage?
  - Acceptable downtime?
  - Maintenance windows?

- **Fault Tolerance**: Can it operate despite failures?
  - Single points of failure?
  - Graceful degradation?
  - Redundancy requirements?

- **Recoverability**: Can it recover from failures?
  - Recovery time objective (RTO)?
  - Recovery point objective (RPO)?
  - Automatic recovery?

**Example Requirements**:
- "System uptime ≥ 99.9% (< 43 minutes downtime/month)"
- "RTO < 1 hour, RPO < 5 minutes"
- "System must gracefully handle individual service failures"
- "Automatic failover to backup systems"
- "Mean time to recovery < 15 minutes"

#### 6. Security

**Definition**: Degree to which product protects information and data.

**Guide Questions**:

- **Confidentiality**: Is sensitive data protected?
  - Data classification scheme?
  - Encryption requirements?
  - Access control granularity?

- **Integrity**: Is data protected from tampering?
  - Data validation?
  - Checksums/signatures?
  - Audit trails?

- **Non-repudiation**: Can actions be proven?
  - Audit logging requirements?
  - Digital signatures?
  - Transaction logs?

- **Accountability**: Can actions be traced to users?
  - User identification?
  - Action logging?
  - Compliance requirements?

- **Authenticity**: Can users/systems be verified?
  - Authentication methods?
  - Multi-factor authentication?
  - Certificate validation?

**Example Requirements**:
- "All sensitive data encrypted at rest (AES-256)"
- "All connections use TLS 1.3"
- "Authentication via MFA for privileged accounts"
- "All user actions logged with timestamp and user ID"
- "Password must meet complexity policy (12+ chars, mixed case, numbers, symbols)"
- "Failed login attempts locked after 5 attempts"
- "Session timeout after 30 minutes of inactivity"

#### 7. Maintainability

**Definition**: Degree of effectiveness and efficiency of modifying the product.

**Guide Questions**:

- **Modularity**: Is it well-decomposed?
  - Clear component boundaries?
  - Low coupling, high cohesion?
  - Component independence?

- **Reusability**: Can components be reused?
  - Generic components?
  - Library/framework approach?
  - API-first design?

- **Analyzability**: Can issues be diagnosed?
  - Logging adequacy?
  - Monitoring capabilities?
  - Debug information?

- **Modifiability**: How easy to change?
  - Time to add typical feature?
  - Impact of changes?
  - Configuration vs. code changes?

- **Testability**: How easy to test?
  - Unit test coverage targets?
  - Integration test coverage?
  - Test automation level?

**Example Requirements**:
- "Unit test coverage ≥ 80% for business logic"
- "All public APIs must have integration tests"
- "New simple feature can be added in ≤ 2 days"
- "Components can be developed and tested independently"
- "Structured logging enables root cause analysis"
- "Configuration via environment variables (12-factor app)"

#### 8. Portability

**Definition**: Degree of effectiveness and efficiency of transferring the product.

**Guide Questions**:

- **Adaptability**: Can it adapt to different environments?
  - Platform requirements?
  - Environment variations?
  - Configuration flexibility?

- **Installability**: How easy to install?
  - Installation time?
  - Installation complexity?
  - Automation level?

- **Replaceability**: Can it replace another product?
  - Data migration requirements?
  - API compatibility?
  - Feature parity?

**Example Requirements**:
- "Must run on Linux, macOS, and Windows"
- "Deployment via Docker containers"
- "Installation automated via single command"
- "Support import from competitor's data format"
- "Must run on AWS, Azure, and GCP with minimal changes"

### Step 4: Define Quality Scenarios

For each important quality requirement, create a quality scenario:

**Scenario Structure**:
- **Stimulus**: What triggers this scenario
- **Environment**: Conditions when it occurs
- **Response**: What the system does
- **Measure**: How to quantify success

**Example Performance Scenario**:
```markdown
**Stimulus**: User requests list of 100 resources

**Environment**: Normal operation, 50 concurrent users

**Response**: System retrieves data from cache if available, otherwise from database

**Measure**:
- p50 response time < 100ms
- p95 response time < 200ms
- p99 response time < 500ms
- Cache hit rate > 80%
```

### Step 5: Create Quality Requirements Document

Create `vault/requirements/quality-attributes.md` with structured content:

```markdown
# Quality Requirements

**Last Updated**: YYYY-MM-DD

## Overview

[Brief summary of quality priorities for this system]

## Quality Priorities

| Rank | Characteristic | Sub-characteristic | Priority | Rationale |
|------|---------------|-------------------|----------|-----------|
| 1 | [e.g., Security] | [e.g., Confidentiality] | Critical | [Why critical] |
| 2 | [e.g., Performance] | [e.g., Time Behavior] | High | [Why high] |
| 3 | [e.g., Reliability] | [e.g., Availability] | High | [Why high] |

## Quality Requirements by Characteristic

[For each important characteristic, create detailed section]

### [Characteristic Name]

**Priority**: Critical / High / Medium

**Rationale**: [Why this is important for this system]

#### Requirements

| ID | Sub-characteristic | Requirement | Metric | Target | Priority |
|----|-------------------|-------------|--------|--------|----------|
| QR-001 | [Sub-char] | [Requirement text] | [How measured] | [Target value] | Critical |
| QR-002 | [Sub-char] | [Requirement text] | [How measured] | [Target value] | High |

#### Quality Scenarios

[Include 2-3 detailed scenarios for this characteristic]

#### Verification

[How these requirements will be verified - tests, reviews, etc.]

## Complete Requirements List

[All requirements in one place for reference]

## Traceability

### To Architecture

- [Requirement ID] → [[../arc42/04-solution|Solution Strategy]]
- [Requirement ID] → [[../arc42/07-deployment|Deployment View]]

### To Specifications

- [Requirement ID] → [[../specifications/features#scenario]]

### To Tests

- [Requirement ID] → `test/performance/api_test.clj`
- [Requirement ID] → `test/security/auth_test.clj`

## Related Documentation

- [[../arc42/10-quality|arc42 Quality Requirements]]
- [[../arc42/01-introduction|Quality Goals]]
- [[../decisions/index|ADRs]] - Architecture decisions driven by quality
```

### Step 6: Link to arc42 Documentation

Update arc42 Section 10 to reference this document:

```markdown
# 10. Quality Requirements

See [[../requirements/quality-attributes|Quality Attributes]] for complete quality requirements.

[Include summary and key scenarios]
```

### Step 7: Create Requirement Template

Create template for future requirements in `vault/templates/requirement.md` (if not already exists).

### Step 8: Report Completion

```
Quality requirements documented in vault/requirements/quality-attributes.md

Characteristics analyzed:
✓ Performance Efficiency - 5 requirements, 3 scenarios
✓ Security - 8 requirements, 4 scenarios
✓ Reliability - 4 requirements, 2 scenarios
✓ Maintainability - 6 requirements, 2 scenarios

Total: 23 quality requirements defined

Next steps:
1. Review with stakeholders
2. Link to architecture decisions (ADRs)
3. Create test specifications for quality scenarios
4. Update arc42 Section 10
5. Track in backlog/sprints
```

## Complete Example

Example `vault/requirements/quality-attributes.md`:

```markdown
# Quality Requirements

**Last Updated**: 2025-01-15

**System**: E-Commerce Platform

## Overview

This document defines the quality requirements for the e-commerce platform. Top priorities are performance (for user satisfaction), security (for customer trust and compliance), and reliability (for business continuity).

## Quality Priorities

| Rank | Characteristic | Sub-characteristic | Priority | Rationale |
|------|---------------|-------------------|----------|-----------|
| 1 | Performance | Time Behavior | Critical | Poor performance drives customers away |
| 2 | Security | Confidentiality | Critical | PCI-DSS compliance required, customer trust |
| 3 | Security | Authenticity | Critical | Prevent fraud, secure transactions |
| 4 | Reliability | Availability | High | Revenue loss during downtime |
| 5 | Usability | Learnability | High | First-time buyers must succeed easily |
| 6 | Maintainability | Testability | Medium | Enable rapid feature development |

## Quality Requirements by Characteristic

### 1. Performance Efficiency

**Priority**: Critical

**Rationale**: User studies show 53% of mobile users abandon sites that take > 3 seconds to load. Every 100ms of latency costs 1% in sales.

#### Requirements

| ID | Sub-characteristic | Requirement | Metric | Target | Priority |
|----|-------------------|-------------|--------|--------|----------|
| QR-001 | Time Behavior | API response time | p95 | < 200ms | Critical |
| QR-002 | Time Behavior | Page load time | p95 | < 2 seconds | Critical |
| QR-003 | Time Behavior | Search response | p95 | < 300ms | High |
| QR-004 | Capacity | Concurrent users | Count | 10,000 | High |
| QR-005 | Resource Util | Database connections | Pool size | 20 per instance | Medium |

#### Quality Scenarios

**Scenario 1: Product Search**

**Stimulus**: User enters search query

**Environment**: Peak shopping hours (Black Friday), 5000 concurrent users

**Response**: System queries search index, returns paginated results

**Measure**:
- p50 < 100ms
- p95 < 300ms
- p99 < 500ms
- Search index response < 50ms

**Scenario 2: Add to Cart**

**Stimulus**: User adds product to cart

**Environment**: Normal operation, 1000 concurrent users

**Response**: System updates cart in cache, confirms to user

**Measure**:
- p95 < 150ms
- Cache write < 10ms
- UI update < 50ms

**Scenario 3: Checkout Flow**

**Stimulus**: User completes checkout

**Environment**: Normal operation

**Response**: System processes order, calls payment gateway, confirms order

**Measure**:
- Total checkout time p95 < 3 seconds
- Payment gateway call < 1 second
- Order confirmation < 500ms

#### Verification

- **Load Testing**: k6 scripts run in CI/CD for every release
- **Monitoring**: CloudWatch metrics tracked in production
- **Alerts**: PagerDuty alert if p95 > 300ms for 5 minutes

### 2. Security

**Priority**: Critical

**Rationale**: PCI-DSS compliance required for payment processing. Customer trust essential for business. Data breach would be catastrophic.

#### Requirements

| ID | Sub-characteristic | Requirement | Metric | Target | Priority |
|----|-------------------|-------------|--------|--------|----------|
| QR-010 | Confidentiality | Data encryption at rest | Algorithm | AES-256 | Critical |
| QR-011 | Confidentiality | Data encryption in transit | Protocol | TLS 1.3 | Critical |
| QR-012 | Authenticity | User authentication | Method | JWT + MFA (admin) | Critical |
| QR-013 | Authenticity | Password strength | Policy | 12+ chars, mixed | Critical |
| QR-014 | Integrity | Payment data integrity | Validation | Checksum verified | Critical |
| QR-015 | Accountability | Audit logging | Coverage | All sensitive actions | High |
| QR-016 | Non-repudiation | Transaction signing | Method | Digital signature | High |
| QR-017 | Confidentiality | Session timeout | Duration | 30 minutes inactive | Medium |

#### Quality Scenarios

**Scenario 1: User Authentication**

**Stimulus**: User attempts to log in with credentials

**Environment**: Production system, internet-facing

**Response**: System validates credentials, generates JWT, creates session

**Measure**:
- Password verified using bcrypt
- Failed attempts logged
- Account locked after 5 failures
- Session token expires in 30 minutes
- MFA required for admin accounts

**Scenario 2: Payment Processing**

**Stimulus**: User submits payment information

**Environment**: Checkout flow, HTTPS connection

**Response**: System validates, tokenizes card data, processes payment

**Measure**:
- All payment data sent over TLS 1.3
- Card data never stored (tokenized)
- Transaction logged with signature
- PCI-DSS compliance verified
- Payment gateway response < 1 second

**Scenario 3: Data Breach Attempt**

**Stimulus**: Attacker attempts SQL injection

**Environment**: Production API

**Response**: System rejects invalid input, logs attempt, notifies security team

**Measure**:
- Input validation rejects malicious input
- Attack logged with IP, timestamp, payload
- Alert sent if >10 attempts from single IP
- Database uses parameterized queries only

#### Verification

- **Penetration Testing**: Annual third-party security audit
- **Automated Scanning**: OWASP ZAP in CI/CD
- **Code Review**: Security review for all auth/payment code
- **Compliance**: Annual PCI-DSS audit

### 3. Reliability

**Priority**: High

**Rationale**: Downtime directly impacts revenue. System unavailability during peak shopping damages reputation and loses sales.

#### Requirements

| ID | Sub-characteristic | Requirement | Metric | Target | Priority |
|----|-------------------|-------------|--------|--------|----------|
| QR-020 | Availability | System uptime | Percentage | ≥ 99.9% | High |
| QR-021 | Recoverability | Recovery time | RTO | < 1 hour | High |
| QR-022 | Recoverability | Data loss | RPO | < 5 minutes | High |
| QR-023 | Fault Tolerance | Service failures | Degradation | Graceful | High |

#### Quality Scenarios

**Scenario 1: Database Failure**

**Stimulus**: Primary database instance crashes

**Environment**: Production, during business hours

**Response**: System automatically fails over to replica

**Measure**:
- Automatic failover < 2 minutes
- No data loss (RPO = 0 for this scenario)
- Application reconnects automatically
- Alerts sent to ops team

**Scenario 2: API Instance Failure**

**Stimulus**: One API server instance crashes

**Environment**: Production, 5 instances running

**Response**: Load balancer removes failed instance, routes to healthy instances

**Measure**:
- Health check detects failure < 30 seconds
- Traffic rerouted < 10 seconds
- No user impact (4 instances handle load)
- New instance auto-scales within 5 minutes

#### Verification

- **Chaos Engineering**: Monthly failure drills
- **Monitoring**: Uptime tracking via Pingdom
- **Backup Testing**: Monthly restore verification

## Requirements Summary

### Critical Requirements (9)

- QR-001: API response time p95 < 200ms
- QR-002: Page load time p95 < 2s
- QR-010: AES-256 encryption at rest
- QR-011: TLS 1.3 for all connections
- QR-012: JWT + MFA authentication
- QR-013: Strong password policy
- QR-014: Payment data integrity verification
- QR-020: 99.9% uptime
- QR-021: RTO < 1 hour

### High Requirements (8)

[List high priority requirements]

### Medium Requirements (6)

[List medium priority requirements]

## Traceability

### To Architecture Decisions

- QR-001, QR-002 → [[../decisions/0004-caching-strategy|ADR-0004: Caching Strategy]]
- QR-012 → [[../decisions/0005-jwt-auth|ADR-0005: JWT Authentication]]
- QR-020, QR-021 → [[../decisions/0020-deployment-strategy|ADR-0020: Blue-Green Deployment]]

### To arc42 Sections

- Performance → [[../arc42/10-quality#performance|arc42 Section 10: Performance]]
- Security → [[../arc42/08-crosscutting#security|arc42 Section 8: Security]]
- Reliability → [[../arc42/07-deployment#disaster-recovery|arc42 Section 7: DR]]

### To Specifications

- QR-001 → [[../specifications/features#performance-tests|Performance Test Scenarios]]
- QR-012 → [[../specifications/features#authentication|Authentication Scenarios]]

### To Tests

- QR-001, QR-002 → `test/performance/load_test.js` (k6)
- QR-012 → `test/security/auth_test.clj`
- QR-020 → `test/reliability/failover_test.clj`

## Related Documentation

- [[../arc42/10-quality|arc42 Quality Requirements]]
- [[../arc42/01-introduction#quality-goals|Quality Goals]]
- [[../security/README|Security Documentation]]
- [[README|Main Vault Index]]

---

*Last updated: 2025-01-15*
*Next review: 2025-04-15*
```

## Quality Requirement Template

Template for individual requirements:

```markdown
**ID**: QR-XXX

**Characteristic**: [ISO 25010 characteristic]

**Sub-characteristic**: [ISO 25010 sub-characteristic]

**Requirement**: [Clear statement of requirement]

**Rationale**: [Why this requirement exists]

**Metric**: [How it will be measured]

**Target**: [Specific measurable target]

**Priority**: Critical / High / Medium / Low

**Verification**: [How it will be verified]

**Related**:
- Architecture: [[../arc42/XX-section]]
- ADRs: [[../decisions/NNNN-decision]]
- Tests: `path/to/test`
```

## Best Practices

1. **Be Specific**: "Fast" is not a requirement. "p95 < 200ms" is.
2. **Be Measurable**: Every requirement needs a metric
3. **Be Testable**: Define how you'll verify each requirement
4. **Prioritize**: Not everything can be critical
5. **Context Matters**: Requirements depend on system type and users
6. **Trade-offs**: Document why you accept lower priority for some qualities
7. **Review Regularly**: Quality requirements evolve with system

## Common Pitfalls

- **Too Vague**: "System should be fast" → Specify exact targets
- **Unmeasurable**: "User-friendly" → Define specific usability metrics
- **Unrealistic**: "100% uptime" → Be realistic about constraints
- **Too Many Critical**: If everything is critical, nothing is
- **No Verification**: How will you prove it meets requirements?

## Integration with Other Skills

- **arc42-docs**: Quality requirements feed into Section 10
- **adr-management**: Quality requirements drive architecture decisions
- **scenari-specs**: Quality scenarios can be expressed as Gherkin scenarios

## Related Standards

- **ISO/IEC 25010**: Software quality model
- **ISO/IEC 25040**: Evaluation process
- **ISO/IEC 25041**: Evaluation guide

#quality #iso25010 #requirements #non-functional
