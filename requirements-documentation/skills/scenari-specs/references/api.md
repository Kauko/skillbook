# Scenari API Reference

Complete reference for Scenari v2 API macros and functions.

## Installation

### deps.edn
```clojure
{io.defsquare/scenari {:mvn/version "2.0.2"}}
```

### project.clj
```clojure
[io.defsquare/scenari "2.0.2"]
```

### Import
```clojure
(ns my-test.core
  (:require [scenari.v2.core :as scenari :refer [defgiven defwhen defthen deffeature]]
            [scenari.v2.test :refer [run-feature]]))
```

## Core Macros

### deffeature

Defines a specification feature by parsing a Gherkin feature file.

**Syntax:**
```clojure
(deffeature feature-name "path/to/feature.txt")
(deffeature feature-name "path/to/feature.txt" options-map)
```

**Parameters:**
- `feature-name` - Symbol naming the feature (used for test execution)
- `path` - String path to feature file (relative or absolute)
- `options-map` - Optional map with:
  - `:default-scenario-state` - Initial state map for all scenarios (default: `{}`)
  - `:pre-run` - Vector of functions to run before feature execution
  - `:post-run` - Vector of functions to run after feature execution
  - `:pre-scenario-run` - Vector of functions to run before each scenario
  - `:post-scenario-run` - Vector of functions to run after each scenario

**Returns:**
Parsed specification as Clojure data structure.

**Examples:**

Basic usage:
```clojure
(deffeature product-spec "test/features/product.feature")
```

With initial state:
```clojure
(deffeature product-spec "test/features/product.feature"
  {:default-scenario-state {:db-conn (create-connection)
                            :base-url "http://localhost:3000"}})
```

With lifecycle hooks:
```clojure
(defn setup-db [] (migrate-db!))
(defn cleanup-db [] (rollback-db!))
(defn before-scenario [] (clear-test-data!))
(defn after-scenario [] (log-scenario-results!))

(deffeature product-spec "test/features/product.feature"
  {:pre-run [#'setup-db]
   :post-run [#'cleanup-db]
   :pre-scenario-run [#'before-scenario]
   :post-scenario-run [#'after-scenario]
   :default-scenario-state {:user-id nil :product-id nil}})
```

### defgiven

Defines a Given step (precondition setup).

**Syntax:**
```clojure
(defgiven "sentence with {string} parameters" [state & captured-params] body)
```

**Parameters:**
- `sentence` - String with matcher placeholders (`{string}`, `{int}`, etc.)
- `[state & captured-params]` - Vector where:
  - First element: output from previous step (nil for first step)
  - Remaining elements: captured values from sentence matchers
- `body` - Implementation code

**Returns:**
Map/value passed as input to next step.

**Examples:**

Simple precondition:
```clojure
(defgiven "the database is clean"
  [_]
  (clear-db!)
  {:db-state :clean})
```

With parameters:
```clojure
(defgiven "a user with email {string} exists"
  [state email]
  (let [user (create-user! {:email email})]
    (assoc state :current-user user)))
```

Multiple parameters:
```clojure
(defgiven "a product with name {string} and price {int}"
  [state name price]
  (let [product (create-product! {:name name :price price})]
    (assoc state :product product)))
```

With EDN data:
```clojure
(defgiven "a user with data {string}"
  [state user-data-edn]
  (let [user-data (read-string user-data-edn)
        user (create-user! user-data)]
    (assoc state :user user)))
```

Chaining state:
```clojure
(defgiven "the system is running"
  [_]
  {:system-state :running})

(defgiven "authentication is enabled"
  [{:keys [system-state] :as state}]
  (when (= system-state :running)
    (enable-auth!)
    (assoc state :auth-enabled true)))
```

### defwhen

Defines a When step (action execution).

**Syntax:**
```clojure
(defwhen "sentence with {string} parameters" [state & captured-params] body)
```

**Parameters:**
Same as `defgiven`.

**Returns:**
Map/value passed as input to next step (typically includes action results).

**Examples:**

Simple action:
```clojure
(defwhen "I create a product"
  [{:keys [current-user] :as state}]
  (let [product (api/create-product! {:name "test" :user-id (:id current-user)})]
    (assoc state :created-product product)))
```

With parameters:
```clojure
(defwhen "I login with email {string} and password {string}"
  [state email password]
  (let [result (auth/login {:email email :password password})]
    (assoc state :auth-result result)))
```

HTTP request:
```clojure
(defwhen "I send a POST request to {string} with data {string}"
  [{:keys [base-url] :as state} path data-edn]
  (let [data (read-string data-edn)
        response (http/post (str base-url path) {:json-body data})]
    (assoc state :response response)))
```

Multiple operations:
```clojure
(defwhen "I create {int} products"
  [state count]
  (let [products (repeatedly count #(create-product!))]
    (assoc state :products products)))
```

### defthen

Defines a Then step (assertion/verification).

**Syntax:**
```clojure
(defthen "sentence with {string} parameters" [state & captured-params] body)
```

**Parameters:**
Same as `defgiven` and `defwhen`.

**Returns:**
Typically returns state unchanged or throws assertion failure.

**Examples:**

Simple assertion:
```clojure
(defthen "the product should be created"
  [{:keys [created-product]}]
  (is (some? (:id created-product)))
  (is (= (:status created-product) :active)))
```

With expected value:
```clojure
(defthen "the status code should be {int}"
  [{:keys [response]} expected-code]
  (is (= (:status response) expected-code)))
```

Complex verification:
```clojure
(defthen "the response should contain product with name {string}"
  [{:keys [response]} expected-name]
  (let [products (:body response)]
    (is (some #(= (:name %) expected-name) products))))
```

Database verification:
```clojure
(defthen "the database should contain user with email {string}"
  [_ email]
  (let [user (db/find-user-by-email email)]
    (is (some? user))
    (is (= (:email user) email))))
```

Multiple assertions:
```clojure
(defthen "the authentication should succeed"
  [{:keys [auth-result]}]
  (is (= (:status auth-result) :success))
  (is (contains? auth-result :token))
  (is (not (nil? (:token auth-result))))
  (is (= (:token-type auth-result) "Bearer")))
```

## Execution Functions

### run-feature (core)

Executes feature and returns detailed execution report.

**Namespace:** `scenari.v2.core`

**Syntax:**
```clojure
(run-feature #'feature-var)
```

**Returns:**
Specification map with execution details including:
- `:scenarios` - List of executed scenarios
  - `:steps` - List of executed steps
    - `:status` - `:pending`, `:success`, or `:failed`
    - `:input-state` - State before step execution
    - `:output-state` - State after step execution
    - `:exception` - Exception if failed

**Example:**
```clojure
(let [result (run-feature #'product-spec)]
  (println "Scenarios:" (count (:scenarios result)))
  (println "Passed:" (count (filter #(= (:status %) :success) (:scenarios result)))))
```

### run-feature (test)

Executes feature and outputs formatted test results.

**Namespace:** `scenari.v2.test`

**Syntax:**
```clojure
(run-feature #'feature-var)
```

**Output:**
Human-readable test results compatible with clojure.test reporting.

**Example:**
```clojure
(require '[scenari.v2.test :refer [run-feature]])

(run-feature #'product-spec)
;; Output:
;; Testing product-spec
;;
;; Scenario: create a new product
;;   When I create a new product with name "iphone" ... OK
;;   Then the product should have an id ... OK
;;
;; Ran 1 scenarios, 2 steps, 0 failures
```

## Sentence Matchers

### Built-in Matchers

#### {string}
Captures quoted strings.

**Example:**
```gherkin
When I create user with name "John Doe"
```

```clojure
(defwhen "I create user with name {string}"
  [state name]
  ;; name = "John Doe" (without quotes)
  ...)
```

#### {int}
Captures integer numbers.

**Example:**
```gherkin
Then the price should be 100
```

```clojure
(defthen "the price should be {int}"
  [state expected-price]
  ;; expected-price = 100 (as integer)
  ...)
```

### EDN Data Evaluation

Parameters wrapped in `{}` are evaluated as EDN data. If evaluation produces a collection, it's passed as structured data.

**Example:**
```gherkin
When I create user with data {:name "John" :age 30}
```

```clojure
(defwhen "I create user with data {string}"
  [state user-data]
  ;; user-data = {:name "John" :age 30} (evaluated map)
  ...)
```

**JSON-like structures:**
```gherkin
When I send request with body {"name": "John", "age": 30}
```

```clojure
(defwhen "I send request with body {string}"
  [state body-data]
  ;; body-data as parsed data structure
  ...)
```

## Step Chaining & State Management

### State Flow

Each step receives the output of the previous step as its first parameter:

```clojure
(defgiven "initial state"
  [_]
  {:counter 0})

(defwhen "increment counter"
  [{:keys [counter] :as state}]
  (assoc state :counter (inc counter)))

(defthen "counter should be {int}"
  [{:keys [counter]} expected]
  (is (= counter expected)))
```

Corresponding scenario:
```gherkin
Scenario: counter increment
Given initial state
When increment counter
Then counter should be 1
```

### State Destructuring

Use destructuring for clean state access:

```clojure
(defwhen "user {string} creates product {string}"
  [{:keys [users products] :as state} user-name product-name]
  (let [user (get users user-name)
        product (create-product! user product-name)]
    (assoc-in state [:products product-name] product)))
```

### Nil State Handling

First Given step receives `nil` as state:

```clojure
(defgiven "the system is initialized"
  [_]  ;; Use _ to ignore nil
  {:system-ready true
   :users []
   :products []})
```

## Lifecycle Hooks

### Hook Functions

Hook functions receive no parameters and return no value:

```clojure
(defn setup-database []
  (println "Setting up test database...")
  (migrate-db!)
  (seed-test-data!))

(defn teardown-database []
  (println "Cleaning up test database...")
  (drop-all-tables!))
```

### Hook Execution Order

1. `:pre-run` hooks (before feature)
2. For each scenario:
   - `:pre-scenario-run` hooks
   - Scenario steps execution
   - `:post-scenario-run` hooks
3. `:post-run` hooks (after feature)

### Multiple Hooks

Hooks execute in order defined:

```clojure
(deffeature my-spec "spec.feature"
  {:pre-run [#'setup-db #'setup-cache #'setup-http-server]
   :post-run [#'cleanup-http-server #'cleanup-cache #'cleanup-db]})
```

## Error Handling

### Undefined Steps

When Scenari encounters undefined steps, it prints skeleton code:

**Scenario:**
```gherkin
When I perform action with "value" and 123
```

**Console output:**
```clojure
Missing step definition:
(defwhen "I perform action with {string} and {int}"
  [state arg0 arg1]
  (do "something"))
```

### Step Failures

Failed assertions propagate normally through clojure.test:

```clojure
(defthen "status should be ok"
  [{:keys [response]}]
  (is (= (:status response) 200)))
  ;; Assertion failure reported by clojure.test
```

### Exception Handling

Uncaught exceptions mark step as `:failed`:

```clojure
(defwhen "I call external API"
  [state]
  (try
    (let [response (http/get "http://external-api.com")]
      (assoc state :api-response response))
    (catch Exception e
      (assoc state :error e))))
```

## Status Values

Steps report one of three statuses:

- **`:pending`** - Step not executed (no matching definition found)
- **`:success`** - Step completed without errors
- **`:failed`** - Assertion failed or exception thrown

## Best Practices

### 1. Use Descriptive State Keys

```clojure
;; Good
{:authenticated-user user
 :created-product product
 :http-response response}

;; Avoid
{:u user
 :p product
 :r response}
```

### 2. Chain State Immutably

```clojure
;; Good
(defwhen "step"
  [state]
  (assoc state :new-key value))

;; Avoid mutation
(defwhen "step"
  [state]
  (swap! some-atom assoc :key value))
```

### 3. Separate Setup from Action

```clojure
;; Good - separate Given and When
(defgiven "user exists" [_] {:user (create-user!)})
(defwhen "user logs in" [{:keys [user]}] (login! user))

;; Avoid combining in When
(defwhen "user logs in" [_]
  (let [user (create-user!)]  ;; Setup should be in Given
    (login! user)))
```

### 4. Use Meaningful Assertions

```clojure
;; Good
(defthen "response should be successful"
  [{:keys [response]}]
  (is (= (:status response) 200) "Expected HTTP 200 OK")
  (is (contains? response :body) "Response should have body"))

;; Less clear
(defthen "it works"
  [{:keys [response]}]
  (is response))
```

### 5. Keep Steps Focused

Each step should do one thing:

```clojure
;; Good - focused steps
(defwhen "I create a product" ...)
(defwhen "I publish the product" ...)

;; Avoid - doing too much
(defwhen "I create and publish a product" ...)
```

## Kaocha Integration

### Configuration

Create `tests.edn`:

```clojure
#kaocha/v1
{:tests [{:id :scenario
          :type :kaocha.type/scenari
          :kaocha/source-paths ["src"]
          :kaocha/test-paths ["test/scenario"]
          :kaocha.type.scenari/glue-paths ["test/scenario/glue"]}]}
```

### REPL Usage

```clojure
(require '[kaocha.repl :as k])

;; Run all scenario tests
(k/run :scenario)

;; Run specific feature
(k/run :scenario {:kaocha.filter/focus ['my.test.namespace/my-feature]})
```

### Command Line

```bash
# Run all tests
clojure -M:kaocha :scenario

# Run with specific reporter
clojure -M:kaocha :scenario --reporter documentation
```

## Troubleshooting

### Issue: Steps Not Found

**Problem:** Scenari reports "Missing step definition" for defined steps.

**Solutions:**
1. Check namespace is required: `(require '[my.test.steps])`
2. Verify sentence matcher syntax exactly matches
3. Ensure `{string}` and `{int}` placeholders are correct
4. Check for typos in step sentence

### Issue: State Not Flowing Between Steps

**Problem:** Next step receives nil or incorrect state.

**Solutions:**
1. Ensure previous step returns state map
2. Check for correct state key usage
3. Verify destructuring patterns match state structure
4. Use `run-feature` from `scenari.v2.core` to inspect state transitions

### Issue: EDN Data Not Parsing

**Problem:** Parameters passed as strings instead of data structures.

**Solutions:**
1. Ensure valid EDN syntax in scenario
2. Check for proper quoting and escaping
3. Use explicit `read-string` if needed
4. Verify data structure format

### Issue: Hooks Not Executing

**Problem:** Lifecycle hooks don't run.

**Solutions:**
1. Use var references: `#'my-fn` not `my-fn`
2. Check hook keys: `:pre-run`, `:post-run`, etc.
3. Verify functions exist and are accessible
4. Add print statements to confirm execution

## Related Documentation

See also:
- `references/gherkin.md` - Gherkin syntax reference
- Main skill: `scenari-specs.md` - Workflow and best practices
