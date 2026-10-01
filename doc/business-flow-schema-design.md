# Honka Business Flow Schema Design v1

## 1. Scope

本書はHonkaにおける `Business Flow` のSemantic ModelおよびJSON Schema設計を定義する。

Business FlowはNodeをBusiness Work UnitとしてCompositionし、Node間に存在するBusiness PathをTransition Graphとして表現するAddressable Semantic Resourceである。

Business FlowはWorkflow Executionを定義しない。

---

# 2. Business Flow Definition

> **Business Flow = Node / ScopeのCompositionと、Node間に存在するBusiness PathのTopologyを定義するGraph Container**

```text
Business Flow
├─ Composition
│  ├─ Unscoped Node
│  └─ Scope ↔ Node
└─ Transition Graph
   └─ Node → Node
```

High-Level Processも同じGraph Container Patternを持つが、Memberの粒度が異なる。

```text
High-Level Process
  member = Business Flow

Business Flow
  member = Node
```

Semantic Modelとしては同型だが、Serializationでは `flows[]` / `nodes[]` のように具体的な型名を用いる。

---

# 3. Resource Envelope

```json
{
  "$schema": "../../schemas/business-flow.v1.schema.json",
  "apiVersion": "honka/v1",
  "kind": "BusinessFlow",
  "id": "opportunity-development",
  "name": "Opportunity Development"
}
```

Business FlowはNavigation、Semantic Graph、MCP、AI Context Compilation等から直接参照されるためStable IDを持つ。

---

# 4. Structure

```text
Business Flow
├─ id
├─ name
├─ description
├─ guidance
├─ composition
│  ├─ nodes[]
│  └─ scopes[]
│     ├─ scope
│     └─ nodes[]
└─ transitions[]
   ├─ from
   ├─ to
   └─ guidance?
```

Business FlowにはExecution State、Current Node、Next Node、Transition Condition、Runtime Branching Logic、Implementation等を持たせない。

---

# 5. Description / Guidance

`description` はBusiness Flowが表す業務全体を説明する。

`guidance` はBusiness Flow全体をどのように理解・運用すべきかを記述する。

GuidanceはTopologyを定義しない。Business Pathは `transitions[]` によってのみ表現する。

---

# 6. Composition

`composition` はBusiness Flowに参加するNodeとScope、およびその包含関係を定義する。

```json
{
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
      }
    ]
  }
}
```

Scopeに属さないNodeを許容する。

配列順序にはExecution Order、Display Order、Priority、Transition Order等のSemantic Meaningを持たせない。

v1では1つのBusiness Flow内において1つのNodeは最大1つのScopeに所属する。

Scope Resource自身はNode Membershipを保持しない。

---

# 7. Transition

TransitionはNode間に存在するpossible Business Pathを表す。

```json
{
  "from": { "ref": "develop-meddic" },
  "to": { "ref": "build-proposal" },
  "guidance": "顧客課題と意思決定構造が十分に把握でき、具体的な提案によって検証を進める場合。"
}
```

> **Transition is a possible Business Path, not an execution command.**

`guidance` はOptionalであり、複数のPathからなぜこのPathを選択するのかを説明する。

Transitionには `condition`、`rules[]`、Expression等を持たせない。

---

# 8. Boundary and Transition Semantics

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

Cross-Scope Transitionでは概念的に以下のContextが関連する。

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

これはExecution Sequenceではない。

---

# 9. Branch / Merge / Loop / Backtracking

専用のGateway、Decision、Branch、Merge Elementは導入しない。

すべてTransition GraphのTopologyとして表現する。

Business FlowをDAGに限定しない。

同一Business Entityが同一Nodeへ繰り返し戻ることも許容し、Node Resource自体は複製しない。

---

# 10. Scope Boundary Crossing

Scope間Transitionは保存しない。

```text
develop-meddic ∈ discovery
build-proposal ∈ proposal
develop-meddic → build-proposal

             ↓ derive

discovery → proposal
```

CompilerまたはSemantic GraphがCompositionとNode Transitionから導出する。

---

# 11. No Semantic Order

`order`、`sequence`、`stepNumber`、`nextNode`、`previousNode` は定義しない。

Canvas上のPosition、Size、Z-order等はLayout Metadataとして別管理する。

---

# 12. Full Example

```json
{
  "$schema": "../../schemas/business-flow.v1.schema.json",
  "apiVersion": "honka/v1",
  "kind": "BusinessFlow",
  "id": "opportunity-development",
  "name": "Opportunity Development",
  "description": "商談開始から顧客理解、提案に至るBusiness Workの関係を表す。",
  "guidance": "商談は必ずしも一方向には進行しない。提案後であっても顧客理解が不足した場合はDiscoveryへ戻る。",
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
      "to": { "ref": "customer-meeting" }
    },
    {
      "from": { "ref": "customer-meeting" },
      "to": { "ref": "develop-meddic" }
    },
    {
      "from": { "ref": "develop-meddic" },
      "to": { "ref": "build-proposal" },
      "guidance": "具体的な提案によって検証を進める場合。"
    },
    {
      "from": { "ref": "review-proposal" },
      "to": { "ref": "develop-meddic" },
      "guidance": "提案内容を再検討するため顧客理解の更新が必要な場合。"
    }
  ]
}
```

---

# 13. Semantic Graph Representation

Compositionでは `HAS_SCOPE`、`HAS_NODE`、`CONTAINS` Relationを生成する。

TopologyではNode間の `TRANSITION` Relationを生成する。

Resource Identity、Composition、Topologyを区別する。

---

# 14. Validation Responsibilities

JSON SchemaはResource Envelope、Composition Structure、Reference Structure、Transition Structure、unknown properties等を検証する。

Semantic Validatorは以下を検証する。

```text
Scope reference exists
Node reference exists
Node belongs to at most one Scope in the Flow
Transition.from exists in Flow Composition
Transition.to exists in Flow Composition
Duplicate Transition detection
Scope Boundary Crossing resolution
```

Analysis LayerはUnreachable Node、Disconnected Subgraph、Cycle、Branch without Transition Guidance等をWarningまたはAnalysis Resultとして検出できる。

---

# 15. Architecture Decision Records

## ADR-F001: Business FlowをExecution Definitionとして扱わない
**Status:** Accepted

Business StructureとBusiness Pathを表現し、Runtime StateやExecution Logicを保持しない。

## ADR-F002: CompositionとTopologyを分離する
**Status:** Accepted

CompositionはScope ↔ Node Membership、TopologyはNode → Node Transitionとして保持する。

## ADR-F003: Node MembershipはBusiness Flowが所有する
**Status:** Accepted

Scope / Node Resource自身にはMembershipを保持しない。

## ADR-F004: 配列順序にSemantic Meaningを持たせない
**Status:** Accepted

Business PathはTransition Graphのみから判断する。

## ADR-F005: TransitionはNode間にのみ定義する
**Status:** Accepted

ScopeはTransition GraphのVertexとしない。

## ADR-F006: Scope間Transitionを保持しない
**Status:** Accepted

Scope Boundary CrossingはNode TransitionとCompositionから導出する。

## ADR-F007: Branch / Merge専用Elementを導入しない
**Status:** Accepted

Graph Topologyとして表現する。

## ADR-F008: Business FlowはCycleを許容する
**Status:** Accepted

Loop、Backtracking、Repeated Node Usageを許容する。

## ADR-F009: TransitionにOptional Guidanceを持たせる
**Status:** Accepted

Path Selectionの意味を記述する。

## ADR-F010: Transition GuidanceをBranch時にも必須としない
**Status:** Accepted

不足時はAnalysis LayerがWarningを生成できる。

## ADR-F011: TransitionにConditionを持たせない
**Status:** Accepted

Boundary ConstraintとPath Selection Contextを分離する。

## ADR-F012: Semantic Orderを保持しない
**Status:** Accepted

Order/Sequence/Next等を持たない。

## ADR-F013: LayoutをSemantic Modelから分離する
**Status:** Accepted

Canvas情報はLayout Metadataとして管理する。

## ADR-F014: 1 Flow内でNodeは最大1 Scopeに所属する
**Status:** Accepted

Scope Boundary Crossingを一意に導出するためZero-or-One Scopeとする。

## ADR-F015: Business FlowをGraph Container Patternとして扱う
**Status:** Accepted

High-Level ProcessとBusiness FlowはComposition + Transition Graphという同型のSemantic Patternを持つ。

ただしJSONを汎用 `members[]` に抽象化せず、Business Flowでは `nodes[]` を用いる。

---

# 16. Separation of Concerns

```text
Node
  Business Work

Scope
  Business Context Boundary

Business Flow
  Graph<Node>

High-Level Process
  Graph<Business Flow>

Layout
  Visual Representation

Runtime
  Execution State
```

Business FlowはNode固有Context、Scope固有Context、Capability Definition、Implementation、Runtime State、Layoutを保持しない。

---

# 17. Summary

```text
Business Flow
├─ Composition
│  ├─ nodes[]
│  └─ scopes[]
│     ├─ scope
│     └─ nodes[]
└─ transitions[]
   ├─ from
   ├─ to
   └─ guidance?
```

> **Business Flow = Graph<Node>**

ScopeはNode群に共通するBusiness Context Boundaryを与える。

Resource、Composition、Topologyを分離することで、Branch、Merge、Loop、Backtrackingを自然に表現しながらWorkflow Execution Engineになることを避ける。
