# Honka Node Schema Design v1

## 1. Scope

本書は、Honkaにおける `Node` のSemantic ModelおよびJSON Schema設計を定義する。

NodeはBusiness Flow上のBusiness Workを表現するSemantic Resourceである。

Nodeは単一実行される処理単位に限定されない。同一Node上で業務を複数回実施し、Business Contextを継続的に更新することを許容する。

---

# 2. Resource Envelope

すべてのNodeは以下のResource Envelopeを持つ。

```json
{
  "$schema": "../../schemas/node.v1.schema.json",
  "apiVersion": "honka/v1",
  "kind": "Node",
  "id": "develop-meddic",
  "name": "Develop MEDDIC"
}
```

## `$schema`

Nodeが準拠するJSON Schemaを指定する。

## `apiVersion`

Honka Semantic Modelのバージョンを指定する。

```text
honka/v1
```

## `kind`

Resource Typeを指定する。Nodeでは固定値とする。

```text
Node
```

## `id`

NodeのStable Identifier。

Nodeは他のSemantic ResourceやNavigationから参照されるためStable IDを持つ。

表示名の変更によって変更しない。

## `name`

Nodeの表示名。

---

# 3. Node Structure

Nodeは以下のSemantic Structureを持つ。

```text
Node
│
├─ description
├─ guidance
│
├─ entry
│   ├─ guidance
│   └─ rules[]
│       └─ description
│
├─ dataContract[]
│   ├─ field
│   ├─ required
│   ├─ usage
│   └─ guidance
│
├─ actions[]
│   ├─ capability
│   ├─ guidance
│   ├─ context[]
│   └─ expectedResponse[]
│
├─ rules[]
│   └─ description
│
├─ exit
│   ├─ guidance
│   └─ rules[]
│       └─ description
│
└─ outcome[]
    ├─ field
    └─ guidance
```

---

# 4. Description

`description` はNodeが表すBusiness Workを説明する。

```json
{
  "description": "顧客との商談活動を通じて、意思決定構造をMEDDICとして明らかにする。"
}
```

`description` はResourceそのものの意味を記述する。

---

# 5. Guidance

`guidance` は対象となるSemantic ResourceまたはContextを、人間またはAIがどのように理解・判断・利用すべきかを記述する。

```json
{
  "guidance": "顧客との会話から確認できた情報をもとにMEDDICを継続的に更新する。確認できていない情報を推測によって補完しない。"
}
```

Guidanceは以下のContextで使用できる。

```text
Node
Entry
Data Contract Item
Action
Action Context
Expected Response
Exit
Outcome
```

Guidanceは機械的な成立条件を定義しない。

明示的なBusiness ConstraintはRuleとして定義する。

---

# 6. Rule

Ruleは、特定のBusiness Workにおいて成立すべきBusiness Constraintを表す。

Ruleは独立した共有Resourceとして定義しない。

Ruleは、それを利用するNodeまたはNode内のBoundaryによって所有される。

```json
{
  "description": "商談がActiveであること"
}
```

Ruleは外部Resourceから直接参照されないためStable IDを持たない。

同一または類似する内容のRuleが複数Nodeに存在する場合も、それぞれ独立したRuleとして保持する。

Ruleの統合や共有Resource化は行わない。

---

# 7. Rule Ownership

Ruleには3種類のOwnership Scopeが存在する。

```text
Entry Rule
Node Rule
Exit Rule
```

Ruleの種類はRule自身の属性として保持せず、配置されるContextによって決定する。

```text
entry.rules[] → Entry Rule
rules[]       → Node Rule
exit.rules[]  → Exit Rule
```

---

# 8. Entry

`entry` はNodeに入るためのBoundaryを定義する。

```json
{
  "entry": {
    "guidance": "顧客との具体的な商談活動が開始され、意思決定構造を把握していく段階。",
    "rules": [
      {
        "description": "商談がActiveであること"
      }
    ]
  }
}
```

`guidance` はNodeへ入る状態のBusiness Contextを説明する。

`rules` はNodeへ入ることを許可するためのConstraintを定義し、Gateとして扱う。

---

# 9. Data Contract

`dataContract` はNodeがBusiness Workを遂行する際に扱うBusiness Dataを定義する。

```json
{
  "dataContract": [
    {
      "field": "Meeting.Notes",
      "required": true,
      "usage": "read",
      "guidance": "顧客の発言からMEDDICに該当する情報を判断するための根拠として利用する。"
    }
  ]
}
```

Data Contract Itemは以下の属性を持つ。

```text
field
required
usage
guidance
```

## 9.1 Field

Business Data Model上のFieldを参照する。

Field自体の意味はBusiness Data Model側で定義する。

Data Contract上のGuidanceは、そのFieldを対象Nodeにおいてどのように扱うかを定義する。

## 9.2 Required

`required` は対象FieldをNodeへのInput Contextとして提供することが契約上必須かを表す。

```text
true
  対象FieldをNodeへのInput Contextとして必ず提供する。

false
  対象FieldをInput Contextとして提供しなくてもNodeを利用できる。
```

`required` は以下を表さない。

```text
Field値がnullではならない
Fieldが重要である
Node終了時に値が必要である
Exit条件である
```

## 9.3 Usage

v1では以下を定義する。

```text
read
write
read-write
```

## 9.4 Data Guidance

対象FieldをこのNodeでどのように解釈・利用・更新するかを定義する。

```json
{
  "field": "Opportunity.Metrics",
  "required": false,
  "usage": "read-write",
  "guidance": "意思決定権を持つ顧客担当者の採用基準を記載する。先方の担当レベルの採用基準は適さない。"
}
```

---

# 10. Action

`actions` はNode上のBusiness Workで利用するBusiness Capabilityを定義する。

```json
{
  "actions": [
    {
      "capability": {
        "ref": "UpdateOpportunity"
      },
      "guidance": "顧客との会話から根拠を確認できたMEDDIC項目を更新する。"
    }
  ]
}
```

Capabilityは共有Semantic Resourceとして扱う。

NodeはImplementationではなくBusiness Capabilityを参照する。

Action自体はNode-local Elementであり、外部から参照されない限りStable IDを要求しない。

---

# 11. Action Context

Capabilityへ渡すBusiness Contextを定義する。

```json
{
  "context": [
    {
      "field": "Opportunity.Id",
      "required": true,
      "guidance": "更新対象となる商談を識別するために使用する。"
    }
  ]
}
```

---

# 12. Expected Response

Capabilityから受け取ることを想定するSemantic Outputを定義する。

```json
{
  "expectedResponse": [
    {
      "output": "updatedOpportunity",
      "guidance": "更新後の商談情報として後続の判断に利用する。"
    }
  ]
}
```

Expected ResponseはImplementation固有のResponse Schemaを表さない。

---

# 13. Node Rules

Node直下の `rules` は、Node上でBusiness Workを行っている間に適用されるConstraintを定義する。

```json
{
  "rules": [
    {
      "description": "顧客から確認できていないMEDDIC情報を推測によって補完しないこと"
    }
  ]
}
```

---

# 14. Exit

`exit` はNodeを離れるためのBoundaryを定義する。

```json
{
  "exit": {
    "guidance": "顧客の意思決定構造がMEDDICとして十分に把握され、後続の商談判断に利用できる状態を目指す。",
    "rules": [
      {
        "description": "後続の商談判断に必要なMEDDIC項目が確認されていること"
      }
    ]
  }
}
```

`guidance` はNode上のBusiness Workによって目指すBusiness Stateを説明する。

`rules` はNodeを離れることを許可するためのConstraintを定義する。

Exit Ruleを持たないNodeも許容する。

---

# 15. Outcome

`outcome` はNodeから後続のBusiness Workへ提供するBusiness Contextを定義する。

OutcomeはNodeの終了条件を表さない。

```json
{
  "outcome": [
    {
      "field": "Contract.SignedDocument",
      "guidance": "締結済みの正式な契約書として後続業務へ引き渡す。"
    },
    {
      "field": "Contract.SignedAt",
      "guidance": "契約が正式に締結された日時として後続業務で利用する。"
    }
  ]
}
```

Outcomeを持たないNodeでは `outcome` を省略できる。

---

# 16. Entry / Exit / Outcome

```text
Entry
  Nodeへ入ってよいか

Node
  Business Workを行う

Exit
  Nodeを離れてよいか

Outcome
  Nodeから何を後続へ提供するか
```

```text
             Entry
       guidance + rules
               │
               ▼
       ┌───────────────┐
       │     Node      │
       │               │
Input ─▶ Data Contract │
       │               │
       │ Guidance      │
       │ Rules         │
       │ Actions       │
       └───────┬───────┘
               │
               ▼
              Exit
       guidance + rules
               │
               ▼
            Outcome
               │
               ▼
     Downstream Context
```

---

# 17. Rule Inspection

RuleはNode-localで保持するが、Semantic Graph上では横断的なInspection対象とする。

Business Rules Viewは独立Rule Resourceの管理画面ではない。

各Nodeに存在するRuleを集約し、検索・分類・比較するためのSemantic Graph Explorerとして扱う。

Business Rules ViewからRuleの所有NodeおよびRuleの配置ContextへNavigationできる。

RuleのInspectionに永続的なRule IDは要求しない。

---

# 18. Semantic Rule Grouping

複数Nodeに存在する同一または類似RuleはRepository上で統合しない。

Semantic Analysis Layerは、それらを類似Constraintとして分類できる。

```text
Semantic Group:
Opportunity State

├─ Develop MEDDIC / Entry
│  「商談がActiveであること」
│
├─ Proposal Preparation / Entry
│  「有効な商談であること」
│
└─ Contract Negotiation / Entry
   「商談が終了していないこと」
```

Semantic Groupは元のRuleのOwnershipを変更しない。

LLMによるSemantic Analysisは以下に利用できる。

```text
類似Ruleの検出
Ruleの分類
Rule間の差分説明
矛盾候補の検出
重複候補の表示
```

---

# 19. Shared Resources and Local Elements

Semantic Elementは外部参照の有無によってResourceとNode-local Elementに分離する。

```text
Addressable Semantic Resources
├─ Node
├─ Data Model
│  └─ Field
└─ Capability

Node-local Semantic Elements
├─ Entry
├─ Rule
├─ Data Contract Item
├─ Action
├─ Exit
└─ Outcome Item
```

Addressable Semantic ResourceはStable IDを持つ。

Node-local Semantic Elementは、外部から独立して参照する必要がない限りStable IDを要求しない。

---

# 20. Repeated Node Usage

Nodeは一回限りの実行単位ではない。

同一Business Entityについて、同一Nodeを複数回利用できる。

```text
Develop MEDDIC

Meeting 1
  → Metrics更新

Meeting 2
  → Economic Buyer更新

Meeting 3
  → Decision Criteria更新

Meeting 4
  → Metrics再更新
```

各利用時にData Contractに従ってContextを提供する。

Exit Ruleが成立するまで同一Node上でBusiness Workを継続できる。

Node自身はLoop、Backtracking、Branchingを定義しない。

同一Nodeへの再訪やNode間のLoop、分岐、合流はBusiness Flowが保持するTransition Graphによって表現する。

---

# 21. MEDDIC Example

```json
{
  "$schema": "../../schemas/node.v1.schema.json",
  "apiVersion": "honka/v1",
  "kind": "Node",
  "id": "develop-meddic",
  "name": "Develop MEDDIC",

  "description": "顧客との商談活動を通じて、意思決定構造をMEDDICとして明らかにする。",

  "guidance": "顧客との会話から確認できた情報をもとにMEDDICを継続的に更新する。確認できていない情報を推測によって補完しない。",

  "entry": {
    "guidance": "顧客との具体的な商談活動が開始され、意思決定構造を把握していく段階。",
    "rules": [
      {
        "description": "商談がActiveであること"
      }
    ]
  },

  "dataContract": [
    {
      "field": "Meeting.Notes",
      "required": true,
      "usage": "read",
      "guidance": "顧客の発言からMEDDICに該当する情報を判断するための根拠として利用する。"
    },
    {
      "field": "Opportunity.Metrics",
      "required": false,
      "usage": "read-write",
      "guidance": "意思決定権を持つ顧客担当者の採用基準を記載する。先方の担当レベルの採用基準は適さない。"
    }
  ],

  "actions": [
    {
      "capability": {
        "ref": "UpdateOpportunity"
      },
      "guidance": "顧客との会話から根拠を確認できたMEDDIC項目を更新する。"
    }
  ],

  "rules": [
    {
      "description": "顧客から確認できていないMEDDIC情報を推測によって補完しないこと"
    }
  ],

  "exit": {
    "guidance": "顧客の意思決定構造がMEDDICとして十分に把握され、後続の商談判断に利用できる状態を目指す。",
    "rules": [
      {
        "description": "後続の商談判断に必要なMEDDIC項目が確認されていること"
      }
    ]
  }
}
```

---

# 22. Contract Execution Example

```json
{
  "$schema": "../../schemas/node.v1.schema.json",
  "apiVersion": "honka/v1",
  "kind": "Node",
  "id": "execute-contract",
  "name": "Execute Contract",

  "description": "承認済みの契約内容について契約当事者による正式な締結を行う。",

  "guidance": "最終承認された契約内容を変更せず、正式な契約書として締結する。",

  "entry": {
    "guidance": "契約内容が確定し、締結可能な状態。",
    "rules": [
      {
        "description": "契約が承認済みであること"
      }
    ]
  },

  "dataContract": [
    {
      "field": "Contract.ApprovedDocument",
      "required": true,
      "usage": "read",
      "guidance": "最終承認された契約書として締結に使用する。"
    },
    {
      "field": "Contract.Customer",
      "required": true,
      "usage": "read",
      "guidance": "契約相手および署名者を特定するために使用する。"
    }
  ],

  "actions": [
    {
      "capability": {
        "ref": "SendContractForSignature"
      },
      "guidance": "承認済み契約書を契約相手へ署名依頼として送信する。"
    }
  ],

  "rules": [
    {
      "description": "承認済みの契約内容を変更しないこと"
    }
  ],

  "exit": {
    "guidance": "契約当事者による正式な締結が完了した状態。",
    "rules": [
      {
        "description": "すべての契約当事者による署名が完了していること"
      }
    ]
  },

  "outcome": [
    {
      "field": "Contract.SignedDocument",
      "guidance": "締結済みの正式な契約書として後続業務へ提供する。"
    },
    {
      "field": "Contract.SignedAt",
      "guidance": "正式な契約締結日時として後続業務へ提供する。"
    }
  ]
}
```

---

# 23. Schema Validation Responsibilities

JSON SchemaはNodeの構造を検証する。

対象には以下を含む。

```text
apiVersion
kind
required properties
property types
enum values
unknown properties
array structure
identifier format for addressable resources
Rule structure
```

Node-local ElementにStable IDの存在は要求しない。

JSON SchemaはResource Referenceの成立性を検証しない。

---

# 24. Semantic Validation Responsibilities

Honka Semantic ValidatorはResource間およびNode内部の意味的整合性を検証する。

対象には以下を含む。

```text
Field reference exists
Capability reference exists
Capability output exists
Data Contract usage is valid for referenced Field
Outcome Field can be resolved
Rule semantics can be analyzed
```

Validation Pipeline:

```text
JSON
  ↓
JSON Schema Validation
  ↓
Typed Node AST
  ↓
Reference Resolution
  ↓
Semantic Validation
  ↓
Semantic Graph
```

RuleはReference Resolutionを必要としない。

---

# 25. Architecture Decision Records

本章はNode Semantic Modelの設計過程で検討された主要な選択肢、採用した設計、および採用しなかった設計を記録する。

設計変更を行う場合は、既存ADRのDecisionを暗黙に上書きせず、新しいADRまたは既存ADRの更新として理由を記録する。

## ADR-001: Nodeを単発の実行単位として扱わない

**Status:** Accepted

### Decision

NodeをBusiness Workを行う意味的な場所として定義する。

Nodeは複数回利用でき、一定期間Business Entityが同じNodeに留まることを許容する。

### Rejected

```text
Node = 一回のAction
Node = 一回のState Transition
Node = 一回のCapability Execution
```

---

## ADR-002: Goalを独立プロパティとして持たない

**Status:** Accepted

### Decision

独立した `goal` は定義しない。

目指すBusiness Stateは `exit.guidance` に記述する。

Nodeを離れるための厳密な条件は `exit.rules[]` に記述する。

### Rejected

GoalとExitを別概念として保持する設計。

---

## ADR-003: OutcomeをExitに統合しない

**Status:** Accepted

### Decision

```text
Exit
  Nodeを離れてよいか

Outcome
  Nodeから後続へ何を提供するか
```

として独立させる。

### Rejected

```text
Outcome = Nodeの終了状態
Outcome = Exit Rule
Outcomeを削除してExitへ統合
```

---

## ADR-004: EntryとExitをBoundaryとして扱う

**Status:** Accepted

### Decision

```text
Entry
├─ guidance
└─ rules[]

Exit
├─ guidance
└─ rules[]
```

として同型のBoundaryとする。

---

## ADR-005: GuidanceとRuleを分離する

**Status:** Accepted

### Decision

```text
Guidance
  どう理解・判断・利用するか

Rule
  何が成立しなければならないか
```

として責務を分離する。

---

## ADR-006: Data Contractのrequiredを値の必須性として扱わない

**Status:** Accepted

### Decision

`dataContract[].required` は以下のみを意味する。

> Node利用時に、そのFieldをInput Contextとして提供することが契約上必須か。

### Rejected

`required` に以下の意味を持たせる設計。

```text
Field値がnullではならない
Fieldが重要である
Node終了時に値が必要である
Goal達成に必要である
Ruleの強度
```

---

## ADR-007: Fieldの意味とNode内でのFieldの意味を分離する

**Status:** Accepted

### Decision

Field自体の意味はData Modelに保持する。

特定NodeでのFieldの解釈・利用方法はData Contract Itemの `guidance` に保持する。

---

## ADR-008: ActionはImplementationではなくCapabilityを参照する

**Status:** Accepted

### Decision

```text
Node
  ↓
Action
  ↓
Capability
  ↓
Implementation
```

とする。

### Rejected

NodeからREST Endpoint、Apex、Flow、MCP Tool等のImplementationを直接参照する設計。

---

## ADR-009: Ruleを共有Resourceとして扱わない

**Status:** Accepted

### Context

Ruleを独立Resourceとして定義し、Nodeから `ref` で参照する設計を検討した。

Ruleの再利用よりもBusiness WorkごとのContext独立性を優先する。

### Decision

RuleはNode-local Constraintとして保持する。

### Rejected

すべてのRuleを独立Resource化し、Nodeから `ref` で参照する方式。

---

## ADR-010: Inline RuleとRule Referenceを選択させない

**Status:** Accepted

### Decision

v1ではRuleを常にNode-localとする。

AuthorはRuleの保存方式を選択しない。

### Rejected

```text
Inline Rule
Shared Rule Reference
```

をRuleごとに選択する方式。

---

## ADR-011: RuleのInspectionに共有Identityを要求しない

**Status:** Accepted

### Decision

各Node-local RuleをSemantic Graph上でInspection可能にする。

同一ResourceへのReferenceを持たなくても、一覧表示、検索、分類、比較、Navigationを可能とする。

### Rejected

Inspectionを可能にするためだけにRuleを共有Resource化する設計。

---

## ADR-012: 類似Ruleを自動統合しない

**Status:** Accepted

### Decision

LLMはRuleのSemantic Analysisに利用する。

```text
類似Ruleの検出
分類
クラスタリング
差分説明
矛盾候補の検出
重複候補の表示
```

元RuleのOwnershipは変更しない。

### Rejected

LLMのSimilarity判定によってRuleを共通Resourceへ自動統合する方式。

---

## ADR-013: Business Rules ViewをCRUD画面として扱わない

**Status:** Accepted

### Decision

Business Rules ViewをSemantic Graph Explorerとして扱う。

各Nodeが所有するRuleを横断的にInspectionする。

### Rejected

Business Rules Viewを独立Rule ResourceのCRUD画面とする設計。

---

## ADR-014: JSONをSemantic Modelそのものとして扱わない

**Status:** Accepted

### Decision

Serialization FormatとしてJSONを使用する。

```text
Honka Semantic Model
        ↓ serialization
       JSON
```

### Rejected

XMLを標準Serialization Formatとする方式。

JSON SchemaのみをSemantic Specificationとみなす方式。

---

## ADR-015: Structural ValidationとSemantic Validationを分離する

**Status:** Accepted

### Decision

```text
JSON Schema Validation
  ↓
Structural Validity

Semantic Validator
  ↓
Semantic Validity
```

とする。

---

## ADR-016: Outcomeを必須にしない

**Status:** Accepted

### Decision

`outcome` はOptionalとする。

Outcomeを持たないNodeを許容する。

### Rejected

すべてのNodeにOutcomeを要求する設計。

---

## ADR-017: Exit Ruleを必須にしない

**Status:** Accepted

### Decision

`exit.rules` はOptionalとする。

明示的なExit Gateを持たないNodeを許容する。

### Rejected

すべてのNodeに機械的なExit Conditionを要求する設計。

---

## ADR-018: Node-local ElementにStable IDを要求しない

**Status:** Accepted

### Context

RuleをSemantic Graph上で識別し、InspectionやGit差分を扱う目的で、各RuleにStable IDを付与する設計を検討した。

```json
{
  "id": "contract-approved",
  "description": "契約が承認済みであること"
}
```

RuleをNode-local Elementとして扱い、外部Resourceから直接参照しない設計では、このIDはBusiness Semantic Model上のIdentityとして使用されない。

同様の性質はAction等のNode-local Elementにも存在する。

### Decision

Stable IDは、外部から独立して参照可能なAddressable Semantic Resourceに要求する。

```text
Addressable Resource
→ Stable ID required

Node-local Element
→ Stable ID not required
```

RuleにはStable IDを持たせない。

Actionについても外部参照要件がない限りStable IDを要求しない。

Data Contract ItemはField Referenceによって対象を識別する。

Outcome ItemもField Referenceによって対象を識別する。

### Rejected

すべてのSemantic ElementへStable IDを付与する方式。

RuleのInspectionを目的としてRule IDを永続化する方式。

Git diffの安定性のみを目的としてBusiness Semantic ModelへIDを追加する方式。

### Consequences

配列内Elementの編集追跡、UI上の一時的な識別、Semantic Graph構築時の内部Identityが必要な場合は、CompilerまたはRuntimeが内部Identityを生成する。

内部IdentityはHonka MetadataのPublic Contractとはしない。

---

## ADR-019: NodeにFlow Topologyを保持しない

**Status:** Accepted

### Context

Business Flowでは、Node間のLoop、Backtracking、Branching、Mergeが発生する。

これらをNode自身の `next`、`previous`、`order`、`scope` 等として保持すると、NodeのBusiness Workとしての意味と、特定Business Flow上の配置・接続関係が混在する。

### Decision

Node SchemaはBusiness WorkそのもののSemantic Contextのみを定義する。

以下はNode Schemaに保持しない。

```text
Scope membership
Node ordering
Node-to-Node transitions
Branching
Merge
Loop structure
Flow topology
```

NodeがどのScopeに配置され、どのNodeへ遷移可能かはBusiness Flow側のCompositionおよびTransition Graphが定義する。

同一Nodeへの再訪やLoopもNode自身の属性ではなく、Transition Graphによって表現する。

### Rejected

```text
node.scope
node.order
node.next
node.previous
node.transitions
node.branches
```

等をNode Schemaに保持する方式。

### Consequences

Nodeは特定のBusiness Flow Topologyから独立したAddressable Semantic Resourceとして扱える。

同一Nodeを異なるFlowまたは異なるScope Compositionから参照することをSemantic Model上妨げない。

---

# 26. Semantic Graph Representation

Node-local ElementはSemantic Graphへ展開できる。

Ruleは独立したAddressable Resourceではないが、所有Nodeおよび配置Contextとの関係としてGraph上で表現する。

```text
[Develop MEDDIC]
       │
       ├── ENTRY_RULE ──▶ [商談がActiveであること]
       │
       └── NODE_RULE ───▶ [未確認情報を推測しないこと]
```

Ruleの永続Stable IDは要求しない。

CompilerまたはSemantic Graph Runtimeは必要に応じて内部Identityを生成できる。

この内部Identityは以下の用途に限定する。

```text
Graph traversal
UI selection
Inspection
Compilation
Analysis
Temporary caching
```

内部IdentityをNode Schemaへ永続化することは要求しない。

---

# 27. Separation of Concerns

Node SchemaにはImplementation固有情報を保持しない。

以下はNode Schemaの対象外とする。

```text
REST endpoint
HTTP method
HTTP response code
MCP server
MCP tool name
Salesforce Flow API Name
Apex class
Authentication
Retry count
Timeout
Prompt implementation
Implementation-specific error
Internal runtime identity
UI selection state
Scope membership
Node ordering
Node-to-Node transitions
Branching
Merge
Loop structure
Flow topology
```

Implementation固有情報はCapability、Implementation、Compiler、RuntimeまたはUI Layerによって管理する。

Scope membershipおよびFlow TopologyはBusiness Flow側のComposition / Transition Graphによって管理する。

Nodeは以下を記述する。

```text
Business Work
Business Context
Business Data Contract
Business Capability
Business Constraint
Entry Boundary
Exit Boundary
Outcome / Context Transfer
```
