# Scenari Behavioral Specifications

Use when writing behavioral specifications. Creates Gherkin feature files using Scenari for executable specifications in Clojure projects.

## Purpose

Creates behavioral specifications using Gherkin (Given-When-Then) syntax with Scenari, a Clojure library for behavior-driven development. Connects specifications to quality requirements and maintains traceability from requirements through tests to implementation.

## When to Use

- Starting BDD (Behavior-Driven Development) for a project
- Documenting user-facing behavior and API contracts
- Creating executable specifications
- Connecting requirements to tests
- Facilitating communication between stakeholders and developers
- Translating critical scenarios to formal specifications (TLA+, Alloy)

## Prerequisites

- Obsidian vault initialized (use init-obsidian-vault skill)
- Clojure project with deps.edn
- Understanding of system requirements

## What is Scenari?

Scenari (v2.0.2) is a Clojure library for writing executable specifications using Gherkin syntax. Unlike Cucumber (Ruby) or Cucumber-JVM, Scenari is designed specifically for Clojure and emphasizes:

- **Simplicity**: Pure Clojure, no separate step definition files
- **Testability**: Integrates with clojure.test and Kaocha
- **Traceability**: Link scenarios to requirements and architecture
- **Expressiveness**: Rich Clojure DSL for step definitions (defgiven, defwhen, defthen)
- **State Management**: Immutable state flow between steps

**Latest version:** `io.defsquare/scenari {:mvn/version "2.0.2"}`

## Reference Documentation

For detailed API and syntax information, see:
- **API Reference**: `references/api.md` - Complete Scenari API (deffeature, defgiven, defwhen, defthen, hooks, state management)
- **Gherkin Reference**: `references/gherkin.md` - Complete Gherkin syntax guide (keywords, tags, patterns, best practices)

## Gherkin Syntax Overview

Gherkin uses a simple, readable syntax with keywords and natural language:

```gherkin
Feature: User Authentication
  As a registered user
  I want to log in to the system
  So that I can access my account

  Background:
    Given the system is running
    And the database is clean

  Scenario: Successful login
    Given a registered user with email "user@example.com"
    And the user has password "SecurePass123!"
    When the user logs in with email "user@example.com" and password "SecurePass123!"
    Then the login should succeed
    And a JWT token should be returned
    And the session should be active

  Scenario: Failed login with wrong password
    Given a registered user with email "user@example.com"
    When the user logs in with email "user@example.com" and password "WrongPass"
    Then the login should fail
    And an error message "Invalid credentials" should be shown
    And no token should be returned
```

### Gherkin Keywords Quick Reference

- **Feature**: High-level description of functionality with user story
- **Background**: Steps run before each scenario (common setup)
- **Scenario**: Specific test case with Given-When-Then
- **Scenario Outline**: Template scenario with data table
- **Examples**: Data table for Scenario Outline
- **Given**: Preconditions and context setup
- **When**: Actions or events
- **Then**: Expected outcomes and assertions
- **And/But**: Continue previous step type

**Tags** for organization:
```gherkin
@smoke @authentication @critical
Scenario: Login validation
```

For complete Gherkin syntax, patterns, and best practices, see `references/gherkin.md`.

## Execution Steps

### Step 1: Check Project Setup

Verify this is a Clojure project:

```bash
ls -la deps.edn project.clj 2>/dev/null || echo "Not a Clojure project"
```

### Step 2: Add Scenari Dependency

Check if Scenari is in dependencies:

```bash
grep -r "scenari" deps.edn project.clj 2>/dev/null || echo "Scenari not found"
```

If not found, guide user to add it to `deps.edn`:

```clojure
{:deps {io.defsquare/scenari {:mvn/version "2.0.2"}}

 :aliases
 {:test {:extra-paths ["test"]
         :extra-deps {io.github.cognitect-labs/test-runner {:git/tag "v0.5.0"
                                                              :git/sha "48c3c67"}}}}}
```

Or for `project.clj`:

```clojure
:dependencies [[io.defsquare/scenari "2.0.2"]]
```

**Important:** Use `io.defsquare/scenari` for the latest v2.x, not the older `scenari/scenari`.

### Step 3: Create Feature Directory Structure

Create directory for feature files:

```bash
mkdir -p test/features
```

Standard structure:
```
test/
├── features/                    # Gherkin feature files
│   ├── authentication.feature
│   ├── user_management.feature
│   └── resource_crud.feature
└── step_definitions/           # Clojure step implementations
    ├── auth_steps.clj
    ├── user_steps.clj
    └── resource_steps.clj
```

### Step 4: Interview User About Scenarios

Gather information for writing scenarios:

1. **What feature are you specifying?**
   - Feature name
   - User story (As a... I want... So that...)

2. **What are the key scenarios?**
   - Happy path
   - Error cases
   - Edge cases
   - Security scenarios

3. **What are the preconditions?**
   - System state
   - Test data needed
   - External dependencies

4. **What are the actions?**
   - User interactions
   - API calls
   - System events

5. **What are the expected outcomes?**
   - Success criteria
   - Error messages
   - State changes
   - Side effects

6. **Are there related quality requirements?**
   - Performance requirements
   - Security requirements
   - Reliability requirements

### Step 5: Write Feature Files

Create feature files in `test/features/` using Gherkin syntax.

**Best Practices**:
- One feature per file
- Clear, business-readable language
- Focus on behavior, not implementation
- Use tags for categorization
- Link to requirements and ADRs

### Step 6: Add Traceability Tags

Use tags to link scenarios to other artifacts:

```gherkin
@requirement:REQ-001
@adr:0005
@quality:performance
@component:authentication
Feature: User Authentication
```

Common tag patterns:
- `@requirement:REQ-XXX` - Link to requirement ID
- `@adr:NNNN` - Link to ADR
- `@quality:CHARACTERISTIC` - Link to quality attribute
- `@component:NAME` - Component being tested
- `@critical` - Critical business scenario
- `@security` - Security-related
- `@performance` - Performance-related
- `@smoke` - Smoke test
- `@regression` - Regression test
- `@wip` - Work in progress

### Step 7: Create Step Definitions

Create Clojure step definition files in `test/step_definitions/`:

```clojure
(ns step-definitions.auth-steps
  (:require [clojure.test :refer :all]
            [scenari.v2.core :refer [defgiven defwhen defthen deffeature]]
            [myapp.auth :as auth]))

;; Given steps set up preconditions
(defgiven "the system is running"
  [_]
  {:system-state :running})

(defgiven "a registered user with email {string}"
  [state email]
  (let [user (create-test-user! email)]
    (assoc state :test-user user)))

(defgiven "the user has password {string}"
  [{:keys [test-user] :as state} password]
  (assoc-in state [:test-user :password] password))

;; When steps perform actions
(defwhen "the user logs in with email {string} and password {string}"
  [state email password]
  (let [result (auth/login {:email email :password password})]
    (assoc state :login-result result)))

;; Then steps verify outcomes
(defthen "the login should succeed"
  [{:keys [login-result]}]
  (is (= :success (:status login-result))))

(defthen "a JWT token should be returned"
  [{:keys [login-result]}]
  (is (contains? login-result :token))
  (is (not (nil? (:token login-result)))))

(defthen "the session should be active"
  [{:keys [login-result]}]
  (is (auth/session-active? (:session-id login-result))))

;; Define feature with lifecycle hooks
(deffeature auth-spec "test/features/authentication.feature"
  {:pre-scenario-run [#'clear-test-data!]
   :post-scenario-run [#'log-test-results!]
   :default-scenario-state {}})
```

**Key points:**
- Import from `scenari.v2.core`
- Use `defgiven`, `defwhen`, `defthen` (not `defsteps`)
- First parameter is state from previous step
- Use `{string}`, `{int}` matchers (not regex)
- State flows immutably between steps
- See `references/api.md` for complete API details

### Step 8: Create Vault Documentation

Create `vault/specifications/features.md` to document and embed features:

```markdown
# Feature Specifications

**Last Updated**: YYYY-MM-DD

## Overview

This document provides an overview of behavioral specifications for the system, written in Gherkin format and executable via Scenari.

## Features

### Authentication

**Location**: `test/features/authentication.feature`

**Related**:
- Requirements: [[../requirements/quality-attributes#security|Security Requirements]]
- Architecture: [[../arc42/08-crosscutting#authentication|Authentication Design]]
- ADRs: [[../decisions/0005-jwt-auth|ADR-0005: JWT Authentication]]

**Scenarios**:
- ✅ Successful login
- ✅ Failed login with wrong password
- ✅ Failed login with non-existent user
- ✅ Account lockout after failed attempts
- ✅ Session expiration

**Feature File**:
```gherkin
![[../../test/features/authentication.feature]]
```

### User Management

**Location**: `test/features/user_management.feature`

[Similar structure for each feature]

## Running Tests

Run all feature tests:
```bash
clojure -M:test
```

Run specific feature:
```bash
clojure -M:test -n authentication
```

Run tagged scenarios:
```bash
clojure -M:test -t smoke
clojure -M:test -t critical
clojure -M:test -t security
```

## Traceability

### By Requirement

- **REQ-001** (User Authentication):
  - `authentication.feature`: Successful login, Failed login
- **REQ-005** (Data Validation):
  - `user_management.feature`: Create user with invalid email

### By Quality Attribute

- **Performance**:
  - `api_performance.feature`: Response time scenarios
- **Security**:
  - `authentication.feature`: Auth scenarios
  - `authorization.feature`: Permission scenarios
  - `security.feature`: Security-specific scenarios

### By Component

- **Auth Component**:
  - `authentication.feature`
  - `authorization.feature`
- **User Component**:
  - `user_management.feature`
  - `user_profile.feature`

## Coverage

| Component | Scenarios | Coverage |
|-----------|-----------|----------|
| Authentication | 12 | 95% |
| User Management | 8 | 87% |
| Resource CRUD | 15 | 92% |
| **Total** | **35** | **91%** |

## Related Documentation

- [[../requirements/README|Requirements]]
- [[../arc42/06-runtime|Runtime View]] - Similar scenarios
- [[../arc42/10-quality|Quality Requirements]]

#specifications #gherkin #scenari #bdd
```

### Step 9: Link Critical Scenarios to Formal Specs

For critical scenarios (especially concurrency, consistency, security), suggest formal specification:

```markdown
## Critical Scenarios Requiring Formal Verification

### Scenario: Concurrent Resource Updates

**Location**: `test/features/concurrency.feature`

**Criticality**: HIGH - Data consistency

**Gherkin Scenario**:
```gherkin
Scenario: Two users update same resource simultaneously
  Given a resource with value "100"
  When user A updates value to "150"
  And user B updates value to "200" at the same time
  Then only one update should succeed
  And the other should get a conflict error
  And no data should be lost
```

**Recommendation**: Specify in TLA+ for formal verification of concurrency invariants.

**TLA+ Specification**: `specs/resource_update.tla`

**Verification Status**: ⏳ Pending formal verification
```

### Step 10: Report Completion

```
Feature specifications created in test/features/

Features created:
✓ authentication.feature - 5 scenarios
✓ user_management.feature - 3 scenarios
✓ resource_crud.feature - 4 scenarios

Documentation:
✓ vault/specifications/features.md - Feature overview with embeds

Next steps:
1. Implement step definitions in test/step_definitions/
2. Run tests: clojure -M:test
3. Add scenarios to CI/CD pipeline
4. Review traceability to requirements
5. Consider formal specs for critical scenarios
```

## Complete Feature File Example

`test/features/authentication.feature`:

```gherkin
@component:authentication
@requirement:REQ-010
@adr:0005
Feature: User Authentication
  As a registered user
  I want to securely log in to the system
  So that I can access my protected resources

  Background:
    Given the system is running
    And the authentication service is available
    And the database is clean

  @smoke @critical
  Scenario: Successful login with valid credentials
    Given a registered user with email "alice@example.com"
    And the user has password "SecurePass123!"
    When the user logs in with email "alice@example.com" and password "SecurePass123!"
    Then the login should succeed
    And a JWT token should be returned
    And the token should expire in 15 minutes
    And the session should be active
    And the user ID should match the registered user

  @security @critical
  Scenario: Failed login with incorrect password
    Given a registered user with email "alice@example.com"
    And the user has password "SecurePass123!"
    When the user logs in with email "alice@example.com" and password "WrongPassword"
    Then the login should fail with status 401
    And an error message "Invalid credentials" should be returned
    And no token should be returned
    And the failed attempt should be logged
    And the attempt count should increase

  @security
  Scenario: Failed login with non-existent user
    Given no user exists with email "nobody@example.com"
    When the user logs in with email "nobody@example.com" and password "AnyPassword"
    Then the login should fail with status 401
    And an error message "Invalid credentials" should be returned
    And no token should be returned
    And the attempt should be logged

  @security @critical
  Scenario: Account lockout after multiple failed attempts
    Given a registered user with email "alice@example.com"
    And the user has password "SecurePass123!"
    When the user fails to login 5 times with wrong password
    Then the account should be locked
    And subsequent login attempts should fail with status 403
    And an error message "Account locked" should be returned
    And an email notification should be sent
    And an admin alert should be triggered

  @quality:performance
  Scenario: Login performance under normal load
    Given 100 registered users
    When 100 users log in simultaneously
    Then all logins should complete
    And 95% of logins should complete in less than 200ms
    And no login should take more than 500ms

  @security
  Scenario: Session expiration after inactivity
    Given a logged-in user with email "alice@example.com"
    And the session timeout is 30 minutes
    When 31 minutes pass without activity
    And the user tries to access a protected resource
    Then the request should fail with status 401
    And an error message "Session expired" should be returned
    And the user should be redirected to login

  @security
  Scenario: JWT token validation with invalid signature
    Given a JWT token with invalid signature
    When the user tries to access a protected resource with the token
    Then the request should fail with status 401
    And an error message "Invalid token" should be returned
    And the attempt should be logged as potential security threat

  @security @critical
  Scenario Outline: Password validation
    When a user registers with password "<password>"
    Then the registration should <result>
    And the error message should be "<message>"

    Examples:
      | password         | result | message                               |
      | short           | fail   | Password must be at least 12 characters |
      | alllowercase123 | fail   | Password must contain uppercase letter  |
      | ALLUPPERCASE123 | fail   | Password must contain lowercase letter  |
      | NoNumbers!      | fail   | Password must contain a number          |
      | NoSpecial123    | fail   | Password must contain special character |
      | ValidPass123!   | succeed | -                                      |

  @security
  Scenario: Prevent timing attacks in authentication
    Given authentication endpoint is available
    When checking login with non-existent user
    And checking login with existing user but wrong password
    Then both requests should take similar time
    And the time difference should be less than 50ms
    And both should return the same error message

  @requirement:REQ-015
  @quality:usability
  Scenario: Clear error messages for login failures
    Given a registered user with email "alice@example.com"
    When the user fails to login
    Then the error message should not reveal if the email exists
    And the error message should be user-friendly
    And the error message should suggest corrective action
    And the message should not expose system internals
```

## Common Step Patterns

Provide reusable step patterns for typical scenarios:

### Setup Steps (Given)

```gherkin
Given the system is running
Given the database is clean
Given a registered user with email "alice@test.com"
Given the user has role "admin"
Given a resource with ID "12345"
Given 100 concurrent users
Given the cache is empty
Given a product with data {:name "Laptop" :price 1000}
```

### Action Steps (When)

```gherkin
When the user logs in with email "user@test.com" and password "pass123"
When the user creates a resource
When the user updates resource "12345"
When the user deletes resource "12345"
When 100 users perform action simultaneously
When I send a POST request to "/api/products"
```

### Assertion Steps (Then)

```gherkin
Then the request should succeed
Then the request should fail with status 401
Then an error message "Invalid credentials" should be returned
Then a JWT token should be returned
Then the database should contain a record
Then the cache should be updated
Then an event should be published
```

### Quality Attribute Steps

**Performance:**
```gherkin
Then the response time should be less than 200ms
Then 95% of requests should complete in less than 500ms
Then the throughput should be at least 1000 requests per second
```

**Security:**
```gherkin
Then the attempt should be logged
Then an admin alert should be triggered
Then the data should be encrypted
Then no sensitive data should be in logs
```

**Reliability:**
```gherkin
Then the system should handle the failure gracefully
Then cached data should be served
Then the circuit breaker should open
```

For more patterns and best practices, see `references/gherkin.md`.

## Integration with Quality Requirements

Link scenarios to quality attributes:

```gherkin
@quality:performance
@requirement:QR-001
Scenario: API response time meets SLA
  Given normal system load
  When a user requests their profile
  Then the response time should be less than 200ms at p95
  And the cache hit rate should be above 80%

@quality:security
@requirement:QR-012
Scenario: Authentication prevents brute force
  Given a user account
  When 5 failed login attempts occur
  Then the account should be locked
  And an alert should be sent

@quality:reliability
@requirement:QR-020
Scenario: System handles database failure gracefully
  Given the system is running
  When the database becomes unavailable
  Then cached data should still be served
  And the circuit breaker should open
  And users should see degraded service notice
```

## Translating to Formal Specifications

For critical scenarios, suggest formal verification:

### When to Use Formal Specs

Consider formal specification (TLA+, Alloy) for:
- **Concurrency scenarios**: Race conditions, deadlocks
- **Consistency requirements**: Database transactions, distributed systems
- **Security properties**: Access control, cryptographic protocols
- **Safety-critical**: Financial transactions, healthcare
- **Complex state machines**: Order processing, workflow engines

### Example Translation

**Gherkin Scenario**:
```gherkin
@formal-spec:required
Scenario: Concurrent resource updates maintain consistency
  Given a resource with value 100
  When user A updates to 150
  And user B updates to 200 concurrently
  Then only one update succeeds
  And the other receives conflict error
  And final value is either 150 or 200
  And no update is lost
```

**Suggest TLA+ Specification**:
```
Create formal spec: specs/resource_update.tla

Invariants to verify:
- AtMostOneUpdateSucceeds
- NoUpdateLost
- ConsistentFinalState

Run TLC model checker to verify all possible interleavings.
```

## Best Practices

1. **Business Language**: Write for stakeholders, not just developers
2. **One Scenario, One Thing**: Test one behavior per scenario
3. **Independent Scenarios**: Each scenario should run independently
4. **Declarative**: Describe what, not how
5. **Traceability**: Use tags to link to requirements and architecture
6. **Maintenance**: Keep scenarios up-to-date with system changes
7. **Data Tables**: Use examples for data-driven scenarios
8. **Performance**: Tag performance scenarios for separate runs
9. **Security**: Explicitly test security properties
10. **Critical Scenarios**: Consider formal verification

## Common Pitfalls

- **Too Much Detail**: Focus on behavior, not implementation
- **UI-Coupled**: Don't tie to specific UI elements
- **Missing Context**: Always provide necessary Given steps
- **Vague Assertions**: Be specific about expected outcomes
- **No Traceability**: Link to requirements and quality attributes
- **Untestable**: Make sure scenarios can be automated

## CI/CD Integration

Add to CI pipeline:

```yaml
# .github/workflows/test.yml
- name: Run Feature Tests
  run: clojure -M:test -t smoke

- name: Run All Tests
  run: clojure -M:test

- name: Run Security Tests
  run: clojure -M:test -t security

- name: Run Performance Tests
  run: clojure -M:test -t performance
  if: github.event_name == 'push' && github.ref == 'refs/heads/main'
```

## Running Tests

### With clojure.test

```bash
# Run all tests
clojure -M:test

# Run specific namespace
clojure -M:test -n my.test.namespace
```

### With Kaocha

Configure `tests.edn`:
```clojure
#kaocha/v1
{:tests [{:id :scenario
          :type :kaocha.type/scenari
          :kaocha/source-paths ["src"]
          :kaocha/test-paths ["test/scenario"]
          :kaocha.type.scenari/glue-paths ["test/scenario/glue"]}]}
```

Run tests:
```bash
# Run all scenario tests
clojure -M:kaocha :scenario

# Run with specific tags
clojure -M:kaocha :scenario --focus-meta :smoke
clojure -M:kaocha :scenario --focus-meta :critical
```

### REPL-Driven Development

```clojure
(require '[scenari.v2.core :refer [run-feature]]
         '[scenari.v2.test :refer [run-feature]])

;; Debug mode - returns detailed execution report
(scenari.v2.core/run-feature #'my-spec)

;; Test mode - outputs formatted results
(scenari.v2.test/run-feature #'my-spec)
```

For complete execution API, see `references/api.md`.

## Scenari vs Cucumber

**Scenari Advantages**:
- Pure Clojure (no JVM complexity)
- Simpler setup (v2: just defgiven/defwhen/defthen)
- Better Clojure integration (immutable state flow)
- Lightweight and fast
- Integrates with Kaocha

**Cucumber Advantages**:
- More mature ecosystem
- Better IDE tooling
- Larger community
- Multi-language support

Choose Scenari for Clojure projects prioritizing simplicity and Clojure-native workflows.

## Documentation Index

- **Main skill**: Current file - Workflow and best practices
- **API Reference**: `references/api.md` - Scenari v2 API documentation
- **Gherkin Reference**: `references/gherkin.md` - Complete Gherkin syntax guide

## Related Skills

- **iso25010-quality**: Quality requirements become scenarios
- **arc42-docs**: Section 6 (Runtime View) references scenarios
- **init-obsidian-vault**: Creates specifications folder

#scenari #gherkin #bdd #specifications #testing
