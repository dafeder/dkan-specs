# Tasks: Minimal Datastore Without Distribution ID Requirement

**Input**: Design documents from `specs/004-minimal-datastore-no-distribution-id/`
**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`, `contracts/`, `quickstart.md`

**Tests**: Included because the specification requires independently testable user stories and measurable outcomes for discovery, import initiation, compatibility, and reporting behavior.

**Path Note**: DKAN implementation paths below are relative to the DKAN repository root. Spec artifacts are relative to the `dkan-specs` repository root.

## Shared Foundation (Complete Before Ticket Work)

**Purpose**: Establish the shared contracts, service wiring, and planning notes that every quickstart ticket depends on.

**CRITICAL**: Complete this section before starting ticket implementation work.

- [x] T001 Create feature implementation notes and validation checklist in `specs/004-minimal-datastore-no-distribution-id/quickstart.md`
- [x] T005 Create resource discovery result value object in `modules/dkan_metastore/src/LifeCycle/ResourceDiscovery/ResourceDiscoveryResult.php`
- [x] T006 [P] Defer dispatch-subsystem cleanup and keep existing datastore initiation flow unchanged for this phase in `modules/dkan_datastore/src/Dispatch/DispatchItem.php`
- [x] T007 [P] Defer dispatch-subsystem cleanup and keep existing datastore initiation flow unchanged for this phase in `modules/dkan_datastore/src/Dispatch/DispatchResult.php`
- [x] T008 Implement dataset resource discovery service logic for top-level `$.distribution[]` `downloadURL` discovery, normalization into discovery candidates, and skip-reason capture in `modules/dkan_metastore/src/LifeCycle/ResourceDiscovery/DatasetResourceDiscovery.php`
- [x] T009 Defer initiation orchestration refactor and keep existing subscriber + datastore service flow unchanged for this phase in `modules/dkan_datastore/src/EventSubscriber/DatastoreSubscriber.php`
- [x] T010 Defer standalone dispatcher service wiring cleanup and keep current service definitions unchanged for this phase in `modules/dkan_datastore/dkan_datastore.services.yml`
- [x] T011 Add unit tests for discovery result object normalization and encounter ordering in `modules/dkan_metastore/tests/src/Unit/LifeCycle/ResourceDiscovery/ResourceDiscoveryResultTest.php`
- [x] T012 [P] Add unit tests for top-level `$.distribution[]` resource discovery and skip reasons in `modules/dkan_metastore/tests/src/Unit/LifeCycle/ResourceDiscovery/DatasetResourceDiscoveryTest.php`

**Checkpoint**: Shared contracts and services are in place; ticket work can proceed.

---

## Ticket 1: Dataset-Save Resource Discovery and Registration in LifeCycle Flow

**Goal**: Implement dataset-save resource discovery and registration in the dataset presave lifecycle path, decoupled from the metadata referencing workflow, using a reusable discovery contract.

**Independent Test**: Save datasets with referenced and non-referenced distribution structures and verify dataset-save discovery identifies valid `downloadURL` values and registers them (triggering datastore processing) while invalid/missing entries are skipped and reported, regardless of whether distribution referencing is enabled; additionally verify `distribution[].describedBy` data-dictionary handling behaves equivalently in both modes.

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

- [ ] T040 [P] [US2] Add unit tests for dashboard row rendering of discovered URL-only resources when distribution IDs are absent in `modules/dkan_datastore/tests/src/Unit/Form/DashboardFormTest.php`
- [ ] T038 [P] [US3] Add unit tests for `PostImportResultFactory` initialization from distribution UUID and dataset+resource_url inputs in `modules/dkan_datastore/tests/src/Unit/Service/PostImportResultTest.php`
- [ ] T039 [P] [US3] Add kernel tests for import status lookup from resource mapping without distribution references in `modules/dkan_datastore/tests/src/Kernel/Service/Info/ImportInfoTest.php`
- [ ] T061 [P] [US2] Add functional dashboard continuity test verifying resource rows and status render without distribution IDs in `modules/dkan_datastore/tests/src/Functional/Form/DashboardFormNoDistributionIdTest.php`

### Implementation for Ticket 5

- [ ] T035 [US2] Update datastore drush command parameter handling and help text for compatibility-only distribution IDs in `modules/dkan_datastore/src/Drush/Commands/DatastoreCommands.php`
- [x] T047 [US2] Update dashboard resource row builder to render discovered resources and status values without distribution IDs in `modules/dkan_datastore/src/Form/DashboardForm.php`

**Checkpoint**: Compatibility-sensitive datastore entry points, post-import status flows, and status lookup surfaces continue to work without requiring distribution IDs as operational keys.

---

## Ticket 6: Importer Modularity Regression Assurance

**Goal**: Preserve the single active importer model with stage-level override and fallback behavior.

**Independent Test**: Configure stage overrides and verify overridden stages run while non-overridden stages keep default behavior.

### Tests for Ticket 6

- [ ] T043 [P] [US3] Add kernel tests proving importer stage override/fallback behavior remains intact after refactor in `modules/dkan_datastore/tests/src/Kernel/Service/ImportServiceEventsTest.php`

### Implementation for Ticket 6

- [ ] T051 [US3] Preserve single active importer with stage override fallback behavior in post-refactor import factory/service flow in `modules/dkan_datastore/src/Service/Factory/ImportServiceFactory.php`

**Checkpoint**: Importer override and fallback behavior remains unchanged after the refactor.

---

## Ticket 7: Cleanup and Orphan Behavior Updates

**Goal**: Remove obsolete mappings and datastore artifacts without depending solely on orphaned distribution references.

**Independent Test**: Run cleanup flows for referenced and non-referenced resources and verify obsolete mappings and artifacts are removed.

### Tests for Ticket 7

- [ ] T042 [P] [US3] Add functional cleanup tests for referenced and non-referenced orphan resource paths in `modules/dkan_metastore/tests/src/Functional/OrphanCheckerTest.php`

### Implementation for Ticket 7

- [ ] T050 [US3] Update orphan cleanup path to remove obsolete mappings/artifacts without requiring orphaned distribution references in `modules/dkan_metastore/src/Plugin/QueueWorker/OrphanResourceRemover.php`

**Checkpoint**: Cleanup works for both referenced and non-referenced distribution workflows.

---

## Ticket 8: Migration Guidance and Custom Importer Hook Updates

**Goal**: Capture the required migration steps for changed payloads and compatibility behavior.

**Independent Test**: Review the migration guidance against changed hook payloads and compatibility behavior and verify each required update is documented.

### Tests for Ticket 8

- [ ] T044 [P] [US3] Add unit tests validating removal of legacy distribution-ID hook payload fields in `modules/dkan_datastore/tests/src/Unit/Service/PostImportResultTest.php`

### Implementation for Ticket 8

- [ ] T052 [US3] Remove legacy distribution-ID hook payload fields from importer event payload shaping in `modules/dkan_datastore/src/Service/ImportService.php`
- [ ] T053 [US3] Update custom importer hook payload contract documentation for dataset/downloadURL/resource context in `docs/source/components/dkan_datastore.rst`
- [ ] T037 [US3] Document compatibility behavior and migration notes for changed operational keys in `docs/source/upgrade.rst`
- [ ] T055 [P] [US3] Capture pre/post compatibility inventory and verify >=80% source-compatible datastore surfaces in `specs/004-minimal-datastore-no-distribution-id/quickstart.md`
- [ ] T058 [P] [US3] Add migration examples for legacy distribution-ID callers and updated importer hook payloads in `docs/source/upgrade.rst`
- [ ] T060 [US3] Update completion checklist and residual risks in `specs/004-minimal-datastore-no-distribution-id/tasks.md`

**Checkpoint**: Migration expectations are documented for downstream maintainers and importer implementers.

---

## Ticket 9: SQL Endpoint Compatibility (Separate Maintenance Track)

**Goal**: Update the SQL endpoint to work with resource-based cache dependencies instead of mandatory distribution IDs, recognizing that this surface is barely-maintained and may require careful review.

**Independent Test**: Execute SQL endpoint queries with resource identifiers and verify cache dependencies are correctly collected without requiring distribution ID lookups.

### Tests for Ticket 9

- [ ] T028 [P] [US2] Add unit tests for SQL endpoint behavior with resource-based execution and compatibility inputs in `modules/dkan_datastore/tests/src/Unit/SqlEndpoint/WebServiceApiTest.php`

### Implementation for Ticket 9

- [ ] T034 [US2] Update SQL endpoint service/controller to avoid distribution-ID operational assumptions in `modules/dkan_datastore/src/SqlEndpoint/WebServiceApi.php`

**Checkpoint**: SQL endpoint accepts resource identifiers and generates correct cache dependencies.

---

## Ticket 10: Referenced-to-Embedded Distribution Migration Action

**Goal**: Provide an executable migration path to convert referenced distributions into embedded non-referenced distributions for operators who choose that model.

**Independent Test**: Run the migration command against representative datasets, verify metadata conversion correctness and idempotency, and confirm datastore behaviors remain valid after conversion.

### Tests for Ticket 10

- [ ] T071 [P] [US3] Add functional Drush-command tests for referenced-to-embedded distribution migration behavior and idempotency in `modules/dkan_datastore/tests/src/Functional/Commands/DatastoreCommandsTest.php`

### Implementation for Ticket 10

- [ ] T070 [US3] Implement a Drush migration command to convert referenced distributions to embedded non-referenced distributions with dry-run and scoped execution options in `modules/dkan_datastore/src/Drush/Commands/DatastoreCommands.php`

**Checkpoint**: Operators can run a supported, repeatable migration action to convert referenced distributions to embedded non-referenced distributions.

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
- **Ticket 10**: Depends on Shared Foundation and should follow Ticket 5 compatibility work so migration execution aligns with finalized operational behavior.
- **Tickets 6-8**: Depend on Shared Foundation; Tickets 6-7 benefit from Ticket 1 resource registration/dispatch behavior, and Ticket 8 should land after payload changes and migration action behavior are settled.
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
- Ticket 5 controller, admin, Drush, dashboard, and status compatibility refactors can split across SQL, Drush, dashboard, and status files once Ticket 4 settles DatasetInfo output.
- Tickets 6 and 7 can proceed in parallel once Ticket 2 resource registration behavior is stable.

## Parallel Example: Ticket 1

```bash
Task: "T002 [P] Add dataset discovery fixture variants in modules/dkan_metastore/tests/src/Functional/OnPreReferenceTest.php"
Task: "T003 [P] Add mixed-validity distribution fixtures in modules/dkan_metastore/tests/src/Kernel/LifeCycle/LifeCycleTest.php"
Task: "T004 [P] Add subscriber multi-URL contract coverage in modules/dkan_datastore/tests/src/Unit/EventSubscriber/DatastoreSubscriberTest.php"
```

## Parallel Example: Ticket 5

```bash
Task: "T028 [US2] Add SQL endpoint unit tests in modules/dkan_datastore/tests/src/Unit/SqlEndpoint/WebServiceApiTest.php"
Task: "T040 [US2] Add dashboard row unit tests in modules/dkan_datastore/tests/src/Unit/Form/DashboardFormTest.php"
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
3. Deliver Ticket 2 registration, initiation, and summary reporting.
4. Validate the combined Ticket 1 and Ticket 2 path independently before taking compatibility/reporting refactors.

### Incremental Delivery

1. Deliver Tickets 1-2 for MVP dispatch trigger correctness.
2. Deliver Ticket 3 cache and invalidation updates.
3. Deliver Ticket 4 DatasetInfo integration before dashboard/reporting compatibility work.
4. Deliver Ticket 5 for compatibility, endpoint, admin continuity, post-import status, and status lookup updates.
5. Deliver Tickets 6-7 for cleanup and importer-regression follow-up work.
6. Deliver Ticket 10 migration action for referenced-to-embedded distribution conversion.
7. Deliver Ticket 8 migration guidance updates.
8. Finish with Release Gate Validation.

### Team Parallelization

1. One engineer on Shared Foundation plus Ticket 1 lifecycle/discovery work.
2. One engineer on Ticket 3 cache work, then Ticket 4 DatasetInfo integration, then Ticket 5 SQL, Drush, dashboard, and post-import compatibility work after Shared Foundation is ready.
3. One engineer on Tickets 6-7 cleanup and importer-regression work after Ticket 2 stabilizes the resource path.

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
- T037 was moved from Ticket 5 to Ticket 8 so migration/upgrade documentation is grouped with migration guidance and custom importer payload updates.
- T070 and T071 were moved into Ticket 10 so referenced-to-embedded distribution migration execution is tracked as a standalone deliverable rather than a documentation-only concern.
- T062 was intentionally removed because the `DispatchResult` class/logic was eliminated as redundant with existing datastore initiation and status/reporting mechanisms; no replacement task is required.
- T063 was intentionally removed because it only validated `DispatchResult` shape/count behavior, which is no longer part of the implementation after `DispatchResult` removal.
- T054 was intentionally removed because it tested dispatch summary counter accuracy which depended on the eliminated `DispatchResult` class; no replacement task is required.
