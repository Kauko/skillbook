# ADR Templates Reference

This document provides comprehensive ADR template formats from various sources.

## Template Comparison

Multiple ADR template formats exist, each with different strengths:

- **Michael Nygard's Template** (2011): The foundational format that popularized ADRs
- **MADR (Markdown Any Decision Records)**: Streamlined, markdown-focused template
- **Y-Statement Format**: Derived from "Sustainable Architectural Decisions" by Zdun et al.
- **AWS Prescriptive Guidance**: Cloud-optimized decision documentation

## 1. Michael Nygard's Template

The original ADR template that established modern practices (from his 2011 blog post "Documenting Architecture Decisions").

### Structure

```markdown
# Title

## Status

What is the status, such as proposed, accepted, rejected, deprecated, superseded, etc.?

## Context

What is the issue that we're seeing that is motivating this decision or change?

## Decision

What is the change that we're proposing and/or doing?

## Consequences

What becomes easier or more difficult to do because of this change?
```

### Key Characteristics

- **Simplicity**: Minimal structure with four core sections
- **Focus**: Emphasizes context and consequences
- **Brevity**: Designed to be completed quickly
- **Immutability**: Once accepted, ADRs should not be modified (create new ones to supersede)

### When to Use

- Quick decision documentation
- Teams new to ADRs
- Decisions with clear context and outcomes
- When speed of documentation is important

### Example

```markdown
# Use Redis for Session Storage

## Status

Accepted

## Context

We need to store user session data for our web application. We have multiple
application servers behind a load balancer, so we need centralized session storage.

Current in-memory sessions don't work with multiple servers. We need:
- Fast read/write performance
- Automatic expiration
- Simple key-value storage
- High availability

## Decision

We will use Redis for session storage with the following configuration:
- Redis Cluster with 3 master nodes
- Automatic failover enabled
- Session TTL of 24 hours
- Store only session ID and user ID, not full user object

## Consequences

**Positive:**
- Fast session lookup (< 5ms)
- Automatic session expiration
- Scales horizontally with our application
- Well-tested and reliable technology

**Negative:**
- Additional infrastructure to maintain
- Need to monitor Redis health
- Sessions lost if Redis cluster fails completely
- Need backup strategy for Redis data

**Neutral:**
- Team needs to learn Redis operations
- Session data must be serializable
```

## 2. MADR (Markdown Any Decision Records)

Modern, structured template with comprehensive sections. Current version: MADR 4.0.0 (September 2024).

### Full MADR Template

```markdown
# [short title of solved problem and solution]

* Status: [proposed | rejected | accepted | deprecated | superseded by [ADR-0005](0005-example.md)]
* Deciders: [list everyone involved in the decision]
* Date: [YYYY-MM-DD when the decision was last updated]
* Technical Story: [description | ticket/issue URL]

## Context and Problem Statement

[Describe the context and problem statement, e.g., in free form using two to three sentences.
You may want to articulate the problem in form of a question.]

## Decision Drivers

* [driver 1, e.g., a force, facing concern, ...]
* [driver 2, e.g., a force, facing concern, ...]
* [...]

## Considered Options

* [option 1]
* [option 2]
* [option 3]
* [...]

## Decision Outcome

Chosen option: "[option 1]", because [justification. e.g., only option, which meets k.o.
criterion decision driver | which resolves force force | ... | comes out best (see below)].

### Consequences

* Good, because [positive consequence, e.g., improvement of quality attribute satisfaction,
  follow-up decisions required, ...]
* Bad, because [negative consequence, e.g., compromising quality attribute, follow-up
  decisions required, ...]
* ...

### Confirmation

[Describe how compliance with the ADR is ensured and how deviations are detected. Options include
manual review, ArchUnit tests, fitness functions, or automated validation.]

## Pros and Cons of the Options

### [option 1]

[example | description | pointer to more information | ...]

* Good, because [argument a]
* Good, because [argument b]
* Bad, because [argument c]
* ...

### [option 2]

[example | description | pointer to more information | ...]

* Good, because [argument a]
* Good, because [argument b]
* Bad, because [argument c]
* ...

### [option 3]

[example | description | pointer to more information | ...]

* Good, because [argument a]
* Good, because [argument b]
* Bad, because [argument c]
* ...

## More Information

[You can use this section to provide additional evidence/confidence for the decision, if that's
useful. Optionally, list the team members who contributed to the decision including their doubts
or concerns. Alternatively, link to relevant discussions or provide other sources of insight.]
```

### MADR Minimal Template

For simpler decisions, MADR provides a minimal variant:

```markdown
# [short title of solved problem and solution]

## Context and Problem Statement

[Describe the context and problem statement]

## Considered Options

* [option 1]
* [option 2]
* [option 3]

## Decision Outcome

Chosen option: "[option 1]", because [justification].
```

### Key Characteristics

- **Structure**: More detailed than Nygard's template
- **Decision Drivers**: Explicit section for forces influencing the decision
- **Pros/Cons Analysis**: Structured comparison of all options
- **Confirmation**: How to verify the decision is being followed
- **Flexibility**: "Minimal" variant for simpler decisions
- **Consistency**: Markdownlint-ready with provided configuration

### Naming Convention

Files follow the pattern: `NNNN-title-with-dashes.md`
- NNNN: Sequential number (0001-9999)
- Example: `0023-use-postgresql-for-data-storage.md`

### Organization Best Practices

**Standard Location**: `docs/decisions/` directory

**Large Projects**: Organize by subdirectory:
```
docs/decisions/
  backend/
    0001-api-framework.md
    0002-database-choice.md
  frontend/
    0001-ui-framework.md
    0002-state-management.md
  infrastructure/
    0001-cloud-provider.md
```

### When to Use

- Complex decisions requiring detailed analysis
- Teams wanting structured decision documentation
- Projects needing clear option comparison
- When verification/compliance tracking is important

## 3. Y-Statement Format

Derived from "Sustainable Architectural Decisions" by Zdun et al.

### Structure

```
In the context of [use case/user story],
facing [concern],
we decided for [option]
to achieve [quality],
accepting [downside].
```

### Example

```
In the context of our e-commerce API,
facing the need for flexible querying by mobile clients,
we decided for GraphQL over REST,
to achieve reduced over-fetching and better mobile performance,
accepting increased backend complexity and learning curve.
```

### Key Characteristics

- **Conciseness**: Single-sentence or paragraph format
- **Context-driven**: Explicitly ties decision to use case
- **Trade-off focused**: Always includes accepted downsides
- **Quality-oriented**: Links decision to quality attributes

### When to Use

- As a summary at the top of longer ADRs
- For decision logs where brevity is critical
- When creating decision indexes or overviews
- For decisions with clear context and trade-offs

## 4. Extended Template (From Current Skill)

The template included in this skill is comprehensive and includes sections for:

- Title and metadata (Status, Date, Deciders, Tags)
- Context (Background, Context Factors, Problem Statement)
- Decision Drivers
- Considered Options (with detailed pros/cons, effort, risk)
- Decision (Rationale, Key Factors, Trade-offs)
- Consequences (Positive, Negative, Neutral)
- Implementation (Action Plan, Migration Strategy, Rollback Plan)
- Validation (Success Criteria, Metrics, Review Date)
- Risks (with mitigation strategies)
- Related Decisions and Documentation
- References
- Discussion Notes

### When to Use

- Critical architectural decisions
- Decisions requiring team alignment
- Decisions with complex implementation plans
- When detailed tracking and validation are needed

## Choosing the Right Template

### Use Nygard's Template When:
- You're starting with ADRs
- Speed of documentation is important
- Decisions are relatively straightforward
- Team wants minimal overhead

### Use MADR When:
- You want standardized, structured documentation
- Multiple options need clear comparison
- Verification/compliance is important
- You're using markdown tooling (markdownlint, etc.)

### Use Y-Statement When:
- Creating decision summaries
- Building decision indexes
- Decisions are simple and context is clear
- Brevity is paramount

### Use Extended Template When:
- Decisions are highly complex
- Implementation tracking is critical
- Risk analysis is needed
- Detailed consequences and validation required

## Template Customization

All templates can be customized for your context:

1. **Start Simple**: Begin with Nygard's or MADR minimal template
2. **Add Sections**: Gradually add sections as needed (implementation, risks, etc.)
3. **Remove Sections**: If sections aren't providing value, remove them
4. **Be Consistent**: Once you choose a format, use it consistently across your project
5. **Evolve**: Templates can evolve as your team's needs change

## Template Metadata

All ADR templates should include:

- **Unique Identifier**: Sequential number (0001, 0002, etc.)
- **Status**: Proposed, Accepted, Superseded, Deprecated, Rejected
- **Date**: When the decision was made or last updated
- **Deciders**: Who participated in the decision

## References

- Michael Nygard, "Documenting Architecture Decisions" (2011)
- MADR 4.0.0 (September 2024) - https://adr.github.io/madr/
- "Sustainable Architectural Decisions" by Zdun et al.
- "Architectural Decision Guidance Across Projects" (WICSA 2015)
- ADR GitHub Organization - https://adr.github.io/
