# Initialize Obsidian Vault

Use when starting a new project to set up the Obsidian vault documentation structure.

## Purpose

Creates a complete Obsidian vault structure for project documentation with folders for architecture, requirements, specifications, decisions, security, and arc42 documentation. Includes templates and README files with wikilinks for easy navigation.

## When to Use

- Starting a new project that needs comprehensive documentation
- Converting existing project to use structured documentation
- Setting up documentation for a team that will use Obsidian
- Before using other requirements-documentation skills (prerequisite)

## Directory Structure

The vault will be created with the following structure:

```
vault/
├── README.md                    # Main vault index with navigation
├── architecture/                # System architecture documentation
│   └── README.md
├── requirements/                # Requirements and constraints
│   └── README.md
├── specifications/              # Behavioral and formal specifications
│   └── README.md
├── decisions/                   # Architectural Decision Records (ADRs)
│   ├── README.md
│   └── index.md                # Auto-generated ADR index
├── security/                    # Security documentation and threat models
│   └── README.md
├── arc42/                       # arc42 architecture sections
│   └── README.md
└── templates/                   # Obsidian templates
    ├── adr-template.md
    ├── meeting-notes.md
    └── requirement.md
```

## Execution Steps

### 1. Check for Existing Vault

First, check if a vault directory already exists:

```bash
ls -la vault/ 2>/dev/null || echo "No vault directory found"
```

If vault exists, this skill is **idempotent** - it will only create missing directories and files, preserving existing content.

### 2. Create Directory Structure

Create all required directories:

```bash
mkdir -p vault/{architecture,requirements,specifications,decisions,security,arc42,templates}
```

### 3. Create Main Vault README

Create `vault/README.md`:

```markdown
# Project Documentation Vault

Welcome to the project documentation vault. This Obsidian vault contains all project documentation organized by category.

## Navigation

- [[architecture/README|Architecture]] - System architecture and design
- [[requirements/README|Requirements]] - Functional and non-functional requirements
- [[specifications/README|Specifications]] - Behavioral specifications and formal specs
- [[decisions/README|Decisions]] - Architectural Decision Records (ADRs)
- [[security/README|Security]] - Security documentation and threat models
- [[arc42/README|arc42]] - Structured architecture documentation

## Quick Links

- [[decisions/index|ADR Index]] - All architectural decisions
- [[requirements/quality-attributes|Quality Requirements]] - ISO 25010 quality attributes
- [[specifications/features|Feature Specifications]] - Gherkin scenarios

## Templates

Use these templates for consistent documentation:

- [[templates/adr-template|ADR Template]] - For architectural decisions
- [[templates/meeting-notes|Meeting Notes]] - For meetings and discussions
- [[templates/requirement|Requirement Template]] - For requirements

## Documentation Practices

1. **Link Everything**: Use wikilinks `[[page]]` to connect related concepts
2. **Use Tags**: Tag content with `#architecture`, `#security`, `#requirement`, etc.
3. **Keep Current**: Update documentation as the system evolves
4. **Review Regularly**: Schedule periodic documentation reviews
5. **Write for Humans**: Documentation is for communication, not just recording

## Tools

This vault is designed to work with:

- **Obsidian**: Primary editing environment
- **Obsidian MCP**: For Claude Code integration
- **arc42**: Architecture documentation template
- **ADR Tools**: For decision record management
- **Scenari**: For Gherkin feature specifications
```

### 4. Create Category READMEs

Create `vault/architecture/README.md`:

```markdown
# Architecture Documentation

System architecture, design decisions, and technical structure.

## Contents

- System context and scope
- Building blocks and components
- Runtime views and scenarios
- Deployment architecture
- Cross-cutting concerns

## Related

- [[../arc42/README|arc42 Documentation]] - Structured architecture docs
- [[../decisions/index|Architecture Decisions]] - ADRs affecting architecture
- [[../requirements/README|Requirements]] - Requirements driving architecture

## arc42 Sections

The architecture is documented using the arc42 template:

1. [[../arc42/01-introduction|Introduction and Goals]]
2. [[../arc42/02-constraints|Constraints]]
3. [[../arc42/03-context|Context and Scope]]
4. [[../arc42/04-solution|Solution Strategy]]
5. [[../arc42/05-building-blocks|Building Block View]]
6. [[../arc42/06-runtime|Runtime View]]
7. [[../arc42/07-deployment|Deployment View]]
8. [[../arc42/08-crosscutting|Cross-cutting Concepts]]
9. [[../arc42/09-decisions|Architecture Decisions]]
10. [[../arc42/10-quality|Quality Requirements]]
11. [[../arc42/11-risks|Risks and Technical Debt]]
12. [[../arc42/12-glossary|Glossary]]

#architecture
```

Create `vault/requirements/README.md`:

```markdown
# Requirements Documentation

Functional and non-functional requirements, constraints, and quality attributes.

## Contents

- [[quality-attributes|Quality Requirements]] - ISO 25010 quality attributes
- Functional requirements
- Non-functional requirements
- Constraints and assumptions
- Stakeholder needs

## Quality Attributes (ISO 25010)

Quality requirements are organized according to ISO/IEC 25010:

1. **Functional Suitability** - Completeness, correctness, appropriateness
2. **Performance Efficiency** - Time behavior, resource utilization, capacity
3. **Compatibility** - Co-existence, interoperability
4. **Usability** - Recognizability, learnability, operability, accessibility
5. **Reliability** - Maturity, availability, fault tolerance, recoverability
6. **Security** - Confidentiality, integrity, non-repudiation, accountability, authenticity
7. **Maintainability** - Modularity, reusability, analyzability, modifiability, testability
8. **Portability** - Adaptability, installability, replaceability

## Related

- [[../arc42/10-quality|arc42 Quality Requirements]]
- [[../specifications/README|Specifications]] - Behavioral specs for requirements
- [[../decisions/index|Decisions]] - How requirements influence decisions

## Traceability

Requirements should be traceable to:

- Specifications (Gherkin scenarios)
- Architecture decisions (ADRs)
- Test cases
- Implementation

#requirements
```

Create `vault/specifications/README.md`:

```markdown
# Specifications

Behavioral specifications using Gherkin (Given-When-Then) and formal specifications.

## Contents

- [[features|Feature Specifications]] - Gherkin scenarios embedded from test files
- Formal specifications (TLA+, Alloy, etc.)
- API specifications (OpenAPI, GraphQL schemas)
- Protocol specifications

## Gherkin Features

Behavioral specifications are written as Gherkin scenarios in `test/features/` and embedded here:

- **Given**: Preconditions and initial context
- **When**: Actions or events
- **Then**: Expected outcomes and postconditions

Use tags for traceability:
- `@requirement:REQ-001` - Link to requirement
- `@adr:0001` - Link to ADR
- `@quality:performance` - Link to quality attribute
- `@security` - Security-related scenarios

## Formal Specifications

For critical components, consider formal specifications:

- **TLA+**: Distributed systems, concurrency
- **Alloy**: System models, invariants
- **Coq/Isabelle**: Proof-carrying code

## Related

- [[../requirements/README|Requirements]] - What we're specifying
- [[../architecture/README|Architecture]] - How it's implemented
- [[../security/README|Security]] - Security specifications

#specifications
```

Create `vault/decisions/README.md`:

```markdown
# Architectural Decision Records

Architectural Decision Records (ADRs) document important architectural decisions and their context, rationale, and consequences.

## Current Decisions

See [[index|ADR Index]] for a complete list of all decisions.

## What is an ADR?

An ADR captures:

- **Context**: The situation requiring a decision
- **Decision**: The chosen solution
- **Status**: Proposed, accepted, superseded, deprecated
- **Consequences**: Positive and negative outcomes

## When to Write an ADR

Write an ADR when you make a decision that:

- Affects the system structure
- Is difficult or expensive to reverse
- Impacts multiple components or teams
- Sets a precedent for future decisions
- Has significant trade-offs

## ADR Numbering

ADRs are numbered sequentially: `0001-title.md`, `0002-title.md`, etc.

When an ADR is superseded, keep the old one and link to the new one.

## Related

- [[../arc42/09-decisions|arc42 Architecture Decisions]]
- [[../architecture/README|Architecture Documentation]]

## Template

Use [[../templates/adr-template|ADR Template]] to create new ADRs.

#adr #decisions
```

Create `vault/decisions/index.md`:

```markdown
# ADR Index

This index lists all Architectural Decision Records in chronological order.

## Active Decisions

(No decisions yet - use the ADR management skill to create your first ADR)

## Superseded Decisions

(None yet)

## Deprecated Decisions

(None yet)

---

*This index is automatically updated when creating new ADRs.*

#adr #index
```

Create `vault/security/README.md`:

```markdown
# Security Documentation

Security architecture, threat models, security controls, and security-related decisions.

## Contents

- Threat models (Threagile, STRIDE)
- Security architecture
- Security controls and mitigations
- Security requirements
- Security testing results
- Incident response procedures

## Threat Modeling

Use tools like Threagile to model threats and generate reports.

Key threat categories (STRIDE):

- **Spoofing**: Identity verification
- **Tampering**: Data integrity
- **Repudiation**: Non-repudiation
- **Information Disclosure**: Confidentiality
- **Denial of Service**: Availability
- **Elevation of Privilege**: Authorization

## Security Requirements

Security is documented in:

- [[../requirements/quality-attributes|Quality Requirements]] - ISO 25010 Security
- [[../specifications/README|Security Specifications]] - Security scenarios
- [[../decisions/index|Security Decisions]] - Security-related ADRs

## Related

- [[../arc42/08-crosscutting|Cross-cutting Security Concerns]]
- [[../architecture/README|Security Architecture]]

#security
```

Create `vault/arc42/README.md`:

```markdown
# arc42 Architecture Documentation

Structured architecture documentation following the arc42 template.

## What is arc42?

arc42 is a template for architecture communication and documentation. It provides 12 sections covering all aspects of software architecture.

## Sections

1. [[01-introduction|Introduction and Goals]] - Requirements overview, quality goals, stakeholders
2. [[02-constraints|Constraints]] - Technical, organizational, and political constraints
3. [[03-context|Context and Scope]] - System boundaries, external interfaces
4. [[04-solution|Solution Strategy]] - Fundamental decisions and solution approaches
5. [[05-building-blocks|Building Block View]] - Static decomposition, component structure
6. [[06-runtime|Runtime View]] - Dynamic behavior, scenarios, interactions
7. [[07-deployment|Deployment View]] - Infrastructure, deployment, operations
8. [[08-crosscutting|Cross-cutting Concepts]] - Overarching concepts and patterns
9. [[09-decisions|Architecture Decisions]] - Important decisions and rationale
10. [[10-quality|Quality Requirements]] - Quality scenarios and requirements
11. [[11-risks|Risks and Technical Debt]] - Known risks and technical debt
12. [[12-glossary|Glossary]] - Important terms and definitions

## Documentation Status

Use the arc42-docs skill to populate these sections with content.

## Related

- [[../architecture/README|Architecture Documentation]]
- [[../decisions/index|Architecture Decisions]]

#arc42 #architecture
```

### 5. Create Templates

Create `vault/templates/adr-template.md`:

```markdown
# ADR-NNNN: [Title]

**Status**: Proposed | Accepted | Superseded | Deprecated

**Date**: YYYY-MM-DD

**Deciders**: [List of people involved]

**Tags**: #adr #architecture

## Context

[Describe the context and problem statement. What is the issue we're trying to solve? What are the forces at play (technical, political, social, project)?]

### Context Factors

- Technical context:
- Business context:
- Team context:

## Decision Drivers

- [Driver 1]
- [Driver 2]
- [Driver 3]

## Considered Options

### Option 1: [Name]

**Description**: [Brief description]

**Pros**:
- [Pro 1]
- [Pro 2]

**Cons**:
- [Con 1]
- [Con 2]

### Option 2: [Name]

**Description**: [Brief description]

**Pros**:
- [Pro 1]
- [Pro 2]

**Cons**:
- [Con 1]
- [Con 2]

### Option 3: [Name]

**Description**: [Brief description]

**Pros**:
- [Pro 1]
- [Pro 2]

**Cons**:
- [Con 1]
- [Con 2]

## Decision

[Describe the chosen option and why. Include the rationale behind the decision.]

**Chosen**: [Option name]

**Rationale**: [Why this option was chosen over others]

## Consequences

### Positive

- [Positive consequence 1]
- [Positive consequence 2]

### Negative

- [Negative consequence 1]
- [Negative consequence 2]

### Neutral

- [Neutral consequence 1]

## Implementation

[How will this decision be implemented? What are the next steps?]

- [ ] Action item 1
- [ ] Action item 2

## Validation

[How will we validate this decision? What metrics will we use?]

## Related Decisions

- Supersedes: [[NNNN-old-decision]]
- Related to: [[NNNN-other-decision]]
- Required by: [[NNNN-dependent-decision]]

## Related Documentation

- Requirements: [[../requirements/README]]
- Architecture: [[../arc42/04-solution]]
- Specifications: [[../specifications/features]]

## References

- [Reference 1]
- [Reference 2]

---

*Last updated: YYYY-MM-DD*
```

Create `vault/templates/meeting-notes.md`:

```markdown
# Meeting Notes: [Topic] - YYYY-MM-DD

**Date**: YYYY-MM-DD HH:MM

**Attendees**:
- [Name 1]
- [Name 2]

**Tags**: #meeting

## Agenda

1. [Topic 1]
2. [Topic 2]
3. [Topic 3]

## Discussion

### Topic 1

[Notes on discussion]

**Key Points**:
- [Point 1]
- [Point 2]

### Topic 2

[Notes on discussion]

**Key Points**:
- [Point 1]
- [Point 2]

## Decisions

- [ ] Decision 1 - Owner: [Name]
- [ ] Decision 2 - Owner: [Name]

## Action Items

- [ ] Action 1 - Owner: [Name] - Due: YYYY-MM-DD
- [ ] Action 2 - Owner: [Name] - Due: YYYY-MM-DD

## Related Documentation

- [[../decisions/index|ADRs]]
- [[../requirements/README|Requirements]]
- [[../architecture/README|Architecture]]

## Next Meeting

**Date**: YYYY-MM-DD

**Topics**:
- [Topic 1]
- [Topic 2]

---

*Next meeting: YYYY-MM-DD*
```

Create `vault/templates/requirement.md`:

```markdown
# REQ-NNN: [Requirement Title]

**ID**: REQ-NNN

**Status**: Draft | Proposed | Accepted | Implemented | Verified

**Priority**: Critical | High | Medium | Low

**Category**: Functional | Quality | Constraint

**Tags**: #requirement

## Description

[Clear, concise description of the requirement]

## Rationale

[Why is this requirement needed? What problem does it solve?]

## Source

- **Stakeholder**: [Who requested this]
- **Origin**: [Meeting, user feedback, analysis, etc.]
- **Date**: YYYY-MM-DD

## Acceptance Criteria

- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3

## Quality Attributes

[If this is a quality requirement, specify the ISO 25010 characteristic]

- **Characteristic**: [e.g., Performance Efficiency]
- **Sub-characteristic**: [e.g., Time Behavior]
- **Metric**: [e.g., Response time < 200ms]
- **Target**: [e.g., 95th percentile < 200ms]

## Constraints

[Any constraints that affect this requirement]

## Dependencies

- Depends on: [[REQ-NNN-other]]
- Conflicts with: [[REQ-NNN-another]]

## Related Documentation

- Specification: [[../specifications/features#feature-name]]
- Architecture: [[../arc42/05-building-blocks]]
- Decisions: [[../decisions/NNNN-related-decision]]

## Verification

[How will this requirement be verified?]

- Unit tests
- Integration tests
- Manual testing
- Performance testing

## Traceability

- **Features**: [[../specifications/features]]
- **Tests**: `test/features/feature-name.feature`
- **Implementation**: [Link to code]

## Notes

[Additional notes, considerations, or discussion]

---

*Last updated: YYYY-MM-DD*
```

### 6. Report Completion

After creating all files, report:

```
Obsidian vault structure created successfully at vault/

Created directories:
- vault/architecture/
- vault/requirements/
- vault/specifications/
- vault/decisions/
- vault/security/
- vault/arc42/
- vault/templates/

Created documentation:
- README files for each section with navigation wikilinks
- ADR index for tracking decisions
- Templates for ADRs, meeting notes, and requirements

Next steps:
1. Open the vault in Obsidian
2. Consider installing Obsidian MCP for Claude Code integration
3. Use arc42-docs skill to populate architecture sections
4. Use adr-management skill to create your first ADR
5. Use iso25010-quality skill to define quality requirements
6. Use scenari-specs skill to write behavioral specifications
```

## Obsidian MCP Integration

For seamless integration with Claude Code, install the Obsidian MCP server:

```bash
# Add to Claude Desktop MCP configuration
# or use the Obsidian MCP installation instructions
```

With Obsidian MCP, Claude can:
- Read and write vault files
- Search vault content
- Create and update notes
- Follow wikilinks
- Maintain vault structure

## Best Practices

1. **Use Wikilinks**: Connect related concepts with `[[page]]` links
2. **Tag Consistently**: Use consistent tags like `#architecture`, `#requirement`, `#adr`
3. **Update Regularly**: Keep documentation current as the system evolves
4. **Review Together**: Use documentation as a communication tool
5. **Link to Code**: Reference implementation in documentation
6. **Version Control**: Commit vault to git for history and collaboration

## Idempotent Operation

This skill is safe to run multiple times:
- Existing directories are not modified
- Existing files are not overwritten
- Only missing directories and files are created
- Preserves any custom content you've added

## Troubleshooting

**Issue**: Vault already exists
**Solution**: This is fine! The skill will only create missing directories and files.

**Issue**: Templates not showing in Obsidian
**Solution**: Enable the Templates core plugin in Obsidian settings and set the template folder to `templates/`.

**Issue**: Wikilinks not working
**Solution**: Ensure the vault is opened in Obsidian and wikilinks are enabled in settings.

#documentation #obsidian #initialization
