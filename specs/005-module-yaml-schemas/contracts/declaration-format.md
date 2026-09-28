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
      path: schema/dataset.json
    ui_schema:
      path: schema/dataset.ui.json
    references:
      keyword:
        type: object_ref
        target_schema: keyword
      distribution[].downloadURL:
        type: resource_url
      distribution[].describedBy:
        type: item_url
        target_schema: data-dictionary
    behaviors:
      modified:
        - import_trigger
      distribution[].downloadURL:
        - resource_mapper
      distribution[].describedBy:
        - data_dictionary
    title: title
  keyword:
    validation_schema:
      inline:
        type: string
        title: Tag
        description: >-
          Tags (or keywords) help users discover your dataset; please include terms that would be used by
          technical and non-technical users.
        minLength: 1
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
- `behaviors` values are non-empty arrays of supported behavior names (`import_trigger`, `resource_mapper`, `data_dictionary`)

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

Reference entries map schema properties that must be dereferenced on retrieval to a reference type.

Required fields:

- `type`: `object_ref`, `item_url`, or `resource_url` for the initial implementation.
- `target_schema`: required for `object_ref` and `item_url`.

Default behavior:

- If `type` is omitted, treat it as `object_ref`.

Disallowed combinations:

- `target_schema` must not be present when `type` is `resource_url`.

Invalid reference definitions exclude the affected declaration and produce an operator warning.

## Behavior Definitions

Behavior entries map schema properties to one or more named system behaviors, independent of that property's reference definition (a property may declare both).

Required shape:

- Value is a non-empty array of supported behavior names.

Supported behavior names for the initial implementation:

- `import_trigger`: property change triggers datastore import for affected resources (replaces the previous `triggers` map).
- `resource_mapper`: property value is a downloadable/importable resource URL that should register or update resource linkage (replaces the previous `resource_mapper_registration` reference behavior).
- `data_dictionary`: property is the data dictionary source used by data-dictionary resolution.

Invalid behavior definitions exclude the affected declaration and produce an operator warning.

## Literal Inline Schema Example

```yaml
schemas:
  keyword:
    validation_schema:
      inline:
        type: string
```

Literal string schemas must not require a standalone JSON schema file.