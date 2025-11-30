---
name: scenari-specs
description: Use when user wants to write BDD tests, create Gherkin feature files, or implement executable specifications in Clojure.
requires:
  tools: []
  skills: []
  deps: [io.defsquare/scenari]
---

# Scenari Behavioral Specifications

## Prerequisites

```bash
grep -q "scenari" deps.edn || echo "Add scenari dependency - see below"
mkdir -p test/features vault/specifications/features
```

Add if missing:
```clojure
{:deps {io.defsquare/scenari {:mvn/version "2.0.2"}}}
```

## Quick Start

### 1. Write Feature File

`test/features/auth.feature`:
```gherkin
Feature: User Authentication

  Scenario: Successful login
    Given a registered user "user@example.com" with password "secret"
    When the user logs in with "user@example.com" and "secret"
    Then the login should succeed
    And a session token is returned

  Scenario: Invalid password
    Given a registered user "user@example.com" with password "secret"
    When the user logs in with "user@example.com" and "wrong"
    Then the login should fail with "invalid_credentials"
```

### 2. Write Step Definitions

`test/auth_steps.clj`:
```clojure
(ns auth-steps
  (:require [scenari.core :refer [defgiven defwhen defthen]]))

(defgiven #"a registered user \"(.+)\" with password \"(.+)\""
  [state email password]
  (assoc state :user {:email email :password password}))

(defwhen #"the user logs in with \"(.+)\" and \"(.+)\""
  [state email password]
  (let [result (auth/login email password)]
    (assoc state :result result)))

(defthen #"the login should succeed"
  [state]
  (assert (= :success (:status (:result state))))
  state)
```

### 3. Run Tests

```clojure
(require '[scenari.core :as scenari])
(scenari/run-feature "test/features/auth.feature")
```

Or with Kaocha:
```clojure
;; tests.edn
{:tests [{:type :scenari :source-paths ["test/features"]}]}
```

## Gherkin Patterns

| Pattern | Purpose |
|---------|---------|
| `Scenario Outline` | Data-driven tests with `Examples` table |
| `Background` | Common setup for all scenarios |
| `@tag` | Filter and organize scenarios |
| `"""` | Doc strings for multiline input |
| `\|` | Data tables for structured input |

## State Flow

Steps receive and return state (immutable map):
```clojure
{:user {...}      ;; Set by Given
 :result {...}    ;; Set by When
 :assertions [...]} ;; Added by Then
```

## Reference Documentation

- `references/api.md` - Complete Scenari API
- `references/gherkin.md` - Gherkin syntax guide

## Success Criteria

- [ ] Feature files created in `test/features/`
- [ ] Step definitions implemented
- [ ] `scenari/run-feature` executes without errors

## Linking Requirements

When creating Gherkin specs:
- Link to components: `Components: [[components/button]], [[components/form]]`
- Link to ADRs: `Decision: [[decisions/0010-validation-approach]]`
- Link to beads task: `Task: [[beads/FEAT-456]]`
- Link to arc42 runtime section: `See [[arc42/06-runtime]]`

## Related Skills

- `iso25010-quality` - Link scenarios to quality requirements
- `recife-modeling` - Translate critical scenarios to formal specs
