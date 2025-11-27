# arc42 Sections - Detailed Guidance

This document provides comprehensive guidance for all 12 arc42 sections, drawing from official arc42 documentation.

## 1. Introduction and Goals (24 tips)

### Purpose
Documents requirements overview, stakeholder expectations, and the top 3-5 quality goals that shape the architecture.

### What to Include

**Requirements Overview**
- Brief description of the system and its main purpose
- Key features and capabilities
- What problem does the system solve?
- Who are the primary users?

**Quality Goals**
- Top 3-5 quality attributes that shape architecture
- Prioritized list (quality goals drive architectural decisions)
- Motivation for each quality goal
- These should be concrete, measurable, and testable

**Stakeholders**
- Table of key stakeholders
- Their roles and responsibilities
- Contact information
- Expectations from the architecture and system

### Best Practices

1. **Keep it concise** - This is an overview, not detailed requirements
2. **Focus on WHY** - Why does the system exist? Why are quality goals important?
3. **Prioritize ruthlessly** - Only top 3-5 quality goals (more dilutes focus)
4. **Make goals measurable** - "Fast" is vague; "p95 response time < 200ms" is concrete
5. **Know your audience** - Different stakeholders care about different aspects

### Common Pitfalls

- Listing too many quality goals (makes prioritization meaningless)
- Vague, non-measurable quality statements
- Missing stakeholder expectations
- Confusing features with requirements

### Tips from arc42

- Use quality scenarios from ISO 25010 to make goals concrete
- Link stakeholder expectations to quality goals
- Update when requirements or stakeholders change
- Keep it to 1-2 pages maximum

---

## 2. Constraints (5 tips)

### Purpose
Documents design limitations that constrain teams in design and implementation decisions. Constraints are "given" - you cannot change them.

### What to Include

**Technical Constraints**
- Hardware limitations
- Operating systems and platforms
- Programming languages mandated
- Required technologies or frameworks
- Database systems
- Integration requirements

**Organizational Constraints**
- Team size and structure
- Budget limitations
- Time constraints and deadlines
- Development process requirements (Scrum, waterfall, etc.)
- Testing and QA requirements
- Release schedule

**Political/Legal Constraints**
- Regulatory compliance (GDPR, HIPAA, etc.)
- Open source license requirements
- Vendor lock-in or requirements
- Security and privacy regulations
- Data sovereignty requirements
- Accessibility standards

**Conventions**
- Coding standards and style guides
- Documentation requirements
- Git workflow and branching strategy
- Naming conventions
- Architecture patterns to follow

### Best Practices

1. **Be specific** - Don't just say "limited budget," quantify if possible
2. **Include rationale** - Why does each constraint exist?
3. **Separate categories** - Technical vs. organizational vs. political
4. **Document workarounds** - How do you work within constraints?
5. **Keep updated** - Constraints can change over time

### Common Pitfalls

- Confusing constraints with design decisions (constraints are unchangeable)
- Listing desires as constraints
- Missing hidden organizational constraints
- Not documenting impact of constraints on architecture

### Tips from arc42

- Constraints limit your design freedom
- They are "given" - you must work within them
- Document even obvious constraints (they may not be obvious to all stakeholders)
- Reference specific standards, laws, or policies
- Update when constraints change or are lifted

---

## 3. Context and Scope (19 tips)

### Purpose
Delimits your system from external communication partners, specifying system boundaries and external interfaces from business and technical perspectives.

### What to Include

**Business Context**
- Diagram showing external entities and their relationships
- Users, administrators, external systems
- Business-level interfaces and data flows
- What information is exchanged with each entity?
- Why does the system need these interactions?

**Technical Context**
- Technical protocols and interfaces
- APIs (REST, GraphQL, gRPC, etc.)
- Message formats (JSON, XML, Protocol Buffers, etc.)
- Network protocols (HTTP, WebSocket, MQTT, etc.)
- Authentication mechanisms
- Data formats exchanged

**System Boundary**
- What is inside vs. outside the system?
- What are you responsible for?
- What is delegated to external systems?
- Clear boundary between system and environment

**External Entities**
- Description of each external entity
- Interface specifications
- Relationship to your system
- Technology used for communication

### Best Practices

1. **Use diagrams** - Context diagrams are visual and easy to understand
2. **Separate business and technical views** - Different audiences need different perspectives
3. **Be consistent** - Use same entity names throughout documentation
4. **Define clearly** - What crosses the boundary? What doesn't?
5. **Show data flow** - What information flows where?

### Common Pitfalls

- Unclear system boundaries
- Missing external dependencies
- Confusing internal and external components
- Too much technical detail in business context
- Not documenting "why" for external dependencies

### Tips from arc42

- Context shows your system as a black box
- Business context is for non-technical stakeholders
- Technical context is for developers and integrators
- Use UML context diagrams, C4 diagrams, or simple boxes
- Update when external dependencies change

---

## 4. Solution Strategy (6 tips)

### Purpose
Summarizes fundamental decisions and solution approaches that shape the architecture. Describes HOW you solve the problem within given constraints.

### What to Include

**Technology Decisions**
- Programming languages and frameworks
- Database choices
- Infrastructure decisions
- Key libraries and tools
- Rationale for each choice
- Links to ADRs

**Architectural Approach**
- Overall architecture style (microservices, monolith, event-driven, etc.)
- Key architectural patterns used
- Why this approach was chosen
- Alternatives considered
- Trade-offs made

**Quality Goal Achievement**
- How does the solution achieve each quality goal?
- Specific strategies and patterns
- Technology choices that support quality goals
- Validation approach

**High-Level Decomposition**
- Overview of main components/modules
- How they relate to each other
- Key responsibilities
- Communication patterns

### Best Practices

1. **Link to quality goals** - Show how solution achieves goals from Section 1
2. **Explain trade-offs** - What did you sacrifice? Why?
3. **Reference constraints** - Show how constraints influenced decisions
4. **Link to ADRs** - Point to detailed decision records
5. **Keep high-level** - Details come in later sections

### Common Pitfalls

- Too much detail (save for Building Block View)
- Not connecting to quality goals
- Missing rationale for decisions
- Not documenting alternatives considered

### Tips from arc42

- This is your architectural "executive summary"
- Should be understandable by non-technical stakeholders
- Typically 1-2 pages maximum
- Update when fundamental decisions change
- Link quality goals (Section 1) to quality requirements (Section 10)

---

## 5. Building Block View (28 tips)

### Purpose
Provides static decomposition of the system through hierarchical white-box/black-box representations showing structural organization.

### What to Include

**Level 1: System Overview**
- Highest-level black-box view of entire system
- Main components and their responsibilities
- External interfaces
- Key relationships

**Level 2: Component Decomposition**
- White-box view of Level 1 components
- Internal structure of each major component
- Interfaces between components
- Technology choices for each

**Level 3+: Detailed Views**
- Deeper decomposition as needed
- Package/module structure
- Class/function organization
- Design patterns applied

**For Each Component**
- Purpose and responsibility
- Interfaces (provided and required)
- Dependencies
- Implementation notes
- Design decisions

### Best Practices

1. **Use hierarchy** - Don't try to show everything at once
2. **Black-box then white-box** - Show external view before internal
3. **Consistent notation** - Use UML, C4, or other standard notation
4. **Document interfaces** - What does each component expose?
5. **Show dependencies** - What does each component need?
6. **Apply patterns** - Document design patterns used

### Common Pitfalls

- Too much detail at once (overwhelming)
- Inconsistent level of abstraction
- Missing interface specifications
- Not documenting component responsibilities
- Confusing runtime and build-time structure

### Tips from arc42

- Start with coarse-grained view (Level 1)
- Refine to finer detail only where needed
- Use consistent notation throughout
- Each level should be on one diagram
- Document important interfaces
- Show how components map to source code
- Update as architecture evolves

---

## 6. Runtime View (11 tips)

### Purpose
Illustrates behavior of building blocks through runtime scenarios showing dynamic interactions and data flows.

### What to Include

**Key Scenarios**
- Most important use cases
- Critical business processes
- Error/exception handling
- State changes and workflows
- Inter-component communication

**For Each Scenario**
- Goal/purpose
- Actors involved
- Preconditions
- Step-by-step flow
- Postconditions
- Alternative/error paths
- Sequence diagrams

**Important Behaviors**
- Startup and shutdown sequences
- Transaction handling
- Concurrency patterns
- Error propagation
- Asynchronous operations

### Best Practices

1. **Select key scenarios** - Don't document every use case
2. **Use sequence diagrams** - Show temporal ordering clearly
3. **Show error handling** - What happens when things go wrong?
4. **Number steps** - Make flow easy to follow
5. **Link to building blocks** - Show which components are involved
6. **Show data flow** - What information is passed?

### Common Pitfalls

- Too many scenarios (focus on important ones)
- Missing error handling
- Too much detail (implementation-level)
- Not showing asynchronous behavior
- Ignoring concurrency concerns

### Tips from arc42

- Focus on architecturally relevant scenarios
- Show interesting or complex interactions
- Document edge cases and error handling
- Use standard UML sequence diagrams
- Can use activity diagrams for complex flows
- Update when behavior changes
- Link to specifications and tests

---

## 7. Deployment View (10 tips)

### Purpose
Details hardware infrastructure, technical environments, deployment topology, and mapping of software to infrastructure.

### What to Include

**Infrastructure Overview**
- Hardware (servers, devices, etc.)
- Network topology
- Deployment environments (dev, test, prod)
- Cloud resources (if applicable)
- Third-party hosting

**Component Mapping**
- Which software runs on which hardware?
- How are components distributed?
- Redundancy and scaling
- Network communication paths
- Storage locations

**Technical Environment**
- Operating systems
- Runtime environments (JVM, .NET, containers, etc.)
- Required software dependencies
- Configuration management
- Monitoring and logging infrastructure

**Deployment Process**
- How is software deployed?
- CI/CD pipeline
- Blue-green, canary, rolling updates?
- Database migration strategy
- Rollback procedures

### Best Practices

1. **Use deployment diagrams** - Show physical/virtual infrastructure
2. **Multiple environments** - Document dev, test, prod differences
3. **Show scaling** - How does system scale horizontally/vertically?
4. **Document dependencies** - What infrastructure is required?
5. **Include networking** - Firewalls, load balancers, DNS, etc.
6. **Show data storage** - Where is data persisted?

### Common Pitfalls

- Missing cloud infrastructure details
- Not documenting different environments
- Ignoring networking complexity
- Missing disaster recovery setup
- Not showing monitoring/logging infrastructure

### Tips from arc42

- Use UML deployment diagrams or similar
- Show physical or virtual nodes
- Document node types and responsibilities
- Include network connections
- Show deployment process
- Document scaling strategies
- Update when infrastructure changes

---

## 8. Crosscutting Concepts (11 tips)

### Purpose
Addresses overall, principal regulations and solution approaches spanning multiple building blocks. Documents recurring patterns and common solutions.

### What to Include

**Domain Concepts**
- Domain model overview
- Core business entities
- Business rules
- Domain-driven design patterns

**Technical Concepts**
- Error handling strategy
- Exception handling patterns
- Transaction management
- Concurrency control
- Caching strategy

**Architecture Patterns**
- Dependency injection
- Repository pattern
- Service layer patterns
- Communication patterns
- Integration patterns

**Development Concepts**
- Testing strategy (unit, integration, e2e)
- Logging and monitoring
- Configuration management
- Internationalization
- Security patterns

**Operational Concepts**
- Deployment strategy
- Scaling approach
- Backup and recovery
- Monitoring and alerting
- Performance optimization

**User Experience**
- UI patterns and standards
- Accessibility requirements
- Responsive design
- Error messaging
- User feedback

### Best Practices

1. **Avoid repetition** - Document once, reference everywhere
2. **Show examples** - Concrete code snippets help
3. **Link to implementation** - Where is this pattern used?
4. **Document rationale** - Why this pattern?
5. **Keep consistent** - All code should follow these patterns
6. **Update regularly** - As patterns evolve

### Common Pitfalls

- Too abstract (need concrete examples)
- Listing patterns without explaining usage
- Not showing code examples
- Missing rationale for choices
- Not enforcing consistency

### Tips from arc42

- These are concepts used throughout the system
- Document to ensure consistency
- Include code examples
- Show how to apply patterns
- Link to specific implementations
- Update when patterns change
- Consider testing, security, operations, etc.

---

## 9. Architecture Decisions (10 tips)

### Purpose
Documents important, costly, critical, or risky decisions made during design with rationale, alternatives, and consequences.

### What to Include

**Decision Summary**
- Table of all architectural decisions
- Status (proposed, accepted, deprecated, superseded)
- Brief summary
- Link to detailed ADR

**Key Decisions**
- Technology choices
- Architecture style selection
- Pattern applications
- Infrastructure decisions
- Security approaches

**For Each Decision**
- Context and problem
- Decision made
- Alternatives considered
- Rationale and justification
- Consequences (positive and negative)
- Status

**Decision Timeline**
- When were decisions made?
- Evolution of architecture
- Superseded decisions

### Best Practices

1. **Use ADR format** - Architecture Decision Records are standard
2. **Document context** - What was the situation?
3. **Show alternatives** - What else was considered?
4. **Explain rationale** - Why this choice?
5. **List consequences** - Both positive and negative
6. **Keep updated** - Mark superseded decisions
7. **Link everywhere** - Reference from other sections

### Common Pitfalls

- Not documenting "why"
- Missing alternatives considered
- No consequences documented
- Not updating when decisions change
- Too much detail (save for ADR)

### Tips from arc42

- This is a summary; details go in ADRs
- Focus on architecturally significant decisions
- Update when decisions are revisited
- Mark superseded decisions clearly
- Show decision timeline
- Link to quality goals and constraints
- Document trade-offs made

---

## 10. Quality Requirements (8 tips)

### Purpose
Details quality scenarios and quality trees, extending quality goals from Section 1 with concrete, measurable requirements.

### What to Include

**Quality Tree**
- Hierarchical breakdown of quality attributes
- Based on ISO/IEC 25010 or similar
- Prioritization (high, medium, low)
- Relationships between attributes

**Quality Scenarios**
- Concrete scenarios for each quality goal
- Stimulus, environment, response, measure
- Success criteria
- Priority
- Validation approach

**Quality Attributes**
- Performance (response time, throughput, latency)
- Security (authentication, authorization, encryption)
- Reliability (availability, fault tolerance, recovery)
- Maintainability (testability, modularity, analyzability)
- Usability (learnability, operability, accessibility)
- Scalability (horizontal, vertical)
- Portability
- Compatibility

**Metrics and Measures**
- How will quality be measured?
- What tools will be used?
- Target values
- Acceptable ranges

### Best Practices

1. **Use ISO 25010** - Standard quality model
2. **Make scenarios concrete** - Specific and measurable
3. **Prioritize ruthlessly** - Not all qualities are equal
4. **Define metrics** - How will you measure?
5. **Link to Section 1** - Elaborate on quality goals
6. **Show trade-offs** - What was sacrificed?
7. **Make testable** - Can you verify?

### Common Pitfalls

- Vague quality statements
- No concrete scenarios
- Missing metrics
- Not prioritizing
- Unmeasurable requirements
- Ignoring trade-offs

### Tips from arc42

- Use quality scenarios (stimulus, response, measure)
- Base on quality goals from Section 1
- Make concrete and measurable
- Use standard quality models
- Show quality tree structure
- Document how to test/verify
- Update as requirements evolve

---

## 11. Risks and Technical Debt (6 tips)

### Purpose
Catalogs known problems, potential risks, and technical debt requiring management attention.

### What to Include

**Technical Risks**
- Identified technical risks
- Probability and impact assessment
- Mitigation strategies
- Monitoring approach
- Status (identified, mitigated, accepted)

**Business Risks**
- Market risks
- Dependency risks (external APIs, vendors)
- Scalability risks
- Security vulnerabilities

**Technical Debt**
- Known shortcuts taken
- Missing tests or documentation
- Code quality issues
- Architecture violations
- Planned refactorings

**For Each Risk/Debt Item**
- Description
- Probability (for risks) or impact (for debt)
- Consequences if not addressed
- Mitigation/remediation plan
- Effort to fix
- Why it exists
- Target date to address

**Risk Management**
- Risk assessment process
- Risk priority matrix
- Review frequency
- Escalation process

### Best Practices

1. **Be honest** - Document real issues
2. **Assess probability and impact** - Prioritize attention
3. **Plan mitigation** - What's the strategy?
4. **Track debt** - Don't let it grow unchecked
5. **Regular reviews** - Quarterly risk assessment
6. **Budget for debt paydown** - Allocate time (e.g., 20% of sprint)

### Common Pitfalls

- Hiding or ignoring problems
- No mitigation plans
- Not tracking technical debt
- Accepting all risks without analysis
- Missing security vulnerabilities

### Tips from arc42

- Be transparent about risks
- Technical debt is normal - manage it
- Use risk matrices (probability × impact)
- Document why debt was incurred
- Plan debt paydown
- Link to threat models
- Update regularly

---

## 12. Glossary (6 tips)

### Purpose
Defines critical domain, business, and technical terminology for consistent stakeholder communication.

### What to Include

**Domain Terms**
- Business concepts and entities
- Domain-specific language
- User roles
- Business processes
- Industry terminology

**Technical Terms**
- Architecture patterns
- Technology acronyms
- Technical concepts
- Component names
- Protocol names

**Abbreviations**
- Common acronyms (API, REST, SQL, etc.)
- Project-specific acronyms
- Organizational acronyms

**For Each Term**
- Clear, concise definition
- Context where used
- Relationships to other terms
- Aliases or synonyms

### Best Practices

1. **Keep alphabetized** - Easy to find terms
2. **Cross-reference** - Link related terms
3. **Be precise** - Clear, unambiguous definitions
4. **Include context** - When/where is term used?
5. **Avoid circular definitions** - Define clearly
6. **Update frequently** - Add terms as they emerge
7. **Include examples** - When helpful

### Common Pitfalls

- Missing obvious terms (not obvious to everyone)
- Circular or vague definitions
- Not updating as language evolves
- Too technical for business terms
- Missing common abbreviations

### Tips from arc42

- Define terms used throughout documentation
- Include both domain and technical terms
- Make searchable (alphabetical order)
- Link from other sections
- Use as reference for all documentation
- Keep updated as project evolves
- Consider different stakeholder perspectives

---

## General arc42 Best Practices

### Documentation Approaches

**Lean Documentation**
- Streamlined for agile environments
- Focus on essential information
- Iterative refinement
- Just enough documentation

**Thorough Documentation**
- Comprehensive for critical systems
- Detailed audit trails
- Formal verification
- Regulatory compliance

**Essential Documentation**
- Balance between lean and thorough
- Non-negotiable information
- Quality goals and decisions
- Flexible depth

### Key Principles

1. **Adapt to context** - Not all sections need same detail
2. **Iterate** - Start simple, refine over time
3. **Keep current** - Documentation must reflect reality
4. **Visual first** - Diagrams over text when possible
5. **Link everything** - Connect related concepts
6. **Know your audience** - Different sections for different stakeholders
7. **Make it searchable** - Use consistent terms and structure
8. **Automate where possible** - Generate diagrams from code/models
9. **Review regularly** - Quarterly updates minimum
10. **Team ownership** - Architecture docs are team responsibility

### Integration with Tools

- **Obsidian**: Use wikilinks to connect concepts
- **Overarch**: Generate diagrams from EDN models
- **ADRs**: Link architectural decisions
- **Threagile**: Reference threat models
- **Mermaid/PlantUML**: Embed diagrams in markdown
- **Tests**: Link to specifications and test suites
- **Code**: Reference implementations

### Common Anti-Patterns

- **Big design up front** - Don't try to complete perfectly initially
- **Out of date** - Docs drift from reality
- **Too detailed** - Can't see forest for trees
- **Too abstract** - No concrete examples
- **One size fits all** - Not adapting to context
- **Write once, forget** - Docs need maintenance
- **Solo documentation** - Should be team effort

### Success Factors

1. **Start early** - Begin documenting from project start
2. **Evolve continuously** - Update as architecture changes
3. **Make it accessible** - Easy to find and navigate
4. **Use standard notation** - UML, C4, etc.
5. **Get feedback** - Review with stakeholders
6. **Link to code** - Show implementation
7. **Celebrate documentation** - Make it part of done
8. **Tool support** - Use appropriate tools
9. **Templates** - Provide structure for consistency
10. **Training** - Help team understand arc42
