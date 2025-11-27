# Threagile Built-in Risk Rules Reference

This document catalogs all built-in risk detection rules in Threagile, organized by security category. Each rule identifies specific threats and vulnerabilities in your architecture model.

## Risk Rule Categories

Threagile organizes risks using the STRIDE threat modeling framework:
- **S**poofing: Identity impersonation
- **T**ampering: Data modification
- **R**epudiation: Denying actions
- **I**nformation Disclosure: Unauthorized information access
- **D**enial of Service: Service availability attacks
- **E**levation of Privilege: Unauthorized privilege gain

## Authentication & Identity Management

### Missing Authentication

**Risk ID:** `missing-authentication`
**STRIDE:** Spoofing, Elevation of Privilege
**CWE:** CWE-306 (Missing Authentication for Critical Function)

**Description:** Technical assets that process or store sensitive data without proper authentication mechanisms.

**Detection Logic:**
- Identifies technical assets with `authentication: none` on communication links
- Checks if assets process confidential or restricted data
- Flags assets accessible from less trusted boundaries

**Mitigation:**
- Implement authentication for all sensitive endpoints
- Use strong authentication methods (token, certificate, two-factor)
- Avoid `authentication: none` for business-critical assets

**Severity Factors:**
- Higher for assets processing strictly-confidential data
- Higher for internet-facing assets
- Higher for assets in less trusted boundaries

---

### Missing Authentication Second Factor

**Risk ID:** `missing-authentication-second-factor`
**STRIDE:** Spoofing, Elevation of Privilege
**CWE:** CWE-308 (Use of Single-factor Authentication)

**Description:** Critical authentication services that don't enforce multi-factor authentication.

**Detection Logic:**
- Identifies authentication services (identity providers, login services)
- Checks if used by human clients
- Flags missing two-factor authentication requirement

**Mitigation:**
- Implement multi-factor authentication (MFA/2FA)
- Use authenticator apps, hardware tokens, or SMS codes
- Enforce MFA for all privileged accounts
- Consider adaptive authentication based on risk

**Severity Factors:**
- Critical for identity providers
- Higher for internet-facing authentication
- Higher for mission-critical systems

---

### Missing Identity Propagation

**Risk ID:** `missing-identity-propagation`
**STRIDE:** Elevation of Privilege, Repudiation
**CWE:** CWE-284 (Improper Access Control)

**Description:** Services that don't propagate end-user identity through the call chain, preventing proper authorization and audit logging.

**Detection Logic:**
- Identifies communication links with `authorization: technical-user`
- Checks if end-user identity should be propagated
- Flags loss of identity context in service chains

**Mitigation:**
- Use `authorization: end-user-identity-propagation`
- Implement token forwarding (JWT, SAML)
- Maintain audit trail with user context
- Use service mesh or API gateway for identity propagation

**Severity Factors:**
- Higher for audit-critical operations
- Higher for authorization decisions
- Higher for compliance requirements (SOX, HIPAA)

---

### Missing Identity Provider Isolation

**Risk ID:** `missing-identity-provider-isolation`
**STRIDE:** Elevation of Privilege, Tampering
**CWE:** CWE-653 (Insufficient Compartmentalization)

**Description:** Identity providers not properly isolated from other components, allowing potential compromise.

**Detection Logic:**
- Identifies identity provider technical assets
- Checks trust boundary isolation
- Flags shared runtimes with non-identity components

**Mitigation:**
- Deploy identity providers in dedicated trust boundary
- Use separate runtime environment
- Implement network segmentation
- Apply strict access controls

**Severity Factors:**
- Critical severity for shared runtimes
- High for weak trust boundary separation
- Higher for multi-tenant environments

---

### Missing Identity Store

**Risk ID:** `missing-identity-store`
**STRIDE:** Spoofing, Tampering
**CWE:** CWE-257 (Storing Passwords in a Recoverable Format)

**Description:** Authentication credentials stored inappropriately or without dedicated identity store.

**Detection Logic:**
- Identifies assets storing user credentials
- Checks for dedicated identity store (LDAP, database)
- Flags applications storing credentials directly

**Mitigation:**
- Use dedicated identity store (LDAP, Active Directory, database)
- Implement proper password hashing (bcrypt, Argon2)
- Use identity provider (OAuth, SAML, OpenID Connect)
- Never store plaintext passwords

**Severity Factors:**
- Critical for plaintext storage
- High for weak hashing algorithms
- Higher for many user accounts

---

## Injection Attacks

### SQL/NoSQL Injection

**Risk ID:** `sql-nosql-injection`
**STRIDE:** Tampering, Information Disclosure, Elevation of Privilege
**CWE:** CWE-89 (SQL Injection), CWE-943 (NoSQL Injection)
**ASVS:** V5.3.4

**Description:** Database queries constructed with unsanitized user input, allowing attackers to execute arbitrary queries.

**Detection Logic:**
- Identifies technical assets communicating with databases
- Checks if assets accept user input (web services, APIs)
- Flags custom-developed components processing user data
- Higher risk for assets accepting JSON, XML, or serialization formats

**Mitigation:**
- Use parameterized queries/prepared statements
- Implement ORM frameworks safely
- Validate and sanitize all input
- Apply principle of least privilege for database accounts
- Use stored procedures with proper escaping
- Implement WAF rules for injection detection

**Severity Factors:**
- Critical if processing strictly-confidential data
- High for internet-facing assets
- High for custom-developed code
- Elevated for databases with many records

---

### LDAP Injection

**Risk ID:** `ldap-injection`
**STRIDE:** Tampering, Information Disclosure, Elevation of Privilege
**CWE:** CWE-90 (LDAP Injection)

**Description:** LDAP queries constructed with unsanitized user input, allowing directory traversal or unauthorized access.

**Detection Logic:**
- Identifies assets communicating via LDAP/LDAPS protocols
- Checks if assets accept user input
- Flags custom-developed directory access code

**Mitigation:**
- Use parameterized LDAP queries
- Validate and escape all user input
- Use LDAP frameworks with built-in protection
- Implement proper input validation
- Apply principle of least privilege

**Severity Factors:**
- High for identity provider access
- Higher for custom-developed code
- Critical if modifying directory entries

---

### Search Query Injection

**Risk ID:** `search-query-injection`
**STRIDE:** Information Disclosure, Tampering
**CWE:** CWE-943 (Improper Neutralization of Special Elements in Data Query Logic)

**Description:** Search queries (Elasticsearch, Solr, etc.) constructed with unsanitized input.

**Detection Logic:**
- Identifies search engines and full-text search services
- Checks if accepting user-provided search terms
- Flags custom query construction

**Mitigation:**
- Use search framework query builders
- Sanitize search input
- Limit query capabilities
- Implement query complexity limits
- Use query templates

**Severity Factors:**
- High for confidential data
- Higher for custom query construction
- Elevated for administrative search functions

---

### XML External Entity (XXE)

**Risk ID:** `xml-external-entity`
**STRIDE:** Information Disclosure, Denial of Service
**CWE:** CWE-611 (XXE Attack)

**Description:** XML parsers processing external entities, allowing file access, SSRF, or DoS attacks.

**Detection Logic:**
- Identifies assets accepting XML input
- Checks `data_formats_accepted` for `xml`
- Flags custom-developed XML processing

**Mitigation:**
- Disable external entity processing
- Use safe XML parser configuration
- Validate XML against schema
- Use JSON instead of XML when possible
- Update XML libraries regularly

**Severity Factors:**
- High for internet-facing assets
- Critical for file system access
- Higher for internal network access

---

### Path Traversal

**Risk ID:** `path-traversal`
**STRIDE:** Information Disclosure, Tampering
**CWE:** CWE-22 (Path Traversal)

**Description:** File operations using unsanitized user input allowing access outside intended directories.

**Detection Logic:**
- Identifies file servers and assets with file system access
- Checks assets accepting file paths or names
- Flags custom file handling code

**Mitigation:**
- Validate and sanitize file paths
- Use whitelisting for allowed paths
- Implement chroot or jail environments
- Avoid direct file path concatenation
- Use safe file API methods

**Severity Factors:**
- Critical for confidential file access
- High for write operations
- Higher for privileged file system access

---

## Web Application Security

### Cross-Site Scripting (XSS)

**Risk ID:** `cross-site-scripting`
**STRIDE:** Tampering, Information Disclosure
**CWE:** CWE-79 (XSS)
**ASVS:** V5.3.3

**Description:** User input rendered in web pages without proper encoding, allowing script injection.

**Detection Logic:**
- Identifies web applications and web services
- Checks if rendering user input
- Flags custom-developed web interfaces

**Mitigation:**
- Encode all output (HTML, JavaScript, CSS, URL)
- Use templating engines with auto-escaping
- Implement Content Security Policy (CSP)
- Sanitize input (secondary defense)
- Use HTTPOnly cookies

**Severity Factors:**
- High for authenticated sessions
- Critical for administrative interfaces
- Higher for stored XSS vs. reflected

---

### Cross-Site Request Forgery (CSRF)

**Risk ID:** `cross-site-request-forgery`
**STRIDE:** Tampering, Elevation of Privilege
**CWE:** CWE-352 (CSRF)

**Description:** State-changing operations executable via GET or without CSRF tokens.

**Detection Logic:**
- Identifies web applications with state-changing operations
- Checks if used by human clients
- Flags missing CSRF protection mechanisms

**Mitigation:**
- Implement CSRF tokens (synchronizer token pattern)
- Use SameSite cookie attribute
- Require POST for state changes
- Verify Origin/Referer headers
- Use custom headers for AJAX requests

**Severity Factors:**
- High for critical operations (transfers, changes)
- Higher for privileged users
- Critical for administrative functions

---

### Server-Side Request Forgery (SSRF)

**Risk ID:** `server-side-request-forgery`
**STRIDE:** Tampering, Information Disclosure, Elevation of Privilege
**CWE:** CWE-918 (SSRF)

**Description:** Server-side code making requests to user-specified URLs, allowing internal network access.

**Detection Logic:**
- Identifies assets making outbound requests based on user input
- Checks for URL parameters or webhook functionality
- Flags custom HTTP client implementations

**Mitigation:**
- Whitelist allowed destinations
- Validate and sanitize URLs
- Disable unnecessary protocols (file://, gopher://)
- Use network segmentation
- Implement request signing
- Block private IP ranges

**Severity Factors:**
- Critical for cloud metadata access
- High for internal network access
- Higher for privileged server accounts

---

### Untrusted Deserialization

**Risk ID:** `untrusted-deserialization`
**STRIDE:** Tampering, Elevation of Privilege
**CWE:** CWE-502 (Deserialization of Untrusted Data)

**Description:** Deserializing untrusted data allowing remote code execution.

**Detection Logic:**
- Identifies assets accepting serialized formats
- Checks `data_formats_accepted` for `serialization`
- Flags Java, .NET, Python pickle deserialization

**Mitigation:**
- Avoid deserializing untrusted data
- Use safe serialization formats (JSON, protobuf)
- Implement integrity checks (signatures, HMACs)
- Use type whitelisting
- Run deserialization in sandboxes

**Severity Factors:**
- Critical severity (RCE potential)
- Higher for internet-facing assets
- Higher for privileged processes

---

## Infrastructure & Deployment Security

### Missing Build Infrastructure

**Risk ID:** `missing-build-infrastructure`
**STRIDE:** Tampering
**CWE:** CWE-494 (Download of Code Without Integrity Check)

**Description:** Custom-developed components without proper build infrastructure or CI/CD pipeline.

**Detection Logic:**
- Identifies custom-developed technical assets
- Checks for corresponding build-pipeline assets
- Flags missing artifact verification

**Mitigation:**
- Implement CI/CD pipeline
- Use source code repository
- Implement build automation
- Sign build artifacts
- Use artifact registry

**Severity Factors:**
- High for production deployments
- Higher for critical business assets
- Critical for security-sensitive components

---

### Unchecked Deployment

**Risk ID:** `unchecked-deployment`
**STRIDE:** Tampering
**CWE:** CWE-494

**Description:** Deployments without security checks, code review, or approval gates.

**Detection Logic:**
- Identifies deployment processes
- Checks for approval mechanisms
- Flags direct production deployment

**Mitigation:**
- Implement deployment approval workflow
- Require code review
- Run security scans (SAST, DAST)
- Use staging environment
- Implement rollback capability

**Severity Factors:**
- High for mission-critical systems
- Higher for internet-facing assets
- Critical for privileged components

---

### Push Instead of Pull Deployment

**Risk ID:** `push-instead-of-pull-deployment`
**STRIDE:** Tampering, Elevation of Privilege
**CWE:** CWE-494

**Description:** Push-based deployments that could be compromised to deploy malicious code.

**Detection Logic:**
- Identifies deployment processes pushing to production
- Checks deployment direction
- Flags privileged push access

**Mitigation:**
- Use pull-based deployment (GitOps)
- Implement image signing and verification
- Use admission controllers
- Require deployment approval
- Limit push credentials

**Severity Factors:**
- High for production systems
- Critical for privileged environments
- Higher for internet-accessible build systems

---

### Container Base Image Backdooring

**Risk ID:** `container-base-image-backdooring`
**STRIDE:** Tampering, Elevation of Privilege
**CWE:** CWE-494

**Description:** Container base images from untrusted sources or without integrity verification.

**Detection Logic:**
- Identifies container-based deployments
- Checks base image sources
- Flags missing image scanning

**Mitigation:**
- Use trusted base images
- Scan images for vulnerabilities
- Sign and verify images
- Use private registry
- Minimize base image layers

**Severity Factors:**
- High for production containers
- Critical for privileged containers
- Higher for internet-facing services

---

### Container Platform Escape

**Risk ID:** `container-platform-escape`
**STRIDE:** Elevation of Privilege
**CWE:** CWE-250 (Execution with Unnecessary Privileges)

**Description:** Containers running with excessive privileges allowing escape to host.

**Detection Logic:**
- Identifies shared container runtimes
- Checks for privileged containers
- Flags containers with host access

**Mitigation:**
- Run containers unprivileged
- Use security policies (Pod Security Standards)
- Implement runtime security
- Use minimal capabilities
- Enable seccomp, AppArmor, SELinux

**Severity Factors:**
- Critical for privileged containers
- High for multi-tenant platforms
- Higher for host network/filesystem access

---

## Data Protection

### Unencrypted Communication

**Risk ID:** `unencrypted-communication`
**STRIDE:** Information Disclosure, Tampering
**CWE:** CWE-319 (Cleartext Transmission of Sensitive Information)
**ASVS:** V9.1.1

**Description:** Communication links transmitting sensitive data without encryption.

**Detection Logic:**
- Identifies communication links with unencrypted protocols (HTTP, FTP, Telnet)
- Checks data asset confidentiality levels
- Flags sensitive data over plaintext channels

**Mitigation:**
- Use TLS 1.3 (minimum TLS 1.2)
- Implement certificate pinning
- Enforce HTTPS redirects
- Use encrypted database protocols
- Enable encryption for message queues

**Severity Factors:**
- Critical for strictly-confidential data
- High for credentials
- Higher for internet transit
- Elevated for cross-boundary communication

---

### Unencrypted Assets

**Risk ID:** `unencrypted-assets`
**STRIDE:** Information Disclosure
**CWE:** CWE-311 (Missing Encryption of Sensitive Data)

**Description:** Data assets at rest without proper encryption.

**Detection Logic:**
- Identifies datastores with `encryption: none` or `encryption: transparent`
- Checks stored data confidentiality
- Flags sensitive data without encryption

**Mitigation:**
- Implement encryption at rest
- Use database encryption features
- Encrypt file systems
- Use hardware security modules (HSM)
- Manage encryption keys properly

**Severity Factors:**
- Critical for strictly-confidential data
- High for credentials and keys
- Higher for compliance requirements (PCI-DSS, HIPAA)

---

### Missing File Validation

**Risk ID:** `missing-file-validation`
**STRIDE:** Tampering, Elevation of Privilege
**CWE:** CWE-434 (Unrestricted Upload of File with Dangerous Type)

**Description:** File uploads without proper validation allowing malicious file execution.

**Detection Logic:**
- Identifies assets accepting file uploads
- Checks for file validation mechanisms
- Flags executable file acceptance

**Mitigation:**
- Validate file types (whitelist)
- Scan files for malware
- Store files outside webroot
- Use virus scanning
- Implement file size limits
- Strip metadata and re-encode

**Severity Factors:**
- High for executable file types
- Critical for server-side processing
- Higher for public upload features

---

### Unnecessary Data Transfer

**Risk ID:** `unnecessary-data-transfer`
**STRIDE:** Information Disclosure
**CWE:** CWE-359 (Exposure of Private Information)

**Description:** Data assets transmitted through links unnecessarily, increasing exposure.

**Detection Logic:**
- Identifies data assets in communication links
- Checks if data transfer is necessary
- Flags excessive data exposure

**Mitigation:**
- Minimize data transfer (need-to-know)
- Implement data filtering
- Use field-level access control
- Apply data minimization principle
- Remove unnecessary data from APIs

**Severity Factors:**
- High for confidential data
- Higher for unnecessary cross-boundary transfer
- Elevated for PII over internet

---

## Access Control & Network Security

### Unguarded Access from Internet

**Risk ID:** `unguarded-access-from-internet`
**STRIDE:** Tampering, Information Disclosure, Denial of Service
**CWE:** CWE-749 (Exposed Dangerous Method or Function)

**Description:** Internal services directly accessible from the internet without proper protections.

**Detection Logic:**
- Identifies assets with `internet: true`
- Checks for WAF, API gateway, or rate limiting
- Flags direct internet exposure

**Mitigation:**
- Use API gateway or reverse proxy
- Implement Web Application Firewall (WAF)
- Add rate limiting and throttling
- Use IP whitelisting where appropriate
- Implement DDoS protection
- Add authentication and authorization

**Severity Factors:**
- Critical for databases and internal services
- High for administrative interfaces
- Higher for sensitive business assets

---

### Unguarded Direct Datastore Access

**Risk ID:** `unguarded-direct-datastore-access`
**STRIDE:** Information Disclosure, Tampering
**CWE:** CWE-284 (Improper Access Control)

**Description:** Datastores accessible without going through application layer controls.

**Detection Logic:**
- Identifies datastore assets
- Checks for direct access from multiple clients
- Flags missing application-layer access control

**Mitigation:**
- Use application-layer access control
- Restrict database network access
- Implement database firewalls
- Use connection pooling
- Apply principle of least privilege

**Severity Factors:**
- High for confidential data
- Critical for multi-tenant datastores
- Higher for internet-accessible databases

---

### Missing Network Segmentation

**Risk ID:** `missing-network-segmentation`
**STRIDE:** Elevation of Privilege, Tampering
**CWE:** CWE-923 (Improper Restriction of Communication Channel)

**Description:** Lack of network segmentation allowing unrestricted lateral movement.

**Detection Logic:**
- Identifies assets in same trust boundary
- Checks for overly broad network access
- Flags flat network architectures

**Mitigation:**
- Implement network segmentation (VLANs, subnets)
- Use micro-segmentation
- Deploy network firewalls
- Implement zero-trust architecture
- Use service mesh for service-to-service security

**Severity Factors:**
- High for mixed criticality assets
- Critical for compliance requirements
- Higher for multi-tenant environments

---

### Missing Hardening

**Risk ID:** `missing-hardening`
**STRIDE:** Elevation of Privilege
**CWE:** CWE-1188 (Initialization of a Resource with an Insecure Default)

**Description:** Systems deployed with default configurations or missing security hardening.

**Detection Logic:**
- Identifies production systems
- Checks for hardening documentation
- Flags default configurations

**Mitigation:**
- Apply CIS benchmarks
- Disable unnecessary services
- Remove default accounts
- Apply security patches
- Use configuration management
- Implement least privilege

**Severity Factors:**
- High for internet-facing systems
- Critical for privileged systems
- Higher for mission-critical assets

---

## Secrets & Vaults

### Accidental Secret Leak

**Risk ID:** `accidental-secret-leak`
**STRIDE:** Information Disclosure
**CWE:** CWE-615 (Inclusion of Sensitive Information in Source Code Comments)

**Description:** Secrets potentially exposed in source code, logs, or error messages.

**Detection Logic:**
- Identifies custom-developed assets
- Checks for credential management practices
- Flags potential secret exposure paths

**Mitigation:**
- Never commit secrets to source control
- Use secret management tools (Vault, AWS Secrets Manager)
- Implement secret scanning in CI/CD
- Use environment variables or config files (gitignored)
- Rotate exposed secrets immediately
- Sanitize logs and error messages

**Severity Factors:**
- Critical for production credentials
- High for API keys and tokens
- Higher for privileged account secrets

---

### Missing Vault

**Risk ID:** `missing-vault`
**STRIDE:** Information Disclosure
**CWE:** CWE-522 (Insufficiently Protected Credentials)

**Description:** Secrets and credentials not stored in dedicated secret management system.

**Detection Logic:**
- Identifies assets managing secrets
- Checks for vault or secret management system
- Flags credential storage in applications

**Mitigation:**
- Implement secret management (HashiCorp Vault, AWS Secrets Manager)
- Centralize secret storage
- Enable secret rotation
- Use dynamic credentials
- Implement access auditing

**Severity Factors:**
- High for production secrets
- Critical for many secrets
- Higher for privileged credentials

---

### Missing Vault Isolation

**Risk ID:** `missing-vault-isolation`
**STRIDE:** Elevation of Privilege
**CWE:** CWE-653 (Insufficient Compartmentalization)

**Description:** Secret management systems not properly isolated from other components.

**Detection Logic:**
- Identifies vault/secret management assets
- Checks trust boundary isolation
- Flags shared runtimes with other components

**Mitigation:**
- Deploy vault in dedicated trust boundary
- Use separate runtime environment
- Implement strict network segmentation
- Apply hardening to vault infrastructure
- Use HSM for key storage

**Severity Factors:**
- Critical for shared runtimes
- High for weak boundary separation
- Higher for mission-critical systems

---

## Cloud Security

### Missing Cloud Hardening

**Risk ID:** `missing-cloud-hardening`
**STRIDE:** Elevation of Privilege, Tampering
**CWE:** CWE-1188

**Description:** Cloud resources deployed without proper security configurations.

**Detection Logic:**
- Identifies cloud-based assets
- Checks for security group configuration
- Flags missing cloud security controls

**Mitigation:**
- Implement cloud security posture management (CSPM)
- Use infrastructure as code with security policies
- Enable cloud provider security features
- Implement least privilege IAM
- Enable logging and monitoring
- Use cloud-native security services

**Severity Factors:**
- High for internet-facing resources
- Critical for data storage
- Higher for production environments

---

### Missing WAF

**Risk ID:** `missing-waf`
**STRIDE:** Tampering, Information Disclosure, Denial of Service
**CWE:** CWE-1008 (Architectural Security Tactics)

**Description:** Internet-facing web applications without Web Application Firewall protection.

**Detection Logic:**
- Identifies web applications with `internet: true`
- Checks for WAF in front of application
- Flags direct internet exposure

**Mitigation:**
- Deploy Web Application Firewall
- Use cloud WAF services (AWS WAF, Cloudflare)
- Enable OWASP Core Rule Set
- Configure custom rules
- Implement rate limiting
- Enable DDoS protection

**Severity Factors:**
- High for sensitive applications
- Critical for known vulnerable apps
- Higher for high-value targets

---

## Other Security Risks

### DoS Risky Access Across Trust Boundaries

**Risk ID:** `dos-risky-access-across-trust-boundary`
**STRIDE:** Denial of Service
**CWE:** CWE-770 (Allocation of Resources Without Limits)

**Description:** Services accepting unbounded requests across trust boundaries without rate limiting.

**Detection Logic:**
- Identifies communication across trust boundaries
- Checks for rate limiting or throttling
- Flags unrestricted resource access

**Mitigation:**
- Implement rate limiting
- Use request throttling
- Add circuit breakers
- Implement backpressure
- Use queue-based load leveling
- Set resource quotas

**Severity Factors:**
- High for internet-facing services
- Critical for critical business functions
- Higher for computationally expensive operations

---

### Incomplete Model

**Risk ID:** `incomplete-model`
**STRIDE:** All categories
**CWE:** N/A

**Description:** Threat model missing key components or details, potentially missing risks.

**Detection Logic:**
- Checks for empty descriptions
- Identifies assets without trust boundaries
- Flags missing communication links
- Detects orphaned assets

**Mitigation:**
- Complete all asset descriptions
- Document all communication links
- Assign all assets to trust boundaries
- Review model for completeness
- Validate model with stakeholders

**Severity Factors:**
- Varies based on missing elements
- Higher for critical system components

---

### Mixed Targets on Shared Runtime

**Risk ID:** `mixed-targets-on-shared-runtime`
**STRIDE:** Elevation of Privilege
**CWE:** CWE-653 (Insufficient Compartmentalization)

**Description:** Components with different criticality or trust levels sharing the same runtime.

**Detection Logic:**
- Identifies shared runtimes
- Compares confidentiality/integrity levels
- Flags mixed criticality components

**Mitigation:**
- Separate components by criticality
- Use dedicated runtimes for sensitive components
- Implement runtime isolation
- Apply principle of least privilege
- Use containers or VMs for isolation

**Severity Factors:**
- High for mixing critical and non-critical
- Critical for different trust levels
- Higher for multi-tenant scenarios

---

### Unnecessary Communication Links

**Risk ID:** `unnecessary-communication-link`
**STRIDE:** Information Disclosure
**CWE:** CWE-1008

**Description:** Communication links that appear unnecessary or overly permissive.

**Detection Logic:**
- Identifies communication links
- Checks if data flow is justified
- Flags redundant or suspicious links

**Mitigation:**
- Remove unnecessary links
- Apply least privilege to communications
- Document link necessity
- Implement network policies
- Use service mesh for fine-grained control

**Severity Factors:**
- Medium to high based on data sensitivity
- Higher for cross-boundary links

---

### Unnecessary Technical Asset

**Risk ID:** `unnecessary-technical-asset`
**STRIDE:** Elevation of Privilege
**CWE:** CWE-1008

**Description:** Technical assets that may not be necessary for system operation.

**Detection Logic:**
- Identifies underutilized assets
- Checks for orphaned components
- Flags experimental or deprecated assets

**Mitigation:**
- Remove unnecessary assets
- Decommission unused components
- Document asset necessity
- Apply principle of least infrastructure
- Regular architecture reviews

**Severity Factors:**
- Medium based on asset exposure
- Higher for internet-facing assets

---

### Wrong Trust Boundary Content

**Risk ID:** `wrong-trust-boundary-content`
**STRIDE:** Elevation of Privilege
**CWE:** CWE-923

**Description:** Assets placed in inappropriate trust boundaries for their function.

**Detection Logic:**
- Checks asset placement in boundaries
- Validates boundary types match asset needs
- Flags security mismatches

**Mitigation:**
- Move assets to appropriate boundaries
- Align boundaries with security zones
- Document boundary decisions
- Review boundary definitions
- Apply defense-in-depth

**Severity Factors:**
- High for sensitive assets in untrusted boundaries
- Critical for exposing internal services

---

### Wrong Communication Link Content

**Risk ID:** `wrong-communication-link-content`
**STRIDE:** Information Disclosure
**CWE:** CWE-923

**Description:** Communication links with inappropriate protocol, authentication, or data for the connection type.

**Detection Logic:**
- Checks protocol appropriateness
- Validates authentication strength
- Flags data sensitivity mismatches

**Mitigation:**
- Use appropriate protocols
- Match authentication to sensitivity
- Encrypt sensitive communications
- Document communication requirements
- Review link configurations

**Severity Factors:**
- High for unencrypted sensitive data
- Critical for weak authentication on critical links

---

## Risk Severity Calculation

Threagile calculates risk severity based on multiple factors:

### Severity Levels
- **Low**: Minor impact, easily mitigated
- **Medium**: Moderate impact, requires attention
- **Elevated**: Significant impact, priority mitigation
- **High**: Serious impact, urgent mitigation
- **Critical**: Severe impact, immediate action required

### Factors Affecting Severity
1. **Data Confidentiality**: Higher for strictly-confidential data
2. **Asset Criticality**: Higher for mission-critical systems
3. **Exploitation Likelihood**: More likely = higher severity
4. **Internet Exposure**: Internet-facing increases severity
5. **Trust Boundary**: Less trusted = higher severity
6. **Custom Development**: Custom code increases risk
7. **Quantity**: More data/users = higher impact
8. **Business Criticality**: Mission-critical increases severity

## Using Risk Rules

### Listing Available Rules
```bash
threagile list-risk-rules
```

### Skipping Specific Rules
```bash
threagile analyze-model --model model.yaml --skip-risk-rules "rule-id-1,rule-id-2"
```

### Custom Risk Rules
You can extend Threagile with custom risk rules:
```bash
threagile analyze-model --model model.yaml --risk-rules-plugins "/path/to/custom-rules.so"
```

## Best Practices

1. **Review All Identified Risks**: Don't ignore any risk flagged by Threagile
2. **Understand Detection Logic**: Know why each risk was identified
3. **Document Acceptance**: If accepting risks, document clear rationale
4. **Track Mitigation**: Use risk_tracking to monitor progress
5. **Regular Reanalysis**: Run analysis after architecture changes
6. **False Positive Handling**: Mark false positives with clear justification
7. **Compliance Mapping**: Link risks to compliance requirements (ASVS, CWE)
8. **Prioritize Critical**: Focus on critical/high risks first

## References

- [OWASP ASVS](https://owasp.org/www-project-application-security-verification-standard/)
- [CWE Database](https://cwe.mitre.org/)
- [STRIDE Threat Model](https://en.wikipedia.org/wiki/STRIDE_(security))
- [Threagile GitHub](https://github.com/Threagile/threagile)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [SANS Top 25](https://www.sans.org/top25-software-errors/)
