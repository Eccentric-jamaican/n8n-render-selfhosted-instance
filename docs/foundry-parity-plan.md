# Foundry-Parity Plan For This n8n Fork

## Goal
Build a practical, open, extensible platform in this fork that approaches key Palantir Foundry capabilities, with special focus on Ontology + AIP-style governed AI workflows.

This is not a clone target. It is a capability target:
- Semantic business object model
- Governed operational actions
- Lineage, auditability, and trust
- AI that can reason over enterprise objects with guardrails

## Capability Gap Matrix

| Foundry Capability | What It Means | Current n8n Baseline | Gap | Priority | Proposed n8n Target |
| --- | --- | --- | --- | --- | --- |
| Ontology Object Types | Typed business entities (Customer, Asset, Incident) | Workflows mostly process untyped JSON | High | P0 | Add object type registry with schema, identity keys, and versioning |
| Link Types | Explicit graph relationships across objects | Relationships are implicit in data payloads | High | P0 | Add typed links (`owns`, `dependsOn`, `assignedTo`) with constraints |
| Action Types | First-class business actions with parameters and policy | Node execution exists, but not action semantics on domain objects | High | P0 | Add action definitions bound to object/link types and executable workflows |
| Action Log / Audit | Immutable who/what/when/why for actions | Execution logs exist, but no business action ledger | High | P0 | Add append-only action log table + API + UI timeline |
| Data Lineage / Provenance | Source-to-consumer trace for data and decisions | Partial at workflow execution level | Medium-High | P1 | Add lineage edges between object snapshots, actions, and workflows |
| Policy/Access Model | Fine-grained permissions by object/action/field | Existing RBAC/permissions, not ontology-granular | High | P1 | Add object/action scopes and policy checks in API + UI gating |
| Operational Applications | Role-specific apps over ontology + actions | n8n editor/ops UI, but limited domain app layer | Medium | P1 | Build operational workspaces in editor-ui tied to ontology objects |
| AI Ontology Grounding (AIP-like) | AI can query/act on governed business objects | AI nodes exist; ontology grounding is limited | High | P1 | Add ontology-aware tools for AI nodes + strict action authorization |
| Evals for AI | Repeatable quality/safety tests for AI behaviors | No first-class eval suite connected to ontology goals | Medium | P2 | Add evaluation harness for prompts/tools/actions with pass thresholds |
| Model Governance | Model routing, approval, and usage guardrails | Basic model config; limited governance workflow | Medium | P2 | Add model policy layer + audit fields + rollout controls |
| Semantic Search / Object Discovery | Find objects and relationships by meaning | Generic search, not ontology-native | Medium | P2 | Add object index + semantic query endpoint + UI search |
| Simulation / What-if | Test action outcomes before production apply | Limited dry-run patterns | Medium | P3 | Add action simulation mode + diff preview |

## Proposed Architecture In This Monorepo

```mermaid
flowchart LR
  A[packages/@n8n/api-types<br/>Ontology contracts] --> B[packages/cli<br/>Ontology API + policy + action runtime]
  B --> C[packages/workflow<br/>Object/link/action traversal utils]
  B --> D[packages/core<br/>Execution + lineage hooks]
  B --> E[packages/@n8n/db<br/>Ontology/action log persistence]
  B --> F[packages/frontend/editor-ui<br/>Ontology & operations UI]
  B --> G[packages/@n8n/nodes-langchain + ai-*<br/>Ontology-aware AI tooling]
  D --> H[Action Log + Lineage Graph]
  G --> B
  F --> B
```

## Package-Level Placement

### `packages/@n8n/api-types`
- Add shared interfaces for:
  - `OntologyObjectType`
  - `OntologyLinkType`
  - `OntologyActionType`
  - `OntologyPolicy`
  - `ActionLogEntry`
  - `LineageEdge`
- Include versioned DTOs to protect FE/BE compatibility.

### `packages/cli`
- Create backend module for ontology (controller/service/repository).
- Endpoints:
  - `GET/POST/PATCH /ontology/object-types`
  - `GET/POST/PATCH /ontology/link-types`
  - `GET/POST /ontology/action-types`
  - `POST /ontology/actions/:id/execute`
  - `GET /ontology/action-log`
- Enforce policy checks for all action execution paths.

### `packages/@n8n/db`
- Add tables/entities:
  - `ontology_object_type`
  - `ontology_link_type`
  - `ontology_action_type`
  - `ontology_action_log`
  - `ontology_lineage_edge`

### `packages/workflow`
- Add traversal helpers for ontology graph navigation.
- Add validation utilities:
  - link cardinality checks
  - action parameter shape validation

### `packages/core`
- Add execution hooks to emit:
  - action log records
  - lineage edges
  - policy decision traces

### `packages/frontend/editor-ui`
- New sections:
  - Ontology Manager (object/link/action definitions)
  - Object Explorer (instances + relationships)
  - Action Timeline (audit view)
- All text via i18n package.

### `packages/@n8n/nodes-langchain` and AI packages
- Introduce ontology-aware tools:
  - `findObjects`
  - `getObjectContext`
  - `executeAction` (policy-checked)
- Restrict AI tools through explicit allowlists and runtime permission checks.

## Delivery Phases

## Phase 0: Foundations (2-3 weeks)
- API types + DB schema + minimal CRUD for object/link/action definitions.
- Basic UI for defining ontology types.
- Deliverable: persistent ontology metadata in system.

## Phase 1: Action Runtime + Audit (2-4 weeks)
- Action execution endpoint tied to workflows.
- Immutable action log and UI timeline.
- Deliverable: governed, traceable business actions.

## Phase 2: Lineage + Object Explorer (3-5 weeks)
- Emit lineage edges from action/workflow execution.
- Add object graph explorer UI.
- Deliverable: trust and explainability layer.

## Phase 3: AI Grounding + Guardrails (3-6 weeks)
- Ontology-aware AI tools and strict authorization.
- Evaluation harness for action correctness and safety.
- Deliverable: AIP-style enterprise AI workflow primitives.

## MVP Scope (First Shippable Slice)

Target one domain object set only:
- `Company`
- `Contact`
- `Deal`
- links: `CONTACT_BELONGS_TO_COMPANY`, `DEAL_FOR_COMPANY`
- actions: `qualifyDeal`, `assignOwner`, `closeDeal`

Success criteria:
- Define types in UI
- Execute action through API/workflow
- View audit log entry with actor + timestamp + payload hash
- Query object relationships in explorer

## First PR Sequence

1. `docs`: add this plan and architecture decisions.
2. `api-types + db`: ontology contracts and entities.
3. `cli`: ontology CRUD endpoints + policy middleware scaffold.
4. `editor-ui`: ontology definition screens (MVP forms + lists).
5. `core/workflow`: action log emitters + lineage hooks.
6. `ai`: ontology-aware tool wrappers for AI nodes.

## Risks And Mitigations

| Risk | Impact | Mitigation |
| --- | --- | --- |
| Scope explosion from broad “Foundry parity” goal | Slow delivery | Strict phased milestones with MVP object set |
| Permission model complexity | Security gaps | Start deny-by-default, add explicit scope grants |
| Performance from lineage graph growth | Query latency | Use append-only edges + indexed query surfaces |
| AI unsafe actions | Production risk | Hard authorization checks + eval gates before rollout |

## Sources Used For Capability Framing

- https://www.palantir.com/docs/foundry/aip
- https://www.palantir.com/docs/foundry/object-link-types/type-reference/
- https://www.palantir.com/docs/foundry/object-link-types/allow-editing
- https://www.palantir.com/docs/foundry/action-types/action-log
- https://www.palantir.com/docs/foundry/ontologies/ontology-permissions
- https://www.palantir.com/docs/foundry/aip-evals/overview/

