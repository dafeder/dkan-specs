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

## Resource Discovery Candidate

Represents one discovered top-level distribution entry before registration/triggering.

Fields:
- `datasetIdentifier`: Dataset identifier for correlation.
- `downloadURL`: Candidate source URL.
- `mimeType`: Optional media type for registration hints.
- `distributionIdentifier`: Optional legacy distribution UUID metadata.
- `encounterIndex`: Stable order index from the dataset `distribution[]` array.

Relationships:
- Produced by Reference-owned discovery from `$.distribution[]`.
- Promoted to `Discovered Resource` only when `downloadURL` is valid.

Validation rules:
- Candidate is skipped with reason when `downloadURL` is missing or invalid.
- Encounter order must be preserved end-to-end.

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
- Resolves to one `Discovered Resource` when `downloadURL` is valid.

Validation rules:
- `downloadURL` must be present and valid enough for existing ResourceMapper/DataResource registration.
- `describedBy` validation/normalization behavior must remain equivalent for referenced and non-referenced distributions.
- Invalid or missing `downloadURL` entries are skipped and reported.
- Repeated `downloadURL` values are processed as encountered; no new deduplication is required.

## Compound Datastore Identifier

Represents the runtime identifier used to correlate discovered resources with datastore processing/status surfaces without requiring distribution UUID lookup.

Fields:
- `resourceId`: Canonical mapped datastore resource identifier.
- `version`: Mapped resource version.
- `perspective`: Source/localized perspective.
- `datasetIdentifier`: Dataset-level correlation key.
- `downloadURL`: Original URL used for discovered-resource mapping and compatibility lookup.

Relationships:
- Produced from `ResourceMapper` resolution during discovered-resource reference mapping.
- Stored/used by compatibility surfaces (status lookup, dashboard/reporting, query context).

Validation rules:
- Must be derivable for valid discovered URLs regardless of referenced/non-referenced distribution mode.
- Must not require distribution UUID presence.

## Discovered Resource

Represents one normalized mapping from discovered metadata to operational datastore identity.

Fields:
- `candidate`: Source `Resource Discovery Candidate`.
- `compoundIdentifier`: Resolved `Compound Datastore Identifier`.
- `dataResource`: Runtime `DataResource` registration payload.

Relationships:
- Produced by Reference-owned discovery/reference-mapping service.
- Consumed by registration, triggering, and reporting paths without re-discovery.

Validation rules:
- Exactly one discovered resource per valid discovered candidate.
- No deduplication collapse in this feature scope.

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

- `ResourceDiscoveryResult`: the normalized Reference-owned discovery and discovered-resource mapping output consumed by downstream registration, triggering, and logging.

Its field-level definition lives in [contracts/dataset-save-dispatch.md](contracts/dataset-save-dispatch.md) because it describes a service boundary rather than a durable domain entity.

This phase does not introduce a new initiation-summary value-object contract; existing datastore initiation/import status surfaces remain in use.

Runtime behavior constraints such as retained ETL stage order and single active importer selection are documented in the spec and planning artifacts, not modeled here as feature data entities.
