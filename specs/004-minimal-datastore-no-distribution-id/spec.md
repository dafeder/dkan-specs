# Feature Specification: Minimal Datastore Without Distribution ID Requirement

**Feature Branch**: `[004-minimal-datastore-no-distribution-id]`  
**Created**: 2026-06-09  
**Status**: Draft  
**Input**: User description: "Let's create a scoped version that removes requirement for distribution IDs while keeping datastore behavior and interfaces as stable as practical"

## Clarifications

### Session 2026-06-09

- Q: On dataset save, should metastore select one or all valid distribution downloadURL values? -> A: Recurse through the dataset distribution array and process all valid downloadURL values.
- Q: Should metastore deduplicate discovered downloadURL values before triggering datastore imports/ETL processes? -> A: No; preserve current behavior and trigger processing for each discovered downloadURL as encountered.
- Q: How should workflow handle invalid or missing distribution downloadURL values during dataset-save discovery? -> A: Skip invalid/missing entries, continue processing valid entries, and log/report skipped entries.

### Session 2026-06-10

- Q: Should User Story 1 be reframed to prioritize dataset-save resource discovery and import triggering rather than generic import/query focus? -> A: Yes, reframe User Story 1 around dataset-save discovery and import triggering without distribution-ID lookup, with import/query expectations retained as supporting behavior.
- Q: How should legacy distribution ID inputs be handled during workflow initiation? -> A: Accept legacy distribution IDs for compatibility, but ignore them for workflow initiation and use dataset distribution downloadURL discovery as the only operational trigger.
- Q: During multi-downloadURL import triggering, should processing continue when one URL fails? -> A: Yes, use best-effort import triggering: continue processing remaining URLs and log/report each per-URL failure.
- Q: How should custom importer hooks that expect legacy distribution-ID fields be handled? -> A: Remove legacy distribution-ID hook fields immediately; custom importers must update to the new dataset/downloadURL-driven hook payloads.
- Q: What observability surface is required for skip/failure outcomes during import triggering? -> A: Require both structured logs and a machine-readable workflow status summary with processed/skipped/failed counts.

### Session 2026-06-15

- Q: How explicit should dashboard continuity be when distribution IDs are absent? -> A: Dashboard continuity must be explicit and testable: rows and status must render for both distribution-backed and URL-only resources without requiring distribution IDs.

### Session 2026-06-16

- Q: Should feature terminology use "triggering" rather than "dispatch" for datastore workflow initiation? -> A: Yes; use "triggering datastore imports/ETL processes" for workflow initiation language, and reserve "dispatch" for event dispatching context.
- Q: For the dataset-save initiated dispatch orchestration service, should the implementation name be `Dispatcher` rather than `DatasetDispatchCoordinator`? -> A: Yes; prefer `Dispatcher` for the service name within the datastore dispatch namespace.
- Q: Should new resource discovery and datastore dispatch classes live in dedicated namespaces rather than the generic `src/Service` namespace? -> A: Yes; place metastore-side discovery classes under `dkan_metastore/src/LifeCycle/ResourceDiscovery/` and datastore-side dispatch classes under `dkan_datastore/src/Dispatch/`.

### Session 2026-06-17

- Q: When quickstart ticket slices and tasks phase groupings diverge, should the task phase structure be changed to match the existing quickstart ticket names? -> A: Yes; keep the current quickstart ticket names, but regroup `tasks.md` so its phase structure matches those ticket slices.

### Session 2026-06-23

- Q: Are datastore dispatch namespace/service refactors in scope for this phase? -> A: No. Keep existing datastore initiation flow for this phase and defer standalone datastore dispatch-subsystem refactors.
- Q: How should legacy distribution-ID-based hook payload field removal be managed? -> A: Through a documented deprecation transition with an explicit migration period and versioned removal gate.
- Note: For this phase, this session supersedes the Session 2026-06-16 datastore dispatch-namespace/class guidance.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Dataset-Save Discovery and Import Triggering Without Distribution IDs (Priority: P1)

As a DKAN administrator, I want dataset-save hooks to discover distribution downloadURL values and trigger datastore processing without requiring distribution-ID lookup so ingestion starts reliably from dataset context.

**Why this priority**: Dataset-save discovery and import triggering is the most affected behavior and the primary risk area for this scoped change.

**Independent Test**: Save datasets with referenced and non-referenced distribution entries, then verify metastore-side discovery identifies all valid downloadURL values and triggers datastore processing without requiring distribution-ID lookup.

**Acceptance Scenarios**:

1. **Given** a dataset save event containing multiple referenced or non-referenced top-level `distribution` entries with valid `downloadURL` fields, **When** metastore workflow hooks run, **Then** datastore processing is triggered for each valid `downloadURL` identified from `$.distribution[]` discovery.
2. **Given** a dataset save event where some distribution entries are invalid or missing `downloadURL`, **When** discovery runs, **Then** invalid entries are skipped, valid entries still trigger datastore imports/ETL processes, and skipped entries are logged or reported.
3. **Given** dataset-save initiated datastore import triggering, **When** processing is triggered, **Then** runtime workflow initiation proceeds without distribution-ID entity lookup.

---

### User Story 2 - Preserve Existing Behavior Where Practical (Priority: P2)

As a maintainer, I want existing datastore class and method surfaces preserved where practical so migration risk stays low while introducing necessary breaking changes only where required.

**Why this priority**: The feature intentionally minimizes architectural churn and should reduce downstream refactor burden.

**Independent Test**: Compare core datastore internal/PHP entry points before and after the change and verify only necessary breaking changes were introduced.

**Acceptance Scenarios**:

1. **Given** an existing datastore extension that does not rely on distribution IDs, **When** import/query execution runs after dataset-save initiated import triggering in the updated workflow, **Then** it continues to operate with minimal or no code changes where practical.
2. **Given** an interface that must change to remove distribution-ID dependency, **When** the change is introduced, **Then** migration guidance clearly documents the required update, including immediate custom importer hook payload changes.
3. **Given** dataset resources that do not have distribution IDs, **When** an administrator opens datastore dashboard/reporting views, **Then** resource rows and status fields render without errors using dataset/resource context.

---

### User Story 3 - Keep Importer Modularity Simple (Priority: P3)

As a module developer, I want import customization to remain straightforward so specialized importers (for example native MySQL loader) can override import behavior without replacing the full workflow.

**Why this priority**: Existing extension patterns should remain viable and easier to evolve.

**Independent Test**: Configure one active importer implementation and verify stage-specific overrides work with default fallback behavior.

**Acceptance Scenarios**:

1. **Given** a custom importer for one pipeline stage, **When** an import runs, **Then** overridden stage logic is used and non-overridden stages fall back to default behavior.
2. **Given** no custom importer stage override, **When** an import runs, **Then** default datastore import behavior is used.

### Edge Cases

- Legacy calls that still supply distribution IDs are accepted for compatibility, but IDs are ignored for workflow initiation.
- When both legacy IDs and dataset distribution `downloadURL` paths are present, `downloadURL` discovery is authoritative for import triggering.
- Distribution references remain supported; referenced and non-referenced dataset structures use the same discovery logic, except non-referenced distributions do not have their own distribution UUIDs.
- Workflows that still attempt runtime distribution dereferencing must report the failure as a migration error and must not block dataset `downloadURL` discovery for other entries.
- Custom importers that expect old distribution-ID-based hooks fail until updated; migration guidance must include required payload changes.
- Distribution entries with invalid or missing `downloadURL` values are skipped and reported while valid entries continue through import triggering.
- During multi-`downloadURL` import triggering, failure for one URL does not block processing of remaining URLs; each failure is logged/reported.
- Dashboard/reporting views must not require distribution IDs to render resource rows or status values.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The datastore workflow MUST remove distribution IDs as a required operational key for import and query execution.
- **FR-002**: Datastore runtime logic MUST avoid dependency on distribution entity dereference/lookup during normal import/query operations.
- **FR-003**: The system MUST preserve existing datastore class names, method signatures, and invocation patterns where practical.
- **FR-004**: Breaking internal/PHP API changes MAY be introduced only when required to remove distribution-ID dependency.
- **FR-005**: Any required breaking change MUST include explicit migration guidance, including required updates for custom importer hook payload consumers.
- **FR-006**: The ETL stage model (`localize`, `import`, `post-import`) MUST remain intact for this scoped feature.
- **FR-007**: Queue-driven default execution and immediate execution overrides MUST remain supported.
- **FR-008**: Import customization MUST support stage-level override behavior with default fallback for non-overridden stages.
- **FR-009**: This feature MUST keep importer selection to one globally configured active importer for runtime execution in this phase.
- **FR-010**: Priority-based multi-plugin importer selection MUST remain out of scope for this feature version.
- **FR-011**: Metastore workflow logic for datastore initiation MUST run through dataset-save hooks or events rather than requiring distribution-save hooks.
- **FR-012**: On dataset save, the workflow MUST inspect the top-level dataset `distribution` array (`$.distribution[]`) to identify valid `downloadURL` values for both referenced and non-referenced distribution structures.
- **FR-013**: The workflow MUST trigger datastore processing for all valid `downloadURL` values discovered during discovery.
- **FR-014**: The workflow MUST preserve existing import-triggering behavior by processing discovered `downloadURL` values as encountered, without introducing new deduplication requirements in this scoped feature.
- **FR-015**: During discovery, entries with invalid or missing `downloadURL` values MUST be skipped without blocking import triggering for valid entries.
- **FR-016**: The workflow MUST provide administrator-visible reporting for skipped invalid/missing `downloadURL` entries, including skip reasons.
- **FR-017**: Distribution referencing and legacy distribution ID inputs MAY be accepted for backward compatibility, but MUST NOT be required or control workflow initiation when dataset distribution `downloadURL` values are available.
- **FR-018**: During multi-`downloadURL` processing, failure of an individual URL MUST NOT prevent import-triggering attempts for remaining valid discovered URLs.
- **FR-019**: The workflow MUST provide administrator-visible reporting for each per-URL processing failure during best-effort import triggering.
- **FR-020**: Legacy distribution-ID-based hook payload fields MUST be removed in this feature scope through a documented deprecation transition, including an explicit migration period and versioned removal gate.
- **FR-021**: The workflow MUST emit structured machine-readable logs for skipped invalid/missing `downloadURL` entries and per-URL processing failures.
- **FR-022**: The workflow MUST produce a machine-readable `DispatchResult` for each dataset-save workflow execution, with counts for processed, skipped, and failed URLs.
- **FR-023**: Existing `ResourceMapper`/resource mapping storage MUST remain the canonical registry for datastore resource identifiers, versions, perspectives, file paths, MIME types, and checksums.
- **FR-024**: Dataset-save discovery MUST register or resolve valid distribution `downloadURL` values as `DataResource`/resource mapping records before triggering datastore imports/ETL processes.
- **FR-025**: Existing resource registration and localization/import event flow MUST remain supported, including resource registration triggering deferred datastore import where the resource is importable.
- **FR-026**: Cache dependency and cache invalidation behavior for datastore summary/query/import status responses MUST be updated so normal operation does not require resource-to-distribution-to-dataset lookup, while preserving compatibility when distribution references are present.
- **FR-027**: Public API, Drush, SQL endpoint, and admin/reporting paths that currently accept or infer distribution IDs MUST be reviewed and either moved to resource/dataset identifiers or explicitly documented as changed behavior.
- **FR-028**: Resource cleanup and purge behavior MUST remove obsolete resource mappings and datastore artifacts from dataset/downloadURL context without depending on orphaned distribution reference entities as the only cleanup trigger.
- **FR-029**: Post-import status/result creation MUST use datastore resource mapping or dataset/downloadURL context rather than requiring a distribution payload.
- **FR-030**: Dashboard and administrator reporting flows MUST render resource rows and status for both distribution-backed and URL-only discovered resources without requiring distribution IDs.

### Key Entities *(include if feature involves data)*

- **Datastore Resource**: The datastore-managed resource used for import/query operations without distribution-ID runtime dependency.
- **Dataset**: The saved metadata object whose `distribution` array is inspected to identify `downloadURL` values for datastore workflow initiation.
- **Distribution Entry**: A dataset distribution object that may be stored as a referenced distribution entity with its own UUID or embedded directly without a distribution UUID; datastore discovery uses the same `downloadURL` discovery in either case.
- **Resource Mapping**: The persisted metastore mapping record that links a datastore resource identifier, version, perspective, file path, MIME type, and checksum.
- **DispatchResult**: The machine-readable result of one dataset-save workflow execution, including discovered distribution `downloadURL` values and processed, skipped, and failed counts.
- **Pipeline Stage**: One of `localize`, `import`, or `post-import`, each with independent execution outcome.
- **Active Importer**: The single configured importer implementation used for runtime stage execution in this phase.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% of in-scope datastore import and query workflows run without distribution-ID requirement.
- **SC-002**: At least 80% of in-scope datastore internal/PHP surfaces remain source-compatible where practical.
- **SC-003**: 100% of required breaking changes include migration guidance before rollout.
- **SC-004**: 100% of in-scope imports execute successfully using the retained ETL stage workflow with default or stage-overridden importer behavior.
- **SC-005**: For dataset save events with valid distribution `downloadURL` values, 100% of valid discovered URLs trigger datastore processing without distribution-ID lookup.
- **SC-006**: For dataset save events containing repeated valid `downloadURL` values, datastore import-triggering behavior remains consistent with existing non-deduplicated processing semantics.
- **SC-007**: For mixed-validity distribution arrays, 100% of valid `downloadURL` entries trigger datastore imports/ETL processes while invalid/missing entries are skipped and reported.
- **SC-008**: For dataset save events with multiple valid discovered `downloadURL` values where at least one URL fails processing, 100% of remaining valid URLs are still attempted, and 100% of per-URL failures are reported.
- **SC-009**: For 100% of dataset-save workflow executions, a `DispatchResult` is produced with accurate processed/skipped/failed counts matching dispatch outcomes.
- **SC-010**: Existing importable resources registered in `ResourceMapper` continue to trigger deferred datastore import through the retained resource registration/localization/import event flow.
- **SC-011**: Datastore summary/query/import-status cache dependencies and invalidations can be exercised without runtime distribution-ID lookup.
- **SC-012**: Resource cleanup tests verify obsolete mappings and datastore artifacts are removed from dataset/downloadURL context when distribution reference entities are absent.
- **SC-013**: For dataset resources lacking distribution IDs, 100% of in-scope dashboard/reporting views render resource rows and status values without runtime errors.

## Assumptions

- This feature is an intentionally scoped alternative to broader datastore rearchitecture work.
- Reducing distribution-ID dependence is the primary objective; large-scale resource model redesign is out of scope.
- Referenced and non-referenced distributions produce the same effective dataset discovery structure for datastore discovery, except non-referenced distributions do not expose separate distribution UUIDs.
- Existing queue and immediate execution modes remain operationally valuable and should be retained.
- Full multi-importer runtime selection is deferred to future scope.
- Legacy distribution-ID-based hook payload field removal follows a documented deprecation transition with a migration period and explicit versioned removal gate.
- For planning artifacts in this feature, the quickstart ticket slices are the canonical work-package grouping; `tasks.md` should be reorganized to mirror those ticket boundaries rather than using a conflicting phase-first structure.
