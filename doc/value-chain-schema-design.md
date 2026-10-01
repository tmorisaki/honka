# Honka Value Chain Schema Design v1

## 1. Scope

本書はHonkaにおける `Value Chain` のSemantic ModelおよびHigh-Level Processとの関係を定義する。

Value ChainはOrganizationがどのようなBusiness Process群によって価値を生み出しているかを粗粒度で表現するAddressable Semantic Resourceである。

---

# 2. Definition

> **Value Chain = Organizationの価値提供を構成するHigh-Level Process群を参照するBusiness Landscape Resource**

Value ChainはHigh-Level Processを所有しない。

同じHigh-Level Processを複数のValue Chainから参照できる。

```text
Customer Lifecycle ───────┐
                          ├──▶ Contract Management
Partner Lifecycle ────────┘
```

---

# 3. Resource Envelope

```json
{
  "$schema": "../../schemas/value-chain.v1.schema.json",
  "apiVersion": "honka/v1",
  "kind": "ValueChain",
  "id": "customer-lifecycle",
  "name": "Customer Lifecycle"
}
```

Value ChainはBusiness Map、Navigation、Semantic Graph、MCP等から直接参照されるためStable IDを持つ。

---

# 4. Structure

```text
Value Chain
├─ id
├─ name
├─ description
├─ guidance
└─ processes[]
   └─ ref
```

v1ではValue Chain自身にTransition Graphを持たせない。

---

# 5. Description / Guidance

`description` はValue Chainが何を表すかを説明する。

`guidance` はこのValue Chainをどのような価値提供の観点で理解すべきかを記述する。

これらはBusiness Landscape、Navigation、Human Understanding、AI Context Retrievalに利用できる。

---

# 6. High-Level Process Composition

```json
{
  "processes": [
    { "ref": "opportunity-to-contract" },
    { "ref": "customer-onboarding" },
    { "ref": "customer-success" }
  ]
}
```

Value ChainはHigh-Level ProcessへのReferenceを保持するが、High-Level Process Resourceを所有しない。

High-Level Process側にもValue Chain Membershipを保持しない。

RelationshipのSource of TruthはValue Chain側のCompositionとする。

---

# 7. Multiple Value Chains

同一High-Level Processを複数Value Chainから参照できる。

これは重複Resourceを意味しない。

High-Level ProcessのBusiness Meaning、Goal、Guidance、Composition、Topologyは1つのResourceとして維持される。

---

# 8. No Semantic Order

`processes[]` の配列順序にはv1でSemantic Meaningを持たせない。

Value Chainは大まかな価値提供の流れとして視覚化できるが、その表示順やCanvas PositionをSemantic Contractとはしない。

Value Chain自体にTopologyが必要となる具体的なBusiness Requirementが生じるまでは導入しない。

---

# 9. Full Example

```json
{
  "$schema": "../../schemas/value-chain.v1.schema.json",
  "apiVersion": "honka/v1",
  "kind": "ValueChain",
  "id": "customer-lifecycle",
  "name": "Customer Lifecycle",
  "description": "顧客との関係構築から契約、導入、継続利用までの価値提供を表す。",
  "guidance": "顧客に提供する価値がどのHigh-Level Processによって形成・継続されるかを俯瞰する。",
  "processes": [
    { "ref": "opportunity-to-contract" },
    { "ref": "customer-onboarding" },
    { "ref": "customer-success" }
  ]
}
```

---

# 10. Semantic Graph Representation

```text
[Customer Lifecycle]
       │
       ├── INCLUDES ──▶ [Opportunity to Contract]
       ├── INCLUDES ──▶ [Customer Onboarding]
       └── INCLUDES ──▶ [Customer Success]
```

High-Level Processは独立Resourceであるため、別Value Chainから同一VertexへRelationを張ることができる。

---

# 11. Validation Responsibilities

JSON SchemaはValue Chain自身の構造を検証する。

Semantic ValidatorはHigh-Level Process Referenceの存在、Resource Type、重複Reference等を検証する。

配列順序はBusiness Semanticsとして検証しない。

---

# 12. Architecture Decision Records

## ADR-V001: Value ChainをAddressable Semantic Resourceとして扱う
**Status:** Accepted

Business Map、Navigation、Semantic Graph等から直接参照するためStable IDを持つ。

## ADR-V002: Value ChainはHigh-Level Processを参照する
**Status:** Accepted

Value ChainはHigh-Level Process群によって構成されるBusiness Landscape Resourceとする。

## ADR-V003: High-Level Processを所有しない
**Status:** Accepted

High-Level Processは独立Resourceとし、複数Value Chainから再利用可能とする。

## ADR-V004: High-Level Process側にValue Chain Membershipを持たせない
**Status:** Accepted

Relationshipを二重管理しない。

## ADR-V005: v1ではValue ChainにTransition Graphを持たせない
**Status:** Accepted

Value ChainはBusiness Landscape上のCompositionを主責務とする。Topologyの必要性が具体化するまで導入しない。

## ADR-V006: processes配列順序にSemantic Meaningを持たせない
**Status:** Accepted

表示順やCanvas LayoutをSemantic Contractとしない。

---

# 13. Relationship Model

```text
Organization
   │
   │ composition / landscape
   ▼
Value Chain
   │
   │ references
   ▼
High-Level Process
   │
   │ Graph<Business Flow>
   ▼
Business Flow
   │
   │ Graph<Node>
   ▼
Node
```

これはResource Ownership Treeではない。

特に `Value Chain → High-Level Process` は再利用可能なReference Relationshipである。

---

# 14. Summary

```text
Value Chain
├─ id
├─ name
├─ description
├─ guidance
└─ processes[]
   └─ ref
```

> **Value Chain = Organizationの価値提供を構成するHigh-Level Process群を参照するBusiness Landscape Resource**

High-Level ProcessはValue Chainから独立したAddressable Semantic Resourceとして扱い、複数Value Chainから参照できる。

v1ではValue ChainをGraph Containerにはせず、Business Landscape上のCompositionに集中させる。
