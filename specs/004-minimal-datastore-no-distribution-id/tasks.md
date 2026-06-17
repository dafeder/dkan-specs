# Tasks: Minimal Datastore Without Distribution ID Requirement

**Input**: Design documents from `specs/004-minimal-datastore-no-distribution-id/`
**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`, `contracts/`, `quickstart.md`

**Tests**: Included because the specification requires independently testable user stories and measurable outcomes for discovery, dispatch, compatibility, and reporting behavior.

**Path Note**: DKAN implementation paths below are relative to `/Users/dan.feder/Sites/dkan`. Spec artifacts are relative to `/Users/dan.feder/Work/dkan-specs`.

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Establish feature scaffolding and shared fixtures used by all stories.

- [ ] T001 Create feature implementation notes and validation checklist in `specs/004-minimal-datastore-no-distribution-id/quickstart.md`
- [ ] T002 [P] Add dataset discovery fixture variants for referenced/non-referenced distributions in `../dkan/modules/dkan_metastore/tests/src/Functional/OnPreReferenceTest.php`
- [ ] T003 [P] Add mixed-validity distribution fixture data (valid, missing, invalid, repeated downloadURL) in `../dkan/modules/dkan_metastore/tests/src/Functional/Api1/DistributionHandlingTest.php`
- [ ] T004 [P] Add datastore dispatch fixture coverage for multi-URL processing in `../dkan/modules/dkan_datastore/tests/src/Unit/EventSubscriber/DatastoreSubscriberTest.php`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Introduce shared discovery/result contracts and service wiring that all user stories depend on.

**CRITICAL**: No user story implementation starts until this phase is complete.

- [ ] T005 Create resource discovery result value object in `../dkan/modules/dkan_metastore/src/LifeCycle/ResourceDiscovery/ResourceDiscoveryResult.php`
- [ ] T006 [P] Create datastore dispatch item value object in `../dkan/modules/dkan_datastore/src/Dispatch/DatastoreDispatchItem.php`
- [ ] T007 [P] Create datastore dispatch result value object in `../dkan/modules/dkan_datastore/src/Dispatch/DatastoreDispatchResult.php`
- [ ] T008 Create dataset distribution discovery service for recursive downloadURL discovery in `../dkan/modules/dkan_metastore/src/LifeCycle/ResourceDiscovery/DatasetDistributionDiscovery.php`
- [ ] T009 Create datastore dispatcher service (register/dispatch/report pipeline) in `../dkan/modules/dkan_datastore/src/Dispatch/DatastoreDispatcher.php`
- [ ] T010 Register discovery and dispatcher services in `../dkan/modules/dkan_datastore/dkan_datastore.services.yml`
- [ ] T011 Add unit tests for discovery result object normalization and encounter ordering in `../dkan/modules/dkan_metastore/tests/src/Unit/LifeCycle/ResourceDiscovery/ResourceDiscoveryResultTest.php`
- [ ] T012 [P] Add unit tests for dataset discovery recursion and skip reasons in `../dkan/modules/dkan_metastore/tests/src/Unit/LifeCycle/ResourceDiscovery/DatasetDistributionDiscoveryTest.php`

**Checkpoint**: Shared contracts and services are in place; user stories can proceed.

---

## Phase 3: User Story 1 - Dataset-Save Discovery and Dispatch Without Distribution IDs (Priority: P1) MVP

**Goal**: Dataset-save hooks discover distribution `downloadURL` values and trigger datastore processing without requiring distribution-ID lookup.

**Independent Test**: Save datasets with referenced and non-referenced distribution structures and verify all valid discovered URLs are dispatched while invalid/missing entries are skipped and reported.

### Tests for User Story 1

- [ ] T013 [P] [US1] Add lifecycle functional tests proving pre-reference discovery executes during dataset presave in `../dkan/modules/dkan_metastore/tests/src/Functional/OnPreReferenceTest.php`
- [ ] T014 [P] [US1] Add unit tests for `DatastoreSubscriber::onPreReference` consumption of discovery result contract in `../dkan/modules/dkan_datastore/tests/src/Unit/EventSubscriber/DatastoreSubscriberTest.php`
- [ ] T015 [P] [US1] Add kernel tests validating URL registration through ResourceMapper without distribution lookup in `../dkan/modules/dkan_metastore/tests/src/Kernel/ResourceMapperTest.php`
- [ ] T016 [P] [US1] Add functional tests for mixed-validity distribution discovery and best-effort dispatch continuation in `../dkan/modules/dkan_metastore/tests/src/Functional/Api1/DistributionHandlingTest.php`
- [ ] T017 [P] [US1] Add kernel tests for queue-driven versus immediate dispatch behavior from discovered URLs in `../dkan/modules/dkan_datastore/tests/src/Kernel/DatastoreServiceEventsTest.php`

### Implementation for User Story 1

- [ ] T018 [US1] Refactor pre-reference datastore trigger path to use metastore discovery service in `../dkan/modules/dkan_datastore/src/EventSubscriber/DatastoreSubscriber.php`
- [ ] T019 [US1] Wire `LifeCycle::EVENT_PRE_REFERENCE` dataset metadata payload to dispatcher integration in `../dkan/modules/dkan_metastore/src/LifeCycle/LifeCycle.php`
- [ ] T020 [US1] Implement ResourceMapper registration/resolution from discovered resource values in `../dkan/modules/dkan_datastore/src/Dispatch/DatastoreDispatcher.php`
- [ ] T021 [US1] Implement best-effort per-URL datastore dispatch loop (continue after per-entry failure) in `../dkan/modules/dkan_datastore/src/Dispatch/DatastoreDispatcher.php`
- [ ] T022 [US1] Emit structured logs (stable fields for dataset, URL, status, reason) for skipped entries and per-URL dispatch failures in `../dkan/modules/dkan_datastore/src/Dispatch/DatastoreDispatcher.php`
- [ ] T023 [US1] Return machine-readable processed/skipped/failed dataset dispatch summary from dispatcher in `../dkan/modules/dkan_datastore/src/Dispatch/DatastoreDispatcher.php`
- [ ] T024 [US1] Remove runtime dependence on distribution entity dereference for dispatch initiation in `../dkan/modules/dkan_datastore/src/EventSubscriber/DatastoreSubscriber.php`

**Checkpoint**: User Story 1 works independently and provides dataset-save discovery/dispatch without distribution-ID requirement.

---

## Phase 4: User Story 2 - Preserve Existing Behavior Where Practical (Priority: P2)

**Goal**: Keep existing datastore surfaces stable where practical while moving operational keys to resource/dataset context.

**Independent Test**: Verify query/import/reporting paths continue to work with legacy inputs accepted for compatibility but not required for operational flow.

### Tests for User Story 2

- [ ] T025 [P] [US2] Add unit tests for query controller dependency extraction without mandatory distribution IDs in `../dkan/modules/dkan_datastore/tests/src/Unit/Controller/AbstractQueryControllerTest.php`
- [ ] T026 [P] [US2] Add kernel tests for import summary cache dependencies using resource/dataset context in `../dkan/modules/dkan_datastore/tests/src/Kernel/Controller/ImportControllerTest.php`
- [ ] T027 [P] [US2] Add functional tests for query download caching/invalidation behavior when distributions are non-referenced in `../dkan/modules/dkan_datastore/tests/src/Functional/Controller/QueryDownloadControllerTest.php`
- [ ] T028 [P] [US2] Add unit tests for SQL endpoint behavior with resource-based execution and compatibility inputs in `../dkan/modules/dkan_datastore/tests/src/Unit/SqlEndpoint/WebServiceApiTest.php`
- [ ] T029 [P] [US2] Add unit tests for drush command compatibility behavior where distribution IDs are optional metadata only in `../dkan/modules/dkan_datastore/tests/src/Functional/Commands/DegradedModeCommandsTest.php`
- [ ] T040 [P] [US2] Add unit tests for dashboard row rendering of discovered URL-only resources when distribution IDs are absent in `../dkan/modules/dkan_datastore/tests/src/Unit/Form/DashboardFormTest.php`
- [ ] T061 [P] [US2] Add functional dashboard continuity test verifying resource rows and status render without distribution IDs in `../dkan/modules/dkan_datastore/tests/src/Functional/Form/DashboardFormNoDistributionIdTest.php`

### Implementation for User Story 2

- [ ] T030 [US2] Update metastore dependency extraction to not require distribution lookup for cache tags/contexts in `../dkan/modules/dkan_datastore/src/Controller/AbstractQueryController.php`
- [ ] T031 [US2] Update query resource and dataset-resource execution paths to prioritize resource/dataset identifiers in `../dkan/modules/dkan_datastore/src/Controller/AbstractQueryController.php`
- [ ] T032 [US2] Refactor import summary dependency generation to support dataset+resource mapping invalidation in `../dkan/modules/dkan_datastore/src/Controller/ImportController.php`
- [ ] T033 [US2] Update query download response caching to align with resource/dataset invalidation strategy in `../dkan/modules/dkan_datastore/src/Controller/QueryDownloadController.php`
- [ ] T034 [US2] Update SQL endpoint service/controller to avoid distribution-ID operational assumptions in `../dkan/modules/dkan_datastore/src/SqlEndpoint/WebServiceApi.php`
- [ ] T035 [US2] Update datastore drush command parameter handling and help text for compatibility-only distribution IDs in `../dkan/modules/dkan_datastore/src/Drush/Commands/DatastoreCommands.php`
- [ ] T036 [US2] Update reimport workflow to resolve resources from dataset/downloadURL context when distribution references are absent in `../dkan/modules/dkan_datastore/src/Drush/Commands/ReimportCommands.php`
- [ ] T037 [US2] Document compatibility behavior and migration notes for changed operational keys in `../dkan/docs/source/upgrade.rst`
- [ ] T047 [US2] Update dashboard resource row builder to render discovered resources and status values without distribution IDs in `../dkan/modules/dkan_datastore/src/Form/DashboardForm.php`

**Checkpoint**: User Story 2 preserves practical compatibility while removing distribution IDs from normal runtime operation.

---

## Phase 5: User Story 3 - Keep Importer Modularity Simple (Priority: P3)

**Goal**: Maintain stage-level importer override model and default fallback behavior.

**Independent Test**: Validate post-import status and cleanup behavior while confirming stage override/fallback behavior is unchanged.

### Tests for User Story 3

- [ ] T038 [P] [US3] Add unit tests for `PostImportResultFactory` initialization from distribution UUID and dataset+resource_url inputs in `../dkan/modules/dkan_datastore/tests/src/Unit/Service/PostImportResultTest.php`
- [ ] T039 [P] [US3] Add kernel tests for import status lookup from resource mapping without distribution references in `../dkan/modules/dkan_datastore/tests/src/Kernel/Service/Info/ImportInfoTest.php`
- [ ] T041 [P] [US3] Add kernel tests for `DatasetInfo` gathering of discovered resources in `../dkan/modules/dkan_common/tests/src/Kernel/DatasetInfoTest.php`
- [ ] T042 [P] [US3] Add functional cleanup tests for referenced and non-referenced orphan resource paths in `../dkan/modules/dkan_metastore/tests/src/Functional/OrphanCheckerTest.php`
- [ ] T043 [P] [US3] Add kernel tests proving importer stage override/fallback behavior remains intact after refactor in `../dkan/modules/dkan_datastore/tests/src/Kernel/Service/ImportServiceEventsTest.php`
- [ ] T044 [P] [US3] Add unit tests validating removal of legacy distribution-ID hook payload fields in `../dkan/modules/dkan_datastore/tests/src/Unit/Service/PostImportResultTest.php`

### Implementation for User Story 3

- [ ] T045 [US3] Extend post-import result factory initialization to support dataset/resource context in `../dkan/modules/dkan_datastore/src/PostImportResultFactory.php`
- [ ] T046 [US3] Add resource-mapper-based status lookup helper for URL-only/distribution-backed resources in `../dkan/modules/dkan_datastore/src/Service/Info/ImportInfoList.php`
- [ ] T048 [US3] Integrate discovered-resource collection into dataset info gathering for reporting surfaces in `../dkan/modules/dkan_common/src/DatasetInfo.php`
- [ ] T049 [US3] Update datastore dataset-info plugin integration for combined distribution/discovered resource display in `../dkan/modules/dkan_datastore/src/Plugin/DatasetInfo/DatastoreInfo.php`
- [ ] T050 [US3] Update orphan cleanup path to remove obsolete mappings/artifacts without requiring orphaned distribution references in `../dkan/modules/dkan_metastore/src/Plugin/QueueWorker/OrphanResourceRemover.php`
- [ ] T051 [US3] Preserve single active importer with stage override fallback behavior in post-refactor import factory/service flow in `../dkan/modules/dkan_datastore/src/Service/Factory/ImportServiceFactory.php`
- [ ] T052 [US3] Remove legacy distribution-ID hook payload fields from importer event payload shaping in `../dkan/modules/dkan_datastore/src/Service/ImportService.php`
- [ ] T053 [US3] Update custom importer hook payload contract documentation for dataset/downloadURL/resource context in `../dkan/docs/source/components/dkan_datastore.rst`

**Checkpoint**: User Story 3 keeps importer modularity stable while preserving status and cleanup behavior.

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Final consistency checks, docs alignment, and validation execution.

- [ ] T054 [P] Add kernel test coverage for dataset dispatch summary counter accuracy (processed/skipped/failed) in `../dkan/modules/dkan_datastore/tests/src/Kernel/DatastoreServiceEventsTest.php`
- [ ] T055 [P] Capture pre/post compatibility inventory and verify >=80% source-compatible datastore surfaces in `specs/004-minimal-datastore-no-distribution-id/quickstart.md`
- [ ] T056 [P] Run metastore/datastore PHPUnit suites for feature validation and record outcomes in `specs/004-minimal-datastore-no-distribution-id/quickstart.md`
- [ ] T057 [P] Run PHPCS for touched datastore/metastore modules and record outcomes in `specs/004-minimal-datastore-no-distribution-id/quickstart.md`
- [ ] T058 [P] Add migration examples for legacy distribution-ID callers and updated importer hook payloads in `../dkan/docs/source/upgrade.rst`
- [ ] T059 Review cache invalidation strategy notes for query/import/streaming endpoints and capture final decisions in `specs/004-minimal-datastore-no-distribution-id/research.md`
- [ ] T060 Update completion checklist and residual risks in `specs/004-minimal-datastore-no-distribution-id/tasks.md`

---

## Dependencies & Execution Order

### Phase Dependencies

- **Phase 1 Setup**: No dependencies.
- **Phase 2 Foundational**: Depends on Phase 1; blocks all user story work.
- **Phase 3 US1 (P1)**: Depends on Phase 2; delivers MVP behavior.
- **Phase 4 US2 (P2)**: Depends on Phase 2 and integrates cleanly after US1 dispatch contracts exist.
- **Phase 5 US3 (P3)**: Depends on Phase 2 and should integrate after US1 resource discovery/dispatch contracts are stable.
- **Phase 6 Polish**: Depends on desired story set completion.

### User Story Dependencies

- **US1**: Independent after Foundational; required MVP.
- **US2**: Operationally independent after Foundational but expects US1 dispatch/discovery contracts for full end-to-end validation.
- **US3**: Operationally independent after Foundational but uses US1 discovery outcomes and benefits from US2 cache/compatibility updates.

### Within Each User Story

- Tests are listed first and should fail before implementation.
- Service/value object contracts precede controller/form integrations.
- Runtime behavior changes precede documentation finalization.
- Complete story checkpoint validation before moving to the next priority.

## Parallel Opportunities

- Setup tasks T002-T004 can run in parallel.
- Foundational value object and test tasks T006-T007 and T011-T012 can run in parallel.
- User-story test tasks marked [P] can run in parallel across separate test files.
- US2 controller refactors can split across query/import/sql/drush files.
- US3 status, dataset-info, and cleanup updates can run in parallel once post-import contract updates are agreed.

## Parallel Example: User Story 1

```bash
Task: "T013 [US1] Add lifecycle functional tests in ../dkan/modules/dkan_metastore/tests/src/Functional/OnPreReferenceTest.php"
Task: "T014 [US1] Add subscriber contract tests in ../dkan/modules/dkan_datastore/tests/src/Unit/EventSubscriber/DatastoreSubscriberTest.php"
Task: "T015 [US1] Add ResourceMapper kernel tests in ../dkan/modules/dkan_metastore/tests/src/Kernel/ResourceMapperTest.php"
```

## Parallel Example: User Story 2

```bash
Task: "T025 [US2] Add query dependency unit tests in ../dkan/modules/dkan_datastore/tests/src/Unit/Controller/AbstractQueryControllerTest.php"
Task: "T027 [US2] Add query download functional tests in ../dkan/modules/dkan_datastore/tests/src/Functional/Controller/QueryDownloadControllerTest.php"
Task: "T040 [US2] Add dashboard row unit tests in ../dkan/modules/dkan_datastore/tests/src/Unit/Form/DashboardFormTest.php"
```

## Parallel Example: User Story 3

```bash
Task: "T043 [US3] Add importer override/fallback kernel tests in ../dkan/modules/dkan_datastore/tests/src/Kernel/Service/ImportServiceEventsTest.php"
Task: "T051 [US3] Preserve single active importer stage overrides in ../dkan/modules/dkan_datastore/src/Service/Factory/ImportServiceFactory.php"
Task: "T049 [US3] Update orphan cleanup in ../dkan/modules/dkan_metastore/src/Plugin/QueueWorker/OrphanResourceRemover.php"
```

## Ticket Split (Quickstart 1-10)

### Shared Foundation (Complete Before Ticket Work)

- T001, T005, T006, T007, T008, T009, T010, T011, T012

### Ticket 1: Pre-Reference Dataset Discovery in LifeCycle Flow

- T002, T003, T004, T013, T014, T018, T019

### Ticket 2: ResourceMapper Registration and Import Trigger Integration

- T015, T016, T017, T020, T021, T023, T024

### Ticket 3: Cache Dependency and Invalidation Updates

- T025, T026, T027, T030, T031, T032, T033, T054, T059

### Ticket 4: API, Drush, SQL Endpoint, and Admin Compatibility Review

- T028, T029, T034, T035, T036, T037, T040, T047, T061

### Ticket 5: PostImportResultFactory Refactoring

- T038, T045

### Ticket 6: ResourceMapper Status Lookup Helper

- T039, T046

### Ticket 7: Importer Modularity Regression Assurance

- T043, T051

### Ticket 8: DatasetInfo Integration for Discovered Resources (Optional Scope)

- T041, T048, T049

### Ticket 9: Cleanup and Orphan Behavior Updates

- T042, T050

### Ticket 10: Migration Guidance and Custom Importer Hook Updates

- T044, T052, T053, T055, T058, T060

### Release Gate Validation (After Desired Tickets Complete)

- T056, T057

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1 and Phase 2.
2. Deliver US1 dataset-save discovery, registration, dispatch, and summary reporting.
3. Validate US1 independently before taking compatibility/reporting refactors.

### Incremental Delivery

1. Deliver US1 (MVP dispatch trigger correctness).
2. Deliver US2 (compatibility + cache and endpoint behavior updates).
3. Deliver US3 (regression assurance for importer modularity plus status/cleanup updates).
4. Complete Polish validation and documentation alignment.

### Team Parallelization

1. One engineer on metastore lifecycle/discovery contracts.
2. One engineer on datastore controller/cache/sql/drush compatibility.
3. One engineer on post-import status/cleanup/reporting (dashboard continuity stays in US2 compatibility work).

## Notes

- [P] marks tasks that can run concurrently because they touch separate files with no direct dependency.
- [US1], [US2], and [US3] map each task to the corresponding user story in `spec.md`.
- Distribution IDs remain accepted only as compatibility metadata; resource/dataset context is the operational key path.
