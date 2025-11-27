# ADR Management

Use when making architectural decisions. Creates and manages Architectural Decision Records (ADRs) in the Obsidian vault.

## Purpose

Documents important architectural decisions with their context, rationale, and consequences. Maintains a chronological record of decisions and their evolution over time.

## When to Use

Create an ADR when you make a decision that:
- Affects the system structure or architecture
- Is difficult or expensive to reverse
- Impacts multiple components or teams
- Sets a precedent for future decisions
- Has significant trade-offs
- Requires team alignment

## Prerequisites

- Obsidian vault initialized (use init-obsidian-vault skill)
- vault/decisions/ directory exists

## ADR Structure

An ADR contains:
- **Title**: Short, descriptive name
- **Status**: Proposed, Accepted, Superseded, Deprecated
- **Context**: The situation requiring a decision
- **Decision**: The chosen solution
- **Consequences**: Positive and negative outcomes
- **Alternatives**: Other options considered

## Execution Steps

### Step 1: Check Prerequisites

Verify vault structure exists:

```bash
ls -d vault/decisions/ 2>/dev/null || echo "ERROR: Run init-obsidian-vault skill first"
```

If vault doesn't exist, suggest running `init-obsidian-vault` skill.

### Step 2: Determine Next ADR Number

Find the highest existing ADR number:

```bash
# Find highest numbered ADR
ls vault/decisions/ | grep -E "^[0-9]{4}-.*\.md$" | sort -r | head -1

# Extract number and increment
# e.g., 0005-some-decision.md -> next is 0006
```

If no ADRs exist, start with `0001`.

### Step 3: Gather Decision Information

Interview the user about the decision:

1. **What decision needs to be made?**
   - What is the problem or situation?
   - What are we trying to achieve?

2. **What is the context?**
   - Technical context
   - Business context
   - Timeline/urgency
   - Stakeholders involved

3. **What options are you considering?**
   - What are the alternatives?
   - What are the pros and cons of each?

4. **What are the decision drivers?**
   - What factors are most important?
   - What constraints exist?
   - What quality attributes are affected?

5. **Have you made a choice?**
   - If yes: Which option and why?
   - If no: Mark as "Proposed" for discussion

### Step 4: Check for ADR CLI Tool

Check if ADR tools are available:

```bash
# Check for adr-tools
which adr || which adr-log || echo "ADR tools not found, using template approach"
```

If ADR tools found, can use them. Otherwise, use template approach.

### Step 5: Create ADR File

Create the ADR file with proper numbering:

```bash
# Example: Creating ADR 0006
FILENAME="vault/decisions/0006-use-graphql-api.md"
```

Use the complete template from `vault/templates/adr-template.md`, filling in:
- Title
- Status (usually "Proposed" or "Accepted")
- Date
- Deciders
- Context with all gathered information
- Decision drivers
- All considered options with pros/cons
- The decision and rationale
- Consequences (positive, negative, neutral)
- Implementation plan
- Related decisions and documentation

### Step 6: Update ADR Index

Update `vault/decisions/index.md` to include the new ADR:

```markdown
## Active Decisions

- [[0006-use-graphql-api|0006: Use GraphQL API]] - YYYY-MM-DD - Proposed
- [[0005-previous-decision|0005: Previous Decision]] - YYYY-MM-DD - Accepted
```

Sort by number (newest first) and group by status.

### Step 7: Link to Related Documentation

Add links from the ADR to:
- arc42 sections (especially [[../arc42/09-decisions]])
- Related requirements ([[../requirements/README]])
- Related ADRs
- Affected architecture documentation

Add links TO the ADR from:
- arc42 Section 9 ([[../arc42/09-decisions]])
- arc42 Section 4 if it affects solution strategy
- Related components in Building Block View

### Step 8: Report Creation

Provide summary:

```
ADR created: vault/decisions/0006-use-graphql-api.md

Status: Proposed
Date: 2025-01-15
Decision: Use GraphQL API for mobile clients

Next steps:
1. Review ADR with team
2. Update status to "Accepted" when agreed
3. Begin implementation
4. Update arc42 documentation
```

## ADR Template

Complete template for an ADR:

```markdown
# ADR-NNNN: [Title]

**Status**: Proposed | Accepted | Superseded | Deprecated

**Date**: YYYY-MM-DD

**Deciders**: [Names of people who participated]

**Tags**: #adr #[category]

## Context

[Describe the context and problem statement. What issue are we trying to solve? What forces are at play?]

### Background

[Provide background information that helps understand the decision]

### Context Factors

- **Technical context**: [Technical aspects affecting the decision]
- **Business context**: [Business drivers and constraints]
- **Team context**: [Team size, skills, preferences]
- **Timeline**: [Time constraints or deadlines]

## Problem Statement

[Clear statement of the problem or opportunity]

## Decision Drivers

- [Driver 1] - [Why it matters]
- [Driver 2] - [Why it matters]
- [Driver 3] - [Why it matters]

## Considered Options

### Option 1: [Name]

**Description**: [Detailed description of this option]

**Pros**:
- ✅ [Advantage 1]
- ✅ [Advantage 2]
- ✅ [Advantage 3]

**Cons**:
- ❌ [Disadvantage 1]
- ❌ [Disadvantage 2]
- ❌ [Disadvantage 3]

**Effort**: [Low / Medium / High]

**Risk**: [Low / Medium / High]

### Option 2: [Name]

**Description**: [Detailed description of this option]

**Pros**:
- ✅ [Advantage 1]
- ✅ [Advantage 2]

**Cons**:
- ❌ [Disadvantage 1]
- ❌ [Disadvantage 2]

**Effort**: [Low / Medium / High]

**Risk**: [Low / Medium / High]

### Option 3: [Name]

**Description**: [Detailed description of this option]

**Pros**:
- ✅ [Advantage 1]
- ✅ [Advantage 2]

**Cons**:
- ❌ [Disadvantage 1]
- ❌ [Disadvantage 2]

**Effort**: [Low / Medium / High]

**Risk**: [Low / Medium / High]

## Decision

**Chosen Option**: Option [N] - [Name]

### Rationale

[Detailed explanation of why this option was chosen over others. Reference decision drivers.]

**Key Factors**:
1. [Factor 1 and how chosen option addresses it]
2. [Factor 2 and how chosen option addresses it]
3. [Factor 3 and how chosen option addresses it]

**Trade-offs Accepted**:
- [Trade-off 1 and why we accept it]
- [Trade-off 2 and why we accept it]

## Consequences

### Positive Consequences

- ✅ [Positive outcome 1]
- ✅ [Positive outcome 2]
- ✅ [Positive outcome 3]

### Negative Consequences

- ❌ [Negative outcome 1] - [How we'll mitigate]
- ❌ [Negative outcome 2] - [How we'll mitigate]

### Neutral Consequences

- ⚪ [Neutral outcome 1]
- ⚪ [Neutral outcome 2]

## Implementation

### Action Plan

- [ ] [Action item 1] - Owner: [Name] - Due: [Date]
- [ ] [Action item 2] - Owner: [Name] - Due: [Date]
- [ ] [Action item 3] - Owner: [Name] - Due: [Date]

### Migration Strategy

[If this decision requires migrating from an existing approach, describe the migration strategy]

### Rollback Plan

[How can we reverse this decision if it doesn't work out?]

**Rollback Effort**: [Low / Medium / High / Impossible]

## Validation

### Success Criteria

[How will we know if this decision was successful?]

- [ ] [Criterion 1]
- [ ] [Criterion 2]
- [ ] [Criterion 3]

### Metrics

[What metrics will we track?]

- [Metric 1]: Target [value]
- [Metric 2]: Target [value]

### Review Date

[When will we review this decision?]

**Review Date**: YYYY-MM-DD

## Risks

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| [Risk 1] | [L/M/H] | [L/M/H] | [How we'll address it] |
| [Risk 2] | [L/M/H] | [L/M/H] | [How we'll address it] |

## Related Decisions

- **Supersedes**: [[NNNN-old-decision|ADR-NNNN]] - [Why this replaces it]
- **Related to**: [[NNNN-other-decision|ADR-NNNN]] - [How they relate]
- **Required by**: [[NNNN-dependent-decision|ADR-NNNN]] - [What depends on this]

## Related Documentation

- **Requirements**: [[../requirements/README]]
- **Architecture**: [[../arc42/04-solution|Solution Strategy]]
- **Quality**: [[../arc42/10-quality|Quality Requirements]]
- **Specifications**: [[../specifications/features]]
- **Security**: [[../security/README]] (if applicable)

## References

- [Reference 1 - Title](URL)
- [Reference 2 - Title](URL)
- [Reference 3 - Title](URL)

## Discussion Notes

[Notes from team discussions about this decision]

### Team Feedback

- [Person 1]: [Feedback]
- [Person 2]: [Feedback]

### Questions Raised

- [Question 1]: [Answer]
- [Question 2]: [Answer]

---

*Last updated: YYYY-MM-DD*

*Next review: YYYY-MM-DD*
```

## Example ADR

Here's a complete example:

```markdown
# ADR-0005: Use JWT for Authentication

**Status**: Accepted

**Date**: 2025-01-15

**Deciders**: Development Team (Alice, Bob, Charlie), Security Architect (David)

**Tags**: #adr #security #authentication

## Context

We need to implement authentication for our REST API. The system will serve both web and mobile clients, and we expect to scale to multiple API server instances.

### Background

- Current MVP has no authentication
- Target: 10,000 users initially, growing to 100,000 in 12 months
- Multiple client types (web, iOS, Android)
- Multiple API server instances for high availability
- RESTful API architecture

### Context Factors

- **Technical context**: Microservices-ready architecture, stateless API servers
- **Business context**: Fast time-to-market, standard security requirements
- **Team context**: Team familiar with JWT from previous projects
- **Timeline**: Authentication needed in 2 weeks for beta launch

## Problem Statement

How should we authenticate users across multiple API server instances while maintaining stateless architecture and supporting multiple client types?

## Decision Drivers

- **Stateless**: API servers should be stateless for easy scaling
- **Standard**: Use well-established, secure standards
- **Multi-client**: Support web, iOS, and Android clients
- **Performance**: Minimize authentication overhead
- **Developer Experience**: Easy for clients to implement
- **Security**: Meet standard security requirements

## Considered Options

### Option 1: Session-based Authentication

**Description**: Traditional server-side sessions stored in database or Redis

**Pros**:
- ✅ Well-understood pattern
- ✅ Easy to invalidate sessions
- ✅ Simple implementation

**Cons**:
- ❌ Requires shared state (database/Redis)
- ❌ More complex for mobile clients
- ❌ Additional latency for session lookup
- ❌ Doesn't scale horizontally as easily

**Effort**: Medium

**Risk**: Low

### Option 2: JWT (JSON Web Tokens)

**Description**: Stateless tokens containing claims, signed by server

**Pros**:
- ✅ Stateless - no server-side storage needed
- ✅ Self-contained - includes all auth info
- ✅ Standard (RFC 7519)
- ✅ Works well with mobile clients
- ✅ Easy horizontal scaling
- ✅ Can include custom claims

**Cons**:
- ❌ Cannot invalidate individual tokens easily
- ❌ Token size larger than session ID
- ❌ Requires careful expiry management
- ❌ Need refresh token strategy

**Effort**: Medium

**Risk**: Low-Medium

### Option 3: OAuth 2.0 with External Provider

**Description**: Delegate authentication to external provider (Auth0, Okta)

**Pros**:
- ✅ Offload authentication complexity
- ✅ Social login support
- ✅ Industry standard
- ✅ Advanced features (MFA, etc.)

**Cons**:
- ❌ External dependency
- ❌ Additional cost
- ❌ Less control over auth flow
- ❌ Overkill for current needs

**Effort**: Medium-High

**Risk**: Medium

## Decision

**Chosen Option**: Option 2 - JWT (JSON Web Tokens)

### Rationale

JWT best fits our requirements for a stateless, scalable architecture:

1. **Stateless Architecture**: JWT aligns with our goal of stateless API servers, enabling easy horizontal scaling without session storage dependencies.

2. **Multi-client Support**: JWT is well-supported across web and mobile platforms, with mature libraries available.

3. **Performance**: No database lookup required for each request - authentication is self-contained in the token.

4. **Team Familiarity**: Team has previous experience implementing JWT successfully.

5. **Standard Solution**: RFC 7519 standard with extensive tooling and security research.

**Trade-offs Accepted**:
- Cannot instantly invalidate tokens (mitigated with short expiry + refresh tokens)
- Slightly larger payload than session IDs (acceptable for our use case)

### Implementation Details

- Access token expiry: 15 minutes
- Refresh token expiry: 7 days
- Use RS256 algorithm (asymmetric)
- Store tokens in httpOnly cookies for web clients
- Store in secure storage for mobile clients

## Consequences

### Positive Consequences

- ✅ Stateless API servers enable easy horizontal scaling
- ✅ No session storage infrastructure needed
- ✅ Fast authentication (no DB lookup per request)
- ✅ Can run multiple API instances without shared state
- ✅ Standard solution with good library support

### Negative Consequences

- ❌ Need to implement refresh token rotation
- ❌ Cannot instantly revoke access tokens (15-minute window)
- ❌ Need to implement token blacklist for immediate revocation scenarios
- ❌ Token size ~500 bytes vs session ID ~32 bytes

### Neutral Consequences

- ⚪ Need to implement token refresh flow
- ⚪ Need secure key management for signing

## Implementation

### Action Plan

- [x] Research JWT libraries for Clojure - Owner: Alice - Due: 2025-01-16
- [x] Implement JWT generation and validation - Owner: Bob - Due: 2025-01-18
- [x] Implement refresh token flow - Owner: Bob - Due: 2025-01-20
- [ ] Implement token blacklist (Redis) - Owner: Charlie - Due: 2025-01-22
- [ ] Add authentication middleware - Owner: Alice - Due: 2025-01-23
- [ ] Update API documentation - Owner: Alice - Due: 2025-01-25
- [ ] Security review - Owner: David - Due: 2025-01-26

### Migration Strategy

This is new functionality (no migration needed).

### Rollback Plan

If JWT proves problematic, we can switch to session-based authentication. The API interface (Authorization header) would remain the same.

**Rollback Effort**: Medium (2-3 days)

## Validation

### Success Criteria

- [ ] Users can successfully authenticate
- [ ] Tokens are properly validated on all endpoints
- [ ] Refresh token flow works correctly
- [ ] Token expiry is enforced
- [ ] No security vulnerabilities in implementation

### Metrics

- Authentication latency: Target < 10ms
- Token validation latency: Target < 1ms
- Failed authentication rate: Target < 5%

### Review Date

**Review Date**: 2025-04-15 (3 months after implementation)

## Risks

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| Token theft | Medium | High | Use httpOnly cookies, short expiry, refresh rotation |
| Cannot revoke tokens | High | Medium | Implement token blacklist in Redis |
| Key compromise | Low | Critical | Rotate keys quarterly, secure key storage |
| XSS attacks | Low | High | CSP headers, input sanitization |

## Related Decisions

- **Related to**: [[0001-api-architecture|ADR-0001: REST API Architecture]]
- **Required by**: [[0006-rate-limiting|ADR-0006: Rate Limiting]] (will use JWT claims)

## Related Documentation

- **Architecture**: [[../arc42/08-crosscutting#authentication|Authentication in Cross-cutting Concepts]]
- **Security**: [[../security/README|Security Documentation]]
- **Implementation**: `src/api/auth/jwt.clj`

## References

- [RFC 7519 - JWT](https://tools.ietf.org/html/rfc7519)
- [JWT.io](https://jwt.io/)
- [OWASP JWT Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html)

## Discussion Notes

### Team Feedback

- Alice: Agrees with JWT, suggests RS256 over HS256
- Bob: Concerned about token revocation, suggests blacklist
- Charlie: Prefers stateless but wants revocation solution
- David: Approves from security perspective with blacklist mitigation

### Questions Raised

- **Q**: How do we handle token revocation for compromised accounts?
  **A**: Implement token blacklist in Redis for immediate revocation scenarios

- **Q**: Should we use symmetric (HS256) or asymmetric (RS256)?
  **A**: RS256 - allows token validation without sharing secret

---

*Last updated: 2025-01-15*

*Next review: 2025-04-15*
```

## Superseding ADRs

When an ADR is superseded:

1. **Update old ADR**:
   - Change status to "Superseded"
   - Add link to new ADR that replaces it
   - Add deprecation date

2. **Reference in new ADR**:
   - In "Related Decisions" section
   - Explain why old decision is being replaced

3. **Update index**:
   - Move old ADR to "Superseded Decisions" section
   - Add new ADR to "Active Decisions"

Example superseded ADR:

```markdown
# ADR-0003: Use REST API

**Status**: Superseded by [[0015-graphql-api|ADR-0015]]

**Date**: 2024-06-15

**Superseded**: 2025-01-15

[Original ADR content...]

## Superseded

This decision has been superseded by [[0015-graphql-api|ADR-0015: Use GraphQL API]].

**Reason**: Mobile clients need more flexible querying and reduced over-fetching. GraphQL better fits mobile use cases.

**Migration**: Existing REST API will be maintained for backward compatibility during 6-month transition period.
```

## ADR Best Practices

1. **Write Timely**: Create ADR when decision is made, not after
2. **Be Specific**: Concrete details, not vague generalities
3. **Show Trade-offs**: Document pros and cons honestly
4. **Consider Alternatives**: Show you evaluated multiple options
5. **Link Everything**: Connect to related docs, ADRs, code
6. **Keep Brief**: Aim for 1-2 pages, not 10
7. **Use Plain Language**: Write for humans, not just experts
8. **Update Status**: Keep status current as decisions evolve
9. **Review Periodically**: Schedule reviews to ensure decisions still make sense

## Common Decision Categories

Use consistent tags to categorize ADRs:

- `#technology` - Technology stack choices
- `#architecture` - Structural decisions
- `#security` - Security-related decisions
- `#deployment` - Infrastructure and deployment
- `#process` - Development process decisions
- `#data` - Data modeling and storage
- `#integration` - External system integration

## Integration with arc42

ADRs complement arc42 documentation:

- **Section 2 (Constraints)**: ADRs explain how constraints influenced decisions
- **Section 4 (Solution Strategy)**: ADRs provide detailed rationale for strategy choices
- **Section 9 (Architecture Decisions)**: Summary of all ADRs with links

Keep Section 9 synchronized with ADR index.

## Related Skills

- **init-obsidian-vault**: Creates the vault structure for ADRs
- **arc42-docs**: References ADRs in architecture documentation

#adr #decisions #documentation
