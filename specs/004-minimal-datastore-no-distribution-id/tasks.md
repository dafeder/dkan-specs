# Tasks: Minimal Datastore Without Distribution ID Requirement

**Input**: Design documents from `specs/004-minimal-datastore-no-distribution-id/`
**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`, `contracts/`, `quickstart.md`

**Tests**: Included because the specification requires independently testable user stories and measurable outcomes for discovery, dispatch, compatibility, and reporting behavior.

**Path Note**: DKAN implementation paths below are relative to `/Users/dan.feder/Sites/dkan`. Spec artifacts are relative to `/Users/dan.feder/Work/dkan-specs`.

## Shared Foundation (Complete Before Ticket Work)

**Purpose**: Establish the shared contracts, service wiring, and planning notes that every quickstart ticket depends on.

**CRITICAL**: Complete this section before starting ticket implementation work.

- [ ] T001 Create feature implementation notes and validation checklist in `specs/004-minimal-datastore-no-distribution-id/quickstart.md`
- [x] T005 Create resource discovery result value object in `../dkan/modules/dkan_metastore/src/LifeCycle/ResourceDiscovery/ResourceDiscoveryResult.php`
- [x] T006 [P] Create dispatch item value object in `../dkan/modules/dkan_datastore/src/Dispatch/DispatchItem.php`
- [x] T007 [P] Create dispatch result value object in `../dkan/modules/dkan_datastore/src/Dispatch/DispatchResult.php`
- [x] T008 Create dataset resource discovery service for recursive downloadURL discovery in `../dkan/modules/dkan_metastore/src/LifeCycle/ResourceDiscovery/DatasetResourceDiscovery.php`
- [x] T009 Create dispatcher service (register/dispatch/report pipeline) in `../dkan/modules/dkan_datastore/src/Dispatch/Dispatcher.php`
- [x] T010 Register discovery and dispatcher services in `../dkan/modules/dkan_datastore/dkan_datastore.services.yml`
- [ ] T011 Add unit tests for discovery result object normalization and encounter ordering in `../dkan/modules/dkan_metastore/tests/src/Unit/LifeCycle/ResourceDiscovery/ResourceDiscoveryResultTest.php`
- [ ] T012 [P] Add unit tests for dataset resource discovery recursion and skip reasons in `../dkan/modules/dkan_metastore/tests/src/Unit/LifeCycle/ResourceDiscovery/DatasetResourceDiscoveryTest.php`

**Checkpoint**: Shared contracts and services are in place; ticket work can proceed.

---

## Ticket 1: Pre-Reference Dataset Discovery in LifeCycle Flow

**Goal**: Implement dataset-save discovery in the pre-reference lifecycle path using a reusable discovery contract.

**Independent Test**: Save datasets with referenced and non-referenced distribution structures and verify pre-reference discovery identifies valid `downloadURL` values while invalid/missing entries are skipped and reported.

### Tests for Ticket 1

- [ ] T002 [P] Add dataset discovery fixture variants and lifecycle coverage for referenced/non-referenced distributions in `../dkan/modules/dkan_metastore/tests/src/Functional/OnPreReferenceTest.php`
- [ ] T003 [P] Add mixed-validity distribution fixture data (valid, missing, invalid, repeated downloadURL) in `../dkan/modules/dkan_metastore/tests/src/Functional/Api1/DistributionHandlingTest.php`
- [ ] T004 [P] Add datastore subscriber fixture and contract coverage for multi-URL processing in `../dkan/modules/dkan_datastore/tests/src/Unit/EventSubscriber/DatastoreSubscriberTest.php`

### Implementation for Ticket 1

- [ ] T018 [US1] Refactor pre-reference datastore trigger path to use metastore discovery service in `../dkan/modules/dkan_datastore/src/EventSubscriber/DatastoreSubscriber.php`
- [ ] T019 [US1] Wire `LifeCycle::EVENT_PRE_REFERENCE` dataset metadata payload to dispatcher integration in `../dkan/modules/dkan_metastore/src/LifeCycle/LifeCycle.php`

**Checkpoint**: Dataset-save discovery runs from the pre-reference lifecycle path and hands off a shared discovery contract.

---

## Ticket 2: ResourceMapper Registration and Import Trigger Integration

**Goal**: Register discovered resources and trigger datastore processing without distribution-ID lookup.

**Independent Test**: Save datasets with valid discovered URLs and verify each valid resource is registered and processed, while failures do not block later URLs.

### Tests for Ticket 2

- [ ] T015 [P] [US1] Add kernel tests validating URL registration through ResourceMapper without distribution lookup in `../dkan/modules/dkan_metastore/tests/src/Kernel/ResourceMapperTest.php`
- [ ] T016 [P] [US1] Add functional tests for mixed-validity distribution discovery and best-effort dispatch continuation in `../dkan/modules/dkan_metastore/tests/src/Functional/Api1/DistributionHandlingTest.php`
- [ ] T017 [P] [US1] Add kernel tests for queue-driven versus immediate dispatch behavior from discovered URLs in `../dkan/modules/dkan_datastore/tests/src/Kernel/DatastoreServiceEventsTest.php`

### Implementation for Ticket 2

- [ ] T020 [US1] Implement dispatcher registration, best-effort per-URL processing, structured logging, and processed/skipped/failed summary reporting in `../dkan/modules/dkan_datastore/src/Dispatch/Dispatcher.php`
- [ ] T024 [US1] Remove runtime dependence on distribution entity dereference for dispatch initiation in `../dkan/modules/dkan_datastore/src/EventSubscriber/DatastoreSubscriber.php`

**Checkpoint**: Resource registration and dataset-save dispatch work end-to-end without requiring distribution-ID lookup.

---

## Ticket 3: Cache Dependency and Invalidation Updates

**Goal**: Update cache dependency and invalidation behavior to operate from dataset/resource context rather than distribution lookups.

**Independent Test**: Exercise query, import summary, and download cache behavior with referenced and non-referenced distributions and verify invalidation stays correct.

### Tests for Ticket 3

- [ ] T025 [P] [US2] Add unit tests for query controller dependency extraction without mandatory distribution IDs in `../dkan/modules/dkan_datastore/tests/src/Unit/Controller/AbstractQueryControllerTest.php`
- [ ] T026 [P] [US2] Add kernel tests for import summary cache dependencies using resource/dataset context in `../dkan/modules/dkan_datastore/tests/src/Kernel/Controller/ImportControllerTest.php`
- [ ] T027 [P] [US2] Add functional tests for query download caching/invalidation behavior when distributions are non-referenced in `../dkan/modules/dkan_datastore/tests/src/Functional/Controller/QueryDownloadControllerTest.php`
- [ ] T054 [P] Add kernel test coverage for dataset dispatch summary counter accuracy (processed/skipped/failed) in `../dkan/modules/dkan_datastore/tests/src/Kernel/DatastoreServiceEventsTest.php`

### Implementation for Ticket 3

- [ ] T030 [US2] Update metastore dependency extraction to not require distribution lookup for cache tags/contexts in `../dkan/modules/dkan_datastore/src/Controller/AbstractQueryController.php`
- [ ] T031 [US2] Update query resource and dataset-resource execution paths to prioritize resource/dataset identifiers in `../dkan/modules/dkan_datastore/src/Controller/AbstractQueryController.php`
- [ ] T032 [US2] Refactor import summary dependency generation to support dataset+resource mapping invalidation in `../dkan/modules/dkan_datastore/src/Controller/ImportController.php`
- [ ] T033 [US2] Update query download response caching to align with resource/dataset invalidation strategy in `../dkan/modules/dkan_datastore/src/Controller/QueryDownloadController.php`
- [ ] T059 Review cache invalidation strategy notes for query/import/streaming endpoints and capture final decisions in `specs/004-minimal-datastore-no-distribution-id/research.md`

**Checkpoint**: In-scope datastore cache surfaces invalidate correctly without requiring runtime distribution-ID lookup.

---

## Ticket 4: DatasetInfo Integration for Discovered Resources

**Goal**: Decide how discovered resources surface through dataset info and reporting views before dashboard compatibility work consumes that shape.

**Independent Test**: Gather dataset info for datasets with discovered resources and verify reporting surfaces render whichever data model is chosen.

### Tests for Ticket 4

- [ ] T041 [P] [US3] Add kernel tests for `DatasetInfo` gathering of discovered resources in `../dkan/modules/dkan_common/tests/src/Kernel/DatasetInfoTest.php`

### Implementation for Ticket 4

- [ ] T048 [US3] Integrate discovered-resource collection into dataset info gathering for reporting surfaces in `../dkan/modules/dkan_common/src/DatasetInfo.php`
- [ ] T049 [US3] Update datastore dataset-info plugin integration for combined distribution/discovered resource display in `../dkan/modules/dkan_datastore/src/Plugin/DatasetInfo/DatastoreInfo.php`

**Checkpoint**: Dataset info surfaces discovered resources in a way dashboard and reporting code can consume consistently.

---

## Ticket 5: API, Drush, SQL Endpoint, and Admin Compatibility Review

**Goal**: Preserve compatibility where practical while moving operational flows to dataset/resource identifiers.

**Independent Test**: Exercise query, SQL, Drush, dashboard/reporting, post-import status, and resource-mapper status lookup paths and verify distribution IDs are optional compatibility metadata rather than required runtime keys, using the DatasetInfo shape established in Ticket 4.

### Tests for Ticket 5

- [ ] T028 [P] [US2] Add unit tests for SQL endpoint behavior with resource-based execution and compatibility inputs in `../dkan/modules/dkan_datastore/tests/src/Unit/SqlEndpoint/WebServiceApiTest.php`
- [ ] T029 [P] [US2] Add unit tests for drush command compatibility behavior where distribution IDs are optional metadata only in `../dkan/modules/dkan_datastore/tests/src/Functional/Commands/DegradedModeCommandsTest.php`
- [ ] T040 [P] [US2] Add unit tests for dashboard row rendering of discovered URL-only resources when distribution IDs are absent in `../dkan/modules/dkan_datastore/tests/src/Unit/Form/DashboardFormTest.php`
- [ ] T038 [P] [US3] Add unit tests for `PostImportResultFactory` initialization from distribution UUID and dataset+resource_url inputs in `../dkan/modules/dkan_datastore/tests/src/Unit/Service/PostImportResultTest.php`
- [ ] T039 [P] [US3] Add kernel tests for import status lookup from resource mapping without distribution references in `../dkan/modules/dkan_datastore/tests/src/Kernel/Service/Info/ImportInfoTest.php`
- [ ] T061 [P] [US2] Add functional dashboard continuity test verifying resource rows and status render without distribution IDs in `../dkan/modules/dkan_datastore/tests/src/Functional/Form/DashboardFormNoDistributionIdTest.php`

### Implementation for Ticket 5

- [ ] T034 [US2] Update SQL endpoint service/controller to avoid distribution-ID operational assumptions in `../dkan/modules/dkan_datastore/src/SqlEndpoint/WebServiceApi.php`
- [ ] T035 [US2] Update datastore drush command parameter handling and help text for compatibility-only distribution IDs in `../dkan/modules/dkan_datastore/src/Drush/Commands/DatastoreCommands.php`
- [ ] T036 [US2] Update reimport workflow to resolve resources from dataset/downloadURL context when distribution references are absent in `../dkan/modules/dkan_datastore/src/Drush/Commands/ReimportCommands.php`
- [ ] T037 [US2] Document compatibility behavior and migration notes for changed operational keys in `../dkan/docs/source/upgrade.rst`
- [ ] T045 [US3] Extend post-import result factory initialization to support dataset/resource context in `../dkan/modules/dkan_datastore/src/PostImportResultFactory.php`
- [ ] T046 [US3] Add resource-mapper-based status lookup helper for URL-only/distribution-backed resources in `../dkan/modules/dkan_datastore/src/Service/Info/ImportInfoList.php`
- [ ] T047 [US2] Update dashboard resource row builder to render discovered resources and status values without distribution IDs in `../dkan/modules/dkan_datastore/src/Form/DashboardForm.php`

**Checkpoint**: Compatibility-sensitive datastore entry points, post-import status flows, and status lookup surfaces continue to work without requiring distribution IDs as operational keys.

---

## Ticket 6: Importer Modularity Regression Assurance

**Goal**: Preserve the single active importer model with stage-level override and fallback behavior.

**Independent Test**: Configure stage overrides and verify overridden stages run while non-overridden stages keep default behavior.

### Tests for Ticket 6

- [ ] T043 [P] [US3] Add kernel tests proving importer stage override/fallback behavior remains intact after refactor in `../dkan/modules/dkan_datastore/tests/src/Kernel/Service/ImportServiceEventsTest.php`

### Implementation for Ticket 6

- [ ] T051 [US3] Preserve single active importer with stage override fallback behavior in post-refactor import factory/service flow in `../dkan/modules/dkan_datastore/src/Service/Factory/ImportServiceFactory.php`

**Checkpoint**: Importer override and fallback behavior remains unchanged after the refactor.

---

## Ticket 7: Cleanup and Orphan Behavior Updates

**Goal**: Remove obsolete mappings and datastore artifacts without depending solely on orphaned distribution references.

**Independent Test**: Run cleanup flows for referenced and non-referenced resources and verify obsolete mappings and artifacts are removed.

### Tests for Ticket 7

- [ ] T042 [P] [US3] Add functional cleanup tests for referenced and non-referenced orphan resource paths in `../dkan/modules/dkan_metastore/tests/src/Functional/OrphanCheckerTest.php`

### Implementation for Ticket 7

- [ ] T050 [US3] Update orphan cleanup path to remove obsolete mappings/artifacts without requiring orphaned distribution references in `../dkan/modules/dkan_metastore/src/Plugin/QueueWorker/OrphanResourceRemover.php`

**Checkpoint**: Cleanup works for both referenced and non-referenced distribution workflows.

---

## Ticket 8: Migration Guidance and Custom Importer Hook Updates

**Goal**: Capture the required migration steps for changed payloads and compatibility behavior.

**Independent Test**: Review the migration guidance against the changed hook payloads and compatibility behavior and verify each required update is documented.

### Tests for Ticket 8

- [ ] T044 [P] [US3] Add unit tests validating removal of legacy distribution-ID hook payload fields in `../dkan/modules/dkan_datastore/tests/src/Unit/Service/PostImportResultTest.php`

### Implementation for Ticket 8

- [ ] T052 [US3] Remove legacy distribution-ID hook payload fields from importer event payload shaping in `../dkan/modules/dkan_datastore/src/Service/ImportService.php`
- [ ] T053 [US3] Update custom importer hook payload contract documentation for dataset/downloadURL/resource context in `../dkan/docs/source/components/dkan_datastore.rst`
- [ ] T055 [P] Capture pre/post compatibility inventory and verify >=80% source-compatible datastore surfaces in `specs/004-minimal-datastore-no-distribution-id/quickstart.md`
- [ ] T058 [P] Add migration examples for legacy distribution-ID callers and updated importer hook payloads in `../dkan/docs/source/upgrade.rst`
- [ ] T060 Update completion checklist and residual risks in `specs/004-minimal-datastore-no-distribution-id/tasks.md`

**Checkpoint**: Migration expectations are documented for downstream maintainers and importer implementers.

---

## Release Gate Validation (After Desired Tickets Complete)

**Purpose**: Run the final feature validation commands after the intended ticket set is complete.

- [ ] T056 [P] Run metastore/datastore PHPUnit suites for feature validation and record outcomes in `specs/004-minimal-datastore-no-distribution-id/quickstart.md`
- [ ] T057 [P] Run PHPCS for touched datastore/metastore modules and record outcomes in `specs/004-minimal-datastore-no-distribution-id/quickstart.md`

---

## Dependencies & Execution Order

### Ticket Dependencies

- **Shared Foundation**: No dependency; blocks implementation work for all tickets.
- **Ticket 1**: Depends on Shared Foundation for implementation tasks T018-T019, while fixture tasks T002-T004 can start earlier.
- **Ticket 2**: Depends on Shared Foundation and builds directly on Ticket 1 discovery handoff.
- **Ticket 3**: Depends on Shared Foundation and benefits from Ticket 2 dispatch/result behavior for end-to-end cache validation.
- **Ticket 4**: Depends on Shared Foundation and benefits from Ticket 2 resource registration behavior so DatasetInfo can surface discovered resources.
- **Ticket 5**: Depends on Shared Foundation and Ticket 4, because dashboard/reporting compatibility work consumes the DatasetInfo shape established there and the same compatibility pass now includes post-import status initialization and status lookup updates.
- **Tickets 6-8**: Depend on Shared Foundation; Tickets 6-7 benefit from Ticket 2 resource registration/dispatch behavior, and Ticket 8 should land after the affected payload changes are settled.
- **Release Gate Validation**: Depends on completion of the intended ticket set.

### Within Each Ticket

- Tests are listed first and should fail before implementation.
- Service/value object contracts precede controller/form integrations.
- Runtime behavior changes precede documentation finalization.
- Complete each ticket checkpoint before moving to the next dependent ticket.

## Parallel Opportunities

- Ticket 1 test tasks T002-T004 can run in parallel.
- Shared Foundation value object and test tasks T006-T007 and T011-T012 can run in parallel.
- Ticket test tasks marked [P] can run in parallel across separate test files.
- Ticket 5 controller, admin, Drush, dashboard, and status compatibility refactors can split across SQL, Drush, dashboard, and status files once Ticket 4 settles DatasetInfo output.
- Tickets 6 and 7 can proceed in parallel once Ticket 2 resource registration behavior is stable.

## Parallel Example: Ticket 1

```bash
Task: "T002 [P] Add dataset discovery fixture variants in ../dkan/modules/dkan_metastore/tests/src/Functional/OnPreReferenceTest.php"
Task: "T003 [P] Add mixed-validity distribution fixtures in ../dkan/modules/dkan_metastore/tests/src/Functional/Api1/DistributionHandlingTest.php"
Task: "T004 [P] Add subscriber multi-URL contract coverage in ../dkan/modules/dkan_datastore/tests/src/Unit/EventSubscriber/DatastoreSubscriberTest.php"
```

## Parallel Example: Ticket 5

```bash
Task: "T028 [US2] Add SQL endpoint unit tests in ../dkan/modules/dkan_datastore/tests/src/Unit/SqlEndpoint/WebServiceApiTest.php"
Task: "T029 [US2] Add drush compatibility tests in ../dkan/modules/dkan_datastore/tests/src/Functional/Commands/DegradedModeCommandsTest.php"
Task: "T040 [US2] Add dashboard row unit tests in ../dkan/modules/dkan_datastore/tests/src/Unit/Form/DashboardFormTest.php"
```

## Parallel Example: Ticket 4 / Ticket 7

```bash
Task: "T041 [US3] Add DatasetInfo gathering tests in ../dkan/modules/dkan_common/tests/src/Kernel/DatasetInfoTest.php"
Task: "T042 [US3] Add orphan cleanup functional tests in ../dkan/modules/dkan_metastore/tests/src/Functional/OrphanCheckerTest.php"
Task: "T048 [US3] Integrate discovered-resource collection in ../dkan/modules/dkan_common/src/DatasetInfo.php"
```

## Implementation Strategy

### MVP First (Tickets 1-2)

1. Complete Shared Foundation.
2. Deliver Ticket 1 dataset-save discovery in the pre-reference lifecycle flow.
3. Deliver Ticket 2 registration, dispatch, logging, and summary reporting.
4. Validate the combined Ticket 1 and Ticket 2 path independently before taking compatibility/reporting refactors.

### Incremental Delivery

1. Deliver Tickets 1-2 for MVP dispatch trigger correctness.
2. Deliver Ticket 3 cache and invalidation updates.
3. Deliver Ticket 4 DatasetInfo integration before dashboard/reporting compatibility work.
4. Deliver Ticket 5 for compatibility, endpoint, admin continuity, post-import status, and status lookup updates.
5. Deliver Tickets 6-7 for cleanup and importer-regression follow-up work.
6. Deliver Ticket 8 migration guidance updates.
7. Finish with Release Gate Validation.

### Team Parallelization

1. One engineer on Shared Foundation plus Ticket 1 lifecycle/discovery work.
2. One engineer on Ticket 3 cache work, then Ticket 4 DatasetInfo integration, then Ticket 5 SQL, Drush, dashboard, and post-import compatibility work after Shared Foundation is ready.
3. One engineer on Tickets 6-7 cleanup and importer-regression work after Ticket 2 stabilizes the resource path.

## Notes

- [P] marks tasks that can run concurrently because they touch separate files with no direct dependency.
- [US1], [US2], and [US3] map each task to the corresponding user story in `spec.md`.
- Distribution IDs remain accepted only as compatibility metadata; resource/dataset context is the operational key path.
