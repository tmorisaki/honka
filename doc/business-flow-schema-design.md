# Honka Business Flow Schema Design v1

## 1. Scope

本書は、Honkaにおける `Business Flow` のSemantic ModelおよびJSON Schema設計を定義する。

Business Flowは、Business Workを表すNodeと、それらを囲うScopeを組み合わせ、業務上存在するBusiness Pathを表現するAddressable Semantic Resourceである。

Business FlowはWorkflow Executionを定義しない。

Business Flowが定義するのは、

```text
どのBusiness Workが存在するか
どのScopeに属するか
どのBusiness WorkからどのBusiness Workへ進み得るか
```

というBusiness Structureである。

---

# 2. Business Flow Definition

Business Flowを以下のように定義する。

> **Business Flow = Node / ScopeのCompositionと、Node間に存在するBusiness PathのTopologyを定義するAddressable Semantic Resource**

Business Flowは大きく2つの構造を持つ。

```text
Business Flow
│
├─ Composition
│   └─ Scope ↔ Node
│
└─ Transition Graph
    └─ Node → Node
```

CompositionはBusiness WorkがどのBusiness Context Boundaryに属するかを、TopologyはBusiness Work間にどのBusiness Pathが存在するかを表す。

---

# 3. Resource Envelope

Business FlowはAddressable Semantic ResourceとしてStable IDを持つ。

```json
{
  "$schema": "../../schemas/business-flow.v1.schema.json",
  "apiVersion": "honka/v1",
  "kind": "BusinessFlow",
  "id": "opportunity-development",
  "name": "Opportunity Development"
}
```

`$schema` は準拠するJSON Schema、`apiVersion` はHonka Semantic Modelのバージョン、`kind` はResource Typeを指定する。

`id` はBusiness FlowのStable Identifierであり、Navigation、Semantic Graph、MCP、AI Context Compilation等から直接参照されるためStable IDを持つ。

`name` は表示名である。

---

# 4. Business Flow Structure

```text
Business Flow
│
├─ id
├─ name
├─ description
├─ guidance
│
├─ composition
│   ├─ scopes[]
│   │   ├─ scope
│   │   └─ nodes[]
│   └─ nodes[]
│
└─ transitions[]
    ├─ from
    ├─ to
    └─ guidance?
```

Business FlowにはExecution State、Current Node、Next Node、Transition Condition、Runtime Branching Logic、Action、Implementation、Retry、Timeout等を持たせない。

---

# 5. Description

`description` はBusiness Flowが表す業務全体を説明する。

```json
{
  "description": "商談開始から顧客理解、提案、契約に至るまでのBusiness Workの関係を表す。"
}
```

---

# 6. Guidance

`guidance` はBusiness Flow全体をどのように理解・運用すべきかを記述する。

```json
{
  "guidance": "商談は必ずしも一方向には進行しない。提案後であっても顧客理解が不足した場合はDiscoveryへ戻り、必要な情報を継続的に更新する。"
}
```

GuidanceはTopologyを定義しない。実際に存在するBusiness Pathは `transitions[]` によって表現する。

---

# 7. Composition

`composition` はBusiness Flowに参加するNodeとScope、およびその包含関係を定義する。

```text
Business Flow
       │
       └─ Composition
            ├─ Discovery
            │   ├─ Understand Needs
            │   ├─ Develop MEDDIC
            │   └─ Customer Meeting
            └─ Proposal
                ├─ Build Proposal
                └─ Review Proposal
```

Compositionは順序を表さない。

---

# 8. Scope Composition

Scopeに属するNodeはBusiness Flow側で定義する。

```json
{
  "composition": {
    "scopes": [
      {
        "scope": { "ref": "discovery" },
        "nodes": [
          { "ref": "understand-needs" },
          { "ref": "develop-meddic" },
          { "ref": "customer-meeting" }
        ]
      }
    ]
  }
}
```

`nodes[]` の配列順序にはExecution Order、Display Order、Priority、Transition Order等のSemantic Meaningを持たせない。

Node間のTopologyは `transitions[]` によってのみ定義する。

---

# 9. Unscoped Nodes

すべてのNodeがScopeに属することを要求しない。

Scopeに属さないNodeは `composition.nodes[]` に保持する。

```json
{
  "composition": {
    "nodes": [
      { "ref": "opportunity-created" }
    ]
  }
}
```

これをUnscoped Nodeと呼ぶ。ScopeはBusiness Context上必要な場合にのみ使用する。

---

# 10. Node Membership

v1では、1つのBusiness Flow内において1つのNodeは最大1つのScopeに所属する。

```text
Node A ∈ Discovery
```

は許容するが、

```text
Node A ∈ Discovery
Node A ∈ Proposal
```

は許容しない。

同じNode Resourceを異なるBusiness Flowから異なるScopeへCompositionすることは許容する。

---

# 11. Transition

`transition` はNode間に存在するBusiness Pathを表す。

```json
{
  "from": { "ref": "develop-meddic" },
  "to": { "ref": "customer-meeting" }
}
```

Transitionは「このBusiness Workから、このBusiness Workへ進むBusiness Pathが存在する」ことを意味する。

TransitionはWorkflow Engineにおける実行命令ではない。

---

# 12. Transition Guidance

TransitionはOptionalな `guidance` を持つことができる。

```json
{
  "from": { "ref": "develop-meddic" },
  "to": { "ref": "build-proposal" },
  "guidance": "顧客課題と意思決定構造が十分に把握でき、具体的な提案によって検証を進める場合。"
}
```

Transition Guidanceは、複数のBusiness Pathの中でなぜこのPathを選択するのかを説明する。

---

# 13. Transition Guidance and Boundary Rules

Transition GuidanceはRuleではない。

```text
Node Exit
  このNodeから離れてよいか

Scope Exit
  このScopeから離れてよいか

Scope Entry
  このScopeへ入ってよいか

Node Entry
  このNodeへ入ってよいか

Transition Guidance
  なぜこのPathを選択するのか
```

Transition自体にConditionまたはRuleを定義しない。

---

# 14. Transition Guidance Requirement

`guidance` はOptionalとする。

単純なBusiness Pathでは記述を要求しない。

一方、1つのNodeから複数のTransitionが存在する場合、Transition GuidanceはPath Selectionを理解する重要なBusiness Contextとなる。

ただしBranch時にGuidanceをStructural Requirementとはしない。

Semantic ValidatorまたはAI Analysis Layerは、複数のOutgoing TransitionにGuidanceが存在しない場合にWarningを生成できる。

---

# 15. Branching

Branchingのための専用構造は定義しない。

```text
       ┌──▶ B
A ─────┤
       └──▶ C
```

は、

```json
[
  { "from": { "ref": "A" }, "to": { "ref": "B" } },
  { "from": { "ref": "A" }, "to": { "ref": "C" } }
]
```

として表現する。

`branch`、`gateway`、`decision` 等のFlow Control Elementは導入しない。

---

# 16. Merge

Mergeについても専用構造は定義しない。

```text
A ──┐
    ├──▶ C
B ──┘
```

は2つのTransitionとして表現する。

---

# 17. Loop and Backtracking

Business FlowはLoopおよびBacktrackingを許容する。

```text
A → B → C
    ▲   │
    └───┘
```

Business FlowはDAGであることを要求しない。

---

# 18. Repeated Node Usage

同一Business Entityが同一Nodeを複数回利用することを許容する。

```text
Develop MEDDIC
      ↓
Customer Meeting
      ↓
Develop MEDDIC
```

これはNode Resourceを複製することを意味しない。

Transition Graph上で同一Nodeへ戻るPathを定義する。Node Resourceは1つのままである。

---

# 19. Cross-Scope Transition

TransitionはScope Boundaryを跨ぐことができる。

Metadata上は通常のNode Transitionとして保持する。

```json
{
  "from": { "ref": "develop-meddic" },
  "to": { "ref": "build-proposal" }
}
```

Scope間Transitionを別途保持しない。

---

# 20. Derived Scope Boundary Crossing

CompilerまたはSemantic GraphはCompositionを利用してScope Boundary Crossingを導出する。

```text
develop-meddic ∈ discovery
build-proposal ∈ proposal

develop-meddic → build-proposal

             ↓ derive

discovery → proposal
```

Scope TopologyはNode Transition Graphから導出される。これをMetadataとして二重管理しない。

---

# 21. Boundary Context

Cross-Scope Transitionを解釈する場合、以下のContextが関連する。

```text
Source Node Exit
       ↓
Source Scope Exit
       ↓
Transition Guidance
       ↓
Destination Scope Entry
       ↓
Destination Node Entry
```

同一Scope内のTransitionでは、

```text
Source Node Exit
       ↓
Transition Guidance
       ↓
Destination Node Entry
```

となる。

これらはExecution Sequenceではなく、Business Pathを判断・理解するためのSemantic Contextである。

---

# 22. Transition Has No Condition

Transitionには `condition` を定義しない。

```json
{
  "from": "develop-meddic",
  "to": "build-proposal",
  "condition": "meddicScore >= 80"
}
```

のようなモデルは採用しない。

ConditionをTransitionへ持たせるとBusiness FlowがWorkflow Execution Definitionへ近づく。

HonkaではEntry / Exit RuleをBusiness BoundaryのConstraint、Transition GuidanceをBusiness Pathを選択するためのContextとして分離する。

---

# 23. No Order

Business FlowにはNodeのSemantic Orderを保存しない。

```text
order
sequence
stepNumber
nextNode
previousNode
```

は定義しない。

Canvas上の `x`、`y`、`width`、`height`、`zIndex` 等はLayout MetadataでありBusiness Flow Semantic Modelには含めない。

---

# 24. Composition vs Topology

CompositionとTopologyは独立して扱う。

CompositionはNodeがどのBusiness Context Boundaryに属するかを表す。

TopologyはNode間にどのBusiness Pathが存在するかを表す。

同じCompositionで異なるTopologyを持つことも概念上可能である。

---

# 25. Full Example

```json
{
  "$schema": "../../schemas/business-flow.v1.schema.json",
  "apiVersion": "honka/v1",
  "kind": "BusinessFlow",
  "id": "opportunity-development",
  "name": "Opportunity Development",
  "description": "商談開始から顧客理解、提案に至るBusiness Workの関係を表す。",
  "guidance": "商談は必ずしも一方向には進行しない。提案後であっても顧客理解が不足した場合はDiscoveryへ戻り、必要な情報を継続的に更新する。",
  "composition": {
    "nodes": [
      { "ref": "opportunity-created" }
    ],
    "scopes": [
      {
        "scope": { "ref": "discovery" },
        "nodes": [
          { "ref": "understand-needs" },
          { "ref": "develop-meddic" },
          { "ref": "customer-meeting" }
        ]
      },
      {
        "scope": { "ref": "proposal" },
        "nodes": [
          { "ref": "build-proposal" },
          { "ref": "review-proposal" }
        ]
      }
    ]
  },
  "transitions": [
    {
      "from": { "ref": "opportunity-created" },
      "to": { "ref": "understand-needs" }
    },
    {
      "from": { "ref": "understand-needs" },
      "to": { "ref": "develop-meddic" }
    },
    {
      "from": { "ref": "develop-meddic" },
      "to": { "ref": "customer-meeting" },
      "guidance": "顧客との対話を通じて追加情報を確認する場合。"
    },
    {
      "from": { "ref": "customer-meeting" },
      "to": { "ref": "develop-meddic" }
    },
    {
      "from": { "ref": "develop-meddic" },
      "to": { "ref": "build-proposal" },
      "guidance": "顧客課題と意思決定構造が十分に把握でき、具体的な提案によって検証を進める場合。"
    },
    {
      "from": { "ref": "build-proposal" },
      "to": { "ref": "review-proposal" }
    },
    {
      "from": { "ref": "review-proposal" },
      "to": { "ref": "develop-meddic" },
      "guidance": "提案内容を再検討するために顧客理解の更新が必要な場合。"
    }
  ]
}
```

---

# 26. Semantic Graph Representation

Business Flow MetadataからSemantic Graphを構築する。

Compositionでは `HAS_SCOPE`、`HAS_NODE`、`CONTAINS` Relationを生成する。

TopologyではNode間の `TRANSITION` Relationを生成する。

Resource Identity、Composition、TopologyをGraph上でも区別する。

---

# 27. Schema Validation Responsibilities

JSON SchemaはBusiness Flowの構造を検証する。

対象には以下を含む。

```text
apiVersion
kind
id
name
description
guidance
composition structure
scope reference structure
node reference structure
transition structure
unknown properties
```

JSON SchemaはReference先の存在を検証しない。

---

# 28. Semantic Validation Responsibilities

Semantic ValidatorはBusiness Flow全体の意味的整合性を検証する。

```text
Scope reference exists
Node reference exists
Referenced Scope is Scope
Referenced Node is Node
Node belongs to at most one Scope in the Flow
Transition.from exists in Flow Composition
Transition.to exists in Flow Composition
Duplicate Transition detection
Scope Boundary Crossing resolution
```

Analysis LayerではNode with no incoming Transition、Node with no outgoing Transition、Unreachable Node、Disconnected subgraph、Cycle、Branch without Transition Guidance、Potential ambiguous path等を検出できる。

これらは必ずしもValidation Errorとはせず、WarningまたはAnalysis Resultとして扱う。

---

# 29. Validation Pipeline

```text
JSON
  ↓
JSON Schema Validation
  ↓
Typed Business Flow AST
  ↓
Reference Resolution
  ↓
Composition Resolution
  ↓
Transition Graph Construction
  ↓
Semantic Validation
  ↓
Semantic Graph
```

---

# 30. Architecture Decision Records

## ADR-F001: Business FlowをExecution Definitionとして扱わない

**Status:** Accepted

Business FlowはBusiness StructureとBusiness Pathを表現する。Workflow Runtime StateやExecution Logicは保持しない。

---

## ADR-F002: CompositionとTopologyを分離する

**Status:** Accepted

CompositionはScope ↔ Node Membership、TopologyはNode → Node Transitionとして独立して保持する。

Scope内のNode配列順序からTopologyを推測しない。

---

## ADR-F003: Node MembershipはBusiness Flowが所有する

**Status:** Accepted

Scope ResourceおよびNode Resource自身にはMembershipを保持しない。

Business Flow CompositionがScopeとNodeの包含関係を定義する。

---

## ADR-F004: Nodeの配列順序にSemantic Meaningを持たせない

**Status:** Accepted

`composition.nodes[]` およびScope Composition内の `nodes[]` の配列順序には意味を持たせない。

Business PathはTransition Graphのみから判断する。

---

## ADR-F005: TransitionはNode間にのみ定義する

**Status:** Accepted

TransitionのVertexはNodeとする。

ScopeはTransition GraphのVertexとしない。

Scope Boundary CrossingはCompositionとNode Transitionから導出する。

---

## ADR-F006: Scope間Transitionを保持しない

**Status:** Accepted

Scope TopologyはNode TransitionとCompositionから導出する。

Scope TransitionをMetadataとして二重管理しない。

---

## ADR-F007: Branch / Merge専用Elementを導入しない

**Status:** Accepted

BranchとMergeはTransition GraphのTopologyとして表現する。

Gateway、Decision Node、Branch Element、Merge Element等のFlow Control Resourceを導入しない。

---

## ADR-F008: Business FlowはCycleを許容する

**Status:** Accepted

Business FlowをDAGに限定しない。

Loop、Backtracking、Repeated Node Usageを許容する。

---

## ADR-F009: TransitionにOptional Guidanceを持たせる

**Status:** Accepted

TransitionはOptionalな `guidance` を持つ。

Transition Guidanceは複数のBusiness Pathから当該Pathを選択する意味を説明する。

---

## ADR-F010: Transition GuidanceをBranch時にも必須としない

**Status:** Accepted

Transition Guidanceは常にOptionalとする。

複数Outgoing TransitionにGuidanceがない場合、Semantic Analysis LayerがWarningを生成できる。

---

## ADR-F011: TransitionにConditionを持たせない

**Status:** Accepted

TransitionにはCondition、Rule、Expressionを持たせない。

Entry / Exit RuleをBoundary Constraint、Transition GuidanceをPath Selection Contextとして分離する。

---

## ADR-F012: Semantic Orderを保持しない

**Status:** Accepted

`order`、`sequence`、`stepNumber`、`nextNode`、`previousNode` をBusiness Flow Semantic Modelに持たせない。

Node間の関係はTransition Graphで表現する。

---

## ADR-F013: LayoutをSemantic Modelから分離する

**Status:** Accepted

Canvas上のPosition、Size、Z-order等はLayout Metadataとして別管理する。

Business Flow Semantic Modelには含めない。

---

## ADR-F014: 1 Flow内でNodeは最大1 Scopeに所属する

**Status:** Accepted

v1では、1つのBusiness FlowにおけるNode MembershipをZero-or-One Scopeとする。

これによりScope Boundary CrossingをCompositionから一意に導出できる。

同一Flow内で1つのNodeが複数Scopeへ所属する方式は採用しない。

---

# 31. Separation of Concerns

Business Flowは以下を保持する。

```text
Flow Identity
Flow Description
Flow Guidance

Composition
  Scope ↔ Node

Topology
  Node → Node
  Transition Guidance
```

Business FlowはNode Business Context、Node Data Contract、Node Action、Node Rule、Scope Guidance、Scope Entry / Exit、Capability Definition、Implementation、Transition Condition、Runtime State、Current Node、Execution History、Retry、Timeout、Canvas Position、Canvas Size、Z-order等を保持しない。

```text
Node
  Business Work

Scope
  Business Context Boundary

Business Flow
  Composition + Topology

Capability
  Business Ability

Implementation
  Technical Realization

Layout
  Visual Representation

Runtime
  Execution State
```

---

# 32. Summary

Business FlowのSemantic Modelは以下とする。

```text
Business Flow
│
├─ id
├─ name
├─ description
├─ guidance
│
├─ composition
│   ├─ nodes[]
│   └─ scopes[]
│       ├─ scope
│       └─ nodes[]
│
└─ transitions[]
    ├─ from
    ├─ to
    └─ guidance?
```

中心原則は、

> **Business Flow = Composition + Transition Graph**

である。

さらに、

> **Transition is a possible Business Path, not an execution command.**

とする。

Resource、Composition、Topologyを分離することで、HonkaはBranch、Merge、Loop、Backtrackingを自然に表現しながら、Workflow Execution Engineになることを避ける。

```text
Node
  What does this Business Work mean?

Scope
  What Business Context surrounds these Works?

Business Flow
  How are these Works composed and connected?
```
