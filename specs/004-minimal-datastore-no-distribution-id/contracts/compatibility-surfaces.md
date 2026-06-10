# Contract: Compatibility Surfaces

## Purpose

Identify existing datastore/metastore surfaces that must be reviewed so distribution UUID lookup is no longer required for normal import/query workflows.

## Surfaces

### Cache Dependencies and Invalidation

Current behavior may infer dataset/distribution context through resource-to-distribution reference lookup.

Required contract:
- Datastore summary/query/import-status responses must have valid cache behavior without mandatory distribution lookup.
- When references exist, metastore-provided context may enrich cache dependencies.
- Fallback cache dependencies must be safe when references are absent.

### Public API, SQL Endpoint, and Drush Lookup

Current behavior may accept or infer distribution IDs.

Required contract:
- Resource identifiers remain operational keys.
- Distribution IDs are compatibility metadata, not required keys.
- Reverse lookup commands are updated to resource/dataset context or documented as metadata-side compatibility helpers.

### Post-Import and Dashboard Reporting

Current behavior may pass distribution-shaped payloads to result factories or dashboard rows.

Required contract:
- Post-import status can be created from datastore resource mapping or dataset/downloadURL context.
- Dashboard rows render datastore status without requiring distribution payloads.
- Dataset/distribution labels are optional enrichments when metastore can resolve them.

### Cleanup and Orphan Behavior

Current behavior may rely on orphaned distribution reference events.

Required contract:
- Obsolete source/localized mappings and datastore artifacts can be cleaned up from dataset/downloadURL context.
- Referenced distribution orphan cleanup remains supported when references exist.
- Non-referenced distribution workflows have an equivalent cleanup path.

### Custom Importer Hooks

Legacy hook payloads may include distribution-ID fields.

Required contract:
- Legacy distribution-ID hook payload fields are removed in this scope.
- New payloads use datastore resource and dataset/downloadURL context.
- Migration guidance documents required custom importer changes.
