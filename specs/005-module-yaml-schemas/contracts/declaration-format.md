# Contract: Module Schema Declaration Format

## File Location

Each enabled module may provide one declaration file at module root:

```text
MODULE.schemas.yml
```

`MODULE` must match the Drupal module machine name.

## Top-Level Shape

```yaml
schemas:
  dataset:
    weight: 0
    validation_schema:
      path: schema/collections/dataset.json
    ui_schema:
      path: schema/collections/dataset.ui.json
    references:
      distribution:
        type: object_ref
        target_schema: distribution
    triggers:
      distribution:
        behavior: datastore_import
  distribution:
    validation_schema:
      path: schema/collections/distribution.json
    references:
      downloadURL:
        type: resource_url
        behavior: resource_mapper_registration
  data-dictionary:
    validation_schema:
      path: schema/collections/data-dictionary.json
```

## Required Behavior

- The system discovers declaration files only from enabled modules.
- The system ignores module declarations when `docroot/schema/collections` exists.
- Each key under `schemas` is a schema machine name.
- Each schema definition must provide validation schema content through `validation_schema.path` or `validation_schema.inline`.
- UI schema content is optional and must be separate from validation schema content.
- `weight` is optional and numeric.
- Missing, unreadable, or malformed schema content invalidates only the affected declaration.
- Invalid declarations are excluded from active selection and included in operator warnings.

## JSON Schema Contract

Machine-validated declaration format is defined in:

```text
contracts/module-schemas.schema.json
```

This JSON Schema validates the parsed YAML structure and enforces:

- required `schemas` map
- required `validation_schema` with exactly one of `path` or `inline`
- optional `ui_schema` with exactly one of `path` or `inline`
- reference `type` values (`object_ref`, `item_url`, `resource_url`)
- `target_schema` required for `object_ref` and `item_url`
- `target_schema` forbidden for `resource_url`

## Duplicate Selection

For declarations with the same schema machine name:

1. Highest explicit `weight` wins.
2. Equal explicit weights, missing weights, or mixed missing/equal weights fall back to standard Drupal module weight/order.
3. The operator report must show all candidates, the winner, and the reason the winner was selected.

## Fixed Core Names

The first implementation phase uses fixed schema machine names for core behavior:

- `dataset`
- `distribution`
- `data-dictionary`
- `catalog`

Role aliases and multiple active schemas for the same core behavior are out of scope.

## Catalog Endpoint Semantics

The `catalog` schema is a special singleton endpoint contract:

- It is served at `/data.json`.
- It is not a metastore item type and does not expose an items collection endpoint.
- The route `/api/1/metastore/schemas/catalog/items` must not be registered or reachable.

This keeps catalog behavior aligned with DCAT-US expectations for a single catalog listing endpoint.

### Controller Structure

Catalog endpoint behavior is implemented through:

- an abstract catalog controller contract in core, defining shared `/data.json` handling behavior;
- a concrete catalog controller implementation provided by the active schema module.

If more than one catalog controller candidate exists, selection must be deterministic and aligned with active schema-module resolution.

## Reference Definitions

Reference entries map schema properties to supported reference behaviors.

Required fields:

- `type`: `object_ref`, `item_url`, or `resource_url` for the initial implementation.
- `target_schema`: required for `object_ref` and `item_url`.

Optional fields:

- `behavior`: named behavior for special handling, such as resource mapper registration.

Default behavior:

- If `type` is omitted, treat it as `object_ref`.

Disallowed combinations:

- `target_schema` must not be present when `type` is `resource_url`.

Invalid reference definitions exclude the affected declaration and produce an operator warning.

## Trigger Definitions

Trigger entries map schema properties to named system behaviors.

Required fields:

- `behavior`: supported behavior name, initially including `datastore_import`.

Invalid trigger definitions exclude the affected declaration and produce an operator warning.

## Literal Inline Schema Example

```yaml
schemas:
  keyword:
    validation_schema:
      inline:
        type: string
```

Literal string schemas must not require a standalone JSON schema file.