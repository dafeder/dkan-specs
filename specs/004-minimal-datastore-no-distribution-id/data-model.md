# Data Model: Minimal Datastore Without Distribution ID Requirement

## Dataset

Represents the saved metadata object that triggers datastore discovery.

Fields:
- `identifier`: Dataset UUID or metastore identifier.
- `distribution`: Top-level array containing distribution entries.
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
- `describedBy`: Optional data-dictionary URI/URL for schema reference.
- `identifier`: Optional distribution UUID when the distribution is referenced.
- `format`/`mediaType`: Optional metadata used to infer MIME type.
- `title`: Optional display metadata.

Relationships:
- May be embedded directly in the dataset.
- May correspond to a referenced distribution entity.
- Resolves to one `Datastore Resource` when `downloadURL` is valid and importable.

Validation rules:
- `downloadURL` must be present and valid enough for existing ResourceMapper/DataResource registration.
- `describedBy` validation/normalization behavior must remain equivalent for referenced and non-referenced distributions.
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

## Execution Contract Note

This feature depends on one execution-contract object during dataset-save processing:

- `ResourceDiscoveryResult`: the normalized discovery output consumed by downstream registration, triggering, and logging.

Its field-level definition lives in [contracts/dataset-save-dispatch.md](contracts/dataset-save-dispatch.md) because it describes a service boundary rather than a durable domain entity.

This phase does not introduce a new initiation-summary value-object contract; existing datastore initiation/import status surfaces remain in use.

Runtime behavior constraints such as retained ETL stage order and single active importer selection are documented in the spec and planning artifacts, not modeled here as feature data entities.
