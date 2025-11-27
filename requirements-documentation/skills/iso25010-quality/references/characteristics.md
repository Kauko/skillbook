# ISO 25010 Quality Characteristics Reference

This document provides detailed definitions of all quality characteristics and sub-characteristics from the ISO/IEC 25010 standard. Use this as a reference when identifying and documenting quality requirements.

## Overview

ISO/IEC 25010 defines a comprehensive quality model with two main dimensions:

1. **Product Quality** (8 characteristics) - Evaluates the inherent properties of the software itself
2. **Quality in Use** (5 characteristics) - Assesses real-world effectiveness and user satisfaction

## Product Quality Model

The product quality model comprises 8 main characteristics and approximately 30 sub-characteristics that relate to the static and dynamic properties of software products.

---

## 1. Functional Suitability

**Definition**: The degree to which a product or system provides functions that meet stated and implied needs when used under specified conditions.

**When This Matters**: All systems require functional correctness, but this becomes critical when:
- Calculations must be precise (financial, scientific, engineering systems)
- Missing functionality would prevent users from completing essential tasks
- Functions must be appropriate for specific user goals and workflows

### Sub-characteristics

#### 1.1 Functional Completeness

**Definition**: The degree to which the set of functions covers all the specified tasks and user objectives.

**Key Questions**:
- Does the system do everything users need?
- Are there gaps in functionality that force workarounds?
- Are all specified requirements implemented?

**Example Requirements**:
- "System must support all CRUD operations for user accounts"
- "All features listed in requirements specification must be implemented"
- "Users must be able to export data in CSV, JSON, and XML formats"

#### 1.2 Functional Correctness

**Definition**: The degree to which a product or system provides the correct results with the needed degree of precision.

**Key Questions**:
- Do calculations produce accurate results?
- What tolerance for errors is acceptable?
- How is data validation performed?

**Example Requirements**:
- "Financial calculations must be accurate to 2 decimal places"
- "Tax calculations must match regulatory requirements exactly"
- "All input data must be validated against defined business rules"

#### 1.3 Functional Appropriateness

**Definition**: The degree to which the functions facilitate the accomplishment of specified tasks and objectives.

**Key Questions**:
- Are there simpler ways to achieve user goals?
- Do functions match user workflows?
- Are features appropriate for the intended use case?

**Example Requirements**:
- "Users must complete checkout in 3 steps or fewer"
- "Common tasks must be accessible within 2 clicks from home screen"
- "Bulk operations must be available for repetitive tasks"

---

## 2. Performance Efficiency

**Definition**: The performance relative to the amount of resources used under stated conditions. Resources can include processing time, memory, disk space, network bandwidth, and energy.

**When This Matters**: Performance becomes critical when:
- User satisfaction depends on response times
- System must handle high load or concurrent users
- Resource costs (cloud, hardware) are significant
- Real-time constraints exist

### Sub-characteristics

#### 2.1 Time Behavior

**Definition**: The degree to which the response and processing times and throughput rates of a product or system meet requirements when performing its functions.

**Key Questions**:
- How fast must API responses be?
- What are acceptable UI interaction delays?
- Are there real-time constraints?
- What are peak load requirements?

**Example Requirements**:
- "API p95 response time < 200ms under normal load"
- "Page load time p95 < 2 seconds"
- "Database queries p95 < 50ms"
- "Real-time notifications delivered within 100ms"

#### 2.2 Resource Utilization

**Definition**: The degree to which the amounts and types of resources used by a product or system meet requirements when performing its functions.

**Key Questions**:
- What are memory consumption limits?
- How efficiently does it use CPU?
- What are storage requirements?
- What network bandwidth is needed?

**Example Requirements**:
- "Memory usage < 4GB per application instance"
- "CPU usage < 70% under normal load"
- "Database storage growth < 1GB per 10,000 users"
- "Network bandwidth < 100Mbps for 1000 concurrent users"

#### 2.3 Capacity

**Definition**: The degree to which the maximum limits of a product or system parameter meet requirements.

**Key Questions**:
- How many concurrent users must be supported?
- What transaction volume is required?
- How much data will the system store?
- What are growth projections?

**Example Requirements**:
- "System must handle 10,000 concurrent users"
- "Support up to 1000 transactions per second"
- "Database must scale to 100 million records"
- "Message queue must handle 50,000 messages per minute"

---

## 3. Compatibility

**Definition**: The degree to which a product, system or component can exchange information with other products, systems or components, and/or perform its required functions while sharing the same hardware or software environment.

**When This Matters**: Compatibility is critical when:
- System must integrate with existing infrastructure
- Multiple systems share the same environment
- Data exchange with external systems is required
- API backward compatibility is needed

### Sub-characteristics

#### 3.1 Co-existence

**Definition**: The degree to which a product can perform its required functions efficiently while sharing a common environment and resources with other products, without detrimental impact on any other product.

**Key Questions**:
- What other software runs in the same environment?
- Are there resource conflicts?
- Are there port or process conflicts?
- Can it coexist with legacy systems?

**Example Requirements**:
- "Must coexist with existing monitoring agents without conflicts"
- "Port usage must be configurable to avoid conflicts"
- "Must not consume more than 50% of available system resources"

#### 3.2 Interoperability

**Definition**: The degree to which two or more systems, products or components can exchange information and mutually use the information that has been exchanged.

**Key Questions**:
- What systems must it integrate with?
- What data formats must be supported?
- What protocols are required?
- What API compatibility is needed?

**Example Requirements**:
- "Must integrate with existing LDAP/Active Directory authentication"
- "Must support OAuth 2.0 for third-party integrations"
- "API must be backward compatible for 2 major versions"
- "Must export data in CSV, JSON, and XML formats"
- "Must support REST and GraphQL APIs"

---

## 4. Usability (Interaction Capability)

**Definition**: The degree to which a product or system can be used by specified users to achieve specified goals with effectiveness, efficiency, and satisfaction in a specified context of use.

Note: In ISO 25010:2023, this is called "Interaction Capability" with updated sub-characteristics.

**When This Matters**: Usability is critical when:
- Users are not technically sophisticated
- Training time must be minimized
- User satisfaction directly impacts adoption
- Accessibility compliance is required

### Sub-characteristics

#### 4.1 Appropriateness Recognizability

**Definition**: The degree to which users can recognize whether a product or system is appropriate for their needs.

**Key Questions**:
- Is the purpose of the system clear?
- Can users quickly understand what it does?
- Is value proposition immediately apparent?

**Example Requirements**:
- "Landing page must communicate core value within 5 seconds"
- "Feature names must clearly indicate their purpose"
- "Dashboard must show relevant information first"

#### 4.2 Learnability

**Definition**: The degree to which a product or system can be used by specified users to achieve specified goals of learning to use the product or system with effectiveness, efficiency, freedom from risk, and satisfaction.

**Key Questions**:
- How quickly can users learn the system?
- Is training required?
- How extensive is documentation needed?
- Can users learn by exploration?

**Example Requirements**:
- "New users must complete first successful task within 5 minutes"
- "80% of users complete onboarding without assistance"
- "Core features learnable without reading documentation"
- "Contextual help available for all complex features"

#### 4.3 Operability

**Definition**: The degree to which a product or system has attributes that make it easy to operate and control.

**Key Questions**:
- How many steps to complete common tasks?
- Is error recovery easy?
- Are undo/redo capabilities available?
- Can workflows be customized?

**Example Requirements**:
- "Common tasks completable in 3 steps or fewer"
- "Undo/redo available for all data modifications"
- "Keyboard shortcuts for all frequent operations"
- "Batch operations for repetitive tasks"

#### 4.4 User Error Protection

**Definition**: The degree to which a system protects users against making errors.

**Key Questions**:
- What input validation is performed?
- Are destructive actions confirmed?
- Are error messages helpful?
- Can errors be prevented proactively?

**Example Requirements**:
- "All destructive actions require confirmation"
- "Input validation provides specific error messages"
- "Error messages suggest corrective actions"
- "Auto-save prevents data loss"
- "Dangerous operations require typing confirmation phrase"

#### 4.5 User Interface Aesthetics

**Definition**: The degree to which a user interface enables pleasing and satisfying interaction for the user.

**Key Questions**:
- Does UI match brand guidelines?
- Is the interface professionally designed?
- Is visual hierarchy clear?
- Is design consistent throughout?

**Example Requirements**:
- "UI must follow company design system"
- "Visual design must be consistent across all pages"
- "Information hierarchy must be clear through typography and spacing"
- "Color palette must match brand guidelines"

#### 4.6 Accessibility

**Definition**: The degree to which a product or system can be used by people with the widest range of characteristics and capabilities to achieve specified goals in a specified context of use.

**Key Questions**:
- What WCAG compliance level is required?
- Is screen reader support needed?
- Is keyboard navigation complete?
- Are color contrast requirements met?

**Example Requirements**:
- "Must meet WCAG 2.1 Level AA compliance"
- "All features accessible via keyboard navigation"
- "Screen reader compatible with ARIA labels"
- "Color contrast ratios ≥ 4.5:1 for normal text"
- "Text resizable to 200% without loss of functionality"
- "Captions for all video content"

---

## 5. Reliability

**Definition**: The degree to which a system, product or component performs specified functions under specified conditions for a specified period of time.

**When This Matters**: Reliability is critical when:
- Downtime has significant business impact
- System handles critical business processes
- 24/7 availability is expected
- Data loss is unacceptable

### Sub-characteristics

#### 5.1 Maturity

**Definition**: The degree to which a system meets needs for reliability under normal operation.

**Key Questions**:
- What is acceptable bug rate?
- What is mean time between failures (MTBF)?
- What defect density targets exist?
- How stable is the system?

**Example Requirements**:
- "MTBF > 720 hours (30 days) in production"
- "Defect density < 1 per 1000 lines of code"
- "Zero critical bugs in production at release"
- "Bug escape rate < 5% from QA to production"

#### 5.2 Availability

**Definition**: The degree to which a system, product or component is operational and accessible when required for use.

**Key Questions**:
- What uptime percentage is required?
- How much downtime is acceptable?
- When are maintenance windows allowed?
- What hours must system be available?

**Example Requirements**:
- "System uptime ≥ 99.9% (< 43 minutes downtime/month)"
- "System uptime ≥ 99.99% during business hours"
- "Planned maintenance only during designated windows"
- "No downtime during peak business periods"

#### 5.3 Fault Tolerance

**Definition**: The degree to which a system, product or component operates as intended despite the presence of hardware or software faults.

**Key Questions**:
- What single points of failure exist?
- Can system degrade gracefully?
- What redundancy is required?
- How are failures isolated?

**Example Requirements**:
- "No single points of failure in critical path"
- "System continues operation with 1 of N nodes failed"
- "Circuit breakers prevent cascading failures"
- "Graceful degradation when external services fail"
- "Database replication with automatic failover"

#### 5.4 Recoverability

**Definition**: The degree to which, in the event of an interruption or failure, a product or system can recover the data directly affected and re-establish the desired state of the system.

**Key Questions**:
- What is Recovery Time Objective (RTO)?
- What is Recovery Point Objective (RPO)?
- Is automatic recovery possible?
- How is data integrity ensured?

**Example Requirements**:
- "RTO < 1 hour for complete system recovery"
- "RPO < 5 minutes (maximum data loss)"
- "Automatic recovery from transient failures"
- "Mean time to recovery (MTTR) < 15 minutes"
- "Automated backups every 15 minutes"
- "Point-in-time recovery capability"

---

## 6. Security

**Definition**: The degree to which a product or system protects information and data so that persons or other products or systems have the degree of data access appropriate to their types and levels of authorization.

**When This Matters**: Security is critical when:
- Sensitive or personal data is handled
- Regulatory compliance is required (GDPR, HIPAA, PCI-DSS)
- System is internet-facing
- Security breach would have severe consequences

### Sub-characteristics

#### 6.1 Confidentiality

**Definition**: The degree to which a product or system ensures that data are accessible only to those authorized to have access.

**Key Questions**:
- What data classification scheme applies?
- What encryption is required?
- How granular is access control?
- How is sensitive data protected?

**Example Requirements**:
- "All sensitive data encrypted at rest with AES-256"
- "All PII encrypted in database"
- "Role-based access control (RBAC) for all resources"
- "Field-level access control for sensitive data"
- "Data encryption keys rotated quarterly"

#### 6.2 Integrity

**Definition**: The degree to which a system, product or component prevents unauthorized access to, or modification of, computer programs or data.

**Key Questions**:
- How is data validation performed?
- Are checksums or signatures used?
- How are audit trails maintained?
- How is tampering detected?

**Example Requirements**:
- "All API payloads validated against schema"
- "Database transactions use checksums"
- "All data modifications logged with before/after values"
- "File integrity monitoring for critical system files"
- "Digital signatures for all software releases"

#### 6.3 Non-repudiation

**Definition**: The degree to which actions or events can be proven to have taken place, so that the events or actions cannot be repudiated later.

**Key Questions**:
- What audit logging is required?
- Are digital signatures needed?
- How are transactions logged?
- What proof of action is needed?

**Example Requirements**:
- "All user actions logged with timestamp and user ID"
- "Transaction logs immutable and cryptographically signed"
- "Audit logs retained for 7 years"
- "Email notifications signed with DKIM"

#### 6.4 Accountability

**Definition**: The degree to which actions of an entity can be traced uniquely to the entity.

**Key Questions**:
- How are users identified?
- What actions are logged?
- Are compliance requirements met?
- How is forensic investigation supported?

**Example Requirements**:
- "All user actions logged with user ID and timestamp"
- "All API calls include authenticated user context"
- "Admin actions logged separately with enhanced detail"
- "Logs support forensic investigation"
- "Cannot delete or modify audit logs"

#### 6.5 Authenticity

**Definition**: The degree to which the identity of a subject or resource can be proved to be the one claimed.

**Key Questions**:
- What authentication methods are used?
- Is multi-factor authentication required?
- How are certificates validated?
- How is identity verified?

**Example Requirements**:
- "Multi-factor authentication (MFA) for privileged accounts"
- "MFA for all users accessing sensitive data"
- "Certificate-based authentication for service accounts"
- "OAuth 2.0/OIDC for third-party authentication"
- "Biometric authentication option for mobile apps"
- "Failed login attempts locked after 5 attempts"
- "Session timeout after 30 minutes of inactivity"

---

## 7. Maintainability

**Definition**: The degree of effectiveness and efficiency with which a product or system can be modified to improve it, correct it, or adapt it to changes in environment and requirements.

**When This Matters**: Maintainability is critical when:
- System will be actively developed for years
- Multiple teams contribute to codebase
- Rapid feature development is required
- Technical debt must be minimized

### Sub-characteristics

#### 7.1 Modularity

**Definition**: The degree to which a system or computer program is composed of discrete components such that a change to one component has minimal impact on other components.

**Key Questions**:
- Are component boundaries clear?
- Is coupling low and cohesion high?
- Can components be developed independently?
- Are dependencies explicit?

**Example Requirements**:
- "Components must have well-defined interfaces"
- "No circular dependencies between modules"
- "Components can be developed and tested independently"
- "Shared code extracted to libraries"
- "Microservices communicate only via APIs"

#### 7.2 Reusability

**Definition**: The degree to which an asset can be used in more than one system, or in building other assets.

**Key Questions**:
- Can components be reused?
- Is code generic enough?
- Are libraries/frameworks used?
- Is API-first design followed?

**Example Requirements**:
- "Common components published as shared libraries"
- "UI components reusable across applications"
- "Business logic separated from presentation"
- "API designed for multiple clients"

#### 7.3 Analyzability

**Definition**: The degree of effectiveness and efficiency with which it is possible to assess the impact on a product or system of an intended change, to diagnose deficiencies or causes of failures, or to identify parts to be modified.

**Key Questions**:
- Is logging adequate?
- Are monitoring capabilities sufficient?
- Is debug information available?
- Can issues be diagnosed easily?

**Example Requirements**:
- "Structured logging with correlation IDs"
- "All errors logged with stack traces"
- "Distributed tracing for all requests"
- "Metrics collected for all critical paths"
- "Log aggregation and search capability"

#### 7.4 Modifiability

**Definition**: The degree to which a product or system can be effectively and efficiently modified without introducing defects or degrading existing product quality.

**Key Questions**:
- How long to add typical features?
- What is impact of changes?
- Can changes be made via configuration?
- How easy is it to refactor?

**Example Requirements**:
- "New simple feature added in ≤ 2 days"
- "Configuration via environment variables (12-factor app)"
- "Feature flags for gradual rollout"
- "Code changes don't require database migrations"

#### 7.5 Testability

**Definition**: The degree of effectiveness and efficiency with which test criteria can be established for a system, product or component and tests can be performed to determine whether those criteria have been met.

**Key Questions**:
- What test coverage is required?
- How automated is testing?
- Can components be tested in isolation?
- Are tests maintainable?

**Example Requirements**:
- "Unit test coverage ≥ 80% for business logic"
- "All public APIs have integration tests"
- "Critical paths have end-to-end tests"
- "All new features include automated tests"
- "Tests run in < 10 minutes in CI/CD"
- "Components can be tested without external dependencies"

---

## 8. Portability (Flexibility)

**Definition**: The degree of effectiveness and efficiency with which a system, product or component can be transferred from one hardware, software or other operational or usage environment to another.

Note: In ISO 25010:2023, this is called "Flexibility" with updated definitions.

**When This Matters**: Portability is critical when:
- System must run in multiple environments
- Cloud-agnostic deployment is required
- Platform independence is needed
- Migration from legacy systems is planned

### Sub-characteristics

#### 8.1 Adaptability

**Definition**: The degree to which a product or system can effectively and efficiently be adapted for different or evolving hardware, software or other operational or usage environments.

**Key Questions**:
- What platforms must be supported?
- How are environment variations handled?
- How flexible is configuration?
- Can it adapt to different infrastructures?

**Example Requirements**:
- "Must run on Linux, macOS, and Windows"
- "Must run on AWS, Azure, and GCP with minimal changes"
- "Environment-specific configuration via environment variables"
- "Support for different database backends (PostgreSQL, MySQL)"

#### 8.2 Installability

**Definition**: The degree of effectiveness and efficiency with which a product or system can be successfully installed and/or uninstalled in a specified environment.

**Key Questions**:
- How long does installation take?
- How complex is installation?
- How automated is deployment?
- What installation options exist?

**Example Requirements**:
- "Installation automated via single command"
- "Deployment via Docker containers"
- "Zero-downtime deployments"
- "Installation time < 5 minutes"
- "Self-contained installation with no external dependencies"

#### 8.3 Replaceability

**Definition**: The degree to which a product can replace another specified software product for the same purpose in the same environment.

**Key Questions**:
- Can it replace existing systems?
- What data migration is required?
- Is API compatibility maintained?
- Is feature parity achieved?

**Example Requirements**:
- "Support import from competitor's data format"
- "API compatible with system being replaced"
- "Migration scripts for legacy data"
- "Feature parity with replaced system"

---

## Quality in Use Model

The Quality in Use model comprises 5 characteristics that relate to the outcome of interaction when a product is used in a particular context of use.

### 1. Effectiveness

**Definition**: The accuracy and completeness with which users achieve specified goals.

**Key Questions**:
- Do users accomplish their goals?
- Are results accurate?
- Are tasks completed successfully?

**Example Requirements**:
- "95% of users complete checkout successfully"
- "Search results relevant to query in 90% of cases"
- "Task completion rate > 85%"

### 2. Efficiency

**Definition**: The resources expended in relation to the accuracy and completeness with which users achieve goals.

**Key Questions**:
- How long do tasks take?
- How much effort is required?
- Are workflows optimized?

**Example Requirements**:
- "Average task completion time < 2 minutes"
- "Users complete purchase in average of 3 clicks"
- "Expert users 50% faster than novices"

### 3. Satisfaction

**Definition**: The degree to which user needs are satisfied when a product or system is used in a specified context of use.

Sub-characteristics include:
- **Usefulness**: Satisfaction with pragmatic goal achievement
- **Trust**: User confidence in intended behavior
- **Pleasure**: Fulfillment of personal needs
- **Comfort**: Physical comfort during use

**Example Requirements**:
- "Net Promoter Score (NPS) > 30"
- "User satisfaction rating > 4.0/5.0"
- "Task frustration rating < 2.0/5.0"
- "95% of users trust system with sensitive data"

### 4. Freedom from Risk

**Definition**: The degree to which a product or system mitigates the potential risk to economic status, human life, health, or the environment.

Sub-characteristics include:
- **Economic risk mitigation**: Protection of financial status and assets
- **Health and safety risk mitigation**: Protection of people
- **Environmental risk mitigation**: Protection of property and environment

**Example Requirements**:
- "No financial loss due to system errors"
- "Safety-critical operations require double confirmation"
- "System cannot be used in way that violates regulations"

### 5. Context Coverage

**Definition**: The degree to which a product or system can be used with effectiveness, efficiency, freedom from risk, and satisfaction in both specified contexts of use and in contexts beyond those initially explicitly identified.

Sub-characteristics include:
- **Context completeness**: Maintaining quality across all use contexts
- **Flexibility**: Effectiveness in unexpected usage scenarios

**Example Requirements**:
- "System usable on mobile, tablet, and desktop"
- "Functions correctly in poor network conditions"
- "Handles unexpected inputs gracefully"
- "Accessible in different locales and languages"

---

## Using This Reference

### When Defining Requirements

1. Review all 8 product quality characteristics
2. Identify which are CRITICAL, HIGH, MEDIUM, or LOW priority for your system
3. For each important characteristic, review sub-characteristics
4. Define specific, measurable requirements using sub-characteristics as guide
5. Create quality scenarios for important requirements

### Quality Characteristic Selection

Not all characteristics are equally important for every system. Prioritize based on:

- **System Type**: Web app, mobile, embedded, API, batch processing
- **Users**: Technical vs. non-technical, internal vs. external
- **Business Context**: Revenue impact, compliance, brand reputation
- **Risk Profile**: What failures would be most damaging

### Trade-offs

Quality characteristics often conflict. Common trade-offs:

- **Security vs. Usability**: More security controls can reduce ease of use
- **Performance vs. Maintainability**: Optimized code may be harder to maintain
- **Reliability vs. Cost**: Higher reliability requires more infrastructure
- **Flexibility vs. Performance**: Generic solutions may be slower

Document these trade-offs in your architecture decisions.

---

## References

- ISO/IEC 25010:2011 - Systems and software engineering — Software product Quality Requirements and Evaluation (SQuaRE) — System and software quality models
- ISO/IEC 25010:2023 - Updated version with revised characteristics
- [ISO 25000 Portal](https://iso25000.com/index.php/en/iso-25000-standards/iso-25010) - Free overview and guidance
- [arc42 Quality Model](https://quality.arc42.org/standards/iso-25010) - Practical quality requirements
- [ZetCode ISO 25010 Tutorial](https://zetcode.com/terms-testing/iso-25010/) - Comprehensive tutorial

---

*This reference is based on freely available ISO 25010 documentation and educational resources. For the complete official standard, consult ISO/IEC 25010:2023.*
