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
- **FR-011**: The metastore MUST be responsible for resolving dataset IDs and distribution IDs to canonical datastore resource identifiers.
- **FR-012**: Datastore operations MUST accept only canonical datastore resource identifiers as operational keys.
- **FR-013**: Datastore components and APIs MUST NOT parse, persist, or depend on dataset IDs or distribution IDs for execution logic.
- **FR-014**: When datastore receives a non-resource identifier, it MUST reject the request with an actionable response directing clients to resolve identifiers through metastore first.
- **FR-015**: URI-based resource identity MUST preserve all query parameters and URI fragments exactly as provided.
- **FR-016**: Additional URI normalization policies (such as redirect canonicalization, host/path case rules, and default-port handling) MUST be explicitly deferred and treated as out of scope for this phase.

### Key Entities *(include if feature involves data)*

- **Datastore Resource**: A queryable data asset managed by the datastore with its own lifecycle, identity, and availability status.
- **Metadata Record**: The catalog entry describing a dataset or distribution, which may reference one or more datastore resources.
- **Distribution Reference**: Optional metadata linkage from a dataset distribution to a datastore resource; not required as an operational key.
- **Resource Status**: The current state of a datastore resource, such as available, stale, missing, or replaced.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: At least 95% of datastore query requests for existing resources succeed even when metadata updates are happening separately.
- **SC-002**: Operators can identify the current datastore resource for a changed dataset without consulting implementation details in under 2 minutes.
- **SC-003**: At least 90% of maintainer-reviewed datastore tasks are correctly classified as datastore-managed or metadata-managed in the published documentation.
- **SC-004**: Support requests caused by confusion between datastore and metadata responsibilities are reduced by 50% after rollout.
- **SC-005**: 100% of datastore operations in scope are executable using only canonical datastore resource identifiers.
- **SC-006**: 100% of dataset/distribution-to-resource identifier resolution in scope is performed by metastore rather than datastore.

## Assumptions

- The feature is focused on reducing coupling between datastore lifecycle management and metadata management, not on replacing the metadata layer.
- Datastore resources are independently addressable operational assets, while metadata remains the discovery and catalog layer.
- Distribution IDs remain useful for metadata discovery and traceability, but are not required for datastore operations.
- Clients resolve dataset/distribution identifiers through metastore before invoking datastore operations.
- URI identity preserves all query parameters and fragments for this phase.
- Broader URI normalization policy decisions are deferred to a later phase.
- Existing datasets and datastore resources remain valid unless explicitly refreshed, replaced, or removed.
- Operators need clear guidance and status reporting more than they need a new user-facing workflow.
- The platform continues to support both datastore and metadata capabilities, but each must have clearer boundaries.