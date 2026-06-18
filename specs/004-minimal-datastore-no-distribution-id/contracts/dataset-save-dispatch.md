# Contract: Dataset-Save Datastore Dispatch

## Purpose

Define the internal service contract for discovering dataset distribution `downloadURL` values and initiating datastore processing without requiring distribution UUIDs.

## Caller

Metastore dataset-save lifecycle hooks/events.

## Inputs

```php
DatasetDispatchInput {
  string $datasetIdentifier;
  object|array $datasetMetadata;
  bool $deferred = true;
}
```

Rules:
- `datasetMetadata` may contain referenced or non-referenced distribution structures.
- Distribution UUIDs may be present but are not required.
- Discovery uses `distribution` entries and nested `downloadURL` values.

## Processing Contract

1. Recursively discover dataset `distribution` data.
2. Normalize discovery findings into a `ResourceDiscoveryResult` in encounter order.
3. For each discovered resource candidate with a valid `downloadURL`, register or resolve a `DataResource` through existing ResourceMapper behavior.
4. Trigger datastore processing for each valid discovered resource in encounter order.
5. Do not deduplicate repeated URLs.
6. Skip invalid/missing entries during discovery and continue.
7. Continue after per-entry dispatch failures.
8. Emit structured logs for skipped and failed entries.
9. Return a machine-readable dispatch summary derived from the `ResourceDiscoveryResult` and downstream execution outcomes.

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

## Output

```php
DispatchResult {
  string $datasetIdentifier;
  int $processedCount;
  int $skippedCount;
  int $failedCount;
  array $items;
}

DispatchItem {
  string|null $downloadUrl;
  string|null $resourceIdentifier;
  string $status; // processed|skipped|failed
  string|null $reason;
}
```

## Error Behavior

- Invalid distribution entry: record skipped entry, continue.
- Missing `downloadURL`: record skipped entry, continue.
- ResourceMapper registration failure: record failed entry, continue.
- Datastore dispatch failure: record failed entry, continue.
- Unexpected fatal dataset-level failure: bubble exception after logging dataset context.

## Compatibility

- Distribution references remain supported.
- Legacy distribution IDs may be accepted as input metadata but cannot control initiation.
- Existing ResourceMapper registration/localization/import events remain supported.
