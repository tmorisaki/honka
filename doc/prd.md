# Honka — Product Requirements Document

> **A shared business context for humans, AI, and software.**

**Status:** v0.1  
**Product:** Honka  
**Document role:** Initial product definition and architectural baseline

---

## 1. Product Concept

Honka is a tool for designing, managing, and exposing the **Business Context** shared by humans, AI, and software.

Honka is not primarily an execution platform. Existing systems remain responsible for executing business operations, including Salesforce Flow/Apex, Slack agents, MCP servers, APIs, SAP, and custom applications.

Honka stores and exposes the meaning behind those implementations:

> **What does this work mean in this organization?**

The durable asset managed by Honka is the organization's own business world model: its processes, concepts, rules, data contracts, capabilities, and implementation mappings.

## 2. Name and Philosophy

The name **Honka** comes from the Japanese literary concept of **本歌 (honka)**, the source poem behind the technique of *honkadori*.

In honkadori, a new expression can invoke a much larger body of meaning because the reader understands the original poem behind it.

Honka applies the same idea to business systems.

```text
                 HONKA
            Business Context
                  |
        +---------+---------+
        |         |         |
      Human       AI     Software
```

A Flow, Apex class, AI agent, diagram, or API is an expression of business intent. Honka preserves the shared context behind those expressions.

> **The same work can have many expressions because they share the same Honka.**

## 3. Problem

Business meaning is currently fragmented across systems and people:

- Salesforce Flow and Apex
- APIs and integration code
- Slack and other collaboration tools
- spreadsheets
- documentation and wikis
- architecture diagrams
- prompts and agent instructions
- meeting decisions
- individual employees' knowledge

An AI asked to "change the contract sending feature" may understand the code but still not reliably know what a Contract means in this organization, why legal approval is required, which actors own each decision, which data is required, what business rules constrain the process, what capability "Send Contract" represents, or why CloudSign is currently used.

Honka makes this organization-specific context explicit, structured, versioned, and queryable.

## 4. Product Thesis

Foundation models will continue to improve. As reasoning improves, some value currently placed in prompt engineering, custom instructions, and narrowly tuned agent skills may decrease.

Organization-specific context is different. A model cannot intrinsically know what a particular company's concepts, rules, processes, and implementation decisions mean.

Therefore Honka optimizes for a durable asset:

> **Company-specific business context, not model-specific intelligence.**

## 5. Product Boundary: Context, Not Execution

Honka must not evolve by default into a workflow engine. Execution remains in the systems best suited to execute it.

```text
                         HONKA
                    Business Context
                          |
          +---------------+---------------+
          |               |               |
     Salesforce       Slack / Agent    Custom App
     Flow / Apex          MCP              API
```

Honka describes what a capability means, what it requires, and which implementation currently realizes it. It does not need to invoke that implementation itself.

This separates:

- **Context / Control Plane — Honka**
- **Execution Plane — existing systems**

This boundary is a core product principle.

## 6. Business Graph

The core representation is a **Semantic Business Graph**.

```text
Value Chain
    |
High-level Process
    |
Business Flow
```

### 6.1 Value Chain

Represents how the organization creates value. This layer is intentionally loose and suitable for broad, exploratory modeling.

### 6.2 High-level Process

Represents major business activities such as sales, contracting, fulfillment, or billing. This layer remains relatively flexible.

### 6.3 Business Flow

Represents concrete business behavior.

```text
Create Contract
      |
Legal Review
      |
Approval
      |
Send Contract
```

Business Flow is where stronger semantics begin.

## 7. Semantic Nodes

A Business Flow node can progressively acquire structured meaning.

A semantic node may contain:

- Description
- Actor / Role
- Data Contract
- Business Rules
- Capability
- Preconditions
- Postconditions
- Exceptions
- Implementation References

Not every field must be required.

> **Loose inside, strict at boundaries.**

Users should be able to begin with an informal model and progressively make it machine-readable.

## 8. Data Model

The Data Model is not embedded inside the Business Flow. Business Flow and Business Data Model are peer concepts.

```text
Business Flow <----> Business Data Model
```

Honka also separates business concepts from physical implementation.

```text
Business Data Model
        |
      mapping
        |
Physical Data Model
```

Example business model:

```text
Contract
 |- customer
 |- amount
 '- approvalStatus
```

Possible Salesforce mapping:

```text
Contract__c
Account__c
TotalAmount__c
ApprovalStatus__c
```

The Business Data Model must remain valid even if Salesforce or another implementation platform is replaced.

## 9. Data Contracts

A Business Flow node does not consume the entire Data Model. It declares the data boundary relevant to that operation.

```yaml
LegalReview:
  requires:
    - Contract.amount
    - Contract.customer

  writes:
    - Contract.legalReviewed
```

The compiler can detect inconsistencies such as:

- a referenced field does not exist;
- downstream requires data that upstream never supplies;
- incompatible data types;
- missing mappings;
- unresolved references.

Honka should support both authoring directions.

### Data-model-first

An existing model is available and a node selects the fields it requires, reads, or writes.

### Contract-first

A node declares a requirement such as `Contract.legalReviewed: boolean`. If the field does not exist, Honka can propose adding it to the Business Data Model.

## 10. Business Rules

Business Rules represent explicit constraints on business behavior.

Rules should be strongly typed and independently referenceable where practical. They are among the strictest semantic boundaries in Honka because they may be consumed by humans, AI, validation, testing, and implementation tooling.

## 11. Capabilities

A Capability represents **what the business can do**, independently of how that capability is implemented.

Example: `SendContract`.

Possible implementation references include Salesforce Apex, CloudSign MCP, REST API, Human Operation, or Agent.

```text
Business Capability != Implementation
```

Honka owns the semantic relationship between them, not necessarily their execution.

## 12. Semantic Compiler

Honka behaves in part like a **Business Context Compiler**.

```text
Metadata
   |
Parser
   |
AST
   |
Resolver
   |
Semantic Graph
   |
Validator
   |
IRs
```

The Semantic Graph is the core product representation. Neither the UI nor AI should treat raw YAML or JSON as the canonical conceptual model.

From the Semantic Graph, Honka can generate purpose-specific intermediate representations:

- Designer IR
- AI Context IR
- Test IR
- Platform IR
- Integration / Reference IR

## 13. AI Context Compilation

AI should not receive the entire repository by default.

For a request involving `SendContract`, Honka should compile a minimal relevant context containing the selected semantic node, nearby upstream and downstream nodes, referenced Data Contracts, Business Rules, Capabilities, and relevant implementation references.

This provides organization-specific context while avoiding unnecessary context-window consumption.

## 14. MCP Interface

Honka should expose its Business Context through an MCP server.

Initial conceptual tools include:

```text
find_process
get_context
get_rules
get_data_contract
get_capabilities
```

Example:

```text
User
  |
"Change the contract sending feature"
  |
Codex / Development Agent
  |
Honka MCP
  |
get_context("contract sending")
  |
Business Context
  |
Code analysis / change
```

The MCP interface allows increasingly capable AI systems to consume stable organization-specific context without Honka having to own the AI model itself.

## 15. AI-Assisted Editing

AI must not directly mutate canonical business metadata.

```text
AI
 |
Metadata Patch Proposal
 |
Compiler / Validator
 |
Human Review
 |
Commit
```

This is conceptually a **Pull Request for Business Context**.

A user may select an area of the canvas and ask whether anything is wrong. AI can inspect the Semantic Graph, identify a problematic path, and present a ghost/preview modification. Accepting the proposal produces a metadata patch that still passes validation and review.

## 16. Git-First Source of Truth

Canonical semantic Business Context is stored as metadata in Git.

A possible repository structure is:

```text
/value-chains
/processes
  /contract
    process.yaml
    /flows
/data-model
/rules
/capabilities
/implementations
```

Semantic metadata and visual layout metadata should be separated:

```text
contract-flow.yaml
contract-flow.layout.yaml
```

Moving a node by 20 pixels must not create noise in semantic diffs.

For technical users, this remains normal Git history. For nontechnical users, the UI can expose Change History, Change Proposal, Review, and Restore Previous Version. Internally these map to metadata patches, validation, commits, and optionally pull requests.

## 17. Canvas UX

The interaction model should feel closer to **Miro than Figma or a rigid BPMN editor**.

The surface should support fluid whiteboard behavior:

- Sticky
- Text
- Group
- Connector
- Semantic Node

A user should be able to begin informally:

```text
[Create contract]
       |
[Ask legal]
       |
[Send]
```

and progressively formalize it:

```text
Sticky
  |
Semantic Node
  |
Data Contract
  |
Business Rule
  |
Capability
  |
Implementation Mapping
```

The surface is a whiteboard. The underlying model is typed metadata.

## 18. Strictness Gradient

Honka should not require the entire business world to be formalized upfront.

| Layer | Strictness |
| --- | --- |
| Value Chain | Very loose |
| High-level Process | Loose |
| Node Description | Loose |
| Flow Transition | Medium |
| Capability Contract | Strict |
| Data Contract | Strict |
| Business Rule | Strict |

The closer information gets to an AI/system boundary, the stronger its semantics should become.

## 19. Frontend Architecture

Initial frontend direction:

- React
- TypeScript
- React Flow or equivalent node-based canvas library
- Zustand or equivalent lightweight local state
- Zod or equivalent schema validation

Canvas interaction should be local-first. Pointer movement, dragging, connecting nodes, and text editing must not require a server round trip.

```text
Browser State
     |
debounced / asynchronous sync
     |
Draft Service
     |
Server
```

Undo/redo should initially be local. Realtime collaboration and CRDT technology such as Yjs are explicitly not MVP requirements.

## 20. Backend Architecture

Start with a **TypeScript modular monolith**.

```text
Honka App
 |- API
 |- Draft
 |- Compiler
 |- Validator
 |- Git Integration
 |- AI Gateway
 |- MCP Server
 '- Job Worker
```

Do not introduce microservices merely to represent these boundaries.

Compiler and Validator packages should remain infrastructure-independent pure TypeScript wherever practical. They must be runnable from CLI and CI without requiring the application database.

## 21. Monorepo Direction

A possible initial structure:

```text
honka/
├─ apps/
│  ├─ designer-web/
│  └─ api/
├─ packages/
│  ├─ business-model/
│  ├─ graph-core/
│  ├─ graph-compiler/
│  ├─ graph-validator/
│  ├─ ai-context/
│  └─ mcp-context-server/
└─ infra/
```

These are architectural boundaries, not a requirement to deploy each package independently.

## 22. Infrastructure

### Local Development

```text
Docker Compose
 |- Honka App
 |- PostgreSQL
 '- Local Git Repository
```

### Production

```text
Frontend
   |
Managed Container
   |
Managed PostgreSQL
   |
Git Provider / Repository
```

Initial architecture should avoid unnecessary operational complexity.

Not required initially:

- Kubernetes
- Neo4j
- Kafka
- dedicated vector database
- function-per-event serverless architecture

A managed container is preferred over a FaaS-heavy architecture to keep execution behavior and cost easier to reason about.

## 23. Git and PostgreSQL Responsibilities

### Git

Git is the authoritative source of truth for semantic Business Context.

### PostgreSQL

PostgreSQL stores application/service state such as Users, Workspaces, Permissions, Drafts, Editing State, Comments, AI Conversations when retained, Git Connection Metadata, Job State, Audit Logs, Indexes, and Projections.

The Semantic Graph should be reconstructable from canonical Git metadata. The operational database is not the semantic source of truth.

Production should use managed PostgreSQL. Local development and integration testing may use PostgreSQL containers.

## 24. MVP

The MVP should prove the core thesis rather than attempt to become a complete enterprise architecture suite.

```text
Canvas
  |
Semantic Node
  |
Data Contract / Rule
  |
Git Metadata
  |
Compiler / Validator
  |
AI Context
  |
Honka MCP
```

The central validation question is:

> **Can humans naturally describe business context, structure it progressively, and provide it to AI in a way that materially improves the AI's understanding of organization-specific work?**

Workflow execution is not an MVP validation target.

## 25. Competitive Boundary

Honka occupies a different layer from workflow engines.

```text
Workflow Engine
"How is the work executed?"
        ^
        |
      HONKA
"What does the work mean?"
```

It also differs from a general-purpose whiteboard.

```text
Whiteboard
Diagram -> Human Understanding

Honka
Business Context
   -> Human Understanding
   -> AI Understanding
   -> Software Understanding
```

The goal is not simply "AI that understands diagrams." The goal is a durable, structured shared context layer.

## 26. Core Design Principles

### 1. Context, not Execution

Honka preserves the meaning of work. Existing systems execute it.

### 2. Semantic Graph is the Core

The canvas, YAML, database, and generated representations are interfaces to or projections of the Semantic Graph.

### 3. Git is the Source of Truth

Organization-specific business context is versioned metadata.

### 4. Loose Inside, Strict at Boundaries

Human thinking remains flexible. Boundaries consumed by AI and software become typed and validated.

### 5. One Honka, Many Expressions

Human-facing canvas, AI Context, tests, platform mappings, and implementation references are generated from or grounded in the same underlying business context.

> **One Honka, Many Expressions.**

## 27. Explicit Non-Goals for v0.1

Honka v0.1 is not intended to be:

- a general workflow runtime;
- a replacement for Salesforce Flow, Camunda, Temporal, or other execution engines;
- a BPMN-first modeling suite;
- a generic diagramming application;
- an AI model training platform;
- a full enterprise data catalog;
- a source-code repository replacement;
- a mandatory graph-database architecture.

These boundaries may be revisited only when they support the core Business Context thesis rather than dilute it.
