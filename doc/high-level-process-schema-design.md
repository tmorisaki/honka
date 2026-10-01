# Honka High-Level Process Schema Design v1

## 1. Scope

本書はHonkaにおける `High-Level Process` のSemantic ModelおよびJSON Schema設計を定義する。

High-Level Processは複数のBusiness FlowをBusiness Work UnitとしてCompositionし、それらの間に存在するBusiness Pathを表現するAddressable Semantic Resourceである。

High-Level Processは、人間およびAIが業務の目的、概要、関連Contextを粗粒度で理解するためのSemantic Layerでもある。

---

# 2. Definition

> **High-Level Process = Business Flow / ScopeのCompositionとBusiness Flow間のTopologyを持ち、業務のGoalとGuidanceを粗粒度のBusiness Contextとして提供するGraph Container**

```text
High-Level Process
├─ goal
├─ guidance
├─ Composition
│  ├─ Unscoped Business Flow
│  └─ Scope ↔ Business Flow
└─ Transition Graph
   └─ Business Flow → Business Flow
```

Business Flowと同じGraph Container Patternを持つが、VertexはBusiness Flowである。

---

# 3. Resource Envelope

```json
{
  "$schema": "../../schemas/high-level-process.v1.schema.json",
  "apiVersion": "honka/v1",
  "kind": "HighLevelProcess",
  "id": "opportunity-to-contract",
  "name": "Opportunity to Contract"
}
```

High-Level ProcessはBusiness Map、Semantic Graph、MCP、AI Context Retrieval等から直接参照されるためStable IDを持つ。

---

# 4. Structure

```text
High-Level Process
├─ id
├─ name
├─ description
├─ goal
├─ guidance
├─ composition
│  ├─ flows[]
│  └─ scopes[]
│     ├─ scope
│     └─ flows[]
└─ transitions[]
   ├─ from
   ├─ to
   └─ guidance?
```

---

# 5. Description

`description` はこのHigh-Level Processが何であるかを説明する。

```json
{
  "description": "商談形成から提案、契約締結までのBusiness Processを表す。"
}
```

---

# 6. Goal

`goal` は、このHigh-Level ProcessによってBusinessとして何が達成されているべきかを記述する。

```json
{
  "goal": "顧客の課題と意思決定構造を理解し、顧客と自社の双方が具体的な提案と契約を判断できる状態にする。"
}
```

GoalはCompletion Contractではない。

```text
Goal
  何が達成されているべきか

Exit Rule
  離れるために何が成立しなければならないか
```

High-Level ProcessのGoalをTransition Conditionや機械的な完了判定として扱わない。

> **High-Level Process Goal describes achievement, not completion.**

---

# 7. Guidance

`guidance` は、このHigh-Level Processで何をするのか、どのような観点でBusiness Flow群を理解・利用するのかを記述する。

```json
{
  "guidance": "顧客との対話や既存情報をもとに、課題、成功条件、意思決定構造を継続的に把握する。必要に応じて過去の顧客事例や社内知識を検索し、提案検討に必要なBusiness Contextを形成する。"
}
```

GuidanceはExecution Contractではない。

人間の業務理解、AI Context Retrieval、Semantic Search、関連Knowledgeの探索等に利用できる粗粒度のBusiness Contextである。

---

# 8. Context Retrieval

High-Level Processの `name`、`description`、`goal`、`guidance` は、関連Business Contextを探索する入口として利用できる。

```text
User / Agent Intent
       ↓
High-Level Process Semantic Context
├─ name
├─ description
├─ goal
└─ guidance
       ↓
Semantic / Vector Retrieval
       ↓
Relevant Business Flow / Scope / Node / Knowledge
```

特定の検索実装、Embedding方式、Vector Engine等はHigh-Level Process Schemaの責務ではない。

---

# 9. Composition

CompositionはHigh-Level Processに参加するBusiness FlowとScope、およびその包含関係を定義する。

```json
{
  "composition": {
    "flows": [],
    "scopes": [
      {
        "scope": { "ref": "discovery" },
        "flows": [
          { "ref": "opportunity-development" },
          { "ref": "customer-discovery" }
        ]
      },
      {
        "scope": { "ref": "commercial" },
        "flows": [
          { "ref": "proposal-development" },
          { "ref": "pricing-negotiation" }
        ]
      }
    ]
  }
}
```

Scopeに属さないBusiness Flowを許容する。

配列順序にはSemantic Meaningを持たせない。

v1では1つのHigh-Level Process内において1つのBusiness Flowは最大1つのScopeに所属する。

Scope Resource自身はFlow Membershipを保持しない。

---

# 10. Scope

High-Level ProcessでもScopeを利用できる。

```text
High-Level Process
├─ Scope: Discovery
│  ├─ Opportunity Development
│  └─ Customer Discovery
├─ Scope: Commercial
│  ├─ Proposal Development
│  └─ Pricing / Negotiation
└─ Scope: Contracting
   └─ Contract Execution
```

Scope ResourceのSchemaはBusiness Flowで使用するScopeと同一である。

包含対象の違いはGraph Container側のCompositionによって表現する。

---

# 11. Transition Graph

TransitionはBusiness Flow間に存在するpossible Business Pathを表す。

```json
{
  "from": { "ref": "opportunity-development" },
  "to": { "ref": "proposal-development" },
  "guidance": "顧客理解が提案による検証を開始できる程度に形成された場合。"
}
```

TransitionはExecution Commandではない。

`guidance` はOptionalであり、Path Selectionの意味を説明する。

Condition、Rule、ExpressionはTransitionに持たせない。

---

# 12. Branch / Merge / Loop / Backtracking

High-Level ProcessはBusiness Flowと同様にBranching、Merge、Loop、Backtracking、Repeated Business Flow Usage、Optional Pathsを許容する。

専用Gatewayは導入しない。TopologyはBusiness Flow間Transitionのみで表現する。

---

# 13. Scope Boundary Crossing

Business Flow間Transitionが異なるScopeを跨ぐ場合、Scope Boundary CrossingをCompositionから導出する。

Scope間Transitionを別途保存しない。

---

# 14. No Semantic Order

`flows[]` の配列順序にExecution OrderやBusiness Orderを持たせない。

`order`、`sequence`、`nextFlow`、`previousFlow` 等を定義しない。

TopologyはTransition Graphのみで表現する。

Canvas Layoutは別Metadataとする。

---

# 15. Full Example

```json
{
  "$schema": "../../schemas/high-level-process.v1.schema.json",
  "apiVersion": "honka/v1",
  "kind": "HighLevelProcess",
  "id": "opportunity-to-contract",
  "name": "Opportunity to Contract",
  "description": "商談形成から提案、契約締結までのBusiness Processを表す。",
  "goal": "顧客の課題と意思決定構造を理解し、顧客と自社の双方が具体的な提案と契約を判断できる状態にする。",
  "guidance": "顧客との対話や既存情報をもとにBusiness Contextを継続的に形成する。必要に応じて過去の顧客事例や社内知識を検索し、判断に利用する。",
  "composition": {
    "flows": [],
    "scopes": [
      {
        "scope": { "ref": "discovery" },
        "flows": [
          { "ref": "opportunity-development" },
          { "ref": "customer-discovery" }
        ]
      },
      {
        "scope": { "ref": "commercial" },
        "flows": [
          { "ref": "proposal-development" },
          { "ref": "pricing-negotiation" }
        ]
      },
      {
        "scope": { "ref": "contracting" },
        "flows": [
          { "ref": "contract-execution" }
        ]
      }
    ]
  },
  "transitions": [
    {
      "from": { "ref": "opportunity-development" },
      "to": { "ref": "customer-discovery" }
    },
    {
      "from": { "ref": "customer-discovery" },
      "to": { "ref": "opportunity-development" }
    },
    {
      "from": { "ref": "opportunity-development" },
      "to": { "ref": "proposal-development" },
      "guidance": "具体的な提案によって顧客との検証を進める場合。"
    },
    {
      "from": { "ref": "proposal-development" },
      "to": { "ref": "opportunity-development" },
      "guidance": "提案を具体化する過程で顧客理解の更新が必要になった場合。"
    },
    {
      "from": { "ref": "proposal-development" },
      "to": { "ref": "pricing-negotiation" }
    },
    {
      "from": { "ref": "pricing-negotiation" },
      "to": { "ref": "contract-execution" }
    }
  ]
}
```

---

# 16. Validation Responsibilities

JSON SchemaはHigh-Level Process自身の構造を検証する。

Semantic ValidatorはScope / Business Flow Reference、Business FlowのZero-or-One Scope Membership、Transition Reference、Duplicate Transition、Scope Boundary Crossing等を検証する。

Analysis LayerはDisconnected Flow、Unreachable Flow、Cycle、Branch without Guidance等をWarningまたはAnalysis Resultとして検出できる。

---

# 17. Architecture Decision Records

## ADR-H001: High-Level ProcessをGraph Containerとして扱う
**Status:** Accepted

Business FlowをVertexとするComposition + Transition Graphとして定義する。

## ADR-H002: High-Level ProcessはScopeを利用できる
**Status:** Accepted

ScopeはBusiness Flow群に共通するBusiness Context Boundaryとして利用する。

## ADR-H003: Scope ResourceをHigh-Level専用に分岐しない
**Status:** Accepted

Business Flowで利用するScopeと同じResource Typeを使用する。MembershipはHigh-Level Process Compositionが保持する。

## ADR-H004: Goalを持つ
**Status:** Accepted

Goalは何が達成されているべきかを表す。Completion ContractやExit Ruleとして扱わない。

## ADR-H005: Guidanceを持つ
**Status:** Accepted

Guidanceは何をするProcessか、どのように理解するかを表し、人間の業務理解やAI Context Retrievalに利用する。

## ADR-H006: Branch / Merge専用Elementを持たない
**Status:** Accepted

TopologyはBusiness Flow間Transitionで表現する。

## ADR-H007: Cycleを許容する
**Status:** Accepted

Loop、Backtracking、Repeated Business Flow Usageを許容する。

## ADR-H008: Transition GuidanceはOptionalとする
**Status:** Accepted

Path Selection Contextとして利用する。

## ADR-H009: Transition Conditionを持たせない
**Status:** Accepted

High-Level ProcessをExecution Definitionにしない。

## ADR-H010: 配列順序にSemantic Meaningを持たせない
**Status:** Accepted

TopologyはTransition Graphのみから判断する。

## ADR-H011: 1 Process内でBusiness Flowは最大1 Scopeに所属する
**Status:** Accepted

Scope Boundary Crossingを一意に導出するためZero-or-One Scopeとする。

## ADR-H012: Graph Container PatternはSemantic Modelとして共通化し、Serializationは具体化する
**Status:** Accepted

概念上はBusiness Flowと同型だが、JSONでは汎用 `members[]` ではなく `flows[]` を使用する。

---

# 18. Separation of Concerns

```text
High-Level Process
  Goal
  Broad Guidance
  Graph<Business Flow>

Business Flow
  Graph<Node>

Scope
  Business Context Boundary

Node
  Business Work
```

High-Level ProcessはBusiness Flow内部のNode Topology、Data Contract、Action、Capability Implementation、Runtime State、Search Implementation、Canvas Layoutを保持しない。

---

# 19. Summary

```text
High-Level Process
├─ id
├─ name
├─ description
├─ goal
├─ guidance
├─ composition
│  ├─ flows[]
│  └─ scopes[]
│     ├─ scope
│     └─ flows[]
└─ transitions[]
   ├─ from
   ├─ to
   └─ guidance?
```

> **High-Level Process = Goal + Guidance + Graph<Business Flow>**

GoalとGuidanceはContractではなく、粗粒度のBusiness Meaningとして人間の業務理解とAIのContext Retrievalを支える。

CompositionとTopologyによって、分岐、合流、繰り返し、戻りを自然に表現する。
