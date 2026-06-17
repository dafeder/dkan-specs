# Implementation Plan: Minimal Datastore Without Distribution ID Requirement

**Branch**: `main` | **Date**: 2026-06-15 | **Spec**: `specs/004-minimal-datastore-no-distribution-id/spec.md`
**Input**: Feature specification from `/specs/004-minimal-datastore-no-distribution-id/spec.md`

## Summary

Remove distribution UUIDs as required runtime keys for datastore initiation and downstream status/query workflows by shifting initiation to dataset-save discovery of distribution downloadURL values, while preserving ResourceMapper/DataResource, ETL stage behavior, queue/immediate execution modes, and single-active-importer modularity. The implementation keeps compatibility inputs where practical, but operational flow becomes dataset plus discovered resource driven.

## Technical Context

**Language/Version**: PHP 8.x on Drupal 10 module architecture
**Primary Dependencies**: DKAN modules (`dkan_metastore`, `dkan_datastore`, `dkan_common`), Drupal service container/event system, existing ResourceMapper/DataResource services
**Storage**: Drupal entities and metastore resource mapping records; datastore table storage remains unchanged
**Testing**: PHPUnit unit, kernel, and functional suites in DKAN module test directories; PHPCS for coding standards
**Target Platform**: DKAN Drupal application runtime (web + CLI/Drush)
**Project Type**: Drupal module feature update across metastore/datastore/common
**Performance Goals**: Preserve current dispatch characteristics; no new deduplication; best-effort multi-URL continuation
**Constraints**: Keep class/method surfaces stable where practical; no multi-importer priority selection; no datastore-owned canonical resource model redesign
**Scale/Scope**: Feature-scoped refactor focused on dataset-save dispatch, compatibility surfaces, cache invalidation, status/reporting, and cleanup behavior

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- Principle I (Metadata-First Design): PASS
  - Feature is metadata-driven and centered on dataset distribution discovery and schema-compatible metadata handling.
- Principle II (Modular Component Architecture): PASS
  - Changes preserve metastore/datastore separation and rely on documented service/event integration, not hidden data coupling.
- Principle III (API-Driven Integration): PASS WITH CONSTRAINTS
  - Public API/SQL/Drush compatibility surfaces are explicitly tracked for migration and behavior updates.
- Principle IV (Standards Compliance & Data Quality): PASS
  - Invalid/missing downloadURL entries are explicitly skipped/reported; structured observability and machine-readable summaries are required.
- Principle V (Extensibility Through Drupal): PASS
  - Importer modularity and stage-level override behavior are retained via existing Drupal extension patterns.
- Principle VI (Test Coverage & Documentation Excellence): PASS WITH REQUIRED WORK
  - Tasks include unit/kernel/functional coverage and migration docs; completion requires executing and recording required suite results.

Post-design re-check:
- Research, data model, contracts, and quickstart artifacts cover discovery, compatibility, cache invalidation, reporting, and cleanup paths.
- No constitutional violations identified; implementation must complete structured logging and migration documentation tasks to preserve Principle VI compliance.

## Project Structure

### Documentation (this feature)

```text
specs/004-minimal-datastore-no-distribution-id/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   ├── compatibility-surfaces.md
│   └── dataset-save-dispatch.md
└── tasks.md
```

### Source Code (repository root)

```text
../dkan/modules/dkan_metastore/
├── src/ResourceDiscovery/
├── src/LifeCycle/
├── src/Plugin/QueueWorker/
└── tests/src/

../dkan/modules/dkan_datastore/
├── src/EventSubscriber/
├── src/Controller/
├── src/Dispatch/
├── src/Service/
├── src/SqlEndpoint/
├── src/Form/
├── src/Drush/Commands/
└── tests/src/

../dkan/modules/dkan_common/
├── src/
└── tests/src/

../dkan/docs/source/
```

**Structure Decision**: Use the existing DKAN multi-module Drupal layout. Implement behavior changes primarily in `dkan_datastore` and `dkan_metastore`, with shared reporting/status data shape adjustments in `dkan_common`, and migration/update documentation in DKAN docs. Place metastore-specific discovery classes under `dkan_metastore/src/ResourceDiscovery/` and datastore-specific dispatch classes under `dkan_datastore/src/Dispatch/` rather than the generic `src/Service/` namespace.

## Complexity Tracking

No constitutional violations require exception handling at this stage.
