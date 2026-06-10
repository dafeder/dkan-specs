# Data Model: Minimal Datastore Without Distribution ID Requirement

## Dataset

Represents the saved metadata object that triggers datastore discovery.

Fields:
- `identifier`: Dataset UUID or metastore identifier.
- `distribution`: Array or nested structure containing distribution entries.
- `metadata`: Raw or dereferenced dataset metadata used during save lifecycle.

Relationships:
- Has zero or more `Distribution Entry` objects.
- May reference standalone distribution entities, but datastore discovery cannot require those entities.

Validation rules:
- Dataset save traversal must tolerate missing or malformed distribution entries.
- Dataset traversal must continue after invalid entries.

## Distribution Entry

Represents a distribution object inside the dataset structure.

Fields:
- `downloadURL`: Source URL or resource identifier candidate discovered during traversal.
- `identifier`: Optional distribution UUID when the distribution is referenced.
- `format`/`mediaType`: Optional metadata used to infer MIME type.
- `title`: Optional display metadata.

Relationships:
- May be embedded directly in the dataset.
- May correspond to a referenced distribution entity.
- Resolves to one `Datastore Resource` when `downloadURL` is valid and importable.

Validation rules:
- `downloadURL` must be present and valid enough for existing ResourceMapper/DataResource registration.
- Invalid or missing `downloadURL` entries are skipped and reported.
- Repeated `downloadURL` values are processed as encountered; no new deduplication is required.

## Datastore Resource

Represents the resource used by datastore import/query workflows.

Fields:
- `identifier`: Resource identifier used by datastore workflows.
- `version`: Resource version.
- `perspective`: Source or localized perspective.
- `filePath`: Original or localized file path.
- `mimeType`: MIME type used for importability checks.
- `checksum`: Localized file checksum when available.

Relationships:
- Stored in `Resource Mapping`.
- Drives ETL stages through existing queue/immediate execution paths.

Validation rules:
- Must not require a distribution UUID to import/query.
- Must remain compatible with existing ResourceMapper registration and localization/import events.

## Resource Mapping

Persisted metastore mapping record used as the scoped registry.

Fields:
- `identifier`
- `version`
- `perspective`
- `filePath`
- `mimeType`
- `checksum`

Relationships:
- Created or reused when dataset-save discovery resolves valid `downloadURL` values.
- Used by datastore localize/import/query/post-import workflows.

Validation rules:
- Duplicate file paths continue to follow existing AlreadyRegistered handling.
- New versions are created only when existing ResourceMapper rules require them.

## Dataset Dispatch Run

Machine-readable summary for one dataset-save datastore-discovery pass.

Fields:
- `datasetIdentifier`: Dataset being processed.
- `processedCount`: Count of valid entries where dispatch was attempted successfully or queued.
- `skippedCount`: Count of entries skipped before dispatch.
- `failedCount`: Count of valid entries whose dispatch attempt failed.
- `entries`: Optional per-entry result objects with URL/path, status, reason, and resource identifier when available.

Relationships:
- Belongs to one dataset save event.
- Contains zero or more per-entry outcomes.

Validation rules:
- Counts must equal the actual traversal outcomes.
- Failures for one entry must not prevent attempts for later valid entries.

## Pipeline Stage

One of the retained ETL stages.

Fields:
- `name`: `localize`, `import`, or `post-import`.
- `executionMode`: Queue-driven or immediate.
- `outcome`: Stage-specific result from existing datastore workflow.

Relationships:
- Executes for a Datastore Resource.
- May be implemented by default behavior or the single active importer.

Validation rules:
- Stage order remains `localize -> import -> post-import`.
- Stage overrides must fall back to defaults for non-overridden stages.

## Active Importer

The single configured importer implementation used at runtime.

Fields:
- `serviceId` or class reference.
- Stage override support.
- Default fallback behavior.

Relationships:
- Used by datastore import factory/service paths.

Validation rules:
- Only one active importer is selected in this feature.
- Priority-based multi-plugin selection remains out of scope.
