# Data Model: Module-Based Schemas With YAML Declarations

## Schema Declaration File

Module-root YAML file named `MODULE.schemas.yml`, where `MODULE` matches the provider module machine name.

Attributes:

- `schemas`: map of schema machine names to schema definitions.
- File provider: inferred from the enabled Drupal module containing the file.

Validation rules:

- File is read only from enabled modules.
- File name must match the module machine name.
- Invalid YAML excludes every declaration in that file and records an operator warning.

Relationships:

- Contains one or more Schema Definitions.
- Belongs to one Drupal module provider.

## Schema Definition

Declared schema entry selected by schema machine name.

Attributes:

- Machine name: stable schema ID used by metastore lookup, derived from the key under `schemas`, such as `dataset`, `distribution`, or `data-dictionary`.
- Provider module: module machine name that owns the declaration.
- Validation schema source: path to JSON schema content or inline schema content.
- UI schema source: optional path or inline schema content for forms.
- Weight: optional numeric weight for duplicate resolution.
- References: optional list/map of Reference Definitions.
- Triggers: optional list/map of Trigger Definitions.
- Status: valid, invalid, active, inactive, or ignored-by-legacy-override.
- Validation messages: warnings or errors discovered during declaration validation.

Validation rules:

- Machine name is required as the schema map key; it is not repeated inside the schema definition body.
- Validation schema source is required and must resolve to parseable JSON-compatible schema content.
- UI schema source is optional, but if present must resolve to parseable JSON-compatible content.
- A declaration cannot define mutually exclusive schema sources in an ambiguous way; inline content and file path rules must be documented and validated.
- Invalid declarations are excluded individually.

Relationships:

- Has zero or more Reference Definitions.
- Has zero or more Trigger Definitions.
- Participates in one Schema Selection for its machine name unless legacy override mode is active.

## Schema Registry

Runtime discovery result for all available schema sources.

Attributes:

- Source mode: `legacy_filesystem` or `module_declarations`.
- Legacy directory: `docroot/schema/collections` when present.
- Declarations: all parsed module declarations when module mode is active.
- Active schemas: selected Schema Definitions by machine name when module mode is active, or filesystem schema IDs when legacy mode is active.
- Invalid declarations: declarations excluded with validation messages.
- Selection report: operator-visible report data.

Validation rules:

- If `docroot/schema/collections` exists, `source_mode` is `legacy_filesystem` and module declarations do not participate in active selection.
- If the legacy directory is absent, enabled module declarations are discovered and active schemas are selected from them.

## Schema Selection

Decision record for one schema machine name.

Attributes:

- Machine name: schema ID being selected.
- Candidates: valid declarations with that machine name.
- Winner: selected active Schema Definition.
- Selection reason: highest weight, Drupal module order fallback, or single candidate.
- Candidate order: ordered candidate list with weights and provider order.

Validation rules:

- Exactly one winner is selected when at least one valid candidate exists.
- Highest explicit declaration weight wins.
- Equal or missing weights fall back to standard Drupal module weight/order.

## Reference Definition

Declarative rule for metastore reference handling.

Attributes:

- Property: source property in the owning schema, derived from the key under `references`.
- Type: schema reference, identifier reference, resource reference, or supported future reference type.
- Target schema: schema machine name when relevant.
- Behavior: named behavior such as distribution reference or resource download URL registration.

Validation rules:

- Property is required as the reference map key; it is not repeated inside the reference definition body.
- Type must be supported.
- Target schema must be known when required by the reference type.

Relationships:

- Belongs to one Schema Definition.
- Consumed by metastore reference and dereference behavior.

## Trigger Definition

Declarative rule for system behavior driven by metadata changes.

Attributes:

- Property: source property that triggers behavior, derived from the key under `triggers`.
- Event: lifecycle point, initially dataset update/pre-reference behavior.
- Behavior: named behavior such as datastore import trigger.

Validation rules:

- Property is required as the trigger map key; it is not repeated inside the trigger definition body.
- Behavior must be supported.
- Initial implementation supports datastore import trigger behavior and fixed core schema names.

Relationships:

- Belongs to one Schema Definition.
- Consumed by datastore event subscriber behavior.

## Operator Report

Readable report for maintainers and site operators.

Attributes:

- Source mode: active discovery mode.
- Legacy override present: whether `docroot/schema/collections` exists.
- Active schemas: selected schemas and sources.
- Ignored declarations: declarations ignored because legacy override is active.
- Duplicate groups: machine names with more than one valid declaration.
- Invalid declarations: skipped declarations and reasons.
- Selection reasons: weight/order explanation for each active schema.

Validation rules:

- Must identify whether active schemas come from filesystem or module YAML.
- Must identify invalid declarations and exclusion reasons.
- Must identify selection reason for duplicate declarations.

## State Transitions

### Discovery Mode

1. `docroot/schema/collections` exists -> legacy filesystem mode.
2. Legacy directory removed -> module declaration mode on next discovery/cache rebuild.
3. Legacy directory added -> legacy filesystem mode on next discovery/cache rebuild, module declarations ignored.

### Declaration Lifecycle

1. Module disabled -> declarations unavailable.
2. Module enabled -> declarations parsed.
3. Declaration invalid -> excluded and warned.
4. Declaration valid but loses duplicate selection -> inactive candidate.
5. Declaration valid and wins selection -> active schema.