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
        type: schema
        target_schema: distribution
    triggers:
      distribution:
        behavior: datastore_import
  distribution:
    validation_schema:
      path: schema/collections/distribution.json
    references:
      downloadURL:
        type: resource
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

Role aliases and multiple active schemas for the same core behavior are out of scope.

## Reference Definitions

Reference entries map schema properties to supported reference behaviors.

Required fields:

- `type`: `schema`, `identifier`, or `resource` for the initial implementation.
- `target_schema`: required for schema or identifier references when a target schema is needed.

Optional fields:

- `behavior`: named behavior for special handling, such as resource mapper registration.

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