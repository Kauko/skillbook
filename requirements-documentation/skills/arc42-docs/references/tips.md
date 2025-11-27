# arc42 Practical Tips and Common Mistakes

This document provides practical tips and identifies common mistakes to avoid when creating arc42 documentation.

## General Tips

### Starting Out

**Tip: Start Simple**
- Don't try to fill all 12 sections perfectly at once
- Begin with Sections 1 (Goals), 3 (Context), and 5 (Building Blocks)
- Iterate and refine as understanding deepens
- Empty sections are better than wrong information

**Tip: Know Your Audience**
- Different sections serve different stakeholders
- Business stakeholders: Sections 1, 3, 4
- Developers: Sections 5, 6, 8
- Operations: Sections 7, 11
- All stakeholders: Sections 9, 10, 12

**Tip: Visual Over Text**
- Always prefer diagrams over textual descriptions
- One diagram is worth 1000 words
- Use standard notations (UML, C4, Mermaid)
- Keep diagrams simple and focused

**Tip: Iterate Continuously**
- Architecture documentation is a living document
- Update after major changes
- Review quarterly minimum
- Mark with last updated date

### Documentation Approach

**Tip: Choose Your Style**
Three arc42 styles based on project needs:

1. **Lean**: Agile projects, small teams, rapid change
   - Minimal but sufficient documentation
   - Focus on decisions and key views
   - Quick to update

2. **Thorough**: Critical systems, regulated industries, large teams
   - Comprehensive coverage
   - Detailed rationale and alternatives
   - Formal review process

3. **Essential**: Middle ground for most projects
   - Core information always documented
   - Depth varies by importance
   - Practical balance

**Tip: Use Templates**
- Provide section templates for consistency
- Include examples and guidance
- Make it easy to get started
- Adapt templates to your context

**Tip: Link Everything**
- Use wikilinks in Obsidian
- Cross-reference between sections
- Link to ADRs, specifications, code
- Create knowledge graph

## Section-Specific Tips

### Section 1: Introduction and Goals

**Tip: Limit Quality Goals to 3-5**
- More goals dilute focus
- Forces prioritization
- Makes trade-offs clear
- Drives architectural decisions

**Tip: Make Goals Measurable**
- ❌ Bad: "System should be fast"
- ✅ Good: "p95 API response time < 200ms"
- Use quality scenarios
- Define metrics and targets

**Tip: Include Stakeholder Table**
- List all key stakeholders
- Capture their expectations
- Include contact information
- Update when stakeholders change

**Common Mistake: Too Many Goals**
- Problem: Listing 10+ quality goals
- Impact: No clear priorities
- Fix: Ruthlessly prioritize top 3-5
- Remember: Quality goals drive architecture

**Common Mistake: Vague Goals**
- Problem: "Good performance" or "Secure"
- Impact: Not actionable, can't verify
- Fix: Concrete scenarios and metrics
- Example: "Support 1000 concurrent users with <200ms response"

### Section 2: Constraints

**Tip: Distinguish Constraints from Decisions**
- Constraints: Given, unchangeable (regulatory, budget, time)
- Decisions: Choices you make (framework, pattern)
- Constraints limit design freedom
- Document rationale for constraints

**Tip: Categorize Constraints**
- Technical (languages, platforms, tools)
- Organizational (team, budget, timeline)
- Political/Legal (regulations, compliance)
- Makes them easier to review and update

**Tip: Document Impact**
- How does constraint affect architecture?
- What alternatives were eliminated?
- Workarounds or mitigation strategies
- Links to affected decisions

**Common Mistake: Confusing Constraints and Decisions**
- Problem: Listing design choices as constraints
- Impact: Blurs accountability
- Fix: Ask "Can we change this?" If yes, it's a decision
- Constraints are imposed from outside

**Common Mistake: Missing Hidden Constraints**
- Problem: Obvious constraints go undocumented
- Impact: New team members miss context
- Fix: Document even "obvious" constraints
- What's obvious to you isn't to everyone

### Section 3: Context and Scope

**Tip: Separate Business and Technical Context**
- Business context: For non-technical stakeholders
- Shows external entities and relationships
- Focus on "what" is exchanged
- Technical context: For developers and integrators
- Shows protocols, formats, technologies
- Focus on "how" communication works

**Tip: Make Boundary Crystal Clear**
- What is inside your system?
- What is delegated to external systems?
- What are you responsible for?
- Use visual boundary box in diagrams

**Tip: Document External Interfaces**
- For each external entity
- What data/information is exchanged?
- What protocols are used?
- Who owns the interface?
- What are dependencies?

**Common Mistake: Unclear System Boundary**
- Problem: Ambiguous what's in vs. out
- Impact: Confusion about responsibilities
- Fix: Explicit boundary in diagrams
- Table listing internal vs. external

**Common Mistake: Missing External Dependencies**
- Problem: Forgetting third-party services
- Impact: Surprises during integration
- Fix: List ALL external interactions
- Include email, payment, analytics, etc.

### Section 4: Solution Strategy

**Tip: Link to Quality Goals**
- Show how solution achieves each quality goal
- Explain trade-offs made
- Document why this approach was chosen
- Reference Section 1 goals

**Tip: Document Alternatives**
- What other approaches were considered?
- Why were they rejected?
- Trade-offs between alternatives
- Links to ADRs for details

**Tip: Keep High-Level**
- This is executive summary
- 1-2 pages maximum
- Details come in later sections
- Focus on fundamental decisions

**Common Mistake: Too Much Detail**
- Problem: Diving into implementation specifics
- Impact: Obscures big picture
- Fix: Save details for Sections 5-8
- Focus on strategy, not tactics

**Common Mistake: Missing Rationale**
- Problem: "We use microservices" without why
- Impact: Can't evaluate decision quality
- Fix: Always explain reasoning
- Document alternatives considered

### Section 5: Building Block View

**Tip: Use Hierarchical Levels**
- Level 1: System overview (3-7 components)
- Level 2: Decomposition of Level 1
- Level 3+: Further refinement as needed
- Don't try to show everything at once

**Tip: Black-Box Then White-Box**
- First show component as black-box (external view)
- Then show white-box (internal structure)
- Document interfaces and dependencies
- Show responsibilities clearly

**Tip: Map to Code**
- Show how components map to source code
- Package structure, modules, services
- Makes implementation clear
- Helps developers navigate codebase

**Common Mistake: Wrong Level of Abstraction**
- Problem: Mixing high-level and low-level
- Impact: Confusing, hard to understand
- Fix: Consistent abstraction per level
- Refine gradually through levels

**Common Mistake: Missing Interfaces**
- Problem: Components without defined interfaces
- Impact: Unclear dependencies and contracts
- Fix: Document provided and required interfaces
- Show protocols, data formats, APIs

### Section 6: Runtime View

**Tip: Select Key Scenarios**
- Don't document every use case
- Focus on architecturally significant
- Critical business processes
- Complex interactions
- Error handling patterns

**Tip: Use Sequence Diagrams**
- Show temporal ordering clearly
- Include components involved
- Show messages and data passed
- Number steps for clarity

**Tip: Show Error Handling**
- What happens when things fail?
- How are errors propagated?
- Recovery mechanisms
- Graceful degradation

**Common Mistake: Too Many Scenarios**
- Problem: Documenting every use case
- Impact: Documentation overload
- Fix: Focus on important/complex scenarios
- Reference specifications for complete list

**Common Mistake: Missing Error Paths**
- Problem: Only showing happy path
- Impact: Surprises when errors occur
- Fix: Document error and exception handling
- Show alternative flows

### Section 7: Deployment View

**Tip: Show Multiple Environments**
- Development environment
- Testing/staging environment
- Production environment
- Document differences

**Tip: Include Infrastructure**
- Hardware or VMs
- Network topology
- Load balancers, firewalls
- Storage systems
- Monitoring and logging infrastructure

**Tip: Document Scaling Strategy**
- How does system scale?
- Horizontal vs. vertical scaling
- Auto-scaling policies
- Capacity planning

**Common Mistake: Missing Cloud Details**
- Problem: "Deployed on AWS" without specifics
- Impact: Can't reproduce environment
- Fix: Document specific services used
- EC2/ECS, RDS, S3, CloudFront, etc.

**Common Mistake: Single Environment Only**
- Problem: Only documenting production
- Impact: Missing dev/test differences
- Fix: Show all environments
- Highlight differences

### Section 8: Crosscutting Concepts

**Tip: Document Once, Reference Everywhere**
- These are patterns used throughout
- Avoids repetition in other sections
- Ensures consistency
- Single source of truth

**Tip: Include Code Examples**
- Don't just describe patterns
- Show concrete code snippets
- Demonstrate usage
- Make it practical

**Tip: Organize by Category**
- Domain concepts
- Technical concepts (error handling, transactions)
- Architecture patterns
- Development concepts (testing, logging)
- Operational concepts (monitoring, deployment)
- UX concepts

**Common Mistake: Too Abstract**
- Problem: Describing patterns without examples
- Impact: Developers don't know how to apply
- Fix: Include code examples
- Show concrete implementations

**Common Mistake: Not Enforcing Consistency**
- Problem: Patterns documented but not followed
- Impact: Inconsistent codebase
- Fix: Code reviews enforce patterns
- Make patterns part of done

### Section 9: Architecture Decisions

**Tip: Use ADR Format**
- Standard Architecture Decision Record format
- Context, decision, consequences
- Status (proposed, accepted, deprecated)
- Links to detailed ADRs

**Tip: Document Alternatives**
- What else was considered?
- Why was it rejected?
- Trade-offs between options
- Shows thoughtful analysis

**Tip: Update When Revisited**
- Mark superseded decisions
- Link to replacement decision
- Explain why changed
- Maintain decision history

**Common Mistake: Missing "Why"**
- Problem: "We chose X" without rationale
- Impact: Can't evaluate or revisit
- Fix: Always document reasoning
- Explain context and constraints

**Common Mistake: Ignored Consequences**
- Problem: Only listing positive outcomes
- Impact: Surprises when negatives appear
- Fix: List both pros and cons
- Be honest about trade-offs

### Section 10: Quality Requirements

**Tip: Use Quality Scenarios**
- Stimulus (what happens)
- Environment (context)
- Response (system behavior)
- Measure (how to quantify)
- Makes requirements concrete and testable

**Tip: Build Quality Tree**
- Hierarchical breakdown of quality attributes
- Based on ISO/IEC 25010
- Shows relationships
- Prioritize branches

**Tip: Make Measurable**
- Define metrics for each quality
- Set target values
- Define acceptable ranges
- Specify measurement method

**Common Mistake: Vague Requirements**
- Problem: "System must be reliable"
- Impact: Can't verify or test
- Fix: Concrete scenarios with metrics
- "99.9% uptime with <2min recovery"

**Common Mistake: No Prioritization**
- Problem: All qualities marked "high priority"
- Impact: No trade-off guidance
- Fix: Ruthlessly prioritize
- Accept some qualities are lower priority

### Section 11: Risks and Technical Debt

**Tip: Use Risk Matrix**
- Probability (low, medium, high)
- Impact (low, medium, high, critical)
- Risk = Probability × Impact
- Prioritize high-risk items

**Tip: Plan Debt Paydown**
- Track all technical debt
- Estimate effort to fix
- Allocate time (e.g., 20% of sprint)
- Prevent debt from growing unchecked

**Tip: Review Regularly**
- Quarterly risk assessment
- Update as risks materialize or are mitigated
- Add new risks as discovered
- Close resolved items

**Common Mistake: Hiding Problems**
- Problem: Not documenting known issues
- Impact: Surprises and crises
- Fix: Be transparent about risks and debt
- Better to manage than ignore

**Common Mistake: No Mitigation Plans**
- Problem: Listing risks without strategy
- Impact: Risks become reality
- Fix: Document mitigation for each risk
- Assign owners and timelines

### Section 12: Glossary

**Tip: Start Early**
- Begin glossary from project start
- Add terms as they emerge
- Don't wait until end
- Easier to maintain incrementally

**Tip: Include Context**
- Where is term used?
- Relationships to other terms
- Examples when helpful
- Aliases or synonyms

**Tip: Make Searchable**
- Alphabetical order
- Consider categorization (domain vs. technical)
- Use consistent formatting
- Link from other sections

**Common Mistake: Circular Definitions**
- Problem: "A is a type of B" and "B uses A"
- Impact: Confusion
- Fix: Clear, self-contained definitions
- Use examples to clarify

**Common Mistake: Missing Obvious Terms**
- Problem: Assuming everyone knows terms
- Impact: New team members confused
- Fix: Define all domain-specific terms
- What's obvious to you isn't to everyone

## Process and Workflow Tips

### Getting Started

**Tip: Use This Order**
1. Section 1 (Goals) - Establish why
2. Section 2 (Constraints) - Understand limits
3. Section 3 (Context) - Define boundaries
4. Section 4 (Solution) - Outline approach
5. Section 5 (Building Blocks) - Show structure
6. Fill other sections as needed

**Tip: Time-Box Initial Draft**
- Don't aim for perfection
- Set time limit (e.g., 1 day per section)
- Get something down
- Iterate and improve

**Tip: Review with Team**
- Architecture docs are team effort
- Get feedback early and often
- Multiple perspectives improve quality
- Builds shared understanding

### Maintaining Documentation

**Tip: Update with Changes**
- When making architectural decision, update Section 9
- When adding component, update Section 5
- When changing deployment, update Section 7
- Make docs part of "done"

**Tip: Quarterly Reviews**
- Schedule regular documentation review
- Check for outdated information
- Update dates
- Add new sections as needed

**Tip: Link to Code**
- Reference actual implementations
- Show where patterns are used
- Link to key files or packages
- Makes docs verifiable

### Tools and Integration

**Tip: Automate Diagram Generation**
- Generate from code (Overarch, PlantUML)
- Reduce manual maintenance
- Keep diagrams in sync with code
- Version control diagram sources

**Tip: Integrate with ADRs**
- Link decisions from Section 9
- Reference constraints in ADRs
- Cross-reference quality goals
- Build knowledge graph

**Tip: Use Version Control**
- Track documentation changes
- See evolution over time
- Branch for major doc updates
- Review doc changes like code

## Quality Assurance

### Documentation Reviews

**Tip: Review Checklist**
- Is it current? (check dates)
- Are diagrams accurate?
- Do links work?
- Is terminology consistent?
- Are examples up to date?
- Does it match code?

**Tip: Involve Stakeholders**
- Business reviews Sections 1, 3, 4
- Developers review Sections 5, 6, 8
- Operations reviews Section 7
- Security reviews relevant sections
- Get diverse feedback

**Tip: Test Documentation**
- Can new team member understand system?
- Can you find information quickly?
- Are diagrams clear?
- Do examples work?
- Onboarding is the test

### Common Overall Mistakes

**Mistake: Big Design Up Front**
- Problem: Trying to complete perfectly at start
- Impact: Wasted effort, out of date immediately
- Fix: Iterative approach
- Document what you know, refine as you learn

**Mistake: Write Once, Forget**
- Problem: Creating docs but never updating
- Impact: Docs diverge from reality
- Fix: Make updates part of workflow
- Review regularly

**Mistake: Solo Effort**
- Problem: One person owns all documentation
- Impact: Limited perspective, bottleneck
- Fix: Team responsibility
- Rotate documentation tasks

**Mistake: Wrong Audience**
- Problem: Too technical for business, too vague for developers
- Impact: Docs don't serve anyone well
- Fix: Tailor sections to audience
- Use appropriate language and detail

**Mistake: Missing Diagrams**
- Problem: Wall of text without visuals
- Impact: Hard to understand
- Fix: Visual first approach
- Diagram per major concept

**Mistake: Too Much Detail**
- Problem: Documenting implementation minutiae
- Impact: Can't see forest for trees
- Fix: Appropriate abstraction level
- Link to code for details

**Mistake: No Standards**
- Problem: Inconsistent notation and terminology
- Impact: Confusion and misunderstanding
- Fix: Choose standards (UML, C4, etc.)
- Use glossary consistently

## Success Patterns

### What Good arc42 Documentation Looks Like

**Well-Structured**
- Clear navigation
- Logical organization
- Consistent formatting
- Easy to find information

**Visual**
- Diagrams illustrate concepts
- Standard notation
- Appropriate detail level
- Complemented by text

**Current**
- Reflects actual architecture
- Updated dates visible
- Obsolete content removed or marked
- Matches codebase

**Linked**
- Cross-references between sections
- Links to ADRs, specs, code
- Builds knowledge graph
- Easy to explore

**Practical**
- Code examples included
- Concrete scenarios
- Actionable guidance
- Helps developers

**Collaborative**
- Multiple authors
- Regular reviews
- Feedback incorporated
- Shared ownership

### Metrics for Good Documentation

**Findability**
- Can you find information in <5 minutes?
- Clear table of contents
- Search works well
- Logical organization

**Accuracy**
- Does it match reality?
- Are diagrams current?
- Do examples work?
- When last updated?

**Completeness**
- Are all sections addressed?
- Key decisions documented?
- Major components covered?
- Appropriate depth?

**Usability**
- Can new team member understand?
- Do stakeholders use it?
- Is it referenced in discussions?
- Does onboarding use it?

**Maintainability**
- How hard to update?
- Are diagrams editable?
- Clear ownership?
- Part of workflow?

## Integration Tips

### With Agile Development

**Tip: Just Enough Documentation**
- Document what's needed now
- Defer details until needed
- Iterative refinement
- Working software over documentation (but documentation is valuable)

**Tip: Document in Sprints**
- Allocate time for documentation
- Update docs with features
- Part of definition of done
- Review in retrospectives

**Tip: Living Documentation**
- Continuous updating
- Not a phase or milestone
- Embedded in workflow
- Team responsibility

### With Other Artifacts

**Tip: Link to Requirements**
- Section 1 links to detailed requirements
- Quality goals drive quality requirements
- Traceability from requirements to architecture

**Tip: Link to ADRs**
- Section 9 summarizes, ADRs have details
- Cross-reference decisions and constraints
- Build decision trail

**Tip: Link to Tests**
- Quality scenarios link to test suites
- Runtime scenarios reference integration tests
- Verify architecture through tests

**Tip: Link to Code**
- Building blocks map to source structure
- Examples show actual code
- Keep documentation close to implementation

## Final Tips

**Start Today**
- Don't wait for perfect moment
- Begin with what you know
- Iterate and improve
- Something is better than nothing

**Keep It Simple**
- Don't overcomplicate
- Use plain language
- Standard notation
- Clear diagrams

**Make It Accessible**
- Easy to find
- Easy to navigate
- Easy to understand
- Easy to update

**Involve the Team**
- Shared ownership
- Regular reviews
- Diverse perspectives
- Collective knowledge

**Adapt to Context**
- One size doesn't fit all
- Tailor to project needs
- Choose appropriate depth
- Focus on value

**Maintain Continuously**
- Not a one-time effort
- Part of development process
- Regular updates
- Living documentation

Remember: Perfect is the enemy of good. Start documenting, iterate, and improve over time. The goal is useful documentation that helps the team and stakeholders understand and evolve the system.
