# Overarch Relationships Reference

This document provides comprehensive details on all relationship types available in Overarch for connecting model elements.

## Common Relationship Properties

All relationships share these common properties:

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `:el` | keyword | Yes | Relationship type (e.g., `:request`, `:rel`) |
| `:id` | namespaced keyword | Yes | Unique identifier for the relationship |
| `:from` | keyword | Yes | Source element ID |
| `:to` | keyword | Yes | Target element ID |
| `:name` | string | No | Relationship label/description |
| `:desc` | string | No | Additional description |
| `:tech` | vector of strings | No | Technologies used (e.g., `["REST" "HTTPS"]`) |
| `:tags` | set of keywords | No | Categorization tags |

## Architecture Model Relations

### Request (Synchronous Call)

Represents a synchronous request from one component to another.

**Element Type:** `:request`

**Semantics:** The source waits for a response from the target.

**Example:**
```clojure
{:el :request
 :id :banking/web-to-api
 :from :banking.internet-banking/web-app
 :to :banking.internet-banking/api-server
 :name "Makes API calls to"
 :tech ["REST" "HTTPS" "JSON"]
 :desc "Fetches account data and executes transactions"}
```

**Common Use Cases:**
- HTTP REST API calls
- gRPC calls
- GraphQL queries
- RPC calls
- Synchronous method invocations

### Response (Synchronous Reply)

Represents a synchronous response to a request.

**Element Type:** `:response`

**Semantics:** Reply from target back to source.

**Example:**
```clojure
{:el :response
 :id :banking/api-to-web-response
 :from :banking.internet-banking/api-server
 :to :banking.internet-banking/web-app
 :name "Returns data to"
 :tech ["JSON"]
 :desc "Returns account details and transaction results"}
```

**Note:** Often implicit in diagrams; request relationships typically imply bidirectional communication.

### Send (Asynchronous Point-to-Point)

Represents asynchronous message sending to a specific target.

**Element Type:** `:send`

**Semantics:** Fire-and-forget or eventual delivery without waiting for immediate response.

**Example:**
```clojure
{:el :send
 :id :banking/send-email
 :from :banking.internet-banking/api-server
 :to :email/email-service
 :name "Sends notification email via"
 :tech ["SMTP"]
 :desc "Sends account alerts and transaction confirmations"}
```

**Common Use Cases:**
- Email sending
- SMS notifications
- Direct message queue sends
- Event publishing to specific consumers

### Publish (Asynchronous Broadcast)

Represents publishing a message to multiple potential subscribers.

**Element Type:** `:publish`

**Semantics:** Broadcast message to topic/channel; one-to-many pattern.

**Example:**
```clojure
{:el :publish
 :id :banking/publish-transaction-event
 :from :banking.transactions/transaction-service
 :to :messaging/event-bus
 :name "Publishes transaction events to"
 :tech ["Apache Kafka" "Avro"]
 :desc "Publishes completed transaction events for downstream processing"}
```

**Common Use Cases:**
- Event-driven architectures
- Pub/sub messaging patterns
- Event sourcing
- Change data capture

### Subscribe (Asynchronous Subscription)

Represents subscribing to messages from a topic/queue.

**Element Type:** `:subscribe`

**Semantics:** Receive messages published to a topic; many-to-one pattern.

**Example:**
```clojure
{:el :subscribe
 :id :banking/subscribe-transaction-events
 :from :banking.analytics/analytics-service
 :to :messaging/event-bus
 :name "Subscribes to transaction events from"
 :tech ["Apache Kafka" "Avro"]
 :desc "Consumes transaction events for real-time analytics"}
```

**Common Use Cases:**
- Event consumers
- Message queue consumers
- Stream processing
- Event handlers

### Dataflow

Represents data flowing between components without specifying the mechanism.

**Element Type:** `:dataflow`

**Semantics:** Generic data transfer; mechanism unspecified.

**Example:**
```clojure
{:el :dataflow
 :id :banking/data-sync
 :from :banking.mainframe/legacy-system
 :to :banking.internet-banking/database
 :name "Syncs account data to"
 :tech ["ETL" "JDBC"]
 :desc "Nightly batch synchronization of account balances"}
```

**Common Use Cases:**
- ETL processes
- Data synchronization
- Batch data transfers
- Data replication

### Generic Relation

Generic relationship when specific semantics don't fit other types.

**Element Type:** `:rel`

**Semantics:** User-defined relationship.

**Example:**
```clojure
{:el :rel
 :id :banking/monitors
 :from :monitoring/prometheus
 :to :banking.internet-banking/api-server
 :name "Monitors"
 :tech ["Prometheus" "HTTP"]
 :desc "Scrapes metrics endpoint"}
```

**Common Use Cases:**
- Monitoring relationships
- Configuration relationships
- Dependencies not covered by specific types
- Cross-cutting concerns

## Hierarchical Relations

### Contained-In

Represents hierarchical containment when elements are defined in separate files.

**Element Type:** `:contained-in`

**Semantics:** Child-parent relationship in the architectural hierarchy.

**Example:**
```clojure
{:el :contained-in
 :id :banking/container-in-system
 :from :banking.internet-banking/web-app
 :to :banking.internet-banking/internet-banking-system}
```

**Common Use Cases:**
- Cross-file model composition
- Explicit hierarchy when not using nested `:ct` collections
- Linking components to containers across files
- Linking containers to systems across files

**Note:** Prefer nested `:ct` collections when possible; use `:contained-in` for cross-file references.

## Traceability Relations

### Implements

Links implementation elements to requirements or specifications.

**Element Type:** `:implements`

**Example:**
```clojure
{:el :implements
 :id :banking/api-implements-transfer
 :from :banking.api/transfer-component
 :to :requirements/money-transfer}
```

**Common Use Cases:**
- Linking components to requirements
- Tracing features to implementations
- Compliance tracking

### Reference

Generic reference from one element to another.

**Element Type:** `:ref`

**Example:**
```clojure
{:el :ref
 :id :banking/api-references-schema
 :from :banking.api/transfer-component
 :to :schemas/transfer-schema}
```

## Use Case Relations

### Uses

Actor or use case using another use case.

**Element Type:** `:uses`

**Example:**
```clojure
{:el :uses
 :id :use-cases/customer-uses-login
 :from :use-cases/customer
 :to :use-cases/login}
```

### Include

Use case includes another use case (mandatory).

**Element Type:** `:include`

**Example:**
```clojure
{:el :include
 :id :use-cases/transfer-includes-auth
 :from :use-cases/transfer-money
 :to :use-cases/authenticate}
```

### Extends

Use case extends another use case (optional).

**Element Type:** `:extends`

**Example:**
```clojure
{:el :extends
 :id :use-cases/premium-extends-standard
 :from :use-cases/premium-transfer
 :to :use-cases/standard-transfer}
```

### Generalizes

Generalization relationship between use cases.

**Element Type:** `:generalizes`

## Code Model Relations

### Association

Bi-directional association between classes.

**Element Type:** `:association`

**Example:**
```clojure
{:el :association
 :id :code/account-customer-assoc
 :from :code/account
 :to :code/customer}
```

### Aggregation

Aggregation relationship (weak ownership).

**Element Type:** `:aggregation`

**Example:**
```clojure
{:el :aggregation
 :id :code/bank-accounts-agg
 :from :code/bank
 :to :code/account}
```

### Composition

Composition relationship (strong ownership).

**Element Type:** `:composition`

**Example:**
```clojure
{:el :composition
 :id :code/order-line-items
 :from :code/order
 :to :code/line-item}
```

### Inheritance

Inheritance/extends relationship.

**Element Type:** `:inheritance`

**Example:**
```clojure
{:el :inheritance
 :id :code/savings-extends-account
 :from :code/savings-account
 :to :code/account}
```

### Implementation

Class implements interface.

**Element Type:** `:implementation`

**Example:**
```clojure
{:el :implementation
 :id :code/account-service-impl
 :from :code/account-service-impl
 :to :code/account-service}
```

### Dependency

Dependency between code elements.

**Element Type:** `:dependency`

**Example:**
```clojure
{:el :dependency
 :id :code/service-depends-repo
 :from :code/account-service
 :to :code/account-repository}
```

## Deployment Relations

### Link

Infrastructure connection between nodes.

**Element Type:** `:link`

**Example:**
```clojure
{:el :link
 :id :infra/app-to-db-network
 :from :infrastructure/app-server
 :to :infrastructure/db-server
 :name "Private network connection"
 :tech ["VPC" "Private Subnet"]}
```

### Deployed-To

Container deployed to infrastructure node.

**Element Type:** `:deployed-to`

**Example:**
```clojure
{:el :deployed-to
 :id :deploy/api-to-k8s
 :from :banking.internet-banking/api-server
 :to :infrastructure/kubernetes-cluster}
```

## Concept Model Relations

### Is-A

Taxonomic relationship (specialization).

**Element Type:** `:is-a`

**Example:**
```clojure
{:el :is-a
 :id :concepts/savings-is-account
 :from :concepts/savings-account
 :to :concepts/account}
```

### Has

Composition or part-of relationship.

**Element Type:** `:has`

**Example:**
```clojure
{:el :has
 :id :concepts/account-has-balance
 :from :concepts/account
 :to :concepts/balance}
```

## Organization Relations

### Responsible-For

Organizational responsibility.

**Element Type:** `:responsible-for`

**Example:**
```clojure
{:el :responsible-for
 :id :org/backend-team-owns-api
 :from :organization/backend-team
 :to :banking.internet-banking/api-server}
```

### Collaborates-With

Team collaboration relationship.

**Element Type:** `:collaborates-with`

**Example:**
```clojure
{:el :collaborates-with
 :id :org/frontend-backend-collab
 :from :organization/frontend-team
 :to :organization/backend-team}
```

### Role-In

Role assignment in organization or process.

**Element Type:** `:role-in`

## Process Model Relations

### Required-For

Requirement dependency.

**Element Type:** `:required-for`

### Input-Of

Data/artifact input to process.

**Element Type:** `:input-of`

**Example:**
```clojure
{:el :input-of
 :id :process/contract-input
 :from :artifacts/customer-contract
 :to :processes/onboarding}
```

### Output-Of

Data/artifact output from process.

**Element Type:** `:output-of`

**Example:**
```clojure
{:el :output-of
 :id :process/account-output
 :from :artifacts/account-record
 :to :processes/onboarding}
```

## Layout Control

### Direction

Control relationship rendering direction in views.

**Property:** `:direction`

**Values:** `:up`, `:down`, `:left`, `:right`

**Example:**
```clojure
{:el :request
 :id :banking/web-to-api
 :from :banking.internet-banking/web-app
 :to :banking.internet-banking/api-server
 :name "Makes API calls to"
 :direction :down}
```

**Use Cases:**
- Improve diagram readability
- Enforce specific layout patterns
- Match existing documentation conventions

## Relationship Naming Best Practices

### Naming Conventions

1. **Active Voice**: "Calls", "Sends", "Publishes" (not "Is called by")
2. **Present Tense**: "Makes requests to" (not "Made requests to")
3. **Descriptive**: "Fetches account balances from" (not just "Uses")
4. **Business Context**: Include what data/action when relevant

### ID Naming

Use descriptive IDs that indicate relationship:
```clojure
:system/from-to-action         ; Good: :banking/web-to-api-request
:system/component1-component2  ; Good: :banking/api-db-connection
```

### Technology Stack

Be specific about communication mechanisms:
```clojure
:tech ["REST" "HTTPS" "JSON"]        ; API calls
:tech ["gRPC" "Protocol Buffers"]    ; RPC
:tech ["Apache Kafka" "Avro"]        ; Event streaming
:tech ["WebSocket" "Socket.IO"]      ; Real-time
:tech ["GraphQL" "HTTPS"]            ; GraphQL API
```

## Relationship Patterns

### Synchronous Request-Response
```clojure
[{:el :request
  :id :app/web-to-api
  :from :app/web-client
  :to :app/api-server
  :name "Fetches data from"
  :tech ["REST" "HTTPS"]}

 {:el :request
  :id :app/api-to-db
  :from :app/api-server
  :to :app/database
  :name "Queries"
  :tech ["SQL" "PostgreSQL"]}]
```

### Event-Driven Architecture
```clojure
[{:el :publish
  :id :app/service-publishes-event
  :from :app/order-service
  :to :app/event-bus
  :name "Publishes order events to"
  :tech ["Kafka"]}

 {:el :subscribe
  :id :app/inventory-subscribes
  :from :app/inventory-service
  :to :app/event-bus
  :name "Subscribes to order events from"
  :tech ["Kafka"]}

 {:el :subscribe
  :id :app/shipping-subscribes
  :from :app/shipping-service
  :to :app/event-bus
  :name "Subscribes to order events from"
  :tech ["Kafka"]}]
```

### Layered Architecture
```clojure
[{:el :request
  :id :app/controller-to-service
  :from :app/account-controller
  :to :app/account-service
  :name "Calls"
  :direction :down}

 {:el :request
  :id :app/service-to-repository
  :from :app/account-service
  :to :app/account-repository
  :name "Uses"
  :direction :down}

 {:el :request
  :id :app/repository-to-db
  :from :app/account-repository
  :to :app/database
  :name "Queries"
  :tech ["JDBC" "SQL"]
  :direction :down}]
```

### Microservices Communication
```clojure
[{:el :request
  :id :services/sync-call
  :from :services/order-service
  :to :services/inventory-service
  :name "Checks inventory via"
  :tech ["gRPC"]}

 {:el :publish
  :id :services/async-event
  :from :services/order-service
  :to :messaging/event-bus
  :name "Publishes order created event"}

 {:el :request
  :id :services/through-gateway
  :from :services/api-gateway
  :to :services/order-service
  :name "Routes requests to"
  :tech ["HTTP" "REST"]}]
```

## Bidirectional Relationships

When modeling bidirectional communication, you have two options:

### Option 1: Single Bidirectional Relation (Simple)
```clojure
{:el :request
 :id :app/web-api-communication
 :from :app/web-app
 :to :app/api-server
 :name "Exchanges data with"
 :tech ["REST" "HTTPS"]}
```

### Option 2: Two Explicit Relations (Detailed)
```clojure
[{:el :request
  :id :app/web-to-api
  :from :app/web-app
  :to :app/api-server
  :name "Makes requests to"}

 {:el :response
  :id :app/api-to-web
  :from :app/api-server
  :to :app/web-app
  :name "Returns responses to"}]
```

**Recommendation:** Use single bidirectional relation for most cases; use two relations when response path has different characteristics (e.g., different technology, security requirements).

## Troubleshooting Relationships

### Common Issues

**Issue:** Relationship not appearing in diagram
- **Check:** Ensure both `:from` and `:to` IDs exist in model
- **Check:** Verify view includes both source and target elements
- **Check:** Check if relationship type is supported in view type

**Issue:** Crossed or messy relationship lines
- **Solution:** Use `:direction` property to guide layout
- **Solution:** Simplify view by splitting into multiple focused views
- **Solution:** Adjust PlantUML `:node-separation` and `:rank-separation`

**Issue:** Missing relationship labels
- **Solution:** Add `:name` property to relationship
- **Solution:** Check if view styling hides labels

## Advanced Relationship Features

### Conditional Relationships

Use tags to create conditional relationships that appear in specific views:

```clojure
{:el :request
 :id :app/monitoring-call
 :from :monitoring/prometheus
 :to :app/api-server
 :name "Monitors"
 :tags #{:monitoring :operational}}
```

Then filter in views:
```clojure
{:el :container-view
 :id :operational-view
 :title "Operational View"
 :spec {:selection {:tags #{:monitoring}}}}
```

### Security Annotations

Mark sensitive relationships:

```clojure
{:el :request
 :id :app/pii-data-flow
 :from :app/api-server
 :to :app/customer-db
 :name "Reads/writes customer data"
 :tech ["JDBC" "TLS 1.3"]
 :tags #{:pii :encrypted :audited}
 :desc "All customer PII access is logged for compliance"}
```
