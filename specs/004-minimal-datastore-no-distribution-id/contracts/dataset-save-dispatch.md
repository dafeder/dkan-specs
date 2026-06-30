# Contract: Dataset-Save Datastore Initiation

## Purpose

Define the internal contract for discovering dataset distribution `downloadURL` values and initiating existing datastore processing without requiring distribution UUIDs.

## Caller

Metastore dataset-save lifecycle hooks/events.

## Inputs

Input parameters:
- `datasetIdentifier` (string)
- `datasetMetadata` (object|array)

Rules:
- `datasetMetadata` may contain referenced or non-referenced distribution structures.
- Distribution UUIDs may be present but are not required.
- Discovery is limited to top-level `$.distribution[]` entries and their `downloadURL` values.
- Initiation is queue/deferred in this feature scope; no caller-supplied mode switch is required.

## Processing Contract

1. Discover dataset resources only from top-level `$.distribution[]` entries.
2. Normalize discovery findings into a `ResourceDiscoveryResult` in encounter order.
3. For each discovered resource candidate with a valid `downloadURL`, register or resolve a `DataResource` through existing ResourceMapper behavior.
4. Trigger datastore processing for each valid discovered resource in encounter order.
5. Do not deduplicate repeated URLs.
6. Skip invalid/missing entries during discovery and continue.
7. Continue after per-entry initiation failures.
8. Emit structured logs for skipped and failed entries.
9. Use existing datastore initiation/status surfaces for downstream observability; no new initiation-summary contract object is required in this phase.
10. Preserve `distribution[].describedBy` data-dictionary URI validation/normalization behavior for referenced and non-referenced distribution configurations; this behavior must not depend on distribution referencing mode.

## Intermediate Output

```php
ResourceDiscoveryResult {
  string $datasetIdentifier;
  array $discoveredResources;
  array $skippedEntries;
  array $invalidEntries;
}
```

Rules:
- This object is the normalized discovery boundary and does not imply registration or dispatch success.
- Downstream registration, trigger, and logging steps must consume this object rather than re-discovering dataset metadata.

Notes:
- A standalone dispatcher service is optional and not required by this contract.
- Implementations may perform initiation orchestration directly in subscriber/service flow while preserving this behavior.

## Error Behavior

- Invalid distribution entry: record skipped entry, continue.
- Missing `downloadURL`: record skipped entry, continue.
- ResourceMapper registration failure: record failed entry, continue.
- Datastore initiation failure: record failed entry, continue.
- Unexpected fatal dataset-level failure: bubble exception after logging dataset context.

## Compatibility

- Distribution references remain supported.
- Legacy distribution IDs may be accepted as input metadata but cannot control initiation.
- Existing ResourceMapper registration/localization/import events remain supported.
