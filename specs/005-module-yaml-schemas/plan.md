# Implementation Plan: Module-Based Schemas With YAML Declarations

**Branch**: `005-add-module-yaml-schemas` | **Date**: 2026-09-24 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/005-module-yaml-schemas/spec.md`

**Note**: This template is filled in by the `/speckit.plan` command. See `.specify/templates/plan-template.md` for the execution workflow.

## Summary

Replace DKAN's filesystem-inferred metastore schema discovery with explicit, module-owned `MODULE.schemas.yml` declarations. Enabled modules declare schema machine names, module-relative validation/UI schema paths, reference rules, datastore-trigger rules, and optional weights. A declaration-aware registry replaces `SchemaRetriever` internals, resolves duplicate machine names deterministically (weight, then Drupal module order), preserves `docroot/schema/collections` as an authoritative legacy override, and exposes administrator-visible reporting for source mode, selection reasons, and invalid declarations.

## Technical Context

**Language/Version**: PHP 8.1+ (Drupal 10 module code)
**Primary Dependencies**: Drupal core module/service system, Symfony YAML component (via Drupal core), existing DKAN `dkan_metastore` and `dkan_datastore` services (`SchemaRetriever`, `Referencer`); `GetDKAN/RootedJsonData` (opis/json-schema 2.x) for schema validation, whose `$ref`/`$id` registration support is a stretch-goal dependency (FR-038–FR-041)
**Storage**: YAML declaration files on disk (module-root `MODULE.schemas.yml`); no new database storage; runtime discovery results are cached via Drupal's cache API
**Testing**: PHPUnit (unit + kernel/integration tests), Behat (existing DKAN feature tests), `composer phpcs` for coding standards
**Target Platform**: Drupal 10 web application (DKAN distribution), Linux server
**Project Type**: Single Drupal distribution project (PHP modules under `modules/`) — no separate frontend/backend split
**Performance Goals**: Schema discovery/registry build must not add noticeable latency to cached metastore API requests; registry snapshot is cacheable so discovery does not re-scan modules per request
**Constraints**: Must preserve existing `SchemaRetriever`/`getAllIds()` consumer API surface where practical; must preserve `docroot/schema/collections` as an authoritative override for backward compatibility; declarations support both file paths and inline schema content, including literal-type inline schemas; the stretch-goal `dependencies`/`$ref` capability (FR-038–FR-041) may require a `GetDKAN/RootedJsonData` release and is not required for this feature's completion
**Scale/Scope**: DKAN core `dkan_metastore` and `dkan_datastore` modules plus a default schema module; scope covers `dataset` and `data-dictionary` as fixed core schema names, with `distribution` supported only when declared, and an optional DCAT-US 1.1 module

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- **I. Metadata-First Design**: PASS. Feature is entirely about how metastore JSON schema structure is declared and discovered; validation schemas remain the authority for metadata structure.
- **II. Modular Component Architecture**: PASS. Schema declarations are owned by modules; `dkan_metastore` and `dkan_datastore` continue to communicate through the existing `SchemaRetriever`/`Referencer` service boundaries rather than reaching into each other's internals. The stretch-goal `dependencies`/`$ref` capability (FR-038–FR-041) is scoped primarily to the external `GetDKAN/RootedJsonData` library rather than DKAN core, preserving this same component boundary; it is optional and does not block this feature's completion if deferred.
- **III. API-Driven Integration**: PASS. Existing `/api/1/metastore/schemas` endpoints remain the external contract; discovery internals change but the documented API surface is preserved (FR-016–FR-018).
- **IV. Standards Compliance & Data Quality**: PASS. Invalid declarations produce actionable administrator-visible validation messages (FR-006, FR-024, FR-030), consistent with quality-gate requirements.
- **V. Extensibility Through Drupal**: PASS. Declarations are standard Drupal module-root files tied to module enable/disable lifecycle; default and custom schemas both use this mechanism.
- **VI. Test Coverage & Documentation Excellence**: PASS (planned). Success criteria SC-004/SC-005/SC-014 require automated test coverage of references, triggers, and data-dictionary resolution; FR-025 requires documentation of the declaration format and migration steps.

No violations requiring Complexity Tracking justification.

## Project Structure

### Documentation (this feature)

```text
specs/[###-feature]/
├── plan.md              # This file (/speckit.plan command output)
├── research.md          # Phase 0 output (/speckit.plan command)
├── data-model.md        # Phase 1 output (/speckit.plan command)
├── quickstart.md        # Phase 1 output (/speckit.plan command)
├── contracts/           # Phase 1 output (/speckit.plan command)
└── tasks.md             # Phase 2 output (/speckit.tasks command - NOT created by /speckit.plan)
```

### Source Code (repository root)

```text
# DKAN Drupal distribution (dkan-noref checkout) — single project, PHP modules
modules/
├── dkan_metastore/
│   ├── src/
│   │   ├── SchemaRetriever.php        # Becomes declaration-aware registry consumer
│   │   ├── Reference/Referencer.php   # Reference rules driven by declarations
│   │   └── ...
│   ├── tests/
│   └── dkan_metastore.services.yml
├── dkan_datastore/
│   ├── src/                           # Trigger behavior consumes declaration triggers
│   └── tests/
├── dkan_sample_content/                # Candidate location for default schema module
│   └── dkan_sample_content.schemas.yml # or new dedicated default-schema module
└── [new/updated custom schema modules]/
    ├── MODULE.schemas.yml
    ├── schema/collections/*.json
    └── schema/collections/*.ui.json

schema/collections/                     # Existing default DCAT-US schema documents
docroot/schema/collections/             # Legacy override location (authoritative when present)

tests/
├── src/                                # PHPUnit unit/kernel tests
└── fixtures/                           # Test module fixtures with sample .schemas.yml files
```

**Structure Decision**: Single-project Drupal distribution (`dkan-noref`). No frontend/backend split is needed—this feature is entirely PHP module code within `modules/dkan_metastore` and `modules/dkan_datastore`, plus new or updated schema-declaration modules. Declaration discovery, the schema registry, and reporting live in `dkan_metastore` services; datastore trigger consumption lives in `dkan_datastore`.

## Complexity Tracking

*No Constitution Check violations identified; this section is not applicable.*
