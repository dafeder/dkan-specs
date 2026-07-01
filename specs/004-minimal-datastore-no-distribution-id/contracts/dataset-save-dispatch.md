# Contract: Dataset-Save Datastore Initiation

## Purpose

Define the internal contract for discovering dataset distribution `downloadURL` values, mapping each valid discovered resource to a compound datastore identifier, and initiating existing datastore processing without requiring distribution UUIDs.

## Caller

Metastore dataset-save lifecycle hooks/events.

## Owner

Metastore Reference-layer service(s) own discovery and reference-mapping behavior. Dataset lifecycle orchestration invokes that service during presave.

## Inputs

Input parameters:
- `datasetIdentifier` (string)
- `datasetMetadata` (object|array)

Rules:
- `datasetMetadata` may contain referenced or non-referenced distribution structures.
- Distribution UUIDs may be present but are not required.
- Discovery is limited to top-level `$.distribution[]` entries and their `downloadURL` values.
- Discovery/reference mapping must be callable even when distribution reference property handling is disabled by metadata reference settings.
- Initiation is queue/deferred in this feature scope; no caller-supplied mode switch is required.

## Processing Contract

1. Discover dataset resources only from top-level `$.distribution[]` entries.
2. Normalize discovery findings into a `ResourceDiscoveryResult` in encounter order.
3. For each discovered candidate with a valid `downloadURL`, produce a discovered resource entry that links metadata context to a compound datastore identifier.
4. For each valid discovered resource entry, register or resolve a `DataResource` through existing ResourceMapper behavior.
5. Trigger datastore processing for each valid discovered resource in encounter order.
6. Do not deduplicate repeated URLs.
7. Skip invalid/missing entries during discovery and continue.
8. Continue after per-entry reference-mapping, registration, or initiation failures (best effort).
9. Emit structured logs for skipped and failed entries.
10. Use existing datastore initiation/status surfaces for downstream observability; no new initiation-summary contract object is required in this phase.
11. Preserve `distribution[].describedBy` data-dictionary URI validation/normalization behavior for referenced and non-referenced distribution configurations; this behavior must not depend on distribution referencing mode.

## Intermediate Output

```php
ResourceDiscoveryResult {
  string $datasetIdentifier;
  array $discoveredResources;
  array $skippedEntries;
  array $failedEntries;
}
```

Rules:
- `discoveredResources[]` entries carry discovered URL metadata plus a compound datastore identifier payload.
- This object is the normalized discovery/mapping boundary and does not imply registration or dispatch success.
- Downstream registration, trigger, and logging steps must consume this object rather than re-discovering or re-mapping dataset metadata.

Notes:
- A standalone dispatcher service is optional and not required by this contract.
- Implementations may perform initiation orchestration directly in subscriber/service flow while preserving this behavior.
- Ownership belongs to metastore Reference-layer logic; lifecycle code is an orchestration/invocation point.

## Error Behavior

- Invalid distribution entry: record skipped entry, continue.
- Missing `downloadURL`: record skipped entry, continue.
- Reference-mapping failure for valid-looking metadata: record failed entry, continue.
- ResourceMapper registration failure: record failed entry, continue.
- Datastore initiation failure: record failed entry, continue.
- Unexpected fatal dataset-level failure: bubble exception after logging dataset context.

## Compatibility

- Distribution references remain supported.
- Legacy distribution IDs may be accepted as input metadata but cannot control initiation.
- Existing ResourceMapper registration/localization/import events remain supported.
- Status/reporting/cache surfaces that previously depended on distribution UUID lookup must consume discovered resource outputs or discovered-resource-derived identifiers.
