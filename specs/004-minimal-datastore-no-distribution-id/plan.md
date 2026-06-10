# Implementation Plan: Minimal Datastore Without Distribution ID Requirement

**Branch**: `[main]` | **Date**: 2026-06-10 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `specs/004-minimal-datastore-no-distribution-id/spec.md`

## Summary
Implement a scoped datastore/metastore change so dataset-save side effects discover distribution `downloadURL` values and initiate datastore processing without requiring distribution UUIDs. The approach keeps existing `ResourceMapper`, `DataResource`, ETL stages, queue-driven execution, and immediate execution overrides while updating initiation, observability, cache/dependency behavior, reporting, and cleanup surfaces that currently assume distribution-ID lookup.

The plan is intentionally split into implementation surfaces that can become tickets sized for one developer to complete manually in a week or less.

## Technical Context

**Language/Version**: PHP 8-compatible Drupal module code; Drupal service container, event subscribers, queue workers, Drush commands  
**Primary Dependencies**: Drupal core services, DKAN metastore/datastore modules, `getdkan/csv-parser`, `getdkan/file-fetcher`, `getdkan/procrastinator`, `getdkan/rooted-json-data`, Guzzle  
**Storage**: Drupal entities and database tables, especially `resource_mapping` entities and datastore database tables  
**Testing**: PHPUnit (`phpunit.xml`), Drupal unit/kernel/functional tests, PHPCS with Drupal/DrupalPractice rules  
**Target Platform**: Drupal-based DKAN installation on PHP web/runtime plus Drush CLI  
**Project Type**: Drupal module feature spanning metastore and datastore components  
**Performance Goals**: Dataset-save traversal should be linear in discovered distribution entries; no new deduplication or global scans are introduced in this scoped feature  
**Constraints**: Preserve existing ResourceMapper/ETL behavior where practical; distribution references remain supported but are not required; no broader datastore-owned canonical resource redesign  
**Scale/Scope**: In-scope code is limited to `modules/dkan_metastore`, `modules/dkan_datastore`, datastore MySQL importer compatibility where needed, tests, and migration documentation

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

### Pre-Research Gate
- **I. Metadata-First Design**: PASS. The feature starts from dataset-save metadata traversal and supports configured dataset distribution structures without changing schema requirements.
- **II. Modular Component Architecture**: PASS WITH WATCH. The feature spans metastore and datastore; contracts document that metastore owns dataset/distribution discovery while datastore receives resource-context execution. Implementation must avoid datastore reaching into metastore internals beyond documented services.
- **III. API-Driven Integration**: PASS WITH WATCH. Public/API/Drush/admin surfaces are explicitly in scope for compatibility review and documentation.
- **IV. Standards Compliance & Data Quality**: PASS. Invalid/missing `downloadURL` values are skipped, reported, and do not block valid entries.
- **V. Extensibility Through Drupal**: PASS. Existing Drupal services, event subscribers, queues, and importer extension points remain the integration mechanism.
- **VI. Test Coverage & Documentation Excellence**: PASS WITH WORK REQUIRED. Plan includes unit/kernel/functional coverage and migration guidance for breaking hook payload changes.

No constitution violations require justification.

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
├── checklists/
│   └── requirements.md
└── tasks.md                 # Created later by /speckit.tasks
```

### Source Code (DKAN repository)

```text
modules/dkan_metastore/
├── src/
│   ├── LifeCycle/LifeCycle.php
│   ├── Reference/Referencer.php
│   ├── ResourceMapper.php
│   ├── EventSubscriber/MetastoreSubscriber.php
│   └── Plugin/QueueWorker/
└── tests/src/

modules/dkan_datastore/
├── src/
│   ├── DatastoreService.php
│   ├── DatastoreLookup.php
│   ├── Controller/ImportController.php
│   ├── Drush.php
│   ├── EventSubscriber/DatastoreSubscriber.php
│   ├── PostImportResultFactory.php
│   ├── Form/DashboardForm.php
│   ├── Service/ImportService.php
│   ├── Service/ResourcePurger.php
│   ├── Service/Info/
│   └── SqlEndpoint/
├── modules/dkan_datastore_mysql_import/
└── tests/src/

docs/source/
└── developer-guide/ or upgrade documentation as appropriate
```

**Structure Decision**: Use existing DKAN Drupal module boundaries. Metastore owns dataset-save traversal and resource registration/resolution. Datastore owns import/query/post-import/reporting/cache/cleanup behavior after a resource context is available. Tests remain beside their respective modules.

## Phase 0: Research Summary

Research is captured in [research.md](research.md). Key decisions:

- Keep `ResourceMapper`/`DataResource` as the scoped resource registry.
- Initiate datastore processing from dataset-save traversal rather than distribution-save side effects.
- Traverse referenced and non-referenced distribution structures the same way.
- Preserve dispatch encounter order and existing non-deduplication behavior.
- Use best-effort dispatch with structured logs and a machine-readable summary.
- Treat distribution references as supported metadata, not required operational keys.
- Split implementation by surface area so follow-up tasks can remain sub-week tickets.

## Phase 1: Design Summary

Design artifacts:

- [data-model.md](data-model.md)
- [contracts/dataset-save-dispatch.md](contracts/dataset-save-dispatch.md)
- [contracts/compatibility-surfaces.md](contracts/compatibility-surfaces.md)
- [quickstart.md](quickstart.md)

Ticket-sized implementation groups:

1. Dataset distribution traversal service.
2. ResourceMapper registration and datastore dispatch integration.
3. Dispatch result summary and structured logging.
4. Cache dependency and invalidation updates.
5. API, Drush, SQL endpoint, and admin compatibility review.
6. Post-import and dashboard reporting updates.
7. Cleanup and orphan behavior updates.
8. Migration guidance and custom importer hook updates.

## Constitution Check: Post-Design

- **I. Metadata-First Design**: PASS. Data model and contracts start from dataset metadata and distribution entries.
- **II. Modular Component Architecture**: PASS. Contracts define the metastore/datastore boundary and identify compatibility surfaces.
- **III. API-Driven Integration**: PASS. Compatibility contract includes public API, Drush, SQL endpoint, and admin/reporting review.
- **IV. Standards Compliance & Data Quality**: PASS. Invalid data handling is explicit and observable.
- **V. Extensibility Through Drupal**: PASS. Existing service/event/queue/importer patterns remain the implementation route.
- **VI. Test Coverage & Documentation Excellence**: PASS WITH REQUIRED TASKS. Follow-up tasks must include tests and migration docs before implementation is complete.

No new constitution violations introduced by design.

## Complexity Tracking

No constitution violations or additional architectural complexity require justification. The plan avoids the broader decoupled datastore identity redesign and keeps the scoped feature inside existing DKAN module patterns.
