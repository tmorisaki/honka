# Honka Scope Schema Design v1

## 1. Scope

本書は、Honkaにおける `Scope` のSemantic ModelおよびJSON Schema設計を定義する。

Scopeは、同一Graph Container内の複数のBusiness Work Unitに共通するBusiness ContextとEntry / Exit Boundaryを表現するAddressable Semantic Resourceである。

Scope自身は包含対象を所有せず、包含対象の順序やTransitionを定義しない。何をScopeに含めるかは、そのScopeを利用するGraph ContainerのCompositionによって定義する。

---

# 2. Scope Definition

> **Scope = 同一Graph Container内の複数のBusiness Work Unitを囲い、それらに共通するBusiness ContextとEntry / Exit Boundaryを定義するAddressable Semantic Resource**

Scopeが囲う対象は利用するGraph Containerによって異なる。

```text
High-Level Process
  Scope ↔ Business Flow

Business Flow
  Scope ↔ Node
```

ScopeはWorkflow EngineにおけるSubprocessではない。

Scope内部のBusiness Work Unitの分岐、合流、Loop、Backtracking等のTopologyは、ScopeではなくGraph Containerが定義する。

---

# 3. Resource Envelope

```json
{
  "$schema": "../../schemas/scope.v1.schema.json",
  "apiVersion": "honka/v1",
  "kind": "Scope",
  "id": "discovery",
  "name": "Discovery"
}
```

ScopeはMCP、Semantic Graph、Navigation、AI Context Compilation等から直接参照されるためStable IDを持つ。

---

# 4. Scope Structure

```text
Scope
├─ id
├─ name
├─ guidance
├─ entry
│  ├─ guidance
│  └─ rules[]
│     └─ description
└─ exit
   ├─ guidance
   └─ rules[]
      └─ description
```

Scopeには以下を持たせない。

```text
description
members
nodes
flows
transitions
order
dataContract
actions
rules
outcome
implementation
```

---

# 5. Guidance

`guidance` はScopeに含まれるBusiness Work Unit群をどのように理解し、進めるべきかを記述する。

```text
Scope Guidance
  このBusiness Work Unit群をどう理解し、どう進めるか

Entry Guidance
  どのようなBusiness StateからこのScopeへ入るか

Exit Guidance
  どのようなBusiness Stateまで進めることを目指すか
```

Scope GuidanceはConstraintではない。明示的なBoundary ConstraintはEntry RuleまたはExit Ruleとして定義する。

---

# 6. Entry / Exit

`entry` はScopeへ入るBoundary、`exit` はScopeを離れるBoundaryを定義する。

```json
{
  "entry": {
    "guidance": "顧客との具体的な商談活動が開始され、顧客課題や意思決定構造を探索できる状態。",
    "rules": [
      { "description": "商談がActiveであること" }
    ]
  },
  "exit": {
    "guidance": "提案方針を判断するために必要な顧客理解が得られている状態。",
    "rules": [
      { "description": "顧客の主要な課題が確認されていること" }
    ]
  }
}
```

RuleはScope-local ElementでありStable IDを持たない。

---

# 7. Scope Composition

Scope自身は包含対象を保持しない。

包含関係はGraph Container側のCompositionとして定義する。

```text
High-Level Process
├─ Composition
│  └─ Scope ↔ Business Flow
└─ Transition Graph
   └─ Business Flow → Business Flow

Business Flow
├─ Composition
│  └─ Scope ↔ Node
└─ Transition Graph
   └─ Node → Node
```

Scope ResourceはScopeそのもののBusiness Meaningを表し、Graph Container CompositionがそのContextにおけるMembershipを表す。

---

# 8. Scope Rules

Scope直下に `rules[]` は定義しない。

Constraintは `entry.rules[]` と `exit.rules[]` に限定する。

Scope内部の個別Business Workに適用されるConstraintは、そのBusiness Work Unit側のSemantic Resourceが所有する。

---

# 9. Boundary Crossing

Transitionが異なるScopeを跨ぐ場合、Scope Boundary Crossingとして解釈できる。

```text
Source Unit Exit
      ↓
Source Scope Exit
      ↓
Transition Guidance
      ↓
Destination Scope Entry
      ↓
Destination Unit Entry / Context
```

具体的なTransitionはGraph ContainerのMember間Transitionとして保持し、Scope間Transitionを重複保存しない。

Semantic GraphまたはCompilerはCompositionからScope Boundary Crossingを導出する。

---

# 10. Data Contract / Actions / Outcome

Scopeには `dataContract`、`actions`、`outcome` を定義しない。

これらは具体的なBusiness Workを表す下位Semantic Resourceの責務である。

Scope単位の集約Viewが必要な場合は、CompositionからContained Membersを解決してDerived Representationとして生成する。

---

# 11. AI Context Boundary

ScopeはAI Context CompilationのBoundaryとして利用できる。

```text
Current Scope
     ↓
Scope Guidance
     ↓
Scope Entry / Exit
     ↓
Container Composition
     ↓
Contained Business Work Units
     ↓
Relevant Lower-Level Context
```

Scope自身はMembershipを保持しないため、Contained MembersはGraph Container Compositionから解決する。

---

# 12. MCP Addressability

ScopeはMCPから直接参照可能なSemantic Resourceとする。

```text
get_scope("discovery")
get_context(scope = "discovery")
```

Scope単体取得ではScope Resourceそのものを返す。Container Contextと組み合わせる場合はCompositionを解決してContained Membersを含むContextを構築できる。

---

# 13. Semantic Graph Representation

Scopeは独立Resourceとして表現する。

Compositionから `CONTAINS` Relationを導出し、Topologyは別の `TRANSITION` Relationとして扱う。

```text
Resource
Composition
Topology
```

を独立した概念として扱う。

---

# 14. Full Example

```json
{
  "$schema": "../../schemas/scope.v1.schema.json",
  "apiVersion": "honka/v1",
  "kind": "Scope",
  "id": "discovery",
  "name": "Discovery",
  "guidance": "顧客の課題、意思決定構造、成功条件を探索する。情報は複数回の顧客接点を通じて継続的に更新する。",
  "entry": {
    "guidance": "顧客との具体的な商談活動が開始され、探索できる状態。",
    "rules": [
      { "description": "商談がActiveであること" }
    ]
  },
  "exit": {
    "guidance": "提案方針を判断するために必要な顧客理解が得られている状態。",
    "rules": [
      { "description": "顧客の主要な課題が確認されていること" }
    ]
  }
}
```

MembershipはこのResourceには含めない。

---

# 15. Validation Responsibilities

JSON SchemaはScope自身の構造を検証する。

Semantic ValidatorはScope ID、Boundary semantics、Rule semantics、および利用するGraph ContainerとのComposition整合性を検証する。

```text
JSON
  ↓
JSON Schema Validation
  ↓
Typed Scope AST
  ↓
Reference Resolution
  ↓
Container Composition Resolution
  ↓
Semantic Validation
  ↓
Semantic Graph
```

---

# 16. Architecture Decision Records

## ADR-S001: Scopeを単なるCanvas Groupとして扱わない

**Status:** Accepted

ScopeをAddressable Semantic Resourceとして定義する。

## ADR-S002: ScopeにStable IDを持たせる

**Status:** Accepted

MCP、AI Context Compilation、Semantic Graph、Navigation等から直接参照するためStable IDを要求する。

## ADR-S003: ScopeにGuidanceを持たせる

**Status:** Accepted

Scope全体のBusiness Work Unit群に共通するContextを記述する。

## ADR-S004: ScopeにはEntryとExitを持たせる

**Status:** Accepted

Scopeは `Entry { guidance, rules[] }` と `Exit { guidance, rules[] }` を持つ。

## ADR-S005: Scope直下にRulesを持たせない

**Status:** Accepted

ConstraintはEntry / Exit Boundaryに限定する。

## ADR-S006: ScopeにData Contractを持たせない

**Status:** Accepted

Data Contractは具体的なBusiness Workを表す下位Resourceが所有する。

## ADR-S007: ScopeにActionを持たせない

**Status:** Accepted

ScopeはCapabilityを直接利用しない。

## ADR-S008: ScopeにOutcomeを持たせない

**Status:** Accepted

ScopeはBoundaryを定義し、具体的なContext Transferは下位Resourceが担う。

## ADR-S009: ScopeにMembershipを保持しない

**Status:** Accepted

MembershipはScopeを利用するGraph ContainerのCompositionが所有する。

## ADR-S010: ScopeにTopologyを保持しない

**Status:** Accepted

Order、Transition、Branching、Merge、Loop、Backtrackingを保持しない。

## ADR-S011: Scope間Transitionを重複保持しない

**Status:** Accepted

Scope Boundary CrossingはMember TransitionとCompositionから導出する。

## ADR-S012: ScopeをSubprocessとして扱わない

**Status:** Accepted

ScopeはExecution ContainerではなくSemantic Context Boundaryとする。

## ADR-S013: ScopeをNode専用Boundaryに限定しない

**Status:** Accepted

ScopeはGraph Container共通のSemantic Boundaryとする。

```text
High-Level Process → Scope<Business Flow>
Business Flow      → Scope<Node>
```

ScopeのSerializationは包含対象の種類に依存しない。Membershipは各Graph Containerが明示的な型名で保持する。

---

# 17. Separation of Concerns

Scope SchemaにはMembership、Member order、Transition、Branching、Merge、Loop、Backtracking、Workflow runtime state、Data Contract、Action、Outcome、Implementation、UI layoutを保持しない。

Scopeは以下のみを記述する。

```text
Scope Identity
Scope Guidance
Entry Boundary
Exit Boundary
```

---

# 18. Summary

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

> **Scope = 同一Graph Container内の複数のBusiness Work Unitを囲い、それらに共通するBusiness ContextとEntry / Exit Boundaryを定義するAddressable Semantic Resource**

Scope自身はMembershipもTopologyも所有しない。

```text
High-Level Process
  Scope ↔ Business Flow

Business Flow
  Scope ↔ Node
```

この一般化により、Scope Resource自体を変えずに複数粒度のBusiness Graphで共通利用できる。
