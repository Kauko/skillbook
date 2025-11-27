# Overarch Model Elements Reference

This document provides comprehensive details on all element types available in Overarch for modeling software architecture.

## Common Element Properties

All model elements share these common properties:

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `:el` | keyword | Yes | Element type (e.g., `:system`, `:container`) |
| `:id` | namespaced keyword | Yes | Unique identifier (e.g., `:banking/web-app`) |
| `:name` | string | Yes | Human-readable display name |
| `:desc` | string | No | Short description shown in diagrams |
| `:doc` | string | No | Extended documentation (not shown in diagrams) |
| `:tech` | vector of strings | No | Technology stack (e.g., `["Java" "Spring Boot"]`) |
| `:tags` | set of keywords | No | Categorization tags for filtering |
| `:maturity` | keyword | No | Lifecycle status (`:proposed`, `:deprecated`) |
| `:external` | boolean | No | Mark as external system/component |

## Architecture Model Elements

### Person

Represents human actors or users who interact with the system.

**Element Type:** `:person`

**Properties:**
- All common properties
- No specific additional properties

**Example:**
```clojure
{:el :person
 :id :banking/personal-customer
 :name "Personal Banking Customer"
 :desc "A customer of the bank with personal accounts"
 :tags #{:external :customer}}
```

**Use Cases:**
- End users of the system
- Administrators
- Support staff
- External stakeholders

### System

Top-level software system boundary. Represents a deployable software system.

**Element Type:** `:system`

**Properties:**
- All common properties
- `:external` - Mark as external system (boolean)
- `:ct` - Container elements (nested collection)

**Example:**
```clojure
{:el :system
 :id :banking.internet-banking/internet-banking-system
 :name "Internet Banking System"
 :desc "Allows customers to view account balances and make payments"
 :tech ["Clojure" "React"]
 :external false}
```

**Use Cases:**
- Your main application system
- External systems (APIs, SaaS platforms)
- Legacy systems
- Third-party integrations

### Container

A separately deployable or executable unit (application, service, database, etc.) within a system.

**Element Type:** `:container`

**Properties:**
- All common properties
- `:subtype` - Specialization (`:database`, `:queue`)
- `:ct` - Component elements (nested collection)

**Subtypes:**
- `:database` - For data storage containers
- `:queue` - For message queues/streams

**Example:**
```clojure
{:el :container
 :id :banking.internet-banking/web-app
 :name "Web Application"
 :desc "Delivers static content and the Internet banking SPA"
 :tech ["React" "TypeScript" "Nginx"]
 :tags #{:frontend :spa}}

{:el :container
 :id :banking.internet-banking/database
 :name "Database"
 :desc "Stores user registration, authentication info, and account transactions"
 :tech ["PostgreSQL" "15.2"]
 :subtype :database}

{:el :container
 :id :messaging/event-bus
 :name "Event Bus"
 :desc "Message broker for async communication"
 :tech ["Apache Kafka"]
 :subtype :queue}
```

**Use Cases:**
- Web applications (SPAs, server-rendered)
- Backend services (REST APIs, GraphQL)
- Databases (SQL, NoSQL)
- Message queues/brokers
- Background workers
- Mobile applications

### Component

A grouping of related functionality within a container.

**Element Type:** `:component`

**Properties:**
- All common properties
- No specific additional properties

**Example:**
```clojure
{:el :component
 :id :banking.api/authentication-component
 :name "Authentication Component"
 :desc "Handles user login, JWT generation, and session management"
 :tech ["JWT" "OAuth2" "bcrypt"]
 :tags #{:security}}
```

**Use Cases:**
- Controllers/handlers
- Service layers
- Repository patterns
- Authentication/authorization modules
- Business logic modules

### Node

Infrastructure or deployment unit (physical or virtual machine, container runtime, etc.).

**Element Type:** `:node`

**Properties:**
- All common properties
- Can contain other nodes (nested deployment structure)
- Can host containers via `:deployed-to` relation

**Example:**
```clojure
{:el :node
 :id :infrastructure/prod-cluster
 :name "Production Kubernetes Cluster"
 :desc "Production deployment environment"
 :tech ["Kubernetes" "1.27"]}

{:el :node
 :id :infrastructure/database-server
 :name "Database Server"
 :desc "Dedicated PostgreSQL server"
 :tech ["AWS RDS" "PostgreSQL 15"]}
```

**Use Cases:**
- Physical servers
- Virtual machines
- Container orchestration platforms (Kubernetes, Docker Swarm)
- Cloud infrastructure (EC2, Azure VMs)
- Edge devices

## Use Case Model Elements

### Actor

Non-architectural actors for use case modeling (distinct from `:person` which is architectural).

**Element Type:** `:actor`

**Properties:**
- All common properties

**Example:**
```clojure
{:el :actor
 :id :use-cases/customer
 :name "Customer"
 :desc "Person using the banking system"}
```

### Use Case

Represents a user goal or system capability.

**Element Type:** `:use-case`

**Properties:**
- All common properties
- `:level` - Goal level (`:summary`, `:user-goal`, `:subfunction`)

**Example:**
```clojure
{:el :use-case
 :id :use-cases/transfer-money
 :name "Transfer Money"
 :desc "Transfer funds between accounts"
 :level :user-goal}
```

## Code Model Elements

### Package/Namespace

Code organization units.

**Element Types:** `:package` (Java/general), `:namespace` (Clojure)

**Example:**
```clojure
{:el :namespace
 :id :code/banking.core
 :name "banking.core"
 :desc "Core banking business logic"}
```

### Class/Interface/Protocol

Type definitions in object-oriented or functional paradigms.

**Element Types:** `:class`, `:interface`, `:protocol`

**Example:**
```clojure
{:el :class
 :id :code/account-service
 :name "AccountService"
 :desc "Manages account operations"}

{:el :interface
 :id :code/repository
 :name "Repository"
 :desc "Data access interface"}

{:el :protocol
 :id :code/handler-protocol
 :name "Handler"
 :desc "Request handler protocol"}
```

### Members

**Element Types:** `:enum`, `:field`, `:method`, `:function`

**Example:**
```clojure
{:el :method
 :id :code/get-balance
 :name "getBalance"
 :desc "Returns account balance"}

{:el :function
 :id :code/calculate-interest
 :name "calculate-interest"
 :desc "Calculates interest for account"}
```

## State Machine Model Elements

### State Machine

Container for state machine definitions.

**Element Type:** `:state-machine`

**Example:**
```clojure
{:el :state-machine
 :id :order/order-lifecycle
 :name "Order Lifecycle"
 :desc "Order processing state machine"}
```

### States

**Element Types:** `:state`, `:start-state`, `:end-state`, `:fork-state`, `:join-state`

**Example:**
```clojure
{:el :start-state
 :id :order/initial
 :name "Initial"}

{:el :state
 :id :order/pending
 :name "Pending"
 :desc "Order awaiting payment"}

{:el :state
 :id :order/confirmed
 :name "Confirmed"
 :desc "Payment received"}

{:el :end-state
 :id :order/complete
 :name "Complete"}
```

### Transition

State changes in state machines.

**Element Type:** `:transition`

**Properties:**
- `:from` - Source state ID
- `:to` - Target state ID
- `:name` - Trigger/condition

**Example:**
```clojure
{:el :transition
 :id :order/pay
 :from :order/pending
 :to :order/confirmed
 :name "Payment Received"}
```

## Concept Model Elements

### Concept

Domain concepts for conceptual modeling.

**Element Type:** `:concept`

**Example:**
```clojure
{:el :concept
 :id :domain/account
 :name "Account"
 :desc "A financial account held by a customer"}

{:el :concept
 :id :domain/transaction
 :name "Transaction"
 :desc "A financial transaction"}
```

## Organization Model Elements

### Organization Units

**Element Types:** `:organization`, `:org-unit`

**Example:**
```clojure
{:el :organization
 :id :company/acme-corp
 :name "ACME Corporation"
 :desc "Parent organization"}

{:el :org-unit
 :id :company/engineering
 :name "Engineering Department"
 :desc "Software development team"}
```

## Process Model Elements

Business process modeling elements.

**Element Types:** `:capability`, `:process`, `:artifact`, `:requirement`, `:decision`, `:information`, `:knowledge`

**Example:**
```clojure
{:el :capability
 :id :business/online-banking
 :name "Online Banking"
 :desc "Provide banking services online"}

{:el :process
 :id :business/onboard-customer
 :name "Customer Onboarding"
 :desc "Process for registering new customers"}

{:el :artifact
 :id :business/customer-contract
 :name "Customer Contract"
 :desc "Legal agreement with customer"}
```

## Element Naming Conventions

### ID Naming
- Use namespaced keywords: `:namespace/element-name`
- Use kebab-case: `:my-system/web-app`
- Organize by domain or layer: `:banking.frontend/web-app`, `:banking.backend/api`

### Display Names
- Use Title Case: "Web Application"
- Keep concise but descriptive
- Avoid abbreviations unless widely understood

### Technology Stack
- Be specific: `["PostgreSQL 15.2"]` not `["Database"]`
- Include version numbers when relevant
- List frameworks and languages: `["React" "TypeScript" "Vite"]`

## Element Organization Best Practices

1. **Hierarchical Structure**: Use namespaces to reflect system hierarchy
   - `:system-name/container-name/component-name`

2. **Consistent Typing**: Use subtypes (`:database`, `:queue`) consistently

3. **External Marking**: Always mark external systems with `:external true`

4. **Technology Documentation**: Include all relevant technologies in `:tech` vector

5. **Tagging Strategy**: Use tags for cross-cutting concerns
   - `:security`, `:monitoring`, `:public-api`, `:deprecated`

6. **Maturity Tracking**: Mark lifecycle status
   - `:proposed` - Planned but not implemented
   - `:deprecated` - Scheduled for removal

7. **Documentation Layers**:
   - `:desc` - Brief, diagram-friendly (1-2 sentences)
   - `:doc` - Extended documentation (multiple paragraphs, implementation details)

## Model Composition

Models can be split across multiple EDN files. Overarch loads all files from the model directory and composes them automatically.

**Example File Organization:**
```
models/
  actors.edn          - Person/actor elements
  systems.edn         - System boundaries
  containers.edn      - Container definitions
  infrastructure.edn  - Node definitions
  relationships.edn   - All relations
  views.edn           - View definitions
```

This allows team collaboration and modular model management.
