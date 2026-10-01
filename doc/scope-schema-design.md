# Honka Scope Schema Design v1

## 1. Scope

本書は、Honkaにおける `Scope` のSemantic ModelおよびJSON Schema設計を定義する。

Scopeは複数のBusiness Workに共通するBusiness ContextとBoundaryを表現するSemantic Resourceである。

ScopeはNodeを所有せず、Nodeの実行順序やTransitionを定義しない。

Business Flow上でどのNodeがScopeに含まれるかは、Business FlowのCompositionによって定義する。

---

# 2. Scope Definition

Scopeを以下のように定義する。

> **Scope = 複数のBusiness Workを囲い、それらに共通するBusiness ContextとEntry / Exit Boundaryを定義するAddressable Semantic Resource**

Nodeとの責務は異なる。

```text
Scope
  複数のBusiness Workが共有するContextとBoundary

Node
  Business Workを行う意味的な場所
```

ScopeはWorkflow EngineにおけるSubprocessではない。

Scope内部のNodeの実行順序、分岐、合流、Loop、BacktrackingなどのFlow TopologyはBusiness Flowが定義する。

---

# 3. Resource Envelope

ScopeはAddressable Semantic ResourceとしてStable IDを持つ。

```json
{
  "$schema": "../../schemas/scope.v1.schema.json",
  "apiVersion": "honka/v1",
  "kind": "Scope",
  "id": "discovery",
  "name": "Discovery"
}
```

## `$schema`

Scopeが準拠するJSON Schemaを指定する。

## `apiVersion`

Honka Semantic Modelのバージョンを指定する。

```text
honka/v1
```

## `kind`

Resource Typeを指定する。Scopeでは固定値とする。

## `id`

ScopeのStable Identifier。

ScopeはMCP、Semantic Graph、Navigation、AI Context Compilation等から直接参照されるAddressable Semantic ResourceであるためStable IDを持つ。

表示名の変更によって変更しない。

## `name`

Scopeの表示名。

---

# 4. Scope Structure

```text
Scope
│
├─ id
├─ name
├─ guidance
│
├─ entry
│   ├─ guidance
│   └─ rules[]
│       └─ description
│
└─ exit
    ├─ guidance
    └─ rules[]
        └─ description
```

Scopeには以下を持たせない。

```text
description
nodes
transitions
order
dataContract
actions
rules
outcome
implementation
```

Scope自身はBusiness WorkやFlow Topologyを定義するResourceではなく、それらを囲うBusiness Context Boundaryである。

---

# 5. Guidance

`guidance` はScopeに含まれるBusiness Work全体をどのように理解し、進めるべきかを記述する。

```json
{
  "guidance": "顧客の課題、意思決定構造、成功条件を探索する。情報は複数回の顧客接点を通じて継続的に更新し、一度の商談ですべてを確定させる必要はない。"
}
```

```text
Scope Guidance
  このBusiness Work群をどう理解し、どう進めるか

Entry Guidance
  どのようなBusiness StateからこのScopeへ入るか

Exit Guidance
  どのようなBusiness Stateまで進めることを目指すか
```

Scope GuidanceはBusiness Constraintではない。明示的なBoundary ConstraintはEntry RuleまたはExit Ruleとして定義する。

---

# 6. Entry

`entry` はScopeへ入るためのBoundaryを定義する。

```json
{
  "entry": {
    "guidance": "顧客との具体的な商談活動が開始され、顧客課題や意思決定構造を探索できる状態。",
    "rules": [
      { "description": "商談がActiveであること" }
    ]
  }
}
```

Entry GuidanceはScopeへ入るBusiness Stateを説明する。

Entry RulesはScopeへ入ることを許可するためのBusiness Constraintを定義する。

RuleはScope-local Elementであり、Stable IDを持たない。

---

# 7. Exit

`exit` はScopeを離れるためのBoundaryを定義する。

```json
{
  "exit": {
    "guidance": "提案方針を判断するために必要な顧客理解が得られている状態。",
    "rules": [
      { "description": "顧客の主要な課題が確認されていること" },
      { "description": "意思決定構造について提案判断に必要な情報が確認されていること" }
    ]
  }
}
```

Exit GuidanceはScope内のBusiness Workによって目指すBusiness Stateを説明する。

Exit RulesはScopeを離れることを許可するためのBusiness Constraintを定義する。

Exit Ruleを持たないScopeも許容する。

---

# 8. Scope Rules

Scope直下に `rules[]` は定義しない。

Scopeに必要なConstraintはBoundaryに限定する。

```text
entry.rules[]
exit.rules[]
```

Scope内部でBusiness Workを行う際に適用されるConstraintは、それぞれのNodeが所有する。

これによりScopeが巨大なNodeになることを防ぐ。

---

# 9. Scope Composition

Scope自身は、包含するNodeを保持しない。

NodeとScopeの包含関係はBusiness FlowがCompositionとして定義する。

```text
Business Flow
│
├─ Composition
│   ├─ Scope: Discovery
│   │   ├─ Node A
│   │   ├─ Node B
│   │   └─ Node C
│   └─ Scope: Proposal
│       ├─ Node D
│       └─ Node E
└─ Transition Graph
```

Scope Schemaには `nodes[]`、`nodeRefs[]`、`children[]` を定義しない。

Scope ResourceはScopeそのもののBusiness Meaningを、Business Flow CompositionはこのFlowにおいてScopeがどのNodeを包含するかを表す。

---

# 10. Scope and Node Ownership

ScopeとNodeはそれぞれ独立したAddressable Semantic Resourceとして扱う。

Business FlowがそれらをCompositionする。

```text
Business Flow
      │
      ├── USES_SCOPE ──▶ Discovery
      ├── USES_NODE ───▶ Develop MEDDIC
      └── USES_NODE ───▶ Customer Meeting

Discovery
      │
      ├── CONTAINS ────▶ Develop MEDDIC
      └── CONTAINS ────▶ Customer Meeting
```

`CONTAINS` はBusiness Flow Compositionから導出される関係である。

Scope Resource自身がNode Referenceを所有することを意味しない。

---

# 11. Flow Topology

ScopeはNode間の順序を定義しない。また、Scope内部のNodeが線形に進行することを前提としない。

以下をすべて許容する。

```text
Branching
Merge
Loop
Backtracking
Repeated Node Usage
Optional Paths
Parallel Business Work
```

これらはBusiness FlowのTransition Graphによって表現する。

ScopeはこのTopologyを保持しない。

---

# 12. Scope Boundary Crossing

Node間のTransitionが異なるScopeを跨ぐ場合、Scope Boundary Crossingとして解釈できる。

Business Flow上では `Node B → Node C` というNode間Transitionとして保持する。

Semantic GraphまたはCompilerはCompositionを参照し、

```text
Node B ∈ Discovery
Node C ∈ Proposal
```

を解決して `Discovery → Proposal` というScope Boundary Crossingを導出できる。

Scope間Transitionを重複してMetadataとして保持する必要はない。

---

# 13. Boundary Evaluation

Scope Boundaryを跨ぐTransitionでは、Node BoundaryとScope Boundaryの双方が意味を持つ。

```text
Node B Exit
     ↓
Discovery Exit
     ↓
Proposal Entry
     ↓
Node C Entry
```

ただし、これはWorkflow Runtimeの実行順序を定義するものではない。

HonkaはそれぞれのBoundaryに存在するBusiness ContextとConstraintを提供する。

それをどのように評価・実行するかは利用するAI、Software、HumanまたはExecution Systemの責務である。

---

# 14. Data Contract

Scopeには `dataContract` を定義しない。

Business Data ContractはBusiness Workを行うNodeが所有する。

Scope単位でData Contractを集約したViewを提供することは可能だが、それはContained Nodeから生成されるDerived RepresentationでありScope Metadataそのものではない。

---

# 15. Actions

Scopeには `actions` を定義しない。

Actionは具体的なBusiness Workを行うNodeの責務である。

```text
Scope
  ↓ composition
Node
  ↓ uses
Action
  ↓ references
Capability
```

ScopeからCapabilityを直接利用するモデルは採用しない。

---

# 16. Outcome

Scopeには `outcome` を定義しない。

具体的なBusiness Contextを後続Business Workへ提供する責務はNodeに属する。

```text
Scope Exit
  このBusiness Work群から離れてよいか

Node Outcome
  このBusiness Workから後続へ何を提供するか
```

---

# 17. AI Context Boundary

ScopeはAI Context CompilationのBoundaryとして利用できる。

```text
Current Scope
     ↓
Scope Guidance
     ↓
Scope Entry / Exit
     ↓
Business Flow Composition
     ↓
Contained Nodes
     ↓
Relevant Node Context
     ↓
Data Contract / Rules / Capabilities
```

Scope自身はNodeを保持しないため、Contained NodesはBusiness Flow Compositionから解決する。

これによりAIにBusiness Flow全体を無条件に提供せず、現在のBusiness Contextに関連する範囲を限定できる。

---

# 18. MCP Addressability

ScopeはMCPから直接参照可能なSemantic Resourceとする。

```text
get_scope("discovery")

get_context(
  scope = "discovery"
)
```

Scope単体を取得した場合にはScope Resourceそのものを返す。

Business Flow Contextと組み合わせる場合には、Compositionを解決してContained Nodesを含むContextを構築できる。

具体的なMCP Tool SchemaはScope Schemaとは分離して定義する。

---

# 19. Semantic Graph Representation

ScopeはSemantic Graph上の独立Resourceとして表現する。

```text
[Business Flow]
      │
      │ USES_SCOPE
      ▼
[Discovery Scope]
```

Business Flow Compositionによって `CONTAINS` Relationを生成できる。

Node間のTopologyは別の `TRANSITION` Relationとして扱う。

したがってSemantic Graphでは、

```text
Resource
Composition
Topology
```

を独立した概念として扱う。

---

# 20. Full Example

```json
{
  "$schema": "../../schemas/scope.v1.schema.json",
  "apiVersion": "honka/v1",
  "kind": "Scope",
  "id": "discovery",
  "name": "Discovery",
  "guidance": "顧客の課題、意思決定構造、成功条件を探索する。情報は複数回の顧客接点を通じて継続的に更新し、一度の商談ですべてを確定させる必要はない。",
  "entry": {
    "guidance": "顧客との具体的な商談活動が開始され、顧客課題や意思決定構造を探索できる状態。",
    "rules": [
      { "description": "商談がActiveであること" }
    ]
  },
  "exit": {
    "guidance": "提案方針を判断するために必要な顧客理解が得られている状態。",
    "rules": [
      { "description": "顧客の主要な課題が確認されていること" },
      { "description": "意思決定構造について提案判断に必要な情報が確認されていること" }
    ]
  }
}
```

Node membershipはこのResourceには含めない。

---

# 21. Schema Validation Responsibilities

JSON SchemaはScopeの構造を検証する。

対象には `apiVersion`、`kind`、`id`、`name`、`guidance`、Entry/Exit構造、Rule構造、unknown propertiesを含む。

Scope SchemaはBusiness Flow Compositionを検証しない。

Node Referenceの成立性やTransition GraphもScope Schemaの対象外とする。

---

# 22. Semantic Validation Responsibilities

Honka Semantic ValidatorはScope ResourceおよびBusiness Flowとの意味的整合性を検証する。

Scope単体ではScope ID validity、Scope structure、Boundary semantics、Rule semanticsを検証できる。

Business Flow ContextではScope reference、Contained Node reference、Composition、Scope Boundary Crossing等を検証する。

```text
JSON
  ↓
JSON Schema Validation
  ↓
Typed Scope AST
  ↓
Reference Resolution
  ↓
Business Flow Composition Resolution
  ↓
Semantic Validation
  ↓
Semantic Graph
```

---

# 23. Architecture Decision Records

## ADR-S001: Scopeを単なるCanvas Groupとして扱わない

**Status:** Accepted

ScopeをAddressable Semantic Resourceとして定義する。ScopeはCanvas上の視覚的なGroupではなく、Business Context Boundaryを表現する。

---

## ADR-S002: ScopeにStable IDを持たせる

**Status:** Accepted

ScopeはMCP、AI Context Compilation、Semantic Graph、Navigation等から直接参照する必要があるため、Addressable Semantic ResourceとしてStable IDを要求する。

---

## ADR-S003: ScopeにGuidanceを持たせる

**Status:** Accepted

Scope全体のBusiness Workをどのように理解し進めるべきかを記述するため、Scope直下に `guidance` を持たせる。

EntryとExitはBoundary Stateを説明し、Scope GuidanceはBoundary間で行われるBusiness Work群全体に共通するContextを説明する。

---

## ADR-S004: ScopeにはEntryとExitを持たせる

**Status:** Accepted

Scopeは `Entry { guidance, rules[] }` と `Exit { guidance, rules[] }` を持つ。

Scopeは複数Nodeを囲むBusiness Context Boundaryであるため、Scopeへ入る条件とScopeから離れる条件を表現できる必要がある。

---

## ADR-S005: Scope直下にRulesを持たせない

**Status:** Accepted

ScopeのConstraintは `entry.rules[]` と `exit.rules[]` に限定する。

Scope内部のBusiness Workに適用されるConstraintはNodeが所有する。

---

## ADR-S006: ScopeにData Contractを持たせない

**Status:** Accepted

Data ContractはNodeが所有する。

Scope単位で必要なData ContractはBusiness Flow CompositionからContained Nodeを解決し、導出できる。

---

## ADR-S007: ScopeにActionを持たせない

**Status:** Accepted

ActionはBusiness Workを行うNodeが所有する。ScopeはCapabilityを直接利用しない。

---

## ADR-S008: ScopeにOutcomeを持たせない

**Status:** Accepted

OutcomeはNodeが後続Business Workへ提供するBusiness ContextとしてNodeに保持する。ScopeはExit Boundaryのみを定義する。

---

## ADR-S009: ScopeにNode Membershipを保持しない

**Status:** Accepted

Scope ResourceにはNode Membershipを保持しない。

NodeとScopeの包含関係はBusiness FlowのCompositionとして保持する。

```text
Scope Resource
  Business Context Boundary

Business Flow Composition
  Scope ↔ Node Membership
```

Scope Resource自身が `nodes[]` を所有する方式は採用しない。

---

## ADR-S010: ScopeにFlow Topologyを保持しない

**Status:** Accepted

ScopeにはNode order、Transition、Branching、Merge、Loop、Backtrackingを保持しない。

これらはBusiness FlowのTransition Graphとして保持する。

---

## ADR-S011: Scope間Transitionを重複保持しない

**Status:** Accepted

TransitionはBusiness Flow上のNode間Transitionとして保持する。

Scope間のBoundary CrossingはNode Membershipから導出する。

Node TransitionとScope Transitionを別々に永続化しない。

---

## ADR-S012: ScopeをSubprocessとして扱わない

**Status:** Accepted

ScopeはExecution ContainerではなくSemantic Context Boundaryとする。

Scope内ではBranching、Merge、Loop、Backtracking、Repeated Node Usage等を許容する。

---

# 24. Separation of Concerns

Scope Schemaには以下を保持しない。

```text
Node membership
Node order
Node-to-Node transition
Branching
Merge
Loop
Backtracking
Workflow runtime state
Transition execution logic
Data Contract
Action
Capability implementation
Outcome
REST endpoint
MCP implementation
Authentication
Retry
Timeout
UI layout
Canvas position
Internal runtime identity
```

Node membershipはBusiness Flow Compositionが管理する。

Node-to-Node transition、Branching、Merge、Loop、BacktrackingはBusiness Flow Transition Graphが管理する。

Scopeは以下のみを記述する。

```text
Scope Identity
Scope Guidance
Entry Boundary
Exit Boundary
```

---

# 25. Summary

ScopeのSemantic Modelは以下とする。

```text
Scope
├─ id
├─ name
├─ guidance
├─ entry
│  ├─ guidance
│  └─ rules[]
└─ exit
   ├─ guidance
   └─ rules[]
```

Scopeは、

> **複数のBusiness Workを囲い、それらに共通するBusiness ContextとEntry / Exit Boundaryを定義するAddressable Semantic Resource**

である。

Scope自身はNodeを所有しない。

Business FlowがScopeとNodeのCompositionを定義する。

Node間の順序を保存するのではなく、Business FlowがTransition Graphとして移動可能なBusiness Pathを保持する。

これにより、

```text
Resource
  Node / Scopeそのものの意味

Composition
  ScopeとNodeの包含関係

Topology
  Node間のTransition
```

を明確に分離する。
