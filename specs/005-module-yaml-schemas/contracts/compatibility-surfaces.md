# Contract: Compatibility Surfaces

## Discovery Source Selection

When `docroot/schema/collections` exists:

- Active schema discovery uses legacy filesystem schemas.
- Module YAML declarations are ignored for active schema selection.
- Operator reporting states that legacy filesystem override mode is active.

When `docroot/schema/collections` is absent:

- Active schema discovery uses enabled module declarations.
- Duplicate declarations are resolved deterministically.
- Invalid declarations are skipped individually.

## Existing Metastore Schema API

Existing routes remain the public schema access surface:

- `GET /api/1/metastore/schemas`
- `GET /api/1/metastore/schemas/{identifier}`

Expected behavior:

- In legacy mode, responses are backed by filesystem schemas.
- In module declaration mode, responses are backed by active declarations.
- Public schema IDs remain fixed for core behavior in this phase.
- Missing schema IDs continue to return the existing schema-not-found behavior or equivalent error semantics.

## Internal Metastore Services

Existing consumers should continue to use metastore services rather than reading files directly:

- `dkan.metastore.schema_retriever`
- `dkan.metastore.valid_metadata`
- `dkan.metastore.schema_properties_helper`
- `dkan.metastore.service`
- metastore API docs generation

Implementation must update service internals or dependencies so these consumers receive active schema content from the selected source mode.

## Reference Handling

Current reference behavior uses configured property lists and property-specific logic. Declaration-based behavior becomes authoritative in module declaration mode.

Required compatibility:

- Dataset-to-distribution references continue to work when declared.
- Distribution-to-resource `downloadURL` handling continues to register resources when declared.
- Data-dictionary identifier/reference behavior continues to work when declared.
- Legacy filesystem mode preserves current behavior as much as practical until migration.

## Datastore Trigger Handling

Current datastore import triggers are selected through `dkan_datastore.settings:triggering_properties` and the datastore settings form.

Required compatibility:

- Declaration-based trigger definitions become authoritative once module declarations are active.
- The settings form path for trigger properties is removed, disabled, or clearly superseded to prevent conflicting runtime configuration.
- Existing rows limit, response stream max-age, degraded mode, and unrelated datastore settings remain available.
- Dataset update/pre-reference behavior still triggers datastore import when declared trigger properties change.

## Operator Reporting

The implementation must expose an operator-visible report or view showing:

- Active source mode: legacy filesystem or module declarations.
- Active schema per machine name.
- Provider module and source path for active declarations.
- Duplicate candidates, weights, module order fallback, and winner reasons.
- Invalid declarations and exclusion reasons.
- Declarations ignored because legacy filesystem override mode is active.

The report may be delivered as an admin page, Drush command, or both, but it must be accessible without reading logs.

## Documentation and Migration

Documentation must update custom schema guidance that currently directs maintainers to copy `schema/collections` into `docroot/schema/collections`.

Required docs:

- Declaration file format and examples.
- Default schema module behavior.
- Custom schema module migration path.
- Legacy override behavior and how to move from filesystem override to module declarations.
- Reference and trigger declaration examples.