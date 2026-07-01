# Implementation Plan: Minimal Datastore Without Distribution ID Requirement

**Branch**: `[004-minimal-datastore-no-distribution-id]` | **Date**: 2026-07-01 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/004-minimal-datastore-no-distribution-id/spec.md`

## Summary

Correct the current implementation direction so dataset-save handling does not only register discovered resources, but also performs the missing reference mapping from `distribution[].downloadURL` to a compound datastore identifier before datastore initiation. To preserve architecture boundaries and avoid regressions, move discovery ownership to the metastore Reference layer (not LifeCycle-only resource registration), while still invoking the workflow during dataset presave so processing is independent of optional distribution referencing toggles.

## Technical Context

**Language/Version**: PHP (Drupal module codebase; PHPUnit 9.6 test harness)  
**Primary Dependencies**: Drupal module/service container patterns, `getdkan/contracts`, existing `ResourceMapper`/`DataResource` workflow, DKAN metastore/datastore modules  
**Storage**: Drupal entities plus existing resource-mapping persistence used by metastore/datastore integration  
**Testing**: PHPUnit unit/kernel/functional suites under `modules/dkan_metastore` and `modules/dkan_datastore`; PHPCS with Drupal + DrupalPractice rules  
**Target Platform**: DKAN on Drupal (PHP runtime in Linux/containerized dev flows)  
**Project Type**: Backend Drupal modules (metastore + datastore integration)  
**Performance Goals**: Process all valid top-level `distribution[].downloadURL` entries in encounter order during dataset save with best-effort continuation  
**Constraints**: No required runtime distribution ID lookup; no URL deduplication change; preserve queue/immediate datastore initiation behavior; preserve `describedBy` behavior in referenced and non-referenced modes  
**Scale/Scope**: Feature-scoped refactor across metastore reference/lifecycle orchestration and datastore initiation/compatibility surfaces

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- **I. Metadata-First Design**: PASS. Discovery/referencing remains driven by dataset metadata (`$.distribution[]`, `downloadURL`, `describedBy`) and preserves schema-dependent behavior.
- **II. Modular Component Architecture**: PASS. Metastore owns the discovered-resource reference contract; datastore consumes mapped resources through existing interfaces.
- **III. API-Driven Integration**: PASS. No new bypass path is introduced; internal orchestration still uses documented Drupal services/events.
- **IV. Standards Compliance & Data Quality**: PASS. Invalid/missing URL and URI validation paths stay explicit and administrator-visible.
- **V. Extensibility Through Drupal**: PASS. Existing importer extension points remain in place; migration guidance addresses payload transitions.
- **VI. Test Coverage & Documentation Excellence**: PASS WITH WORK ITEMS. Requires updated unit/kernel/functional coverage plus contract and upgrade docs for the corrected flow.

Post-Phase-1 re-check: PASS. The updated model and contracts keep component boundaries explicit while restoring required reference semantics.

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

### Source Code (implementation target)

```text
modules/dkan_metastore/
├── src/LifeCycle/
├── src/Reference/
└── tests/src/
    ├── Unit/
    ├── Kernel/
    └── Functional/

modules/dkan_datastore/
├── src/EventSubscriber/
├── src/Service/
├── src/Controller/
├── src/Drush/
└── tests/src/
    ├── Unit/
    ├── Kernel/
    └── Functional/

docs/source/
├── upgrade.rst
└── components/dkan_datastore.rst
```

**Structure Decision**: Keep the existing DKAN module layout and update metastore reference-layer contracts and datastore consumers in place. Do not introduce a new top-level package or dispatch subsystem in this scope.

## Complexity Tracking

No constitution violations requiring justification.
