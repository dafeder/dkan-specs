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
- Dataset save discovery must tolerate missing or malformed distribution entries.
- Dataset discovery must continue after invalid entries.

## Distribution Entry

Represents a distribution object inside the dataset structure.

Fields:
- `downloadURL`: Source URL or resource identifier candidate discovered during discovery.
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

## ResourceDiscoveryResult

Represents the normalized metastore-side discovery output emitted before datastore triggering.

Fields:
- `datasetIdentifier`: Dataset being inspected.
- `discoveredResources`: Ordered list of valid discovered resource candidates, each carrying source `downloadURL` and any metadata needed for downstream registration.
- `skippedEntries`: Ordered list of skipped distribution entries.
- `invalidEntries`: Ordered list of malformed entries that could not be normalized.

Relationships:
- Belongs to one dataset save discovery pass.
- Contains zero or more candidate items derived from `Distribution Entry` objects.
- Acts as the single input contract for downstream registration, triggering, and logging flows.

Validation rules:
- Encounter order must match recursive discovery order in dataset metadata.
- Valid discovered resource candidates must preserve enough source context to support ResourceMapper registration and later datastore triggering.
- Skipped and invalid entries must retain a machine-readable reason for downstream logging and summary generation.
- This object must not imply that ResourceMapper registration, queueing, or datastore execution has already happened.
- Downstream components must consume this object rather than re-discovering dataset metadata.

## DispatchResult

Machine-readable downstream execution summary derived from one `ResourceDiscoveryResult` after registration/trigger attempts have been made.

Fields:
- `datasetIdentifier`: Dataset being processed.
- `processedCount`: Count of discovered resource candidates whose registration and dispatch path completed successfully or was queued successfully.
- `skippedCount`: Count of discovery items that were not attempted downstream because they were already marked skipped or invalid in the paired `ResourceDiscoveryResult`.
- `failedCount`: Count of discovered resource candidates whose registration or dispatch attempt failed after discovery succeeded.
- `items`: Per-item execution outcomes keyed back to the corresponding discovery candidate, with URL/path, status, reason, and resolved resource identifier when available.

Relationships:
- Belongs to one dataset save event.
- Is produced from one `ResourceDiscoveryResult`.
- Contains zero or more per-item execution outcomes.

Validation rules:
- Counts must reconcile against the paired `ResourceDiscoveryResult` and actual downstream attempt outcomes.
- Successfully discovered candidates may still appear as failed here if registration or triggering fails.
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
