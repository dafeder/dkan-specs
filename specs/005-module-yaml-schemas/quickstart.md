# Quickstart: Module-Based Schemas With YAML Declarations

This quickstart describes how to validate the planned behavior after implementation.

## Prerequisites

- DKAN checkout with tests installable through the existing project setup.
- A site without `docroot/schema/collections` for module declaration mode tests.
- A site with `docroot/schema/collections` for legacy override tests.

## 1. Add a Module Declaration

Create or use a test module with a module-root declaration file:

```text
test_schema_provider/test_schema_provider.schemas.yml
```

Example:

```yaml
schemas:
  dataset:
    weight: 10
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
```

Enable the module and clear caches.

## 2. Verify Module Declaration Mode

With `docroot/schema/collections` absent:

- `GET /api/1/metastore/schemas` includes active declared schemas.
- `GET /api/1/metastore/schemas/dataset` returns the selected dataset validation schema.
- UI schema retrieval uses the declared UI schema when present.
- Invalid declarations are absent from active schemas and visible in the administrator report.
- Duplicate declarations select the highest `weight`; equal weights fall back to Drupal module order.

## 3. Verify Legacy Override Mode

Create `docroot/schema/collections` with legacy schema JSON files, then clear caches.

Expected results:

- Active schemas come from `docroot/schema/collections`.
- Module YAML declarations do not participate in active selection.
- Administrator reporting states that legacy filesystem override mode is active.
- Ignored module declarations are visible as ignored or inactive because of the override.

## 4. Verify Reference Behavior

Save metadata containing declared references.

Expected results:

- Dataset distribution references are processed according to declaration metadata.
- Distribution `downloadURL` resource behavior registers resources when declared.
- Existing metastore reference API behavior remains stable for supported core schema names.

## 5. Verify Datastore Trigger Behavior

Update metadata properties declared with `datastore_import` trigger behavior.

Expected results:

- Dataset update/pre-reference behavior detects declared trigger changes.
- Datastore import behavior runs for affected resources.
- The old datastore trigger property settings UI is removed, disabled, or clearly superseded.

## 6. Run Focused Test Suites

Suggested checks after implementation:

```bash
composer test -- --filter Schema
composer test -- --filter Metastore
composer test -- --filter Datastore
composer phpcs
```

If this project uses different local aliases for PHPUnit or PHPCS, use the commands documented in the DKAN checkout.

## Manual Ticket Guidance

Keep implementation pull requests small enough for one developer to finish in a week or less:

1. Parser and value objects.
2. Registry and legacy override source selection.
3. Retriever/API/UI schema compatibility.
4. Duplicate selection and administrator reports.
5. Default schema module packaging.
6. Reference declarations.
7. Datastore trigger declarations and settings UI cleanup.
8. Docs and migration coverage.