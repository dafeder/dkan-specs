# Quickstart: Planning and Ticket Breakdown for 004

## Goal

Implement the minimal datastore change so dataset-save discovery triggers datastore processing from distribution `downloadURL` values without requiring distribution UUIDs, while preserving current ResourceMapper, ETL, queue, and importer behavior where practical.

## Suggested Manual Ticket Slices

Each ticket is intended to be small enough for one developer to complete in a week or less with manual coding and review.

### Ticket 1: Pre-Reference Dataset Discovery in LifeCycle Flow

Scope:
- Implement discovery where dataset saves already trigger datastore-related decisions: `LifeCycle::referenceMetadata()` -> `LifeCycle::EVENT_PRE_REFERENCE` -> `DatastoreSubscriber::onPreReference()`.
- Add a dataset resource discovery helper used by the pre-reference handler to inspect top-level dataset `distribution` entries (`$.distribution[]`) before reference conversion and emit normalized resource candidates.
- Support referenced and non-referenced distribution structures with the same discovery logic.
- Place metastore-specific discovery classes under `dkan_metastore/src/LifeCycle/ResourceDiscovery/` and keep initiation orchestration in the existing datastore subscriber/service flow rather than introducing a parallel dispatch subsystem.
- Emit a normalized `ResourceDiscoveryResult` (valid discovered resource values, skipped entries, invalid entries, and reasons) in encounter order for downstream trigger/logging tickets.
- Use the emitted `ResourceDiscoveryResult` as the single input contract for: (a) deciding whether datastore-trigger criteria are met, (b) executing resource registration and import-trigger work, and (c) producing structured summary/log outputs in later tickets.

Acceptance:
- Unit tests cover referenced/non-referenced top-level distribution entries, repeated URLs, and invalid/missing `downloadURL` entries.
- Subscriber/lifecycle tests verify discovery runs from the pre-reference event path during dataset presave.
- Contract tests verify downstream components consume the `ResourceDiscoveryResult` object (rather than re-discovering metadata) for trigger decisions and registration/import planning.

### Ticket 2: ResourceMapper Registration and Import Trigger Integration

Scope:
- Register or resolve valid discovered resource values as `DataResource`/ResourceMapper records.
- Trigger existing datastore processing from resolved resources.
- Preserve queue-driven default and immediate override behavior.
- Handle import-trigger failures gracefully (best-effort: one URL failure does not block others).
- Log failures with standard logger (no new structured logging framework in this ticket).

Acceptance:
- Kernel/integration tests verify valid discovered resource values trigger datastore processing without distribution lookup.
- Tests verify import-trigger failures are logged and do not block subsequent URL processing.

### Ticket 3: Cache Dependency and Invalidation Updates

Scope:
- Map current cache dependency inputs in code and define their replacements for dataset/URL-driven discovery:
	- `AbstractQueryController::extractMetastoreDependencies()`, `queryResource()`, and `queryDatasetResource()` (currently distribution-heavy dependency arrays passed to `cachedJsonResponse()`).
	- `ImportController::summary()` / `getDependencies()` and SQL endpoint query responses (currently inferring or tagging `distribution` dependencies).
	- `QueryDownloadController` streaming responses (currently max-age header caching via `addCacheHeaders()`, without metastore dependency tags).
- Define the target dependency strategy per endpoint family (query JSON, SQL endpoint JSON, import summary/status, streaming downloads) so invalidation is correct when only dataset + discovered URLs are available.
- Update invalidation triggers so cache entries are invalidated when discovered URL sets, resource mappings, or import outcomes change, without requiring resource-to-distribution-to-dataset resolution.
- Keep distribution-reference-based dependencies as optional backward-compatible augmentation only when distribution references exist; they can add extra invalidation links, but they must not be required for invalidation correctness.
- Document which cache tags/keys are invalidated by dataset changes vs. resource/import state changes.

Acceptance:
- Tests verify invalidation behavior for both referenced and non-referenced distribution workflows, including dataset-only discovery paths.
- Tests prove cache freshness remains correct when distribution references are absent.

### Ticket 4: DatasetInfo Integration for Discovered Resources

Scope:
- Decide whether `DatasetInfo::gather()` merges discovered resources into the `distributions[]` array, or provides a separate array for discovered resources.
- If merged: minimal reporting changes; discovered resources appear alongside distribution-backed resources.
- If separate: dashboard/reporting iterates both arrays, combining rows for display.
- Complete this ticket before DashboardForm compatibility work so dashboard row rendering has a settled DatasetInfo shape to consume.

Acceptance:
- Dashboard correctly displays resources from whichever data model is chosen.
- Integration tests cover both dataset structures.

### Ticket 5: API, Drush, SQL Endpoint, and Admin Compatibility Review

Scope:
- Audit paths that accept or infer distribution IDs.
- Move operational flows to resource/dataset identifiers where needed.
- Explicitly cover dashboard/reporting continuity for URL-only resources where distribution IDs are absent.
- Treat DatasetInfo output from Ticket 4 as an input dependency for dashboard/reporting work.
- Modify `PostImportResultFactory::initializeFromDistribution()` to accept either distribution-based lookups (existing) or dataset+resource_url/resource-identifier (new) because the affected status flows are primarily consumed through admin and Drush interfaces.
- Enable post-import status retrieval for discovered resources without distribution references as part of the same compatibility pass.
- Create the ResourceMapper-based status lookup helper needed by dashboard, reporting, and admin/Drush surfaces as part of the same compatibility pass.
- Document which status fields are available from URL-only resource lookups vs. distribution-backed resources.
- Document compatibility-only distribution ID behavior.

Acceptance:
- Updated tests or docs show distribution IDs are not required operational keys.
- Dashboard/reporting views render rows and status for distribution-backed and URL-only resources without runtime errors.

### Ticket 6: Importer Modularity Regression Assurance

Scope:
- Validate stage-level override and fallback behavior remains unchanged after dataset/downloadURL-driven dispatch refactors.
- Confirm single active importer selection remains intact for runtime execution.
- Verify no importer-path logic depends on distribution IDs as operational keys.

Acceptance:
- Kernel/unit tests demonstrate stage override/fallback behavior is preserved.
- No regression in importer selection or stage invocation order.

### Ticket 7: Cleanup and Orphan Behavior Updates

Scope:
- Ensure obsolete resource mappings and datastore artifacts can be removed without relying only on orphaned distribution reference entities.
- Preserve referenced distribution orphan cleanup when references exist.

Acceptance:
- Tests verify cleanup for referenced and non-referenced distribution workflows.

### Ticket 8: Migration Guidance and Custom Importer Hook Updates

Scope:
- Document breaking hook payload changes.
- Document legacy distribution ID compatibility behavior.
- Provide examples for custom importers moving to dataset/downloadURL/resource context.

Acceptance:
- Migration docs cover all required breaking changes from the spec.

## Validation Commands

From the DKAN repo root:

```bash
vendor/bin/phpunit modules/dkan_metastore/tests modules/dkan_datastore/tests
vendor/bin/phpcs --standard=phpcs.xml modules/dkan_metastore modules/dkan_datastore
```

Use narrower PHPUnit paths while developing individual tickets, then run the broader metastore/datastore suites before merging the full feature.

## Future Improvement: Structured Logging and Import Outcome Reporting

After 004 ships, add a follow-up task to build:
- Machine-readable import-trigger outcome summaries.
- Structured logging for skipped entries and per-URL processing failures.
- Operator-visible reports for import outcomes per dataset.

This deferred work keeps 004 focused on the minimal refactor and lets logging infrastructure be added cleanly in a future feature.

## Planning Notes

- Keep `ResourceMapper` as the resource registry.
- Do not introduce datastore-owned canonical resource IDs in this feature; that belongs to the broader decoupled datastore work.
- Do not add priority-based multi-importer selection.
- Do not deduplicate repeated discovered URLs. (?)
- Treat distribution references as supported metadata, not required operational keys.
- Avoid a standalone dispatch subsystem when existing datastore initiation flow already provides the required behavior.
- Prefer the smallest class set that preserves clear discovery/dispatch contracts; collapse abstractions that do not protect a real boundary, reuse point, or test seam.
