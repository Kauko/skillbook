# Threagile Model Schema Reference

This document provides comprehensive reference for the Threagile YAML model schema. The model defines your application's architecture, assets, trust boundaries, and data flows for threat analysis.

## Model File Structure

```yaml
threagile_version: "1.0.0"
title: "Application Name"
date: "YYYY-MM-DD"
author:
  name: "Author Name"
  homepage: "www.example.com"
business_criticality: "important"  # See criticality levels below
management_summary_comment: "Executive summary (supports HTML)"

business_overview:
  description: "Business context (supports HTML)"
  images:
    - filename: "image-description"

technical_overview:
  description: "Technical architecture (supports HTML)"
  images:
    - filename: "image-description"

questions: {}
abuse_cases: {}
security_requirements: {}

data_assets: {}
technical_assets: {}
trust_boundaries: {}
shared_runtimes: {}
individual_risk_categories: {}
risk_tracking: {}
```

## Business Criticality Levels

Defines the importance of the system to business operations:

- **archive**: Historical data, minimal business impact
- **operational**: Day-to-day operations affected
- **important**: Significant business impact
- **critical**: Major business disruption
- **mission-critical**: Business cannot function without it

## Questions and Abuse Cases

### Questions
Document security-relevant questions and their answers:

```yaml
questions:
  "How is authentication handled?": "OAuth 2.0 with JWT tokens"
  "What data retention policies apply?": "90 days for logs, 7 years for transactions"
```

### Abuse Cases
Define threat scenarios using attacker perspective:

```yaml
abuse_cases:
  "Credential Theft": "As an attacker I want to steal user credentials in order to access their accounts"
  "Data Exfiltration": "As an insider I want to export customer data in order to sell it"
```

## Data Assets

Data assets represent information that flows through or is stored by your system.

```yaml
data_assets:
  user-credentials:
    id: "user-credentials"
    description: "User passwords and authentication tokens"
    usage: "business"  # business | devops
    tags:
      - "authentication"
      - "pii"
    origin: "User registration"
    owner: "Security Team"
    quantity: "many"  # See quantity levels
    confidentiality: "strictly-confidential"  # See CIA levels
    integrity: "critical"
    availability: "important"
    justification_cia_rating: "Contains authentication secrets; compromise enables account takeover"
```

### Data Asset Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | Yes | Unique identifier (kebab-case) |
| `description` | string | Yes | Purpose and content description |
| `usage` | enum | Yes | `business` or `devops` |
| `tags` | list | No | Categorization tags |
| `origin` | string | No | Where this data comes from |
| `owner` | string | No | Team/person responsible |
| `quantity` | enum | Yes | Data volume (see below) |
| `confidentiality` | enum | Yes | Confidentiality level (see below) |
| `integrity` | enum | Yes | Integrity criticality (see below) |
| `availability` | enum | Yes | Availability criticality (see below) |
| `justification_cia_rating` | string | Yes | Rationale for CIA ratings |

### Quantity Levels

- **very-few**: < 100 records
- **few**: 100 - 10,000 records
- **many**: 10,000 - 1,000,000 records
- **very-many**: > 1,000,000 records

### Confidentiality Levels

- **public**: Publicly accessible information
- **internal**: Internal use only
- **restricted**: Limited access required
- **confidential**: Sensitive information
- **strictly-confidential**: Highly sensitive (credentials, keys, PII)

### Integrity and Availability Levels

- **archive**: Historical data, can tolerate loss/corruption
- **operational**: Day-to-day operations affected
- **important**: Significant impact if compromised
- **critical**: Major impact if compromised
- **mission-critical**: Business cannot function if compromised

## Technical Assets

Technical assets represent system components (services, databases, applications).

```yaml
technical_assets:
  user-service:
    id: "user-service"
    description: "User management and authentication service"
    type: "process"  # See asset types
    usage: "business"
    used_as_client_by_human: false
    out_of_scope: false
    justification_out_of_scope: ""
    size: "service"  # See size categories
    technology: "web-service-rest"  # See technologies
    tags:
      - "authentication"
      - "user-management"
    internet: false
    machine: "container"  # See machine types
    encryption: "data-with-symmetric-shared-key"  # See encryption types
    owner: "Backend Team"
    confidentiality: "confidential"
    integrity: "critical"
    availability: "important"
    justification_cia_rating: "Handles user credentials and session management"
    multi_tenant: false
    redundant: true
    custom_developed_parts: true
    data_assets_processed:
      - "user-credentials"
      - "user-profile-data"
    data_assets_stored:
      - "user-credentials"
    data_formats_accepted:
      - "json"
      - "xml"
    communication_links:
      user-api:
        target: "api-gateway"
        description: "Exposes user API endpoints"
        protocol: "https"
        authentication: "token"
        authorization: "end-user-identity-propagation"
        tags: []
        vpn: false
        ip_filtered: false
        readonly: false
        usage: "business"
        data_assets_sent:
          - "user-profile-data"
        data_assets_received:
          - "user-credentials"
      database-connection:
        target: "user-database"
        description: "Stores user data"
        protocol: "jdbc"
        authentication: "credentials"
        authorization: "technical-user"
        tags: []
        vpn: false
        ip_filtered: true
        readonly: false
        usage: "business"
        data_assets_sent:
          - "user-credentials"
        data_assets_received:
          - "user-credentials"
```

### Technical Asset Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | Yes | Unique identifier (kebab-case) |
| `description` | string | Yes | Purpose and functionality |
| `type` | enum | Yes | Asset type (see below) |
| `usage` | enum | Yes | `business` or `devops` |
| `used_as_client_by_human` | boolean | No | Human users interact directly |
| `out_of_scope` | boolean | No | Excluded from analysis |
| `justification_out_of_scope` | string | No | Why excluded |
| `size` | enum | Yes | System size (see below) |
| `technology` | enum | Yes | Technology stack (see below) |
| `tags` | list | No | Categorization tags |
| `internet` | boolean | Yes | Exposed to internet |
| `machine` | enum | Yes | Deployment type (see below) |
| `encryption` | enum | Yes | Encryption approach (see below) |
| `owner` | string | No | Responsible team |
| `confidentiality` | enum | Yes | Confidentiality level |
| `integrity` | enum | Yes | Integrity criticality |
| `availability` | enum | Yes | Availability criticality |
| `justification_cia_rating` | string | Yes | Rationale for CIA ratings |
| `multi_tenant` | boolean | No | Multi-tenant deployment |
| `redundant` | boolean | No | High availability setup |
| `custom_developed_parts` | boolean | Yes | Contains custom code |
| `data_assets_processed` | list | No | Data assets handled |
| `data_assets_stored` | list | No | Data assets persisted |
| `data_formats_accepted` | list | No | Input formats accepted |
| `communication_links` | map | No | Connections to other assets |

### Asset Types

- **external-entity**: External system/user outside your control
- **process**: Active processing component (service, application)
- **datastore**: Data storage component (database, file system)

### Size Categories

- **system**: Entire system/platform
- **service**: Independent service
- **application**: Standalone application
- **component**: Component within larger system

### Technology Types

**Web Technologies:**
- `browser`
- `web-server`
- `web-service-rest`
- `web-service-soap`
- `web-application`

**Data Storage:**
- `database`
- `file-server`
- `local-file-system`
- `ldap`

**Infrastructure:**
- `load-balancer`
- `reverse-proxy`
- `waf`
- `ids-ips`
- `service-mesh`
- `message-queue`
- `container-platform`
- `build-pipeline`
- `sourcecode-repository`
- `artifact-registry`
- `vault`
- `hsm`

**Monitoring & Management:**
- `monitoring`
- `log-server`
- `identity-provider`
- `identity-store-ldap`
- `identity-store-database`
- `erp`
- `cms`
- `gateway`
- `iot-device`
- `client-system`
- `tool`
- `cli`
- `task`
- `function`
- `library`
- `scheduler`
- `mainframe`
- `block-storage`
- `mail-server`

### Machine Types

- **physical**: Physical hardware
- **virtual**: Virtual machine
- **container**: Container (Docker, etc.)
- **serverless**: Serverless/function-as-a-service

### Encryption Types

- **none**: No encryption
- **transparent**: Transparent encryption (disk/network level)
- **data-with-symmetric-shared-key**: Symmetric encryption with shared key
- **data-with-asymmetric-shared-key**: Asymmetric encryption with shared key
- **data-with-end-user-individual-key**: End-to-end encryption with user keys

### Communication Link Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `target` | string | Yes | Target asset ID |
| `description` | string | Yes | Purpose of communication |
| `protocol` | enum | Yes | Communication protocol (see below) |
| `authentication` | enum | Yes | Authentication method (see below) |
| `authorization` | enum | Yes | Authorization approach (see below) |
| `tags` | list | No | Categorization tags |
| `vpn` | boolean | No | Uses VPN |
| `ip_filtered` | boolean | No | IP filtering enabled |
| `readonly` | boolean | No | Read-only access |
| `usage` | enum | Yes | `business` or `devops` |
| `data_assets_sent` | list | No | Data sent through link |
| `data_assets_received` | list | No | Data received through link |

### Protocol Types

**Secure Protocols:**
- `https`
- `ssh`
- `sftp`
- `ldaps`
- `smtp-tls`
- `pop3-tls`
- `imap-tls`

**Insecure Protocols:**
- `http`
- `ftp`
- `telnet`
- `ldap`
- `smtp`
- `pop3`
- `imap`

**Database & Messaging:**
- `jdbc`
- `jdbc-encrypted`
- `odbc`
- `odbc-encrypted`
- `sql-access-protocol`
- `sql-access-protocol-encrypted`
- `nosql-access-protocol`
- `nosql-access-protocol-encrypted`
- `mqtt`
- `mqtt-encrypted`
- `amqp`
- `amqp-encrypted`

**Other:**
- `binary`
- `binary-encrypted`
- `text`
- `text-encrypted`
- `iiop`
- `iiop-encrypted`
- `jrmp`
- `jrmp-encrypted`
- `in-process-library-call`
- `local-file-access`
- `nfs`
- `smb`
- `container-spawning`

### Authentication Types

- **none**: No authentication
- **credentials**: Username/password
- **session-id**: Session identifier
- **token**: Token-based (JWT, OAuth)
- **client-certificate**: TLS client certificate
- **two-factor**: Multi-factor authentication

### Authorization Types

- **none**: No authorization
- **technical-user**: Service account/technical user
- **end-user-identity-propagation**: End-user identity passed through

## Trust Boundaries

Trust boundaries group assets by security domain or network zone.

```yaml
trust_boundaries:
  internet:
    id: "internet"
    description: "Public internet"
    type: "network-cloud-provider"
    tags: []
    technical_assets_inside:
      - "cdn"
      - "api-gateway"
    trust_boundaries_nested: []

  dmz:
    id: "dmz"
    description: "Demilitarized zone"
    type: "network-on-prem"
    tags: []
    technical_assets_inside:
      - "web-server"
      - "load-balancer"
    trust_boundaries_nested: []

  internal:
    id: "internal"
    description: "Internal corporate network"
    type: "network-on-prem"
    tags: []
    technical_assets_inside:
      - "user-service"
      - "database"
    trust_boundaries_nested:
      - "database-zone"
```

### Trust Boundary Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | Yes | Unique identifier |
| `description` | string | Yes | Purpose and scope |
| `type` | enum | Yes | Boundary type (see below) |
| `tags` | list | No | Categorization tags |
| `technical_assets_inside` | list | Yes | Assets within boundary |
| `trust_boundaries_nested` | list | No | Nested boundaries |

### Trust Boundary Types

- **network-cloud-security-group**: Cloud security group
- **network-cloud-provider**: Cloud provider boundary
- **network-on-prem**: On-premises network
- **execution-environment**: Runtime/execution boundary

## Shared Runtimes

Shared runtimes group assets running in the same execution environment.

```yaml
shared_runtimes:
  kubernetes-cluster:
    id: "kubernetes-cluster"
    description: "Production Kubernetes cluster"
    tags:
      - "production"
      - "kubernetes"
    technical_assets_running:
      - "user-service"
      - "api-gateway"
      - "order-service"
      - "payment-service"
```

### Shared Runtime Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | Yes | Unique identifier |
| `description` | string | Yes | Purpose and details |
| `tags` | list | No | Categorization tags |
| `technical_assets_running` | list | Yes | Assets in runtime |

## Individual Risk Categories

Define custom risk categories beyond built-in rules.

```yaml
individual_risk_categories:
  gdpr-compliance-risk:
    id: "gdpr-compliance-risk"
    description: "GDPR compliance requirements not met"
    impact: "Regulatory fines and legal consequences"
    asvs: "V1.2.1"
    cheat_sheet: "https://cheatsheetseries.owasp.org/cheatsheets/GDPR_Cheat_Sheet.html"
    action: "Implement GDPR compliance controls"
    mitigation: "Add consent management, data retention policies, and right-to-erasure"
    check: "Verify GDPR requirements implementation"
    function: "business-side"
    stride: "information-disclosure"
    detection_logic: "Check for PII handling without consent management"
    risk_assessment: "Assess GDPR compliance for data assets"
    false_positives: "Non-EU customers may not require GDPR compliance"
    model_failure_possible_reason: false
    cwe: 359
    risks_identified:
      user-service-gdpr:
        severity: "high"
        exploitation_likelihood: "likely"
        exploitation_impact: "high"
        data_breach_probability: "probable"
        data_breach_technical_assets:
          - "user-service"
        most_relevant_data_asset: "user-profile-data"
        most_relevant_technical_asset: "user-service"
        most_relevant_communication_link: ""
        most_relevant_trust_boundary: ""
        most_relevant_shared_runtime: ""
```

### Individual Risk Category Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | Yes | Unique identifier |
| `description` | string | Yes | Risk description |
| `impact` | string | Yes | Potential impact |
| `asvs` | string | No | ASVS reference |
| `cheat_sheet` | string | No | Reference URL |
| `action` | string | Yes | Required action |
| `mitigation` | string | Yes | Mitigation approach |
| `check` | string | Yes | Verification method |
| `function` | enum | Yes | Function category (see below) |
| `stride` | enum | Yes | STRIDE category (see below) |
| `detection_logic` | string | Yes | How to detect |
| `risk_assessment` | string | Yes | Assessment approach |
| `false_positives` | string | No | Common false positives |
| `model_failure_possible_reason` | boolean | No | Model may be incomplete |
| `cwe` | integer | No | CWE identifier |
| `risks_identified` | map | Yes | Identified risk instances |

### Function Categories

- **business-side**: Business logic layer
- **architecture**: Architectural decisions
- **development**: Development practices
- **operations**: Operations and infrastructure

### STRIDE Categories

- **spoofing**: Identity spoofing
- **tampering**: Data tampering
- **repudiation**: Action repudiation
- **information-disclosure**: Information disclosure
- **denial-of-service**: Denial of service
- **elevation-of-privilege**: Privilege escalation

### Risk Instance Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `severity` | enum | Yes | Risk severity (see below) |
| `exploitation_likelihood` | enum | Yes | Likelihood (see below) |
| `exploitation_impact` | enum | Yes | Impact level (see below) |
| `data_breach_probability` | enum | Yes | Breach probability (see below) |
| `data_breach_technical_assets` | list | No | Assets at risk |
| `most_relevant_data_asset` | string | No | Most relevant data asset |
| `most_relevant_technical_asset` | string | No | Most relevant technical asset |
| `most_relevant_communication_link` | string | No | Most relevant link |
| `most_relevant_trust_boundary` | string | No | Most relevant boundary |
| `most_relevant_shared_runtime` | string | No | Most relevant runtime |

### Severity Levels

- **low**: Minor impact, easily mitigated
- **medium**: Moderate impact, requires attention
- **elevated**: Significant impact, priority mitigation
- **high**: Serious impact, urgent mitigation
- **critical**: Severe impact, immediate action required

### Exploitation Likelihood

- **unlikely**: Difficult to exploit
- **likely**: Can be exploited with effort
- **very-likely**: Easily exploitable
- **frequent**: Actively exploited

### Exploitation Impact

- **low**: Minor consequences
- **medium**: Moderate consequences
- **high**: Significant consequences
- **very-high**: Severe consequences

### Data Breach Probability

- **improbable**: Very unlikely to cause breach
- **possible**: Could potentially cause breach
- **probable**: Likely to cause breach

## Risk Tracking

Track identified risks and their mitigation status.

```yaml
risk_tracking:
  "sql-nosql-injection@user-service":
    status: "in-progress"
    justification: "Implementing parameterized queries"
    ticket: "SEC-123"
    date: "2024-01-15"
    checked_by: "Security Team"

  "unencrypted-communication@*@user-database":
    status: "mitigated"
    justification: "Enabled TLS 1.3 for all database connections"
    ticket: "SEC-124"
    date: "2024-01-20"
    checked_by: "Security Team"

  "missing-authentication@*@legacy-api@*":
    status: "accepted"
    justification: "Legacy API scheduled for decommission Q2 2024"
    ticket: "TECH-456"
    date: "2024-01-10"
    checked_by: "Architecture Team"
```

### Risk Tracking Fields

Risk tracking keys use the format: `risk-id@element[@*][@element]`
- Wildcards (`*`) match any value
- Example: `sql-injection@*@user-service@*` matches all SQL injection risks for user-service

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `status` | enum | Yes | Current status (see below) |
| `justification` | string | Yes | Status rationale |
| `ticket` | string | No | Tracking ticket ID |
| `date` | string | Yes | Status date (YYYY-MM-DD) |
| `checked_by` | string | Yes | Person who verified |

### Risk Tracking Status

- **unchecked**: Not yet reviewed
- **in-discussion**: Under review/discussion
- **accepted**: Risk accepted (won't fix)
- **in-progress**: Mitigation in progress
- **mitigated**: Risk mitigated
- **false-positive**: Not a real risk

## Best Practices

### Model Organization

1. **Use Descriptive IDs**: Use kebab-case, descriptive identifiers
2. **Complete Documentation**: Fill all description fields thoroughly
3. **Accurate CIA Ratings**: Carefully assess confidentiality, integrity, availability
4. **Explicit Data Flows**: Document all data assets in communication links
5. **Proper Trust Boundaries**: Group assets by security domains

### Data Asset Modeling

1. **Granular Assets**: Create separate assets for different data types
2. **Realistic Quantities**: Estimate data volumes accurately
3. **Justify Ratings**: Provide clear rationale for CIA ratings
4. **Track Ownership**: Assign clear data ownership

### Technical Asset Modeling

1. **Complete Links**: Document all communication links
2. **Technology Accuracy**: Use correct technology types
3. **Encryption Details**: Specify encryption for all links
4. **Authentication**: Document authentication/authorization

### Risk Management

1. **Track All Risks**: Don't ignore any identified risks
2. **Justify Acceptance**: Provide clear rationale for accepted risks
3. **Link to Tickets**: Track mitigation work
4. **Regular Updates**: Keep risk tracking current

## Example: Complete Minimal Model

```yaml
threagile_version: "1.0.0"
title: "Example Application"
date: "2024-01-01"
author:
  name: "Security Team"
business_criticality: "important"

data_assets:
  user-data:
    id: "user-data"
    description: "User profile information"
    usage: "business"
    quantity: "many"
    confidentiality: "confidential"
    integrity: "important"
    availability: "important"
    justification_cia_rating: "Contains user PII"

technical_assets:
  web-app:
    id: "web-app"
    description: "Web application"
    type: "process"
    usage: "business"
    size: "application"
    technology: "web-application"
    internet: true
    machine: "virtual"
    encryption: "none"
    confidentiality: "public"
    integrity: "important"
    availability: "important"
    justification_cia_rating: "Public web application"
    custom_developed_parts: true
    data_assets_processed:
      - "user-data"
    communication_links:
      database:
        target: "database"
        description: "Database access"
        protocol: "jdbc-encrypted"
        authentication: "credentials"
        authorization: "technical-user"
        usage: "business"
        data_assets_sent:
          - "user-data"
        data_assets_received:
          - "user-data"

  database:
    id: "database"
    description: "User database"
    type: "datastore"
    usage: "business"
    size: "service"
    technology: "database"
    internet: false
    machine: "virtual"
    encryption: "data-with-symmetric-shared-key"
    confidentiality: "confidential"
    integrity: "critical"
    availability: "important"
    justification_cia_rating: "Stores all user data"
    custom_developed_parts: false
    data_assets_stored:
      - "user-data"

trust_boundaries:
  internet:
    id: "internet"
    description: "Public internet"
    type: "network-cloud-provider"
    technical_assets_inside:
      - "web-app"

risk_tracking: {}
```

## References

- [Threagile GitHub Repository](https://github.com/Threagile/threagile)
- [Threagile Website](https://threagile.io)
- [STRIDE Threat Model](https://en.wikipedia.org/wiki/STRIDE_(security))
- [OWASP ASVS](https://owasp.org/www-project-application-security-verification-standard/)
- [CWE Database](https://cwe.mitre.org/)
