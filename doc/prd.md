# Honka — プロダクト要求仕様書

> **人間・AI・ソフトウェアが共有する Business Context。**

**ステータス:** v0.1  
**プロダクト:** Honka  
**文書の位置づけ:** 初期プロダクト定義およびアーキテクチャの基準文書

---

## 1. プロダクトコンセプト

Honka は、人間・AI・ソフトウェアが共有する **Business Context（業務文脈）** を設計・管理・提供するためのツールである。

Honka は業務の実行基盤そのものではない。実際の業務処理は、Salesforce Flow / Apex、Slack Agent、MCP Server、API、SAP、独自アプリケーションなど、既存の実行システムが担う。

Honka が保持するのは、それらの実装の背後にある意味である。

> **「この組織では、この仕事は何を意味するのか」**

Honka が蓄積する長期的な資産は、その組織固有の Business World Model、すなわち業務プロセス、概念、ルール、Data Contract、Capability、実装との対応関係である。

## 2. 名前と思想

**Honka** という名称は、和歌の本歌取りにおける **「本歌」** に由来する。

本歌取りでは、過去の歌を踏まえた表現を用いることで、読み手が本歌を知っていれば、短い表現からその背後にある情景・意味・文脈まで共有できる。

Honka は、この構造を業務システムへ持ち込む。

```text
                 HONKA
            Business Context
                  |
        +---------+---------+
        |         |         |
      Human       AI     Software
```

Flow、Apex、AI Agent、業務図、APIなどは、それぞれ業務意図の「表現」である。Honka は、それらの表現の背後にある共通の文脈を保持する。

> **同じ本歌を知っているから、異なる表現でも同じ意味を理解できる。**

## 3. 解決する問題

現在の企業では、業務の意味がさまざまな場所に分散している。

- Salesforce Flow / Apex
- API・連携コード
- Slackなどのコラボレーションツール
- Excel・スプレッドシート
- ドキュメント・Wiki
- アーキテクチャ図
- Prompt・Agent Instructions
- 会議で決まったルール
- 担当者個人の知識

たとえばAIに「契約送信機能を変更して」と指示しても、コードだけから次のことを確実に理解できるとは限らない。

- この会社における「契約」とは何か
- なぜ法務確認が必要なのか
- 誰がどの判断を担うのか
- どのデータが必要なのか
- どのBusiness Ruleに制約されるのか
- 「契約送信」というCapabilityが何を意味するのか
- なぜ現在CloudSignを利用しているのか
- どこまでが業務要件で、どこからが実装都合なのか

Honka は、こうした組織固有の文脈を明示的・構造的・バージョン管理可能・検索可能な形にする。

## 4. プロダクト仮説

Foundation Modelの推論能力は今後も改善していく。

それに伴い、Prompt Engineering、細かなCustom Instructions、モデル能力を補うための限定的なAgent Skillなどの一部は、モデル性能の向上によって相対的な価値が低下する可能性がある。

しかし、組織固有の文脈は異なる。

どれだけモデルが賢くなっても、特定企業における概念・ルール・業務・実装判断の意味を、外部から与えずに知ることはできない。

したがってHonkaが最適化する対象は、

> **モデル固有の知能ではなく、企業固有のBusiness Contextである。**

## 5. プロダクト境界 — Context, not Execution

Honka は、原則としてWorkflow Engineへ進化させない。

業務の実行は、それを最も適切に実行できる既存システムに任せる。

```text
                         HONKA
                    Business Context
                          |
          +---------------+---------------+
          |               |               |
     Salesforce       Slack / Agent    Custom App
     Flow / Apex          MCP              API
```

Honka はCapabilityが何を意味し、何を必要とし、現在どの実装によって実現されているかを記述する。

Honka自身が、その実装を呼び出す必要はない。

つまり、

- **Context / Control Plane — Honka**
- **Execution Plane — 既存システム**

と分離する。

この境界はHonkaの中核的な設計原則である。

## 6. Business Graph

Honkaの中心表現は **Semantic Business Graph** とする。

初期階層は次の3層とする。

```text
Value Chain
    |
High-level Process
    |
Business Flow
```

### 6.1 Value Chain

組織がどのように価値を生み出しているかを表現する。

この層は、探索的に業務を整理できるよう、意図的に自由度を高くする。

### 6.2 High-level Process

営業、契約、請求、提供など、大きな業務活動を表現する。

この層も比較的柔軟に扱う。

### 6.3 Business Flow

具体的な業務の流れを表現する。

```text
[契約作成]
     |
[法務確認]
     |
[承認]
     |
[契約送信]
```

Business Flowから、より強いSemanticを持たせる。

## 7. Semantic Node

Business FlowのNodeには、段階的に構造化された意味を付与できる。

Semantic Nodeは、必要に応じて次の情報を持つ。

- Description
- Actor / Role
- Data Contract
- Business Rules
- Capability
- Preconditions
- Postconditions
- Exceptions
- Implementation References

すべてを必須にはしない。

設計思想は、

> **中は柔らかく、境界は硬く。**

ユーザーは曖昧な業務図から始め、必要に応じて徐々に機械可読なモデルへ育てられる。

## 8. Data Model

Data ModelはBusiness Flow内部に埋め込まない。

Business FlowとBusiness Data Modelは対等な概念として扱う。

```text
Business Flow <----> Business Data Model
```

さらに、業務上の概念と物理実装を分離する。

```text
Business Data Model
        |
      mapping
        |
Physical Data Model
```

Business Data Modelの例:

```text
Contract
 |- customer
 |- amount
 '- approvalStatus
```

Salesforce上での物理実装例:

```text
Contract__c
Account__c
TotalAmount__c
ApprovalStatus__c
```

Salesforce等の実装プラットフォームが置き換わっても、Business Data Model自体は維持できなければならない。

## 9. Data Contract

Business FlowのNodeはData Model全体を扱うのではなく、その業務操作に必要なデータ境界を宣言する。

```yaml
LegalReview:
  requires:
    - Contract.amount
    - Contract.customer

  writes:
    - Contract.legalReviewed
```

Compilerは、たとえば次の不整合を検出できる。

- 参照しているFieldがData Modelに存在しない
- 下流が要求するDataを上流が供給していない
- Data Typeが互換でない
- Mappingが存在しない
- Referenceを解決できない

Authoringは両方向を許容する。

### Data Model First

既存Data Modelから、Nodeが必要・参照・更新するFieldを選択する。

### Contract First

Node側から、

```text
Contract.legalReviewed: boolean
```

のような要件を宣言する。

Fieldが存在しなければ、HonkaがBusiness Data Modelへの追加を提案できる。

## 10. Business Rule

Business Ruleは、業務上の明示的な制約を表現する。

可能な限り型を持ち、独立して参照可能にする。

Business Ruleは人間だけでなく、AI、Validator、Test、実装Toolingから利用される可能性があるため、Honkaの中でも特に厳密なSemantic Boundaryとする。

## 11. Capability

Capabilityは、実装方法とは独立した、

> **「業務として何ができるか」**

を表す。

例:

```text
SendContract
```

現在のImplementation Referenceとして、たとえば次を関連付けられる。

- Salesforce Apex
- CloudSign MCP
- REST API
- Human Operation
- Agent

重要なのは、

```text
Business Capability != Implementation
```

という分離である。

Honkaが管理するのは両者の意味上の対応関係であり、必ずしもExecutionではない。

## 12. Semantic Compiler

Honkaは **Business Context Compiler** としての性質を持つ。

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

Honkaの中核表現はSemantic Graphである。

UIもAIも、Raw YAML / JSONを概念上のSource of Truthとして直接扱わない。

Semantic Graphから、用途ごとのIntermediate Representationを生成する。

- Designer IR
- AI Context IR
- Test IR
- Platform IR
- Integration / Reference IR

## 13. AI Context Compilation

AIにRepository全体を無条件に渡さない。

たとえば `SendContract` に関する依頼なら、Honkaは必要なContextだけをCompileする。

- 対象Semantic Node
- 近傍の上流・下流Node
- 関連Data Contract
- Business Rule
- Capability
- 関連Implementation Reference

これにより、企業固有の文脈をAIへ提供しつつ、Context Windowの浪費を抑える。

## 14. MCP Interface

HonkaはBusiness Contextを外部AIへ提供するため、MCP Serverを持つ。

初期のConceptual Toolは次のようなものを想定する。

```text
find_process
get_context
get_rules
get_data_contract
get_capabilities
```

たとえば、

```text
User
  |
「契約送信機能を変更して」
  |
Codex / Development Agent
  |
Honka MCP
  |
get_context("contract sending")
  |
Business Context
  |
Code Analysis / Change
```

という流れになる。

MCP Interfaceは、Honka自身がAI Modelを所有しなくても、より高性能になっていくAIへ安定した企業固有Contextを供給できる重要な出口である。

## 15. AIによる編集

AIはCanonicalなBusiness Metadataを直接変更しない。

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

これは概念的には、

> **Business Contextに対するPull Request**

である。

たとえばユーザーがCanvas上の一部を選択して、

> 「この辺、なんかおかしくない？」

と尋ねる。

AIはSemantic Graphを読み、承認を迂回する経路などを発見し、Canvas上へGhost / Previewとして変更案を提示する。

Acceptされた変更案はMetadata Patchとなり、ValidationとReviewを経て反映される。

## 16. Git First

CanonicalなSemantic Business Contextは、MetadataとしてGitに保存する。

Repository構成例:

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

Semantic MetadataとVisual Layout Metadataは分離する。

```text
contract-flow.yaml
contract-flow.layout.yaml
```

Nodeを20px移動しただけでSemantic Diffを汚してはならない。

技術者には通常のGit Historyとして扱える。

非技術者にはUI上で、

- 変更履歴
- 変更案
- レビュー
- 以前の状態に戻す

などとして提示する。

内部ではMetadata Patch、Validation、Commit、必要に応じてPull Requestへ対応する。

## 17. Canvas UX

操作感は **FigmaよりMiroに近いもの** とする。

BPMN Editorのように最初から厳密な形式を要求しない。

Canvasには次のようなObjectを配置できる。

- Sticky
- Text
- Group
- Connector
- Semantic Node

ユーザーは最初、

```text
[契約つくる]
      |
[法務に確認]
      |
[送る]
```

程度から始めてよい。

そこから徐々に、

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

へSemanticを強化する。

> **表面はホワイトボード、裏面は型付きMetadata。**

## 18. Strictness Gradient

Honkaは、最初から企業活動のすべてを形式化することを要求しない。

初期の厳密さは概ね次の勾配とする。

| Layer | Strictness |
| --- | --- |
| Value Chain | Very Loose |
| High-level Process | Loose |
| Node Description | Loose |
| Flow Transition | Medium |
| Capability Contract | Strict |
| Data Contract | Strict |
| Business Rule | Strict |

AIやSystemとのBoundaryへ近づくほど、Semanticを強くする。

## 19. Frontend Architecture

初期Frontendは次を基本案とする。

- React
- TypeScript
- React Flow または同等のNode-based Canvas Library
- Zustand または同等の軽量Local State
- Zod または同等のSchema Validation

Canvas InteractionはLocal Firstとする。

Pointer Movement、Drag、Node Connection、Text EditingのたびにServer Round Tripを発生させない。

```text
Browser State
     |
debounced / asynchronous sync
     |
Draft Service
     |
Server
```

Undo / RedoもまずLocalで実現する。

Yjs等を利用したRealtime Collaboration / CRDTはMVP要件に含めない。

## 20. Backend Architecture

初期Backendは **TypeScript Modular Monolith** とする。

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

論理的なModule Boundaryを表現するためだけにMicroservices化しない。

Compiler / Validatorは、可能な限りInfrastructure IndependentなPure TypeScript Packageとして実装する。

CLIやCIから、Application Databaseなしでも実行できる状態を維持する。

## 21. Monorepo

初期構成案:

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

これはArchitecture Boundaryであり、それぞれを独立Deploymentすることを意味しない。

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

初期段階では不要な運用複雑性を持ち込まない。

少なくとも初期には次を必要としない。

- Kubernetes
- Neo4j
- Kafka
- Dedicated Vector Database
- Function-per-event型のServerless Architecture

Execution BehaviorとCostを予測しやすくするため、FaaS中心ではなくManaged Containerを基本とする。

## 23. GitとPostgreSQLの責務

### Git

Semantic Business ContextのAuthoritative Source of Truth。

### PostgreSQL

Application / Service運用上の状態を保持する。

例:

- Users
- Workspaces
- Permissions
- Drafts
- Editing State
- Comments
- 必要に応じたAI Conversations
- Git Connection Metadata
- Job State
- Audit Logs
- Indexes / Projections

Semantic GraphはCanonicalなGit Metadataから再構築可能でなければならない。

Operational DatabaseをSemantic Source of Truthにはしない。

ProductionではManaged PostgreSQLを利用する。

Local DevelopmentおよびIntegration TestではPostgreSQL Containerを利用してよい。

## 24. MVP

MVPでは巨大なEnterprise Architecture Suiteを作るのではなく、Honkaの中核仮説を検証する。

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

MVPで検証すべき問いは、

> **人間が自然に記述したBusiness Contextを段階的に構造化し、それをAIへ提供することで、AIが企業固有の仕事をより正しく理解できるか。**

である。

Workflow ExecutionはMVPの検証対象ではない。

## 25. Competitive Boundary

HonkaはWorkflow Engineとは異なるLayerを担当する。

```text
Workflow Engine
「業務をどう実行するか」
        ^
        |
      HONKA
「その業務は何を意味するか」
```

また、汎用Whiteboardとも異なる。

```text
Whiteboard
Diagram -> Human Understanding

Honka
Business Context
   -> Human Understanding
   -> AI Understanding
   -> Software Understanding
```

目標は単なる「AIがDiagramを理解すること」ではない。

**永続的で構造化されたShared Business Context Layer** を作ることである。

## 26. Core Design Principles

### 1. Context, not Execution

Honkaは業務を実行するのではなく、その意味を保持する。Executionは既存システムが担う。

### 2. Semantic Graph is the Core

Canvas、YAML、Database、各種生成表現は、Semantic GraphへのInterfaceまたはProjectionである。

### 3. Git is the Source of Truth

企業固有のBusiness ContextをVersioned Metadataとして管理する。

### 4. Loose Inside, Strict at Boundaries

人間の思考は柔軟なまま保ち、AI / SoftwareとのBoundaryだけを型付け・検証する。

### 5. One Honka, Many Expressions

Human-facing Canvas、AI Context、Test、Platform Mapping、Implementation Referenceは、すべて同じBusiness ContextにGroundingされる。

> **One Honka, Many Expressions.**

## 27. v0.1の明示的なNon-goals

Honka v0.1は、次のものを目指さない。

- 汎用Workflow Runtime
- Salesforce Flow、Camunda、Temporal等のExecution Engineの置き換え
- BPMN FirstのModeling Suite
- 汎用Diagram Application
- AI Model Training Platform
- 完全なEnterprise Data Catalog
- Source Code Repositoryの置き換え
- Graph Databaseの採用を前提としたArchitecture

これらは、HonkaのBusiness Contextという中核仮説を強化する場合にのみ、将来再検討する。
