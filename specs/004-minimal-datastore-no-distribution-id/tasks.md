# Tasks: Minimal Datastore Without Distribution ID Requirement

**Input**: Design documents from `specs/004-minimal-datastore-no-distribution-id/`
**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`, `contracts/`, `quickstart.md`

**Tests**: Included because the specification requires independently testable user stories and measurable outcomes for discovery, import initiation, and compatibility behavior.

**Path Note**: DKAN implementation paths below are relative to the DKAN repository root. Spec artifacts are relative to the `dkan-specs` repository root.

## Shared Foundation (Complete Before Ticket Work)

**Purpose**: Establish the shared contracts, service wiring, and planning notes that every quickstart ticket depends on.

**CRITICAL**: Complete this section before starting ticket implementation work.

- [x] T001 Create feature implementation notes and validation checklist in `specs/004-minimal-datastore-no-distribution-id/quickstart.md`
- [x] T005 Create resource discovery result value object in `modules/dkan_metastore/src/LifeCycle/ResourceDiscovery/ResourceDiscoveryResult.php`
- [x] T006 [P] Defer dispatch-subsystem cleanup and keep existing datastore initiation flow unchanged for this phase in `modules/dkan_datastore/src/Dispatch/DispatchItem.php`
- [x] T007 [P] Defer dispatch-subsystem cleanup and keep existing datastore initiation flow unchanged for this phase in `modules/dkan_datastore/src/Dispatch/`
- [x] T008 Implement dataset resource discovery service logic for top-level `$.distribution[]` `downloadURL` discovery, normalization into discovery candidates, and skip-reason capture in `modules/dkan_metastore/src/LifeCycle/ResourceDiscovery/DatasetResourceDiscovery.php`
- [x] T009 Defer initiation orchestration refactor and keep existing subscriber + datastore service flow unchanged for this phase in `modules/dkan_datastore/src/EventSubscriber/DatastoreSubscriber.php`
- [x] T010 Defer standalone dispatcher service wiring cleanup and keep current service definitions unchanged for this phase in `modules/dkan_datastore/dkan_datastore.services.yml`
- [x] T011 Add unit tests for discovery result object normalization and encounter ordering in `modules/dkan_metastore/tests/src/Unit/LifeCycle/ResourceDiscovery/ResourceDiscoveryResultTest.php`
- [x] T012 [P] Add unit tests for top-level `$.distribution[]` resource discovery and skip reasons in `modules/dkan_metastore/tests/src/Unit/LifeCycle/ResourceDiscovery/DatasetResourceDiscoveryTest.php`

**Checkpoint**: Shared contracts and services are in place; ticket work can proceed.

---

## Ticket 1: Dataset-Save Resource Discovery and Registration in LifeCycle Flow

**Goal**: Implement dataset-save resource discovery and registration in the dataset presave lifecycle path, decoupled from the metadata referencing workflow, using a reusable discovery contract.

**Independent Test**: Save datasets with referenced and non-referenced distribution structures and verify dataset-save discovery identifies valid `downloadURL` values and registers them (triggering datastore processing) while invalid/missing entries are skipped, regardless of whether distribution referencing is enabled; additionally verify `distribution[].describedBy` data-dictionary handling behaves equivalently in both modes.

### Tests for Ticket 1

- [x] T002 [P] [US1] Add dataset discovery fixture variants and lifecycle coverage for referenced/non-referenced distributions in `modules/dkan_metastore/tests/src/Functional/OnPreReferenceTest.php`
- [x] T003 [P] [US1] Add two-distribution fixture coverage where one baseline valid distribution is always present and the second distribution varies across valid/missing/invalid downloadURL cases for dataset presave discovery/registration coverage in `modules/dkan_metastore/tests/src/Kernel/LifeCycle/LifeCycleTest.php`
- [x] T066 [P] [US1] Add dual-mode (`distribution` vs `0`) `describedBy` data-dictionary API behavior coverage in `modules/dkan_metastore/tests/src/Functional/Api1/DistributionHandlingTest.php`
- [x] T004 [P] [US1] Add datastore subscriber fixture and contract coverage for multi-URL processing in `modules/dkan_datastore/tests/src/Unit/EventSubscriber/DatastoreSubscriberTest.php`

### Implementation for Ticket 1

- [x] T018 [US1] Decouple datastore import triggering from the referencing/pre-reference path so registration no longer depends on distribution referencing being enabled, and ensure the triggering-property "new revision" decision is determined before resource registration in `modules/dkan_datastore/src/EventSubscriber/DatastoreSubscriber.php`
- [x] T019 [US1] Add a dedicated dataset-save discovery + resource registration step in `LifeCycle::datasetPresave()` (before `referenceMetadata()`) that runs `DatasetResourceDiscovery` over top-level `$.distribution[]` and registers valid `downloadURL` values via `ResourceMapper`, independent of the referencing workflow, in `modules/dkan_metastore/src/LifeCycle/LifeCycle.php`
- [x] T064 [US1] Make resource registration single-owner so the referencer consumes already-registered resources instead of re-registering during referencing (avoiding duplicate registration and version churn) in `modules/dkan_metastore/src/Reference/Referencer.php`
- [x] T065 [US1] Ensure `distribution[].describedBy` data-dictionary URI validation/normalization is not gated by distribution referencing mode so non-referenced distributions retain existing behavior in `modules/dkan_metastore/src/Reference/Referencer.php`

**Checkpoint**: Dataset-save discovery and resource registration run from the dataset presave path, decoupled from referencing, hand off a shared discovery contract, and preserve `describedBy` data-dictionary behavior in referenced and non-referenced modes.

---

## Ticket 2: Data Dictionary Reference Compatibility

**Goal**: Ensure data-dictionary discovery and reference lookup work correctly across referenced and non-referenced distribution modes.

**Independent Test**: Save datasets with `describedBy` data-dictionary references in both referenced and non-referenced distribution structures and verify dictionary discovery and lookup behavior remains equivalent.

### Implementation for Ticket 2

- [x] T067 [P] Make `ReferenceLookup::getReferencers()` recursive to support nested property matching in metadata objects for compound datastore identifiers in `modules/dkan_metastore/src/Reference/ReferenceLookup.php`
- [x] T068 [P] Fix `DataDictionaryDiscoveryTest` with proper reflection and mock setup for `extractDictionaryId()` private method and URL/URI generator mappings in `modules/dkan_metastore/tests/src/Unit/DataDictionary/DataDictionaryDiscoveryTest.php`
- [x] T069 [P] Update dictionary-related test coverage for non-referenced distribution behavior in `modules/dkan_datastore/tests/src/Unit/Controller/QueryDownloadControllerTest.php`, `modules/dkan_metastore/tests/src/Unit/Enforcement/DictionaryEnforcerTest.php`, and `modules/dkan_datastore/tests/src/Kernel/Plugin/DatasetInfo/StrictModeOffDictionaryEnforcerTest.php`

**Checkpoint**: Data-dictionary discovery and reference lookup work correctly in both referenced and non-referenced distribution modes.

---

## Ticket 3: Cache Dependency and Invalidation Updates

**Goal**: Update cache dependency and invalidation behavior to operate from dataset/resource context rather than distribution lookups.

**Independent Test**: Exercise query, import summary, and download cache behavior with referenced and non-referenced distributions and verify invalidation stays correct.

### Tests for Ticket 3

- [x] T025 [P] [US2] Add unit tests for query controller dependency extraction without mandatory distribution IDs in `modules/dkan_datastore/tests/src/Unit/Controller/AbstractQueryControllerTest.php`
- [x] T026 [P] [US2] Add kernel tests for import summary cache dependencies using resource/dataset context in `modules/dkan_datastore/tests/src/Kernel/Controller/ImportControllerTest.php`

### Implementation for Ticket 3

- [x] T030 [US2] Update metastore dependency extraction to not require distribution lookup for cache tags/contexts in `modules/dkan_datastore/src/Controller/AbstractQueryController.php`
- [x] T031 [US2] Update query resource and dataset-resource execution paths to prioritize resource/dataset identifiers in `modules/dkan_datastore/src/Controller/AbstractQueryController.php`
- [x] T032 [US2] Refactor import summary dependency generation to support dataset+resource mapping invalidation in `modules/dkan_datastore/src/Controller/ImportController.php`
- [x] T059 [US2] Capture final cache strategy decisions for query/import dependency-tag invalidation and streaming download TTL/header behavior (no dependency-tag invalidation) in `specs/004-minimal-datastore-no-distribution-id/research.md`

**Checkpoint**: In-scope datastore cache surfaces invalidate correctly without requiring runtime distribution-ID lookup.

---

## Ticket 4: DatasetInfo Integration for Discovered Resources

**Goal**: Decide how discovered resources surface through dataset info and reporting views before dashboard compatibility work consumes that shape.

**Independent Test**: Gather dataset info for datasets with discovered resources and verify reporting surfaces render whichever data model is chosen.

### Tests for Ticket 4

- [x] T041 [P] [US3] Add kernel tests for `DatasetInfo` gathering of discovered resources in `modules/dkan_common/tests/src/Kernel/DatasetInfoTest.php`

### Implementation for Ticket 4

- [x] T048 [US3] Integrate discovered-resource collection into dataset info gathering for reporting surfaces in `modules/dkan_common/src/DatasetInfo.php`
- [x] T049 [US3] Update datastore dataset-info plugin integration for combined distribution/discovered resource display in `modules/dkan_datastore/src/Plugin/DatasetInfo/DatastoreInfo.php`

**Checkpoint**: Dataset info surfaces discovered resources in a way dashboard and reporting code can consume consistently.

**Status**: ✅ COMPLETE — Implemented on feat-non-ref-distro branch (PR #4748)

---

## Ticket 5: API, Drush, and Admin Compatibility Review

**Goal**: Preserve compatibility where practical while moving operational flows to dataset/resource identifiers.

**Independent Test**: Exercise query, Drush, dashboard/reporting, post-import status, and resource-mapper status lookup paths and verify distribution IDs are optional compatibility metadata rather than required runtime keys, using the DatasetInfo shape established in Ticket 4.

### Tests for Ticket 5

- [x] T040 [P] [US2] Add unit tests for dashboard row rendering of discovered URL-only resources when distribution IDs are absent in `modules/dkan_datastore/tests/src/Unit/Form/DashboardFormTest.php`

### Implementation for Ticket 5

- [x] T035 [US2] Confirm resource-identifier-first datastore Drush parameter handling and update help text to clarify distribution IDs are compatibility metadata (not operational keys) in `modules/dkan_datastore/src/Drush/Commands/DatastoreCommands.php`
- [x] T047 [US2] Update dashboard resource row builder to render discovered resources and status values without distribution IDs in `modules/dkan_datastore/src/Form/DashboardForm.php`
- [ ] T073 [US2] Update datastore API docs generation to remove hard distribution-ID assumptions or provide resource-ID alternatives in parameters/examples for query surfaces in `modules/dkan_datastore/src/Plugin/DkanApiDocs/DatastoreApiDocs.php`, `modules/dkan_metastore/src/Plugin/DkanApiDocs/MetastoreApiDocs.php`, and `modules/dkan_datastore/docs/openapi_spec.json`

**Checkpoint**: Compatibility-sensitive datastore entry points, post-import status flows, and status lookup surfaces continue to work without requiring distribution IDs as operational keys.

## Ticket 7: Cleanup and Orphan Behavior Updates

**Goal**: Remove obsolete mappings and datastore artifacts without depending solely on orphaned distribution references.

**Independent Test**: Run cleanup flows for referenced and non-referenced resources and verify obsolete mappings and artifacts are removed.

### Tests for Ticket 7

- [x] T042 [P] [US3] Add functional cleanup tests for referenced and non-referenced orphan resource paths in `modules/dkan_metastore/tests/src/Functional/OrphanCheckerTest.php`

### Implementation for Ticket 7

- [x] T050 [US3] Update orphan cleanup path to remove obsolete mappings/artifacts for non-referenced distribution mode, via lifecycle/subscriber/resource-purger flow in `modules/dkan_metastore/src/LifeCycle/LifeCycle.php`, `modules/dkan_metastore/src/LifeCycle/LifeCycleEvent.php`, `modules/dkan_metastore/src/EventSubscriber/MetastoreSubscriber.php`, and `modules/dkan_datastore/src/Service/ResourcePurger.php`

**Checkpoint**: Cleanup works for both referenced and non-referenced distribution workflows.

---

## Ticket 9: SQL Endpoint Deprecation

**Goal**: Deprecate the SQL endpoint and announce/document that it is deprecated and should not be used if distributions are unreferenced / non-referenced.

**Independent Test**: Verify deprecation annotations and `Deprecation`/`Link` HTTP response headers are in place on the SQL endpoint controller/service and documented in the datastore documentation.

### Implementation for Ticket 9

- [ ] T034 [US2] Add deprecation annotations and `Deprecation`/`Link` HTTP response headers to SQL endpoint service/controller in `modules/dkan_datastore/src/SqlEndpoint/WebServiceApi.php`
- [ ] T074 [US2] Document deprecation of the SQL endpoint and announce that it should not be used with unreferenced/non-referenced distributions in `docs/source/components/dkan_datastore.rst`

**Checkpoint**: SQL endpoint is deprecated with deprecation response headers (`Deprecation`, `Link`) and explicit documentation advising against its use with unreferenced/non-referenced distributions.

---

## Ticket 10: Optional Unreference-Datasets Migration and Guidance

**Goal**: Provide an optional executable migration path via `dkan:metastore:unreference-datasets` for referenced-to-embedded distribution conversion, and deliver separate documentation for caller compatibility guidance and content migration execution.

**Independent Test**: Run the optional migration command against representative datasets, verify referenced-to-embedded conversion correctness and idempotency, and confirm documentation covers both (a) caller operational-key transition behavior and (b) content migration runbook guidance outside the major-version upgrade guide.

### Tests for Ticket 10

- [x] T071 [P] [US3] Add functional Drush-command tests for `dkan:metastore:unreference-datasets` referenced-to-embedded distribution migration behavior and idempotency in `modules/dkan_metastore/tests/src/Functional/Drush/Commands/UnreferenceDatasetCommandsTest.php`

### Implementation for Ticket 10

- [x] T070 [US3] Implement the `UnreferenceDatasetCommands` Drush migration command to convert referenced distributions to embedded non-referenced distributions in `modules/dkan_metastore/src/Drush/Commands/UnreferenceDatasetCommands.php`
- [x] T072 [US3] Document concise content-migration guidance in `docs/source/components/dkan_metastore.rst` for referenced-to-embedded distribution conversion, including command purpose, `orphan` vs `--delete` behavior, and a cautionary note to test in staging with backup/rollback readiness.

**Checkpoint**: Maintainers can optionally run the supported, repeatable `dkan:metastore:unreference-datasets` migration action and follow complete migration guidance for compatibility and payload transitions.

---

## Release Gate Validation (After Desired Tickets Complete)

**Purpose**: Run the final feature validation commands after the intended ticket set is complete.

- [ ] T056 [P] Run metastore/datastore PHPUnit suites for feature validation and record outcomes in `specs/004-minimal-datastore-no-distribution-id/quickstart.md`
- [ ] T057 [P] Run PHPCS for touched datastore/metastore modules and record outcomes in `specs/004-minimal-datastore-no-distribution-id/quickstart.md`

---

## Dependencies & Execution Order

### Ticket Dependencies

- **Shared Foundation**: No dependency; blocks implementation work for all tickets.
- **Ticket 2**: Depends on Shared Foundation; dictionary behavior fixes are blocking prerequisites for Tickets 3-5.
- **Ticket 1**: Depends on Shared Foundation for implementation tasks T018-T019 and T064, while fixture tasks T002-T004 can start earlier.
- **Ticket 3**: Depends on Shared Foundation and Ticket 2 (dictionary fixes), then benefits from Ticket 1 resource registration behavior for end-to-end cache validation.
- **Ticket 4**: Depends on Shared Foundation and benefits from Ticket 1 resource registration behavior so DatasetInfo can surface discovered resources.
- **Ticket 5**: Depends on Shared Foundation and Ticket 4, because dashboard/reporting compatibility work consumes the DatasetInfo shape established there and the same compatibility pass now includes post-import status initialization and status lookup updates.
- **Ticket 9**: Depends on Shared Foundation; SQL endpoint work is on a separate maintenance track because the surface is barely-maintained. Can proceed in parallel with other tickets once Shared Foundation is ready.
- **Ticket 10**: Depends on Shared Foundation and should follow Ticket 5 compatibility work so migration execution and documentation align with finalized operational behavior.
- **Ticket 7**: Depends on Shared Foundation and benefits from Ticket 1 resource registration/dispatch behavior.
- **Release Gate Validation**: Depends on completion of the intended ticket set.

### Within Each Ticket

- Tests are listed first and should fail before implementation.
- Service/value object contracts precede controller/form integrations.
- Runtime behavior changes precede documentation finalization.
- Complete each ticket checkpoint before moving to the next dependent ticket.

## Parallel Opportunities

- Ticket 1 test tasks T002-T004 can run in parallel.
- Shared Foundation test tasks T011-T012 can run in parallel.
- Ticket test tasks marked [P] can run in parallel across separate test files.
- Ticket 5 controller, admin, Drush, dashboard, and status compatibility refactors can split across Drush, dashboard, and status files once Ticket 4 settles DatasetInfo output.
- Ticket 7 can proceed once Ticket 2 resource registration behavior is stable.

## Parallel Example: Ticket 1

```bash
Task: "T002 [P] Add dataset discovery fixture variants in modules/dkan_metastore/tests/src/Functional/OnPreReferenceTest.php"
Task: "T003 [P] Add mixed-validity distribution fixtures in modules/dkan_metastore/tests/src/Kernel/LifeCycle/LifeCycleTest.php"
Task: "T004 [P] Add subscriber multi-URL contract coverage in modules/dkan_datastore/tests/src/Unit/EventSubscriber/DatastoreSubscriberTest.php"
```

## Parallel Example: Ticket 4 / Ticket 7

```bash
Task: "T041 [US3] Add DatasetInfo gathering tests in modules/dkan_common/tests/src/Kernel/DatasetInfoTest.php"
Task: "T042 [US3] Add orphan cleanup functional tests in modules/dkan_metastore/tests/src/Functional/OrphanCheckerTest.php"
Task: "T048 [US3] Integrate discovered-resource collection in modules/dkan_common/src/DatasetInfo.php"
```

## Implementation Strategy

### MVP First (Tickets 1-2)

1. Complete Shared Foundation.
2. Deliver Ticket 1 dataset-save discovery and registration in the dataset presave lifecycle flow.
3. Deliver Ticket 2 registration and initiation.
4. Validate the combined Ticket 1 and Ticket 2 path independently before taking compatibility refactors.

### Incremental Delivery

1. Deliver Tickets 1-2 for MVP dispatch trigger correctness.
2. Deliver Ticket 3 cache and invalidation updates.
3. Deliver Ticket 4 DatasetInfo integration before dashboard/reporting compatibility work.
4. Deliver Ticket 5 for compatibility, endpoint, admin continuity, post-import status, and status lookup updates.
5. Deliver Ticket 7 for cleanup and orphan-resource follow-up work.
6. Deliver Ticket 10 migration action and migration guidance updates.
7. Finish with Release Gate Validation.

### Team Parallelization

1. One engineer on Shared Foundation plus Ticket 1 lifecycle/discovery work.
2. One engineer on Ticket 3 cache work, then Ticket 4 DatasetInfo integration, then Ticket 5 SQL, Drush, dashboard, and post-import compatibility work after Shared Foundation is ready.
3. One engineer on Ticket 7 cleanup follow-up work after Ticket 2 stabilizes the resource path.

## Notes

- [P] marks tasks that can run concurrently because they touch separate files with no direct dependency.
- [US1], [US2], and [US3] map each task to the corresponding user story in `spec.md`.
- Distribution IDs remain accepted only as compatibility metadata; resource/dataset context is the operational key path.
- T015 was intentionally removed because `ResourceMapper` and `ResourceMapperTest` are already decoupled from distribution lifecycle concerns; lifecycle behavior is covered in datastore/metastore integration tests.
- T016 was intentionally removed because mixed-validity discovery/continuation behavior is already covered in lifecycle/subscriber tests; `DistributionHandlingTest` remains focused on API contract behavior (not duplicated workflow coverage).
- T017 was intentionally removed because testing queue-driven versus immediate datastore initiation behavior is out of scope for Ticket 2 and is better covered by the lifecycle behavior and subscriber unit tests from Ticket 1.
- T020 and T024 (original Ticket 2 implementation tasks) are now considered complete and merged into the broader discovery/initiation flow covered by Ticket 1; Ticket 2 has been repurposed for data dictionary reference compatibility fixes.
- T024 is considered complete: `DatastoreSubscriber` no longer performs direct runtime distribution entity dereference for datastore initiation, and remaining coupling is event ordering via `EVENT_PRE_REFERENCE` rather than distribution lookup.
- T020 is considered complete for initiation behavior: best-effort per-URL datastore initiation from `ResourceMapper::EVENT_REGISTRATION` is in place.
- T027 was intentionally removed because streaming download responses are TTL/header-cached and do not participate in dependency-tag invalidation behavior.
- T029 was intentionally removed because `DegradedModeCommands` toggles degraded query mode state only and does not touch distribution-ID compatibility behavior.
- T033 was intentionally removed because `QueryDownloadController` returns streamed responses with cache headers and does not use dependency-tag invalidation like JSON endpoints.
- T046 was intentionally removed because `ImportInfoList`/`ImportInfo` already operate on resource identifiers and job-store state, with no distribution-node lookup dependency to refactor.
- T045 was intentionally removed because post-import status initialization already consumes resource/dataset information from `DatasetInfo` output and does not require distribution node references.
- T036 was intentionally removed because `ReimportCommands` already operates correctly in non-referenced mode and does not depend on distribution-node/UUID-specific logic.
- T038 was intentionally removed because `PostImportResultFactory` already initializes status retrieval from distribution entries via `resource_id`, and this path does not require distribution-node/UUID-specific behavior changes.
- T039 was intentionally removed because `ImportInfo` status lookup is already resource-mapping based and exercised by existing `ImportInfoTest` coverage without requiring distribution references.
- Ticket 8 (T044/T052/T053) was removed because importer hook payload changes are no longer required for this scope; `ImportService` distribution handling retained for cache-tag invalidation behavior.
- T037 was intentionally removed because caller query parameters and examples are already covered by API docs generation in T073, while migration runbook guidance is covered in T072.
- T070 and T071 were moved into Ticket 10 so referenced-to-embedded distribution migration execution is tracked together with migration guidance deliverables.
- T028 was intentionally removed because Ticket 9 scope was reduced from refactoring SQL endpoint cache-tag handling to deprecating the endpoint and documenting that it should not be used with unreferenced/non-referenced distributions.
- T055 was intentionally removed because tracking an arbitrary percentage-based compatibility inventory was deemed unnecessary.
- T060 was intentionally removed because Ticket 10 implementation and runbook documentation are complete without requiring an administrative risks section in tasks.md.
- Ticket 6 (T043/T051) was intentionally removed because the non-referenced-distribution refactor did not modify importer factory/stage-override behavior, and existing dual-mode `DatasetBTBTest` coverage (including `MySqlDatasetBTBTest` inheritance) already validates end-to-end importer behavior across both distribution modes.
- T061 was intentionally removed because adding functional test coverage for `DashboardForm` is out of scope for this round of work. The actually changes to `DatasetForm` are minimal to non-existant, and we did add some unit test coverage to capture side-effects from changes to DatasetInfo.

- T042/T050 implementation evidence: `orphan_resource_remover` is now effectively dead code for this flow (no active queue producer in the current branch path), and orphan cleanup is handled through lifecycle/subscriber/resource-purger orchestration. Branch changes in `MetastoreSubscriber`, `LifeCycle`, `LifeCycleEvent`, and `ResourcePurger` implement non-referenced orphan cleanup behavior without requiring orphaned distribution-reference queue processing. Functional confirmation is covered by `OrphanCheckerTest::testOrphanedResourceCleanup()` and draft workflow coverage in `tests/src/Functional/DatasetBTBTest.php`.