# Quickstart: Planning and Ticket Breakdown for 004

## Goal

Implement the minimal datastore change so dataset-save discovery triggers datastore processing from distribution `downloadURL` values without requiring distribution UUIDs, while preserving current ResourceMapper, ETL, queue, and importer behavior where practical.

## Suggested Manual Ticket Slices

Each ticket is intended to be small enough for one developer to complete in a week or less with manual coding and review.

### Ticket 1: Dataset Distribution Traversal Service

Scope:
- Add or adapt a metastore-side service that recursively traverses dataset `distribution` entries.
- Support referenced and non-referenced structures with the same traversal logic.
- Collect valid, skipped, and invalid entries in encounter order.

Acceptance:
- Unit tests cover referenced, embedded, nested, repeated, invalid, and missing `downloadURL` entries.

### Ticket 2: ResourceMapper Registration and Dispatch Integration

Scope:
- Register or resolve valid `downloadURL` values as `DataResource`/ResourceMapper records.
- Trigger existing datastore processing from resolved resources.
- Preserve queue-driven default and immediate override behavior.

Acceptance:
- Kernel/integration tests verify valid discovered URLs trigger datastore processing without distribution lookup.

### Ticket 3: Dispatch Result Summary and Structured Logging

Scope:
- Create the machine-readable dispatch summary.
- Add structured logs for skipped entries and per-URL failures.
- Ensure best-effort processing continues after individual failures.

Acceptance:
- Tests verify processed/skipped/failed counts and per-entry reasons.

### Ticket 4: Cache Dependency and Invalidation Updates

Scope:
- Review datastore summary, query, import status, and import completion invalidation paths.
- Remove mandatory resource-to-distribution-to-dataset lookup from normal operation.
- Preserve compatibility/enrichment when distribution references exist.

Acceptance:
- Tests exercise cache behavior with and without distribution references.

### Ticket 5: API, Drush, SQL Endpoint, and Admin Compatibility Review

Scope:
- Audit paths that accept or infer distribution IDs.
- Move operational flows to resource/dataset identifiers where needed.
- Document compatibility-only distribution ID behavior.

Acceptance:
- Updated tests or docs show distribution IDs are not required operational keys.

### Ticket 6: Post-Import and Dashboard Reporting Updates

Scope:
- Update post-import result creation to use resource mapping or dataset/downloadURL context.
- Update dashboard/admin reporting so distribution payloads are optional.

Acceptance:
- Dashboard/reporting tests pass for referenced and non-referenced distributions.

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

## Planning Notes

- Keep `ResourceMapper` as the scoped resource registry.
- Do not introduce datastore-owned canonical resource IDs in this feature; that belongs to the broader decoupled datastore work.
- Do not add priority-based multi-importer selection.
- Do not deduplicate repeated discovered URLs.
- Treat distribution references as supported metadata, not required operational keys.
