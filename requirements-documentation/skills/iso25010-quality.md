---
name: iso25010-quality
description: Use when user needs to define non-functional requirements, quality attributes, or measurable quality scenarios for a system.
requires:
  tools: []
  skills: [init-obsidian-vault]
---

# ISO 25010 Quality Requirements

## Prerequisites

```bash
[ -d vault/requirements ] || { echo "Run init-obsidian-vault first"; exit 1; }
```

### 2. Interview Stakeholders

Ask for top 3-5 quality priorities from:
- Performance (response time, throughput)
- Security (authentication, data protection)
- Reliability (uptime, fault tolerance)
- Maintainability (testability, modularity)
- Usability (learnability, accessibility)

### 3. Create Quality Scenarios

For each priority, create a scenario in `vault/requirements/quality/`:

```markdown
# QS-001: API Response Time

**Characteristic**: Performance Efficiency > Time Behavior

**Scenario**:
- Source: End user
- Stimulus: API request
- Environment: Normal load (100 concurrent users)
- Response: System returns result
- Measure: p95 < 200ms, p99 < 500ms

**Priority**: HIGH
**Rationale**: User satisfaction, SLA requirement
```

### 4. Define Metrics

| Characteristic | Metric | Target | Monitoring |
|----------------|--------|--------|------------|
| Availability | Uptime | 99.9% | Health checks |
| Performance | p95 latency | < 200ms | APM |
| Security | Auth failures | < 1% | Audit logs |

### 5. Link to Architecture

Reference quality scenarios in:
- `vault/arc42/10-quality.md`
- ADRs that address quality requirements
- Threat model for security requirements

## Quality Model Overview

```
ISO 25010 Product Quality
├── Functional Suitability (completeness, correctness)
├── Performance Efficiency (time, resources, capacity)
├── Compatibility (co-existence, interoperability)
├── Usability (learnability, operability, accessibility)
├── Reliability (availability, fault tolerance, recoverability)
├── Security (confidentiality, integrity, authenticity)
├── Maintainability (modularity, testability, modifiability)
└── Portability (adaptability, installability)
```

## Reference Documentation

- `references/characteristics.md` - Complete characteristic definitions and examples

## Success Criteria

- [ ] Quality scenarios created in `vault/requirements/quality/`
- [ ] Metrics defined with measurable targets
- [ ] Linked to arc42 section 10

## Linking Requirements

When defining quality requirements:
- Link to arc42 section 10: `See [[arc42/10-quality]]`
- Link to related ADRs: `Decision: [[decisions/0007-performance-target]]`
- Link to SLOs if defined: `SLO: [[requirements/slo-response-time]]`

## Related Skills

- `arc42-docs` - Section 10 uses quality scenarios
- `scenari-specs` - Behavioral specs for quality testing
- `threagile-analysis` - Security quality requirements
