# Feature Specification: Decoupled Datastore

**Feature Branch**: `[001-decoupled-datastore]`  
**Created**: 2026-06-09  
**Status**: Draft  
**Input**: User description: "I want to specify a new feature, \"decoupled datastore\""

## Clarifications

### Session 2026-06-09

- Q: Should decoupled datastore be only operational, or should datastore resources have their own stable identity and lifecycle? → A: Datastore resources get their own stable identity and can outlive metadata revisions.
- Q: Should canonical datastore lookup require distribution IDs? → A: Use a datastore-owned resource identifier as canonical; distribution ID is optional reference metadata.
- Q: Should datastore handle distribution-ID-only requests during migration? → A: No. Datastore has no knowledge of distribution IDs; metastore performs all dataset/distribution-to-resource mapping.
- Q: What URI normalization is in scope now? → A: Keep all query parameters and fragments as part of URI identity; defer other normalization policies for now.
- Q: What refresh policy should be used for URI-backed resources? → A: Policy-driven refresh using ETag first, Last-Modified fallback, and explicit re-import when neither signal is usable.
- Q: What should be the canonical identity model for stored resources? → A: Datastore issues opaque canonical IDs on URI registration and reuses them for the same URI (idempotent create).
- Q: Should ETL and workflow states be explicitly captured and reported? → A: Yes. ETL run states, workflow transitions, and operator-facing reporting are required parts of this feature.
- Q: How should the ETL pipeline execute by default? → A: Queue-driven default `localize -> import -> post-import`, with immediate execution allowed via admin/API/CLI overrides.
- Q: Should ETL state transitions and report fields be specified now? → A: Yes. Define required transitions and minimum report fields now; defer storage/entity implementation details to planning.
- Q: Are datastore API breaks acceptable, and what compatibility should be preserved? → A: Some breaking changes are acceptable (especially internal/PHP API), but when practical we preserve existing class names, method signatures, and operational call patterns.
- Q: How should import customization be modularized? → A: Use an importer plugin architecture with stage-level extension points (`localize`, `import`, `post-import`) and clear default fallback behavior.
- Q: How should importer selection work in this phase? → A: Use a single globally configured active importer for now to reduce scope; defer priority-based deterministic multi-plugin selection to a future phase.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Independent Data Operations (Priority: P1)

As an operator, I want datastore import and query operations to continue independently of active changes in the metadata layer so that data access is less tightly coupled to metadata management and datastore resources retain their own operational identity.

**Why this priority**: This is the core value of the feature. It reduces coordination overhead and lowers the risk that metadata changes interrupt data access.

**Independent Test**: Verify that a datastore resource can be imported, queried, and removed using its datastore-facing identifier even when metadata updates are occurring separately.

**Acceptance Scenarios**:

1. **Given** a dataset distribution has been made available for import, **When** the datastore is imported and queried, **Then** the data can be retrieved without requiring a synchronous metadata write in the same operation.
2. **Given** the metadata layer is temporarily unavailable, **When** an existing datastore resource is queried, **Then** the datastore still returns the stored data and a clear status if metadata-backed details cannot be refreshed.

---

### User Story 2 - Stable Resource Lifecycle (Priority: P2)

As an operator, I want datastore resources to have a stable lifecycle that can be managed separately from metadata records so that data operations remain predictable across metadata revisions and the datastore remains addressable even when metadata changes.

**Why this priority**: Stable lifecycle management is required for maintainability once the datastore is decoupled.

**Independent Test**: Create, refresh, and remove a datastore resource while metadata revisions change independently, then confirm the datastore resource remains addressable by its own identifier.

**Acceptance Scenarios**:

1. **Given** a datastore resource exists, **When** the related metadata record is revised, **Then** the existing datastore resource remains addressable until it is explicitly refreshed or removed.
2. **Given** a datastore resource is no longer valid, **When** an operator requests it, **Then** the system reports that the resource is unavailable and indicates the datastore resource identifier that should be used instead.

---

### User Story 3 - Clear Operational Boundaries (Priority: P3)

As a maintainer, I want the datastore boundary to be clearly documented so that teams know which operations belong to datastore management versus metadata management.

**Why this priority**: Clear boundaries reduce regression risk and make the decoupled design easier to support.

**Independent Test**: Review the documented datastore and metadata responsibilities and verify that each common operation is assigned to one side of the boundary.

**Acceptance Scenarios**:

1. **Given** a common datastore task such as import, query, refresh, or removal, **When** the task is reviewed in the documentation, **Then** it is clearly assigned to datastore management.
2. **Given** a metadata-only task such as schema validation or metadata editing, **When** the task is reviewed in the documentation, **Then** it is clearly assigned to the metadata layer.

### Edge Cases

- What happens when metadata is updated while a datastore resource is being queried?
- How does the system handle a datastore resource whose metadata reference no longer exists?
- What happens when an operator attempts to refresh a datastore resource whose source definition changed since the last import?
- How are partially available or stale datastore resources reported to operators?
- How does the system respond when a client sends only a distribution ID and no datastore resource identifier?
- What happens when a client calls datastore directly with a dataset ID or distribution ID instead of a resource ID?
- What happens when cache dependency or invalidation code still relies on resource-to-distribution-to-dataset lookup?
- What happens when Drush reverse lookup commands infer dataset identity through distribution IDs?
- What happens when post-import status, dashboard, or reporting code still expects distribution-shaped payloads?
- What happens when cleanup or orphan-processing workflows depend on distribution reference entities that may not exist?
- What happens when a source URI does not support HEAD or does not return ETag/Last-Modified metadata?
- What happens when an ETL run fails after resource identity is resolved but before a new version is finalized?
- What happens when an ETL run is retried while a prior run for the same resource is still in progress?
- What happens when `localize` succeeds but `import` fails, or `import` succeeds but `post-import` fails?
- What happens when immediate execution is requested while queue jobs already exist for the same resource?
- What happens when an invalid ETL state transition is attempted (for example, failed -> running without retry)?
- What happens when preserving a legacy class or method signature conflicts with required decoupling behavior?
- What happens when no custom importer plugin is available for a resource and stage?
- What happens when a custom importer plugin overrides one stage but relies on default behavior for the others?
- What happens when multiple importer plugins are installed but only one is configured as active?

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The system MUST allow datastore import, refresh, query, and removal to be managed independently of metadata editing.
- **FR-002**: The system MUST preserve a stable datastore resource identifier that remains usable across metadata revisions until the resource is explicitly replaced or removed.
- **FR-003**: The system MUST allow an existing datastore resource to be queried even when the metadata layer is unavailable, provided the datastore resource itself is available.
- **FR-004**: The system MUST return a clear status when datastore metadata cannot be refreshed or is no longer in sync with the current metadata record.
- **FR-005**: The system MUST clearly distinguish datastore-managed operations from metadata-managed operations in user-facing documentation and operational guidance.
- **FR-006**: The system MUST report when a datastore resource is stale, missing, or replaced so operators can identify the current valid resource.
- **FR-007**: The system MUST avoid requiring a metadata rewrite as part of the normal datastore query path.
- **FR-008**: The system MUST support an explicit operator action to refresh datastore state when the underlying source or metadata definition changes.
- **FR-009**: The system MUST treat the datastore-owned resource identifier as the canonical lookup key for datastore import, query, refresh, and removal operations.
- **FR-010**: The system MUST NOT require a distribution ID as input to execute datastore operations against an existing datastore resource.
- **FR-011**: Datastore registration by URI MUST be idempotent: if a URI identity already exists, the system MUST return the existing canonical datastore resource identifier.
- **FR-012**: Datastore MUST issue opaque canonical resource identifiers for newly registered URI identities.
- **FR-013**: Datastore operations MUST accept canonical datastore resource identifiers as operational keys and MUST NOT require distribution IDs.
- **FR-014**: Datastore components and APIs MUST NOT parse, persist, or depend on dataset IDs or distribution IDs for execution logic; metadata-initiated datastore workflows MUST resolve dataset/distribution context to canonical datastore resource identifiers before datastore execution begins.
- **FR-015**: URI-based resource identity MUST preserve all query parameters and URI fragments exactly as provided.
- **FR-016**: Additional URI normalization policies (such as redirect canonicalization, host/path case rules, and default-port handling) MUST be explicitly deferred and treated as out of scope for this phase.
- **FR-017**: Refresh behavior MUST be policy-driven and MUST check ETag first when available.
- **FR-018**: If ETag is unavailable, refresh logic MUST use Last-Modified as the fallback signal for change detection.
- **FR-019**: If neither ETag nor Last-Modified is usable, the system MUST NOT auto-reimport and MUST require an explicit import request to create a new version.
- **FR-020**: The system MUST capture ETL run state transitions for each import attempt, including at minimum queued, running, succeeded, and failed states.
- **FR-021**: The system MUST record workflow state ownership boundaries so it is clear which states apply to canonical resources, resource versions, and ETL runs.
- **FR-022**: The system MUST expose operator-readable reporting for ETL runs, including current state, last successful run, last failure reason, and run timestamps.
- **FR-023**: Failed ETL runs MUST NOT mark a new resource version as successful, and failure outcomes MUST remain queryable for diagnostics.
- **FR-024**: Retried ETL runs MUST produce auditable run history linked to the same canonical resource identifier.
- **FR-025**: The ETL pipeline MUST define three ordered stages: `localize` (extract/local caching), `import` (load parsed data), and `post-import` (optional schema/data-dictionary adjustments).
- **FR-026**: In normal non-admin workflows, stage execution MUST be queue-driven, where successful completion of each stage enqueues or triggers the next stage in order.
- **FR-027**: The system MUST allow immediate execution of the same stages through admin/API/CLI-triggered operations without requiring queue-only orchestration.
- **FR-028**: Stage-level state and outcomes MUST be captured and reportable independently so operators can identify the exact stage of failure or completion.
- **FR-029**: When both queued and immediate execution requests target the same resource workflow, the system MUST enforce deterministic handling to avoid duplicate stage execution.
- **FR-030**: ETL run state transitions MUST follow a defined transition contract, including valid transitions from queued to running, and from running to terminal states succeeded or failed.
- **FR-031**: Retries MUST be represented as new ETL runs linked to the same canonical resource, rather than mutating a terminal run back to running.
- **FR-032**: Operator reporting MUST expose, at minimum, canonical resource identifier, run identifier, stage name, current state, terminal outcome, start timestamp, end timestamp (if terminal), and failure reason (when failed).
- **FR-033**: Reporting endpoints and operational views MUST support listing run history for a canonical resource in chronological order.
- **FR-034**: The feature MAY introduce breaking changes to datastore APIs when required to satisfy decoupling and workflow requirements, including internal/PHP APIs.
- **FR-035**: When practical, existing class names, method signatures, and operational invocation patterns MUST be preserved to reduce migration impact.
- **FR-036**: If a breaking change is introduced where practical preservation is not feasible, the system MUST provide explicit migration guidance and deprecation notes for affected consumers.
- **FR-037**: The system MUST provide an importer plugin architecture that allows extension modules to customize import behavior without replacing the entire default workflow.
- **FR-038**: Import customization MUST support stage-level extension points for `localize`, `import`, and `post-import`.
- **FR-039**: If a custom plugin does not implement a stage override, the system MUST fall back to default stage behavior.
- **FR-040**: The system MUST support selecting exactly one globally configured active importer for runtime execution in this phase.
- **FR-041**: If multiple importer plugins are installed, only the configured active importer MUST execute, while others remain available but inactive.
- **FR-042**: Priority-based deterministic multi-plugin selection is explicitly out of scope for this phase and deferred to a future enhancement.
- **FR-043**: Cache dependency and cache invalidation behavior for datastore summary, query, import status, and reporting responses MUST NOT require runtime resource-to-distribution-to-dataset lookup by datastore components.
- **FR-044**: When metadata references are present, metastore MAY provide cache dependencies that relate datastore resources back to datasets or distributions, but datastore execution MUST remain valid when that metadata linkage is absent.
- **FR-045**: Drush reverse lookup and other operational lookup commands that currently derive dataset identity through distribution IDs MUST be redesigned around canonical datastore resource identifiers or clearly documented as metadata-side compatibility tools.
- **FR-046**: Post-import status/result creation MUST use canonical datastore resource identifiers, ETL run records, or datastore resource metadata, and MUST NOT require distribution-shaped payloads.
- **FR-047**: Dashboard and admin reporting code MUST display datastore status from canonical datastore resources and ETL run reporting, with optional dataset/distribution context added only when metastore can resolve it.
- **FR-048**: Resource cleanup, purge, and orphan behavior MUST remove obsolete datastore resources, versions, mappings, and artifacts from datastore resource lifecycle context without depending on orphaned distribution reference entities as the only cleanup trigger.
- **FR-049**: Migration guidance MUST identify existing code paths that currently depend on distribution IDs, including datastore lookup helpers, Drush reverse lookup, cache invalidation, post-import/dashboard reporting, and cleanup/orphan processors.

### Key Entities *(include if feature involves data)*

- **Datastore Resource**: A queryable data asset managed by the datastore with an opaque canonical identifier and lifecycle state.
- **URI Identity**: The URI-derived identity used for idempotent resource registration and canonical ID reuse.
- **ETL Run**: A single execution attempt to import or refresh a resource, with lifecycle states and timestamps.
- **Pipeline Stage**: A named ETL stage (`localize`, `import`, or `post-import`) with ordered execution semantics and stage-level status.
- **Workflow State**: A state record associated with a canonical resource, resource version, or ETL run that indicates processing outcome and progression.
- **State Transition Contract**: The allowed ETL run state changes and retry semantics used to ensure consistent workflow behavior and reporting.
- **Importer Plugin**: A modular extension unit that can override one or more import pipeline stages while interoperating with default datastore behavior.
- **Stage Override**: A plugin-provided implementation for a specific pipeline stage that supersedes the default stage logic for selected workflows.
- **Metadata Record**: The catalog entry describing a dataset or distribution, which may reference one or more datastore resources.
- **Distribution Reference**: Optional metadata linkage from a dataset distribution to a datastore resource; not required as an operational key.
- **Metadata-Initiated Import**: A metastore-owned initiation path that resolves dataset distribution `downloadURL` values to canonical datastore resources before datastore execution begins.
- **Cache Dependency Context**: Optional metadata-provided context used to invalidate or cache datastore-facing responses without making dataset/distribution IDs datastore execution keys.
- **Operational Lookup Command**: A CLI or API helper, such as reverse dataset lookup, that resolves relationships for operators while preserving canonical datastore resource identifiers as operational keys.
- **Cleanup/Orphan Workflow**: Lifecycle processing that removes obsolete datastore resources or artifacts independently of whether a distribution reference entity exists.
- **Resource Status**: The current state of a datastore resource, such as available, stale, missing, or replaced.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: At least 95% of datastore query requests for existing resources succeed even when metadata updates are happening separately.
- **SC-002**: Operators can identify the current datastore resource for a changed dataset without consulting implementation details in under 2 minutes.
- **SC-003**: At least 90% of maintainer-reviewed datastore tasks are correctly classified as datastore-managed or metadata-managed in the published documentation.
- **SC-004**: Support requests caused by confusion between datastore and metadata responsibilities are reduced by 50% after rollout.
- **SC-005**: 100% of repeated URI registration requests in scope return the same canonical datastore resource identifier.
- **SC-006**: 100% of dataset/distribution-to-resource identifier resolution in scope is performed by metastore rather than datastore.
- **SC-007**: For refresh checks where metadata headers are available, at least 99% of change decisions are made using ETag or Last-Modified without full re-import.
- **SC-008**: For refresh checks where both ETag and Last-Modified are unavailable, 100% of new versions are created only via explicit import requests.
- **SC-009**: 100% of ETL runs in scope emit a terminal state (succeeded or failed) with timestamps and are retrievable in operator reporting.
- **SC-010**: At least 95% of failed ETL runs in scope include a machine-readable failure reason that supports operator troubleshooting without direct code inspection.
- **SC-011**: 100% of in-scope ETL executions expose stage-level outcome visibility for `localize`, `import`, and `post-import`.
- **SC-012**: At least 99% of queue-driven workflows execute stages in declared order without duplicate stage completion records.
- **SC-013**: 100% of ETL runs in scope follow the defined state transition contract with invalid transitions rejected and logged.
- **SC-014**: At least 99% of ETL run records in scope include the minimum required reporting fields for operator diagnostics.
- **SC-015**: For affected datastore internal/PHP surfaces in scope, at least 80% remain source-compatible when practical, with documented migration guidance for exceptions.
- **SC-016**: At least two distinct importer implementations in scope (for example, default parser and MySQL-native importer) can run through the same workflow contract without custom queue-worker replacement.
- **SC-017**: 100% of workflows in scope with partial stage overrides correctly execute overridden stages and default fallback stages in declared order.
- **SC-018**: 100% of in-scope runtime executions use the configured active importer and do not invoke inactive importer plugins.
- **SC-019**: 100% of datastore summary, query, import-status, and reporting responses in scope can compute cache dependencies or safe fallback dependencies without datastore-side distribution-ID lookup.
- **SC-020**: 100% of operational lookup commands in scope either accept canonical datastore resource identifiers or are documented as metadata-side compatibility commands.
- **SC-021**: 100% of post-import status and dashboard/reporting views in scope can render datastore status without requiring distribution-shaped payloads.
- **SC-022**: 100% of cleanup/orphan workflows in scope can remove obsolete datastore artifacts when distribution reference entities are absent.

## Assumptions

- The feature is focused on reducing coupling between datastore lifecycle management and metadata management, not on replacing the metadata layer.
- Datastore resources are independently addressable operational assets, while metadata remains the discovery and catalog layer.
- Distribution IDs remain useful for metadata discovery and traceability, but are not required for datastore operations.
- Metadata may continue to support referenced distributions, but datastore initiation receives canonical datastore resource identifiers after metastore resolves dataset distribution `downloadURL` values.
- Cache dependencies, reverse lookup helpers, dashboard/reporting code, and cleanup/orphan processors are part of the decoupling surface because current implementations may still infer dataset context through distribution IDs.
- Clients may store canonical datastore resource IDs for direct operations, or re-register by URI and reuse the returned canonical ID.
- URI identity preserves all query parameters and fragments for this phase.
- Broader URI normalization policy decisions are deferred to a later phase.
- Refresh checks prefer ETag and use Last-Modified fallback; absent both, explicit import is required for version creation.
- ETL and workflow states are first-class operational outputs and must be reportable without querying internal implementation details.
- Queue-driven stage orchestration is the default execution model, while immediate execution remains available for operational and administrative workflows.
- Storage-level entity modeling for ETL and workflow state records is intentionally deferred to planning.
- API compatibility is a pragmatic goal rather than an absolute constraint: preserving existing surfaces is preferred when practical, but decoupling correctness takes priority.
- Import extension behavior should prefer stage-level overrides over full workflow replacement to reduce maintenance cost for extension modules.
- Importer selection is intentionally single-active-plugin for this phase; priority-based multi-plugin selection is deferred to future scope.
- Existing datasets and datastore resources remain valid unless explicitly refreshed, replaced, or removed.
- Operators need clear guidance and status reporting more than they need a new user-facing workflow.
- The platform continues to support both datastore and metadata capabilities, but each must have clearer boundaries.