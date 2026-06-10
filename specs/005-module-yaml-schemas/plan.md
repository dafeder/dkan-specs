# Implementation Plan: Module-Based Schemas With YAML Declarations

**Branch**: `[005-add-module-yaml-schemas]` | **Date**: 2026-06-10 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `specs/005-module-yaml-schemas/spec.md`

## Summary

Introduce explicit module-root `MODULE.schemas.yml` declarations for metastore schema discovery when `docroot/schema/collections` is absent, while preserving the legacy filesystem directory as an authoritative backward-compatible override when present. The implementation replaces hard-coded filesystem discovery in `SchemaRetriever` with a declaration-aware registry, moves reference and datastore trigger rules into schema metadata, keeps fixed core schema machine names for this phase, exposes operator-visible selection and validation reports, and documents migration from legacy filesystem schemas to module-owned declarations.

This plan is shaped for manual implementation. Each implementation group is intended to become a ticket that one developer can complete in a week or less, with focused tests and documentation acceptance criteria.

## Technical Context

**Language/Version**: PHP 8-compatible Drupal module code; YAML declaration files parsed through Drupal/Symfony YAML tooling  
**Primary Dependencies**: Drupal core services (`extension.list.module`, module handler, service container, routing, logger, cache), DKAN metastore/datastore modules, JSON schema validation through existing DKAN metadata validation stack  
**Storage**: Module-root YAML files, JSON schema files or inline schema content, existing Drupal config/state for compatibility only, Drupal cache for discovery results if needed  
**Testing**: PHPUnit (`phpunit.xml`), Drupal unit/kernel/functional tests, PHPCS with Drupal/DrupalPractice rules  
**Target Platform**: Drupal-based DKAN installation on PHP web/runtime plus Drush CLI  
**Project Type**: Drupal module feature centered in metastore with datastore integration and documentation updates  
**Performance Goals**: Schema discovery should scan enabled modules once per discovery/cache rebuild and avoid per-request recursive filesystem scans; duplicate selection should be O(number of declarations for a schema machine name)  
**Constraints**: `docroot/schema/collections` remains authoritative when present; fixed core schema names (`dataset`, `distribution`, `data-dictionary`) drive business behavior in this phase; declaration failures exclude only the invalid declaration; one active schema per machine name  
**Scale/Scope**: In-scope code is limited to `modules/dkan_metastore`, `modules/dkan_datastore`, default schema packaging, docs, and related tests; no schema authoring UI or role/alias system in this phase

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

### Pre-Research Gate

- **I. Metadata-First Design**: PASS. The feature is explicitly about schema-controlled metadata validation, references, UI schema lookup, and datastore trigger metadata.
- **II. Modular Component Architecture**: PASS WITH WATCH. Metastore owns schema declaration discovery and reference rules; datastore consumes declared trigger behavior through documented service contracts and must not read metastore internals directly.
- **III. API-Driven Integration**: PASS WITH WATCH. Existing metastore schema API responses remain public surfaces; any new operator report or Drush/admin surface requires documented output.
- **IV. Standards Compliance & Data Quality**: PASS. Invalid declarations are excluded individually with actionable warnings, and valid declarations continue to participate.
- **V. Extensibility Through Drupal**: PASS. The feature uses enabled Drupal modules as the extension boundary and module-root declaration files as the installable/removable source of truth.
- **VI. Test Coverage & Documentation Excellence**: PASS WITH REQUIRED WORK. Follow-up tickets must include unit/kernel/functional coverage, declaration examples, and migration documentation before implementation is complete.

No constitution violations require justification.

## Project Structure

### Documentation (this feature)

```text
specs/005-module-yaml-schemas/
├── plan.md              # This file (/speckit.plan command output)
├── research.md          # Phase 0 output (/speckit.plan command)
├── data-model.md        # Phase 1 output (/speckit.plan command)
├── quickstart.md        # Phase 1 output (/speckit.plan command)
├── contracts/           # Phase 1 output (/speckit.plan command)
│   ├── declaration-format.md
│   └── compatibility-surfaces.md
├── checklists/
│   └── requirements.md
└── tasks.md             # Phase 2 output (/speckit.tasks command - NOT created by /speckit.plan)
```

### Source Code (DKAN repository)

```text
modules/dkan_metastore/
├── src/
│   ├── SchemaRetriever.php
│   ├── SchemaPropertiesHelper.php
│   ├── ValidMetadataFactory.php
│   ├── MetastoreService.php
│   ├── Reference/
│   ├── Controller/
│   └── Plugin/DkanApiDocs/
├── dkan_metastore.services.yml
├── dkan_metastore.routing.yml
├── config/schema/
└── tests/src/

modules/dkan_datastore/
├── src/
│   ├── EventSubscriber/DatastoreSubscriber.php
│   ├── Form/DatastoreSettingsForm.php
│   └── Service/
├── config/install/dkan_datastore.settings.yml
├── config/schema/dkan_datastore.schema.yml
└── tests/src/

schema/collections/
└── default schema JSON files to be packaged through module declarations

docs/source/
├── components/
├── developer-guide/
└── user-guide/
```

**Structure Decision**: Keep implementation inside existing DKAN Drupal module boundaries. Metastore owns declaration discovery, schema retrieval, validation/UI schema lookup, reference definitions, and operator schema reports. Datastore consumes declared trigger rules through metastore-facing services and removes or supersedes its trigger configuration UI. Default schemas remain in the DKAN codebase but become module-declared rather than inferred by a global hard-coded directory lookup.

## Phase 0: Research Summary

Research is captured in [research.md](research.md). Key decisions:

- Replace hard-coded `SchemaRetriever::getAllIds()` and filesystem-only retrieval with a declaration registry that can select active declarations.
- Treat `docroot/schema/collections` as a source-mode override; when it exists, module declarations are ignored for active schema selection.
- Use enabled module discovery for `MODULE.schemas.yml` files, with optional declaration `weight` and Drupal module weight/order fallback for duplicate machine names.
- Keep fixed core schema machine names for this phase and defer schema roles/aliases.
- Move reference and datastore trigger rules into declaration metadata while preserving the public metastore API shape where practical.
- Exclude invalid declarations individually and expose both validation warnings and schema selection reports to operators.

## Phase 1: Design Summary

Design artifacts:

- [data-model.md](data-model.md)
- [contracts/declaration-format.md](contracts/declaration-format.md)
- [contracts/compatibility-surfaces.md](contracts/compatibility-surfaces.md)
- [quickstart.md](quickstart.md)

Manual ticket-sized implementation groups:

1. Legacy override detection and source-mode reporting for `docroot/schema/collections`.
2. Enabled-module declaration discovery for module-root `MODULE.schemas.yml` files.
3. Declaration parsing and normalized value objects for schema, UI schema, references, triggers, and weights.
4. Declaration validation and warning collection with invalid-declaration exclusion.
5. Duplicate machine-name resolution by declaration weight and Drupal module order fallback.
6. `SchemaRetriever` active ID and validation schema retrieval integration.
7. UI schema retrieval integration for form-building consumers.
8. Metastore schema API and API-doc compatibility updates.
9. Default DCAT-US-style schema packaging as module declarations.
10. Reference declaration integration for dataset, distribution, resource, and identifier behavior.
11. Datastore trigger declaration integration and trigger settings UI cleanup.
12. Operator diagnostics for source mode, winners, ignored declarations, and invalid declarations.
13. Custom schema and upgrade documentation, including legacy override migration.
14. Cross-surface test coverage for legacy override, module mode, duplicates, invalid declarations, references, triggers, UI schemas, and API compatibility.

## Constitution Check: Post-Design

- **I. Metadata-First Design**: PASS. Data model and contracts make schema declarations the governing metadata source when the legacy override is absent.
- **II. Modular Component Architecture**: PASS. Contracts define metastore-owned declaration services and datastore consumption of trigger rules.
- **III. API-Driven Integration**: PASS. Existing `/api/1/metastore/schemas` behavior is preserved where practical, with operator reporting documented separately.
- **IV. Standards Compliance & Data Quality**: PASS. Invalid declarations are reported and excluded without blocking valid schemas.
- **V. Extensibility Through Drupal**: PASS. Enabled modules provide declarations, and Drupal module ordering participates in deterministic conflict resolution.
- **VI. Test Coverage & Documentation Excellence**: PASS WITH REQUIRED TASKS. Follow-up tickets must include tests for each declaration path and migration docs.

No new constitution violations introduced by design.

## Complexity Tracking

No constitution violations or extra architectural complexity require justification. The plan avoids schema role aliases, a schema authoring UI, and broad datastore redesign in this phase.
