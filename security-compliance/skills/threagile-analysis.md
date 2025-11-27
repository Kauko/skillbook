---
description: Use when performing threat modeling or security analysis. Runs Threagile to identify risks and required mitigations.
---

# Threagile Threat Modeling Analysis

## Reference Documentation

Detailed reference documentation is available:

- **[Model Schema Reference](threagile-analysis/references/model-schema.md)**: Complete YAML schema for threat models including all fields, enumerations, and data structures
- **[Risk Rules Reference](threagile-analysis/references/risk-rules.md)**: Built-in risk detection rules organized by STRIDE categories with detection logic and mitigations
- **[Outputs Reference](threagile-analysis/references/outputs.md)**: All output formats (PDF, JSON, Excel, diagrams) with integration examples

Use these references when creating or modifying threat models, understanding risk findings, or integrating Threagile outputs.

## Prerequisites Check

Before starting threat analysis, verify Threagile is available:

1. Check for Threagile CLI:
   ```bash
   which threagile
   ```

2. If not found, guide installation:
   ```bash
   # macOS
   brew install threagile

   # Linux
   wget https://github.com/threagile/threagile/releases/latest/download/threagile-linux-amd64
   chmod +x threagile-linux-amd64
   sudo mv threagile-linux-amd64 /usr/local/bin/threagile

   # Windows
   # Download from: https://github.com/threagile/threagile/releases/latest
   ```

3. Verify installation:
   ```bash
   threagile version
   ```

## Threagile Model Location

Check for existing Threagile model:

1. Look for `vault/security/threagile.yaml`
2. If missing, check if Overarch model exists in `models/`
3. If Overarch model exists, suggest translating it to Threagile format:
   - Map Overarch components to Threagile technical assets
   - Map Overarch relations to Threagile data flows
   - Map Overarch trust boundaries to Threagile trust boundaries

## Model Freshness Check

If both Overarch and Threagile models exist:

1. Compare modification times:
   ```bash
   stat -f "%m %N" models/*.edn vault/security/threagile.yaml
   ```

2. If Overarch is newer, WARN user:
   ```
   ⚠️  WARNING: Architecture models are newer than threat model

   The Overarch models in models/ were modified more recently than
   vault/security/threagile.yaml. The threat model may be out of sync.

   Consider running sync-architecture to update the threat model.
   ```

## Running Threagile Analysis

Execute Threagile threat analysis:

```bash
cd vault/security
threagile analyze-model --model threagile.yaml --output threat-analysis
```

This generates multiple output files (see [Outputs Reference](threagile-analysis/references/outputs.md) for details):
- `threat-analysis/report.pdf` - Comprehensive PDF report
- `threat-analysis/risks.json` - Machine-readable risk data
- `threat-analysis/risks.xlsx` - Excel spreadsheet for tracking
- `threat-analysis/tags.xlsx` - Tag-based risk grouping
- `threat-analysis/technical-assets.json` - Asset inventory
- `threat-analysis/stats.json` - Analysis statistics
- `threat-analysis/data-flow-diagram.png` - Data flow visualization
- `threat-analysis/data-asset-diagram.png` - Asset relationships

## Parse and Report Findings

Parse the risk output and categorize by severity. See [Risk Rules Reference](threagile-analysis/references/risk-rules.md) for detailed information about each risk category and [Outputs Reference](threagile-analysis/references/outputs.md) for output format details.

1. Extract risks from `risks.json`:
   ```bash
   jq '.risks | group_by(.severity)' threat-analysis/risks.json
   ```

2. For each risk, extract:
   - **Risk ID**: Unique identifier
   - **Category**: Type of threat (see STRIDE categories in Risk Rules Reference)
   - **Severity**: Critical, High, Elevated, Medium, Low
   - **Title**: Short description
   - **Affected Asset**: Which component is at risk
   - **Impact**: What could happen
   - **Likelihood**: Probability of exploitation
   - **Mitigation**: Required countermeasures
   - **ASVS/CWE References**: Standards mapping

3. Report findings in structured format:

```
CRITICAL RISKS (Immediate Action Required):
- [RISK-001] SQL Injection in User API
  Component: user-service
  Impact: Data breach, unauthorized access
  Mitigation: Implement parameterized queries, input validation
  Reference: CWE-89, ASVS V5.3.4

HIGH RISKS (Address Soon):
- [RISK-002] Unencrypted Data in Transit
  Component: api-gateway → backend-service
  Impact: Data interception, man-in-the-middle attacks
  Mitigation: Enforce TLS 1.3, implement certificate pinning
  Reference: CWE-319, ASVS V9.1.1

MEDIUM RISKS:
- [RISK-003] Missing Rate Limiting
  Component: public-api
  Impact: DoS attacks, resource exhaustion
  Mitigation: Implement rate limiting, request throttling
  Reference: CWE-770, ASVS V11.1.4

LOW RISKS:
- [RISK-004] Verbose Error Messages
  Component: api-gateway
  Impact: Information disclosure
  Mitigation: Generic error responses, detailed logging backend-only
  Reference: CWE-209, ASVS V7.4.1
```

## Generate Threat Model Documentation

Create comprehensive documentation at `vault/security/threat-model.md`:

```markdown
# Threat Model

Generated: [TIMESTAMP]
Source: vault/security/threagile.yaml
Analysis: Threagile v[VERSION]

## Executive Summary

[Summarize overall security posture]
- Total Risks Identified: [COUNT]
- Critical: [COUNT] | High: [COUNT] | Medium: [COUNT] | Low: [COUNT]
- Most Affected Components: [LIST]
- Priority Mitigations: [TOP 3]

## Architecture Overview

[Embedded data flow diagram]
![Data Flow](threat-analysis/data-flow-diagram.png)

## Risk Analysis

### Critical Risks

#### [RISK-ID]: [Title]

**Affected Component**: [[component-id]]
**Threat Category**: [STRIDE category]
**Attack Vector**: [How the attack could occur]

**Impact**:
- Confidentiality: [High/Medium/Low]
- Integrity: [High/Medium/Low]
- Availability: [High/Medium/Low]

**Likelihood**: [High/Medium/Low]
**Risk Score**: [Calculated score]

**Mitigation Strategy**:
1. [Action item 1]
2. [Action item 2]
3. [Action item 3]

**Verification**:
- [ ] Mitigation implemented
- [ ] Security testing completed
- [ ] Policy enforcement configured

**References**:
- CWE: [CWE-XXX](https://cwe.mitre.org/data/definitions/XXX.html)
- ASVS: V[X.Y.Z]
- Related ADR: [[adr-XXX]]

---

[Repeat for each risk, grouped by severity]

## Component Risk Matrix

| Component | Critical | High | Medium | Low | Total |
|-----------|----------|------|--------|-----|-------|
| [component-1] | X | X | X | X | Total |
| [component-2] | X | X | X | X | Total |

## Mitigation Roadmap

### Phase 1: Critical Risks (Immediate)
- [ ] [RISK-001]: [Short description]
- [ ] [RISK-002]: [Short description]

### Phase 2: High Risks (Sprint 1-2)
- [ ] [RISK-003]: [Short description]
- [ ] [RISK-004]: [Short description]

### Phase 3: Medium Risks (Next Quarter)
- [ ] [RISK-005]: [Short description]

### Phase 4: Low Risks (Backlog)
- [ ] [RISK-006]: [Short description]

## Compliance Mapping

### OWASP ASVS
- V5.3.4: SQL Injection Prevention → [RISK-001]
- V9.1.1: Data in Transit → [RISK-002]

### NIST CSF
- PR.DS-2: Data at rest protection → [RISK-007]
- DE.CM-1: Network monitoring → [RISK-008]

## Security Decisions Required

Based on this analysis, the following Architecture Decision Records are recommended:

- **ADR: API Authentication Strategy** - To address [RISK-001, RISK-003]
- **ADR: Encryption Standards** - To address [RISK-002, RISK-007]
- **ADR: Logging and Monitoring** - To address [RISK-008, RISK-009]

Create these ADRs to document security decisions and rationale.
```

## Link Risks to Architecture Components

For each risk, create bidirectional links:

1. In the threat model, link to architecture components:
   ```markdown
   **Affected Component**: [[components.user-service]]
   ```

2. Consider adding security notes to component documentation:
   ```markdown
   ## Security Considerations

   See [[threat-model#risk-001]] for SQL injection risks.
   Mitigation status: In Progress
   ```

## Suggest Architecture Decision Records

Identify security decisions that need documentation:

```
SUGGESTED ADRs:

1. ADR: Authentication and Authorization Strategy
   Rationale: Multiple risks (RISK-001, RISK-003, RISK-005) relate to
   access control. We need to document our approach to AuthN/AuthZ.

2. ADR: Data Encryption Standards
   Rationale: RISK-002 and RISK-007 require encryption decisions.
   Document which algorithms, key management approach, rotation policy.

3. ADR: Security Logging and Monitoring
   Rationale: Detection capabilities needed for RISK-008, RISK-009.
   Define what to log, retention, alerting thresholds.

Create these with: adr-new "Title" in vault/decisions/
```

## Embed Risk Diagrams

Copy generated diagrams to vault for documentation:

```bash
# Copy diagrams
cp threat-analysis/data-flow-diagram.png vault/security/images/
cp threat-analysis/data-asset-diagram.png vault/security/images/

# Update paths in threat-model.md to use vault-relative paths
sed -i 's|threat-analysis/|images/|g' vault/security/threat-model.md
```

## Next Steps Recommendations

After completing threat analysis:

1. **Review with Team**: Schedule threat model review session
2. **Prioritize Mitigations**: Assign risks to sprints/epics
3. **Generate Policies**: Run `policy-as-code` skill to create OPA policies
4. **Track Progress**: Add mitigation tasks to project board
5. **Schedule Re-analysis**: Set reminder to re-run after architecture changes

## Output Summary

Always provide a summary of the analysis:

```
✓ Threat Analysis Complete

Files Generated:
- vault/security/threat-model.md (Comprehensive documentation)
- vault/security/threat-analysis/report.html (Interactive report)
- vault/security/images/data-flow-diagram.png (Architecture visualization)

Findings:
- Critical Risks: [COUNT] - Immediate action required
- High Risks: [COUNT] - Address in current sprint
- Medium Risks: [COUNT] - Plan for next quarter
- Low Risks: [COUNT] - Backlog

Priority Actions:
1. [Most critical finding]
2. [Second critical finding]
3. [Third critical finding]

Next Steps:
- Review threat-model.md with security team
- Create ADRs for security decisions
- Run policy-as-code skill to generate enforcement policies
- Schedule follow-up analysis after mitigation
```

## Common Threagile Model Patterns

When creating or reviewing `threagile.yaml`, ensure it includes proper structure. See [Model Schema Reference](threagile-analysis/references/model-schema.md) for complete field definitions.

### Model Structure Overview

```yaml
threagile_version: "1.0.0"
title: "Application Name"
date: "YYYY-MM-DD"
business_criticality: "important"

data_assets:
  user-credentials:
    id: "user-credentials"
    description: "User passwords and authentication tokens"
    usage: "business"
    quantity: "many"
    confidentiality: "strictly-confidential"
    integrity: "critical"
    availability: "critical"
    justification_cia_rating: "Contains authentication secrets"

technical_assets:
  user-service:
    id: "user-service"
    description: "User management service"
    type: "process"
    usage: "business"
    size: "service"
    technology: "web-service-rest"
    internet: false
    machine: "container"
    encryption: "data-with-symmetric-shared-key"
    confidentiality: "confidential"
    integrity: "critical"
    availability: "important"
    justification_cia_rating: "Handles user credentials"
    custom_developed_parts: true
    data_assets_processed:
      - "user-credentials"
    communication_links:
      database:
        target: "user-database"
        description: "Database access"
        protocol: "jdbc-encrypted"
        authentication: "credentials"
        authorization: "technical-user"
        usage: "business"
        data_assets_sent:
          - "user-credentials"

trust_boundaries:
  internal:
    id: "internal"
    description: "Internal network"
    type: "network-on-prem"
    technical_assets_inside:
      - "user-service"
      - "user-database"
```

**Key Points:**
- All assets require CIA (Confidentiality, Integrity, Availability) ratings with justification
- Communication links must specify protocol, authentication, and data flows
- Trust boundaries group assets by security domain
- See [Model Schema Reference](threagile-analysis/references/model-schema.md) for all available fields and enumerations

## Troubleshooting

### Threagile Parse Errors
- Validate YAML syntax: `yamllint vault/security/threagile.yaml`
- Check Threagile schema: `threagile check --model vault/security/threagile.yaml`

### Missing Risk Details
- Ensure all technical assets have `data_assets_processed`
- Define data flows between assets
- Specify trust boundaries

### Report Generation Fails
- Check output directory permissions
- Ensure sufficient disk space
- Verify Threagile version compatibility

## Best Practices

1. **Regular Updates**: Re-run threat analysis after significant architecture changes
2. **Version Control**: Track changes to threagile.yaml over time
3. **Team Review**: Don't analyze in isolation - involve developers and security team
4. **Action-Oriented**: Every identified risk should have a mitigation plan
5. **Traceability**: Link risks to requirements, components, and policies
6. **Continuous**: Threat modeling is ongoing, not one-time
7. **Documentation**: Keep threat-model.md up to date with current status
8. **Complete Models**: Use comprehensive model structure (see [Model Schema Reference](threagile-analysis/references/model-schema.md))
9. **Understand Risks**: Review risk categories and detection logic (see [Risk Rules Reference](threagile-analysis/references/risk-rules.md))
10. **Integrate Outputs**: Leverage all output formats for different audiences (see [Outputs Reference](threagile-analysis/references/outputs.md))
