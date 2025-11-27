# Gherkin Syntax Reference

Complete reference for Gherkin syntax used in Scenari feature files.

## Overview

Gherkin is a plain-text language for writing executable specifications. It uses natural language with a small set of keywords to structure scenarios in a Given-When-Then format.

## Basic Structure

```gherkin
Feature: Short feature description
  Longer description explaining the feature
  Can span multiple lines

  Scenario: Specific test case description
    Given precondition or context
    When action or event
    Then expected outcome
```

## Keywords

### Feature

Describes the feature being specified. Appears once at the beginning of each feature file.

**Syntax:**
```gherkin
Feature: Feature name
  Optional description
  Can be multiple lines
  Usually includes user story
```

**Example:**
```gherkin
Feature: User Authentication
  As a registered user
  I want to log in to the system
  So that I can access my protected resources
```

**Best practices:**
- Use clear, business-focused language
- Include user story format (As a... I want... So that...)
- Keep description concise
- Focus on business value

### Scenario

Describes a specific test case or example of the feature's behavior.

**Syntax:**
```gherkin
Scenario: Scenario description
  Given preconditions
  When actions
  Then outcomes
```

**Example:**
```gherkin
Scenario: Successful login with valid credentials
  Given a registered user with email "alice@example.com"
  And the user has password "SecurePass123"
  When the user logs in
  Then the login should succeed
  And a session token should be returned
```

**Best practices:**
- One scenario tests one specific behavior
- Use descriptive names
- Keep scenarios independent
- Test both happy path and error cases

### Given

Sets up preconditions and initial context for the scenario.

**Purpose:** Establish the state of the system before the action.

**Example:**
```gherkin
Given the database is clean
Given a registered user exists
Given the system is in maintenance mode
Given a product with ID 12345 exists
```

**Best practices:**
- Focus on state, not actions
- Set up all necessary preconditions
- Use declarative language
- Avoid implementation details

### When

Describes the action or event being tested.

**Purpose:** Trigger the behavior being specified.

**Example:**
```gherkin
When the user logs in
When I create a new product
When the user submits the form
When a payment is processed
```

**Best practices:**
- Use active voice
- Describe one key action
- Focus on what happens, not how
- Keep it simple and clear

### Then

Describes the expected outcome or assertion.

**Purpose:** Verify the system behaves as expected.

**Example:**
```gherkin
Then the login should succeed
Then a product with ID 12345 should exist
Then an error message should be displayed
Then the order status should be "completed"
```

**Best practices:**
- State observable outcomes
- Be specific about expectations
- Test one main outcome per Then
- Use "should" for clarity

### And

Continues the previous step type (Given, When, or Then).

**Purpose:** Add additional steps of the same type without repeating the keyword.

**Example:**
```gherkin
Given a registered user exists
And the user has verified email
And the user is not locked out

When the user logs in
And enters correct credentials

Then the login should succeed
And a session should be created
And the user should be redirected to dashboard
```

**Best practices:**
- Use for related steps
- Maintains readability
- Keep logical grouping
- Don't overuse (max 3-4 And per section)

### But

Negative version of And, used for contrast.

**Purpose:** Express negative conditions or exceptions.

**Example:**
```gherkin
Given a registered user exists
But the user has not verified their email

Then the login should fail
But no account lockout should occur
```

**Best practices:**
- Use sparingly
- Emphasizes contrast
- Makes negative conditions clear
- Alternative to And for readability

### Background

Steps that run before each scenario in the feature.

**Purpose:** Avoid repeating common setup across scenarios.

**Syntax:**
```gherkin
Feature: User Management

  Background:
    Given the system is running
    And the database is clean
    And an admin user is logged in

  Scenario: Create user
    When I create a new user
    Then the user should exist

  Scenario: Delete user
    When I delete a user
    Then the user should not exist
```

**Best practices:**
- Keep Background short (1-4 steps)
- Only include truly common setup
- Avoid complex logic
- Consider if scenarios should be split into separate features

### Scenario Outline

Template for data-driven testing with multiple examples.

**Purpose:** Run the same scenario with different data sets.

**Syntax:**
```gherkin
Scenario Outline: Description with <placeholders>
  Given some context with "<parameter>"
  When action with <value>
  Then outcome with "<result>"

  Examples:
    | parameter | value | result |
    | data1     | 100   | success |
    | data2     | 200   | failure |
```

**Example:**
```gherkin
Scenario Outline: Password validation
  When a user registers with password "<password>"
  Then the registration should <result>
  And the error message should be "<message>"

  Examples:
    | password        | result  | message                              |
    | short          | fail    | Password must be at least 12 chars   |
    | alllowercase12 | fail    | Password must contain uppercase      |
    | ALLUPPERCASE12 | fail    | Password must contain lowercase      |
    | NoNumbers!     | fail    | Password must contain a number       |
    | ValidPass123!  | succeed | -                                    |
```

**Best practices:**
- Use for testing multiple similar cases
- Keep table columns manageable (3-5)
- Use descriptive column headers
- Consider readability vs. coverage

### Examples

Data table for Scenario Outline.

**Syntax:**
```gherkin
Examples:
  | column1 | column2 | column3 |
  | value1a | value2a | value3a |
  | value1b | value2b | value3b |
```

**Multiple example groups:**
```gherkin
Scenario Outline: User login
  When user "<username>" logs in with "<password>"
  Then the result should be "<outcome>"

  Examples: Valid credentials
    | username | password    | outcome |
    | alice    | secret123   | success |
    | bob      | password456 | success |

  Examples: Invalid credentials
    | username | password | outcome |
    | alice    | wrong    | failure |
    | unknown  | any      | failure |
```

**Best practices:**
- Group related examples
- Use descriptive group names
- Keep data sets focused
- Consider separate scenarios for very different cases

## Comments

Use `#` for comments. Comments are ignored during execution.

**Example:**
```gherkin
# This is a comment
Feature: Product Management
  # Users should be able to manage products

  Scenario: Create product
    # Setup test data
    Given a product category exists
    # Perform creation
    When I create a product
    Then the product should exist
```

**Best practices:**
- Use sparingly
- Explain why, not what
- Document complex scenarios
- Remove outdated comments

## Tags

Tags organize and filter scenarios for execution.

**Syntax:**
```gherkin
@tag-name
@another-tag
Feature: Feature name

@scenario-tag
Scenario: Scenario name
```

**Example:**
```gherkin
@authentication @security @critical
Feature: User Authentication

  @smoke @happy-path
  Scenario: Successful login
    Given valid credentials
    When user logs in
    Then login succeeds

  @error-handling @edge-case
  Scenario: Failed login
    Given invalid credentials
    When user logs in
    Then login fails
```

**Common tag patterns:**

**By priority:**
```gherkin
@critical
@high-priority
@low-priority
@smoke
```

**By type:**
```gherkin
@security
@performance
@integration
@unit
@regression
```

**By status:**
```gherkin
@wip          # Work in progress
@skip         # Skip this test
@manual       # Manual test only
@automated    # Automated test
```

**By component:**
```gherkin
@authentication
@user-management
@payment-processing
@api
@ui
```

**By requirement:**
```gherkin
@requirement:REQ-001
@adr:0005
@story:JIRA-123
```

**Best practices:**
- Use consistent naming conventions
- Tag at feature and/or scenario level
- Enable filtering in test runs
- Document tag meanings

## Parameters and Data

### String Parameters

Quoted strings capture text values:

```gherkin
Given a user with email "alice@example.com"
When I search for "laptop"
Then the message should be "Success"
```

Maps to Scenari:
```clojure
(defgiven "a user with email {string}" [state email] ...)
(defwhen "I search for {string}" [state term] ...)
(defthen "the message should be {string}" [state msg] ...)
```

### Numeric Parameters

Numbers capture integer values:

```gherkin
Given a product with price 100
When I order 5 items
Then the total should be 500
```

Maps to Scenari:
```clojure
(defgiven "a product with price {int}" [state price] ...)
(defwhen "I order {int} items" [state quantity] ...)
(defthen "the total should be {int}" [state total] ...)
```

### EDN Data Structures

Clojure data structures can be embedded:

```gherkin
Given a user with data {:name "Alice" :age 30 :role :admin}
When I send request with body {:product-id 123 :quantity 5}
Then the response should match {:status 200 :success true}
```

Maps to Scenari with automatic EDN evaluation:
```clojure
(defgiven "a user with data {string}"
  [state user-data]
  ;; user-data is evaluated as Clojure map
  (let [user (create-user! user-data)]
    (assoc state :user user)))
```

### Multi-line Strings (Doc Strings)

Use `"""` for multi-line text:

```gherkin
Given a document with content:
  """
  This is a multi-line
  document that can contain
  any text including "quotes"
  """

When I send request with JSON:
  """
  {
    "name": "Product",
    "description": "Long description",
    "tags": ["tag1", "tag2"]
  }
  """
```

### Data Tables

Inline data tables for structured data:

```gherkin
Given the following users exist:
  | name  | email           | role  |
  | Alice | alice@test.com  | admin |
  | Bob   | bob@test.com    | user  |
  | Carol | carol@test.com  | user  |

When I import products:
  | name    | price | category    |
  | Laptop  | 1000  | Electronics |
  | Mouse   | 25    | Accessories |
  | Keyboard| 75    | Accessories |
```

## Language and Style

### Declarative vs. Imperative

**Declarative (Preferred):**
```gherkin
Given a user is logged in
When the user creates a product
Then the product should exist in the database
```

**Imperative (Avoid):**
```gherkin
Given I click the login button
And I enter "user" in the username field
And I enter "pass" in the password field
And I click submit
When I click the "New Product" button
And I fill in the form fields
And I click "Save"
Then I should see the product in the list
```

**Why declarative is better:**
- Focuses on what, not how
- More maintainable
- Less brittle to UI changes
- More readable for non-technical stakeholders

### Business Language

Use domain language that stakeholders understand:

**Good:**
```gherkin
Given an active subscription
When the billing cycle completes
Then the customer should be charged
```

**Avoid technical jargon:**
```gherkin
Given a record in subscriptions table with status=1
When the cron job runs daily_billing_task
Then insert row into transactions table and update balance
```

### Consistent Terminology

Use the same terms throughout:

**Good:**
```gherkin
Given a user account exists
When the user logs in
Then the user should see their dashboard
```

**Inconsistent:**
```gherkin
Given a user account exists
When the customer signs in
Then the member should see their homepage
```

## Feature File Organization

### File Structure

```gherkin
# Comment about feature purpose

@feature-level-tags
Feature: Feature Name
  User story or description
  Can be multiple paragraphs

  Background:
    Given common setup
    And more common setup

  @scenario-tags
  Scenario: First scenario
    Given specific context
    When action occurs
    Then outcome verified

  @scenario-tags
  Scenario: Second scenario
    Given different context
    When different action
    Then different outcome

  @scenario-outline-tags
  Scenario Outline: Parameterized scenario
    Given context with "<param>"
    When action with <value>
    Then outcome "<result>"

    Examples:
      | param | value | result |
      | foo   | 1     | success |
      | bar   | 2     | failure |
```

### File Naming

**Convention:**
- Use lowercase with underscores: `user_authentication.feature`
- Or kebab-case: `user-authentication.feature`
- Match feature domain: `product_management.feature`

**Directory structure:**
```
test/features/
├── authentication/
│   ├── login.feature
│   ├── logout.feature
│   └── password_reset.feature
├── products/
│   ├── create_product.feature
│   ├── update_product.feature
│   └── delete_product.feature
└── users/
    ├── registration.feature
    └── profile_management.feature
```

## Advanced Patterns

### State Setup Chains

Build complex state through chained steps:

```gherkin
Scenario: Admin creates user account with permissions
  Given the system is initialized
  And an admin user is logged in
  And the user management module is enabled
  And a "manager" role exists with permissions:
    | permission      |
    | create_product  |
    | edit_product    |
    | view_analytics  |
  When the admin creates a user with role "manager"
  Then the user should have all manager permissions
  And the user should be able to login
```

### Testing Negative Cases

Explicitly test failures:

```gherkin
Scenario: Cannot create product without authentication
  Given no user is logged in
  When an anonymous user tries to create a product
  Then the request should be rejected with status 401
  And an error message "Authentication required" should be shown
  And no product should be created

Scenario: Cannot delete product owned by another user
  Given user Alice owns product "Laptop"
  And user Bob is logged in
  When Bob tries to delete Alice's product
  Then the request should be rejected with status 403
  And an error message "Forbidden" should be shown
  And the product should still exist
```

### Performance Scenarios

Specify performance requirements:

```gherkin
@quality:performance
Scenario: API responds within SLA
  Given 100 concurrent users
  When each user requests their profile
  Then 95% of requests should complete within 200ms
  And no request should exceed 500ms
  And the error rate should be below 0.1%

@quality:performance
Scenario: System handles peak load
  Given the system is under normal load
  When load increases to 10x normal
  Then the system should continue serving requests
  And response time should remain below 1 second
  And no requests should be dropped
```

### Security Scenarios

Specify security requirements:

```gherkin
@quality:security
Scenario: Prevent SQL injection
  Given a product search endpoint
  When a user searches for "'; DROP TABLE users; --"
  Then the query should be safely escaped
  And no database tables should be dropped
  And an empty search result should be returned

@quality:security
Scenario: Prevent XSS attacks
  Given a user profile page
  When a user sets their name to "<script>alert('XSS')</script>"
  Then the script should not execute
  And the name should be HTML-escaped in the display
```

### Error Recovery

Specify error handling and recovery:

```gherkin
@quality:reliability
Scenario: Graceful degradation when database unavailable
  Given the database connection fails
  When a user requests their profile
  Then cached data should be served
  And a warning "Using cached data" should be shown
  And the system should attempt reconnection

@quality:reliability
Scenario: Transaction rollback on partial failure
  Given a multi-step order process
  When payment succeeds but inventory update fails
  Then the payment should be refunded
  And the order should be cancelled
  And no inventory should be reserved
  And the user should be notified of the failure
```

## Best Practices Summary

### Do's

1. **Write for humans** - Use natural business language
2. **Be specific** - Clear, testable outcomes
3. **Stay declarative** - Focus on what, not how
4. **Keep independent** - Scenarios shouldn't depend on each other
5. **Use tags** - Organize and filter effectively
6. **Test edge cases** - Not just happy paths
7. **Link to requirements** - Maintain traceability
8. **Keep scenarios focused** - One behavior per scenario
9. **Use consistent terminology** - Same words for same concepts
10. **Make them maintainable** - Avoid brittle implementation details

### Don'ts

1. **Avoid UI details** - Don't couple to specific widgets
2. **Don't use technical jargon** - Write for domain experts
3. **Don't write novels** - Keep scenarios concise
4. **Don't repeat yourself** - Use Background for common setup
5. **Don't test implementation** - Test behavior
6. **Don't make assumptions** - Explicit preconditions
7. **Don't skip negative cases** - Test failures too
8. **Don't forget tags** - Tag for organization
9. **Don't mix concerns** - One scenario, one thing
10. **Don't ignore maintainability** - Scenarios evolve with system

## Gherkin Anti-Patterns

### Anti-Pattern: Imperative Steps

**Bad:**
```gherkin
When I open the login page
And I type "user" into the username field
And I type "pass" into the password field
And I click the "Login" button
```

**Good:**
```gherkin
When the user logs in with valid credentials
```

### Anti-Pattern: Too Many Scenarios

**Bad:**
```gherkin
Feature: Calculator with 50 scenarios for every operation
```

**Good:**
```gherkin
Feature: Calculator
  Scenario Outline: Basic operations
    When I calculate <a> <op> <b>
    Then the result should be <result>

    Examples:
      | a | op | b | result |
      | 2 | +  | 3 | 5      |
      | 5 | -  | 3 | 2      |
      | 4 | *  | 3 | 12     |
```

### Anti-Pattern: Testing Multiple Things

**Bad:**
```gherkin
Scenario: User registration and login and profile update
  Given I register a user
  When I login
  And I update my profile
  And I upload an avatar
  Then everything should work
```

**Good:**
Split into separate scenarios:
```gherkin
Scenario: User registration
  ...

Scenario: User login after registration
  ...

Scenario: Profile update after login
  ...
```

### Anti-Pattern: Vague Assertions

**Bad:**
```gherkin
Then it should work
Then the data should be correct
Then the system should be good
```

**Good:**
```gherkin
Then the user should be created with ID 12345
Then the response status should be 200
Then the product should have name "Laptop" and price 1000
```

## Related Documentation

See also:
- `references/api.md` - Scenari API reference
- Main skill: `scenari-specs.md` - Workflow and best practices
