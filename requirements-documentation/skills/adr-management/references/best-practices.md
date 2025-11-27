# ADR Best Practices

Comprehensive guidelines for writing effective Architecture Decision Records.

## Core Principles

### 1. Document at Decision Time

**Write when the decision is made, not after implementation.**

**Why:**
- Fresh context and rationale
- Accurate representation of options considered
- Captures uncertainty and discussion that led to decision
- Prevents hindsight bias

**How:**
- Create ADR as soon as decision is made
- If decision takes time, create with "Proposed" status
- Update when decision is finalized

**Anti-pattern:**
```
❌ Documenting decisions months after they were made
❌ Writing ADRs only when asked about why something was done
❌ Treating ADRs as retrospective documentation
```

**Good Practice:**
```
✅ Create ADR during or immediately after decision meeting
✅ Use "Proposed" status for decisions under discussion
✅ Update ADR when decision is accepted/rejected
```

### 2. Be Specific and Concrete

**Use concrete details, not vague generalities.**

**Why:**
- Future readers need actionable information
- Vague language doesn't help understand trade-offs
- Specific details enable evaluation of whether decision still applies

**Anti-pattern:**
```markdown
❌ We chose the better technology.
❌ This option is more scalable.
❌ We need better performance.
```

**Good Practice:**
```markdown
✅ We chose PostgreSQL to handle 100,000 writes/second with ACID guarantees.
✅ Redis Cluster scales horizontally to 1000 nodes supporting 1M ops/sec.
✅ Current API response time is 500ms; target is <100ms for 95th percentile.
```

### 3. Show Trade-offs Honestly

**Document both advantages and disadvantages of the chosen option.**

**Why:**
- Every decision has trade-offs
- Honest documentation builds trust
- Helps future teams understand what was accepted and why
- Shows thorough evaluation

**What to Include:**
- What you're gaining
- What you're giving up
- What becomes harder
- What becomes easier
- What risks you're accepting

**Anti-pattern:**
```markdown
❌ Decision: Use Microservices

Consequences:
- Better scalability
- Better modularity
- Better team independence
```

**Good Practice:**
```markdown
✅ Decision: Use Microservices

Positive Consequences:
- Independent deployment of services
- Teams can choose appropriate tech for each service
- Failure isolation between services

Negative Consequences:
- Increased operational complexity (need service mesh, monitoring)
- Distributed system challenges (eventual consistency, network failures)
- More difficult to debug cross-service issues
- Higher initial development cost vs monolith
```

### 4. Consider Multiple Options

**Show that you evaluated alternatives before deciding.**

**Why:**
- Demonstrates thorough evaluation
- Helps readers understand why rejected options weren't chosen
- Provides fallback options if decision needs revisiting
- Shows what was known at decision time

**Minimum:**
- Document at least 2-3 options
- Include the "do nothing" option when relevant
- Show pros and cons of each option

**Anti-pattern:**
```markdown
❌ Decision: Use MongoDB

We decided to use MongoDB for our database.
```

**Good Practice:**
```markdown
✅ Considered Options:

1. PostgreSQL
   Pros: ACID, mature, excellent JSON support, strong consistency
   Cons: Vertical scaling limits, more complex sharding

2. MongoDB
   Pros: Horizontal scaling, flexible schema, native JSON
   Cons: Eventual consistency challenges, less mature transaction support

3. DynamoDB
   Pros: Fully managed, predictable performance, auto-scaling
   Cons: Vendor lock-in, limited query flexibility, cost at scale

Decision: MongoDB, because our primary need is horizontal scalability
for document storage with evolving schema, and we can handle eventual
consistency in our use cases.
```

### 5. Link Everything

**Connect ADRs to related documentation, code, and other ADRs.**

**Why:**
- Creates navigable documentation
- Shows relationship between decisions
- Helps readers find related information
- Tracks decision evolution

**What to Link:**
- Related ADRs (supersedes, related to, required by)
- Architecture documentation (arc42 sections)
- Requirements documents
- Code implementations
- External references

**Good Practice:**
```markdown
## Related Decisions

- **Supersedes**: [[0003-rest-api|ADR-0003: REST API]]
- **Related to**: [[0005-authentication|ADR-0005: JWT Authentication]]
- **Required by**: [[0012-mobile-api|ADR-0012: Mobile API Design]]

## Related Documentation

- **Architecture**: [[../arc42/04-solution|Solution Strategy]]
- **Requirements**: [[../requirements/mobile-app|Mobile App Requirements]]
- **Implementation**: `src/api/graphql/schema.graphql`

## References

- [GraphQL Best Practices](https://graphql.org/learn/best-practices/)
- [GitHub GraphQL API](https://docs.github.com/en/graphql)
```

### 6. Keep it Brief

**Aim for 1-3 pages, not 10.**

**Why:**
- More likely to be read
- Easier to maintain
- Forces clarity and focus
- Faster to write

**Guidelines:**
- 1-2 pages for simple decisions
- 2-3 pages for complex decisions
- Move extensive details to appendices or separate docs
- Use links to detailed documentation

**Anti-pattern:**
```
❌ 15-page ADR with implementation details, full API specs, complete code examples
❌ Copying entire sections from product documentation
❌ Including extensive background that isn't decision-relevant
```

**Good Practice:**
```
✅ Concise context (2-3 paragraphs)
✅ Clear options with bullet points
✅ Focused consequences (5-10 items)
✅ Links to detailed documentation for deep dives
```

### 7. Use Plain Language

**Write for humans, not just experts.**

**Why:**
- New team members need to understand decisions
- Future readers may not have current context
- Non-technical stakeholders may need to review
- Clarity benefits everyone

**Guidelines:**
- Avoid unnecessary jargon
- Explain acronyms on first use
- Assume reader has general technical knowledge
- Provide context for specialized terms

**Anti-pattern:**
```
❌ "We'll implement CQRS with ES using DDD aggregates in the BC,
    leveraging saga orchestration via the SCP."
```

**Good Practice:**
```
✅ "We'll use Command Query Responsibility Segregation (CQRS) with
    Event Sourcing (ES). This means separating our read and write
    models, and storing all changes as events.

    We'll organize code using Domain-Driven Design (DDD) principles,
    with aggregates defining consistency boundaries within each
    bounded context (BC).

    For distributed transactions, we'll use saga orchestration via
    the Saga Coordination Pattern (SCP)."
```

### 8. Update Status as Decisions Evolve

**Keep ADR status current to reflect reality.**

**Why:**
- Shows which decisions are active
- Helps readers know what's current
- Tracks decision evolution over time
- Prevents confusion from outdated information

**Status Values:**
- **Proposed**: Under discussion, not yet decided
- **Accepted**: Decision is made and active
- **Superseded**: Replaced by another ADR (link to it)
- **Deprecated**: No longer recommended but not replaced
- **Rejected**: Considered but decided against

**When to Update:**
- Decision is accepted: Proposed → Accepted
- New decision replaces old: Accepted → Superseded
- Decision no longer applies: Accepted → Deprecated
- Proposal is rejected: Proposed → Rejected

**Good Practice:**
```markdown
# ADR-0003: Use REST API

**Status**: Superseded by [[0015-graphql-api|ADR-0015]]
**Date**: 2024-06-15
**Superseded**: 2025-01-15

[Original content...]

## Superseded

This decision has been superseded by [[0015-graphql-api|ADR-0015: GraphQL API]].

**Reason**: Mobile clients need more flexible querying. GraphQL reduces
over-fetching and improves mobile performance.

**Migration**: REST API maintained for 6 months during transition.
```

### 9. Review Periodically

**Schedule reviews to ensure decisions still make sense.**

**Why:**
- Technology and requirements change
- Assumptions may no longer hold
- New options may be available
- Team learns from implementation

**How:**
- Add review date to each ADR
- Schedule review meeting (quarterly or semi-annually)
- Check if decision is still valid
- Update or create new ADR if needed

**Good Practice:**
```markdown
## Validation

### Review Date

**Review Date**: 2025-06-15 (6 months after implementation)

**Review Criteria:**
- Are we meeting target performance metrics?
- Has team successfully adopted GraphQL?
- Are mobile clients benefiting as expected?
- Have any significant issues emerged?

**Next Review**: 2025-12-15
```

## Decision-Making Process Best Practices

### When to Create an ADR

Create an ADR when a decision:
- Affects system structure or architecture
- Is difficult or expensive to reverse
- Impacts multiple components or teams
- Sets a precedent for future decisions
- Has significant trade-offs
- Requires team alignment
- Answers "Why did we do it this way?"

### When NOT to Create an ADR

Don't create ADRs for:
- Trivial decisions (variable names, minor refactorings)
- Decisions easily reversed (can be changed in one commit)
- Purely local decisions (one function, one file)
- Decisions already documented in code/comments
- Decisions that are obvious from context

### Decision Drivers

**Always document what influenced the decision:**

- **Technical Drivers:**
  - Performance requirements
  - Scalability needs
  - Security requirements
  - Integration constraints
  - Technology limitations

- **Business Drivers:**
  - Time to market
  - Budget constraints
  - Regulatory requirements
  - Business strategy
  - Competitive pressure

- **Team Drivers:**
  - Team expertise
  - Learning goals
  - Team size
  - Hiring considerations
  - Operational capacity

**Good Practice:**
```markdown
## Decision Drivers

- **Performance**: Need <100ms API response time for 95th percentile
- **Scalability**: Must handle 10x growth in next 12 months
- **Team Experience**: Team has 3 years React experience, no Vue experience
- **Time to Market**: Must launch MVP in 8 weeks
- **Maintainability**: Small team (3 developers) needs simple architecture
```

## Writing Quality Best Practices

### Use Active Voice

**Anti-pattern:**
```
❌ "It was decided that MongoDB would be used."
❌ "The system will be deployed to AWS."
```

**Good Practice:**
```
✅ "We decided to use MongoDB."
✅ "We will deploy to AWS."
```

### Be Honest About Uncertainty

**Document what you don't know:**

```markdown
## Decision

We chose Option B, but we have uncertainty about:

- Whether it will scale to 1M users (we've only tested to 100K)
- Team's ability to learn the technology in 8 weeks
- Long-term maintenance cost

**Mitigation:**
- Will implement monitoring to track scale issues
- Plan 2-week learning sprint before implementation
- Schedule 3-month review to assess costs
```

### Show Reasoning, Not Just Conclusion

**Anti-pattern:**
```
❌ "We chose PostgreSQL because it's better."
```

**Good Practice:**
```
✅ "We chose PostgreSQL over MongoDB because:

1. We need ACID transactions for financial data
2. Our data model is highly relational (users, accounts, transactions)
3. We require strong consistency for account balances
4. Team has 5 years PostgreSQL experience vs 0 years MongoDB
5. Our query patterns are primarily relational joins

While MongoDB offers better horizontal scaling, our scalability
needs (100K users) are well within PostgreSQL's capabilities with
read replicas."
```

### Include Timeline and Context

**Always date decisions and explain urgency:**

```markdown
## Context Factors

- **Date**: 2025-01-15
- **Timeline**: Need production deployment by 2025-03-01
- **Team**: 3 developers, 1 DevOps engineer
- **Current State**: Prototype in development, 50% complete
- **Urgency**: Committed to beta customers for March 1 launch
```

## Organizational Best Practices

### File Naming

**Consistent naming:**
```
✅ 0001-use-postgresql.md
✅ 0002-implement-jwt-auth.md
✅ 0023-migrate-to-kubernetes.md

❌ adr-postgres.md
❌ decision-about-authentication.md
❌ 23-kubernetes.md
```

### Directory Structure

**Standard structure:**
```
docs/decisions/          # or vault/decisions/
  index.md              # Overview and index of all ADRs
  0001-first.md
  0002-second.md
  ...
  templates/
    adr-template.md     # Template for new ADRs
```

**For large projects:**
```
docs/decisions/
  index.md
  backend/
    0001-api-framework.md
    0002-database.md
  frontend/
    0001-ui-framework.md
  infrastructure/
    0001-cloud-provider.md
```

### Maintain an Index

**Create and maintain index.md:**

```markdown
# Architecture Decision Records

## Active Decisions

- [[0015-graphql-api|0015: GraphQL API]] - 2025-01-15 - Accepted
- [[0012-kubernetes|0012: Kubernetes Deployment]] - 2024-12-01 - Accepted
- [[0010-jwt-auth|0010: JWT Authentication]] - 2024-11-15 - Accepted

## Proposed Decisions (Under Review)

- [[0016-event-sourcing|0016: Event Sourcing]] - 2025-01-20 - Proposed

## Superseded Decisions

- [[0003-rest-api|0003: REST API]] - 2024-06-15 - Superseded by ADR-0015

## Deprecated Decisions

- [[0002-mongodb|0002: MongoDB]] - 2024-05-01 - Deprecated

## By Category

### Architecture
- 0015, 0012, 0003

### Security
- 0010, 0008, 0005

### Data
- 0016, 0006, 0002
```

## Integration with Other Documentation

### Arc42 Integration

**Section 9 (Architecture Decisions):**
```markdown
# 9. Architecture Decisions

## Overview

All detailed architecture decisions are documented as ADRs in
[[../decisions/index|the decisions directory]].

## Key Decisions

### API Design
- [[../decisions/0015-graphql-api|ADR-0015: GraphQL API]]

### Data Storage
- [[../decisions/0006-postgresql|ADR-0006: PostgreSQL]]

### Authentication
- [[../decisions/0010-jwt-auth|ADR-0010: JWT Authentication]]

## Decision Categories

See [[../decisions/index|ADR Index]] for complete list organized by category.
```

**Section 4 (Solution Strategy):**
Link to ADRs that explain strategy choices.

### Requirements Integration

**Link from requirements to ADRs:**
```markdown
# Mobile App Requirements

## Performance
- API response time < 100ms (95th percentile)
- Related: [[../decisions/0015-graphql-api|ADR-0015]]
```

**Link from ADRs to requirements:**
```markdown
## Related Documentation

- **Requirements**: [[../requirements/mobile-app|Mobile App Requirements]]
```

## Team Collaboration Best Practices

### Involve Right People

**Deciders should include:**
- Technical leads
- Affected team members
- Architecture owners
- Key stakeholders with relevant expertise

**Document who decided:**
```markdown
**Deciders**: Alice (Tech Lead), Bob (Backend Engineer),
              Charlie (DevOps), David (Security Architect)
```

### Capture Discussion

**Include key points from discussion:**
```markdown
## Discussion Notes

### Key Points
- Bob raised concerns about MongoDB transactions
- Charlie highlighted operational complexity of Kubernetes
- David approved JWT approach with token blacklist

### Questions Raised
- Q: How do we handle token revocation?
  A: Implement Redis-based token blacklist for critical revocations

- Q: What's the migration strategy from sessions?
  A: Gradual rollout - new users on JWT, migrate existing users over 2 weeks
```

### Async Decision Making

For distributed teams:
```markdown
**Status**: Proposed

**Review Period**: 2025-01-15 to 2025-01-22

**How to Comment**: Add comments to [[0015-graphql-api-discussion]]

**Decision Date**: 2025-01-22 (1 week review period)
```

## Common Pitfalls to Avoid

### 1. "Resume-Driven Development"

**Anti-pattern:**
```
❌ Choosing technology because it looks good on resume
❌ Using new technology without clear benefit
❌ Ignoring team expertise for "shiny new thing"
```

**Good Practice:**
```
✅ Choose based on actual project needs
✅ Consider team expertise seriously
✅ New technology only when clear benefit outweighs cost
```

### 2. Analysis Paralysis

**Anti-pattern:**
```
❌ Spending months documenting every possible option
❌ Creating 20-page ADRs
❌ Delaying decision indefinitely
```

**Good Practice:**
```
✅ Set decision deadline
✅ Good enough > perfect
✅ Can always supersede if needed
```

### 3. Post-hoc Justification

**Anti-pattern:**
```
❌ Writing ADR to justify decision already implemented
❌ Omitting downsides of chosen option
❌ Making unchosen options look worse than they were
```

**Good Practice:**
```
✅ Write ADR when decision is made
✅ Be honest about all options
✅ Document real reasoning, not ideal reasoning
```

### 4. No Follow-up

**Anti-pattern:**
```
❌ Never reviewing decisions
❌ Not updating superseded ADRs
❌ Ignoring when decision isn't working
```

**Good Practice:**
```
✅ Schedule reviews
✅ Update statuses promptly
✅ Create new ADR when situation changes
```

## References

- Michael Nygard, "Documenting Architecture Decisions" (2011)
- MADR 4.0.0 Best Practices - https://adr.github.io/madr/
- "Sustainable Architectural Decisions" by Zdun et al.
- AWS Prescriptive Guidance on ADRs
- "Architectural Decision Guidance Across Projects" (WICSA 2015)
- OWASP on Documenting Security Decisions
