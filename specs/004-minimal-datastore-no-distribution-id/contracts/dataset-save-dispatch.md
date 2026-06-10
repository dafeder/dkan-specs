# Contract: Dataset-Save Datastore Dispatch

## Purpose

Define the internal service contract for discovering dataset distribution `downloadURL` values and initiating datastore processing without requiring distribution UUIDs.

## Caller

Metastore dataset save lifecycle side effect.

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
- Traversal uses `distribution` entries and nested `downloadURL` values.

## Processing Contract

1. Recursively traverse dataset `distribution` data.
2. For each entry with a valid `downloadURL`, register or resolve a `DataResource` through existing ResourceMapper behavior.
3. Trigger datastore processing for each valid discovered resource in encounter order.
4. Do not deduplicate repeated URLs.
5. Skip invalid/missing entries and continue.
6. Continue after per-entry dispatch failures.
7. Emit structured logs for skipped and failed entries.
8. Return a machine-readable status summary.

## Output

```php
DatasetDispatchResult {
  string $datasetIdentifier;
  int $processedCount;
  int $skippedCount;
  int $failedCount;
  array $entries;
}

DatasetDispatchEntryResult {
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
