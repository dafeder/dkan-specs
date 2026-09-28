# Tasks: Module-Based Schemas With YAML Declarations

**Input**: Design documents from `specs/005-module-yaml-schemas/`
**Prerequisites**: [plan.md](plan.md), [spec.md](spec.md), [research.md](research.md), [data-model.md](data-model.md), [contracts/](contracts/), [quickstart.md](quickstart.md)

**Tests**: Included because the feature specification explicitly requires automated coverage for declared references, datastore triggers, duplicate selection, invalid declarations, UI schema retrieval, legacy override behavior, and API compatibility.

**Path Note**: DKAN implementation paths below are relative to `/Users/dan.feder/Work/dkan`. Spec artifacts are relative to `/Users/dan.feder/Work/dkan-specs`.

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Create reusable test fixtures and documentation anchors needed by all stories.

- [ ] T001 Create schema declaration test fixture module metadata in `../dkan/modules/dkan_metastore/tests/modules/dkan_test_schema_provider/dkan_test_schema_provider.info.yml`
- [ ] T002 [P] Create default fixture declaration in `../dkan/modules/dkan_metastore/tests/modules/dkan_test_schema_provider/dkan_test_schema_provider.schemas.yml`
- [ ] T003 [P] Create duplicate fixture module metadata in `../dkan/modules/dkan_metastore/tests/modules/dkan_test_schema_override/dkan_test_schema_override.info.yml`
- [ ] T004 [P] Create invalid fixture module metadata in `../dkan/modules/dkan_metastore/tests/modules/dkan_test_schema_invalid/dkan_test_schema_invalid.info.yml`
- [ ] T005 [P] Add fixture schema JSON files in `../dkan/modules/dkan_metastore/tests/modules/dkan_test_schema_provider/schema/collections/dataset.json`
- [ ] T006 [P] Add fixture UI schema JSON file in `../dkan/modules/dkan_metastore/tests/modules/dkan_test_schema_provider/schema/collections/dataset.ui.json`
- [ ] T007 [P] Add default declaration documentation stub in `../dkan/docs/source/developer-guide/schema_declarations.rst`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Define shared declaration models and service wiring that all user stories depend on.

**CRITICAL**: No user story work can begin until this phase is complete.

- [ ] T008 Create schema declaration value object in `../dkan/modules/dkan_metastore/src/Schema/SchemaDeclaration.php`
- [ ] T009 [P] Create schema source value object in `../dkan/modules/dkan_metastore/src/Schema/SchemaSource.php`
- [ ] T010 [P] Create reference definition value object in `../dkan/modules/dkan_metastore/src/Schema/ReferenceDefinition.php`
- [ ] T011 [P] Create behavior definition value object in `../dkan/modules/dkan_metastore/src/Schema/BehaviorDefinition.php`
- [ ] T012 Create schema registry result object in `../dkan/modules/dkan_metastore/src/Schema/SchemaRegistry.php`
- [ ] T013 Create declaration warning result object in `../dkan/modules/dkan_metastore/src/Schema/SchemaDeclarationWarning.php`
- [ ] T014 Register schema declaration services in `../dkan/modules/dkan_metastore/dkan_metastore.services.yml`
- [ ] T015 Update service constructor compatibility for schema registry consumers in `../dkan/modules/dkan_metastore/src/SchemaRetriever.php`

**Checkpoint**: Declaration model and service skeletons exist; user story implementation can proceed.

---

## Phase 3: User Story 1 - Discover Schemas From Enabled Modules (Priority: P1) MVP

**Goal**: Enabled modules declare schemas in `MODULE.schemas.yml`; active schema lookup no longer depends on unrelated filesystem JSON files when `docroot/schema/collections` is absent.

**Independent Test**: Enable a fixture module with `MODULE.schemas.yml`, verify declared schemas appear in metastore schema lookup, disable the module, and verify those schemas no longer participate. Also verify `docroot/schema/collections` overrides module declarations when present.

### Tests for User Story 1

- [ ] T016 [P] [US1] Add unit tests for declaration parsing and module-relative schema paths in `../dkan/modules/dkan_metastore/tests/src/Unit/Schema/SchemaDeclarationDiscoveryTest.php`
- [ ] T017 [P] [US1] Add unit tests for declaration validation warnings and invalid declaration exclusion in `../dkan/modules/dkan_metastore/tests/src/Unit/Schema/SchemaDeclarationValidatorTest.php`
- [ ] T018 [P] [US1] Add unit tests for duplicate schema selection by weight and Drupal module order in `../dkan/modules/dkan_metastore/tests/src/Unit/Schema/SchemaDeclarationResolverTest.php`
- [ ] T019 [P] [US1] Add kernel tests for enabled and disabled module declaration discovery in `../dkan/modules/dkan_metastore/tests/src/Kernel/Schema/SchemaDeclarationDiscoveryKernelTest.php`
- [ ] T020 [P] [US1] Add functional API tests for `/api/1/metastore/schemas` declaration-backed responses in `../dkan/modules/dkan_metastore/tests/src/Functional/Api1/SchemaDeclarationApiTest.php`
- [ ] T021 [P] [US1] Add functional tests for legacy `docroot/schema/collections` override behavior in `../dkan/modules/dkan_metastore/tests/src/Functional/Api1/LegacySchemaOverrideTest.php`

### Implementation for User Story 1

- [ ] T022 [US1] Implement enabled module declaration discovery in `../dkan/modules/dkan_metastore/src/Schema/SchemaDeclarationDiscovery.php`
- [ ] T023 [US1] Implement declaration YAML parsing and module-relative path normalization in `../dkan/modules/dkan_metastore/src/Schema/SchemaDeclarationDiscovery.php`
- [ ] T024 [US1] Implement declaration validation and invalid declaration warning collection in `../dkan/modules/dkan_metastore/src/Schema/SchemaDeclarationValidator.php`
- [ ] T025 [US1] Implement duplicate declaration resolution by `weight` and Drupal module order in `../dkan/modules/dkan_metastore/src/Schema/SchemaDeclarationResolver.php`
- [ ] T026 [US1] Implement legacy `docroot/schema/collections` source-mode override detection in `../dkan/modules/dkan_metastore/src/Schema/SchemaRegistryFactory.php`
- [ ] T027 [US1] Replace hard-coded active schema IDs with registry-backed IDs in `../dkan/modules/dkan_metastore/src/SchemaRetriever.php`
- [ ] T028 [US1] Update validation schema retrieval to read active declaration schema content in `../dkan/modules/dkan_metastore/src/SchemaRetriever.php`
- [ ] T029 [US1] Update metastore schema listing to use registry-backed IDs in `../dkan/modules/dkan_metastore/src/MetastoreService.php`
- [ ] T030 [US1] Update schema API docs generation for declaration-backed schema IDs in `../dkan/modules/dkan_metastore/src/Plugin/DkanApiDocs/MetastoreApiDocs.php`
- [ ] T031 [US1] Add administrator source and selection report service in `../dkan/modules/dkan_metastore/src/Schema/SchemaSelectionReporter.php`
- [ ] T032 [US1] Add admin or route wiring for schema selection reporting in `../dkan/modules/dkan_metastore/dkan_metastore.routing.yml`

**Checkpoint**: User Story 1 is independently functional: module declarations discover active schemas, duplicates are deterministic, invalid declarations warn without blocking valid ones, schema APIs work, and legacy filesystem override wins when present.

---

## Phase 4: User Story 2 - Configure References and Behaviors Declaratively (Priority: P2)

**Goal**: Schema declarations define reference rules and property behaviors (`import_trigger`, `resource_mapper`, `data_dictionary`) so metastore and datastore behavior no longer depends on property-name-only conditionals or admin trigger config.

**Independent Test**: Declare dataset references, a `resource_mapper` behavior on a distribution download property, an `import_trigger` behavior, and a `data_dictionary` behavior; save/update metadata and verify reference handling, datastore import triggering, resource mapping, and data-dictionary resolution all follow those declarations.

### Tests for User Story 2

- [ ] T033 [P] [US2] Add unit tests for reference definition parsing in `../dkan/modules/dkan_metastore/tests/src/Unit/Schema/ReferenceDefinitionTest.php`
- [ ] T034 [P] [US2] Add kernel tests for declaration-backed dataset-to-distribution references in `../dkan/modules/dkan_metastore/tests/src/Kernel/Reference/DeclarationReferencerTest.php`
- [ ] T035 [P] [US2] Add functional tests for `resource_mapper` behavior resource registration from declaration metadata in `../dkan/modules/dkan_metastore/tests/src/Functional/Api1/DeclarationDistributionHandlingTest.php`
- [ ] T036 [P] [US2] Add unit tests for behavior definition parsing (`import_trigger`, `resource_mapper`, `data_dictionary`) in `../dkan/modules/dkan_datastore/tests/src/Unit/Schema/BehaviorDefinitionTest.php`
- [ ] T037 [P] [US2] Add unit tests for declaration-backed `import_trigger` datastore update triggering in `../dkan/modules/dkan_datastore/tests/src/Unit/EventSubscriber/DeclarationDatastoreSubscriberTest.php`
- [ ] T038 [P] [US2] Add kernel tests proving trigger settings config is no longer authoritative in `../dkan/modules/dkan_datastore/tests/src/Kernel/DeclarationDatastoreTriggerTest.php`
- [ ] T039 [P] [US2] Add unit tests for data-dictionary resolution from the `data_dictionary` behavior in `../dkan/modules/dkan_metastore/tests/src/Unit/Schema/DataDictionaryResolverTest.php`
- [ ] T040 [P] [US2] Add kernel tests proving data-dictionary resolution is deterministic and errors actionably when no property declares `data_dictionary` in `../dkan/modules/dkan_metastore/tests/src/Kernel/Schema/DataDictionaryResolverKernelTest.php`

### Implementation for User Story 2

- [ ] T041 [US2] Add reference and behavior definition access methods to schema registry in `../dkan/modules/dkan_metastore/src/Schema/SchemaRegistry.php`
- [ ] T042 [US2] Update reference property discovery to use declaration reference definitions in `../dkan/modules/dkan_metastore/src/Reference/HelperTrait.php`
- [ ] T043 [US2] Update reference processing for declared schema and identifier references in `../dkan/modules/dkan_metastore/src/Reference/Referencer.php`
- [ ] T044 [US2] Update resource handling for the declared `resource_mapper` behavior in `../dkan/modules/dkan_metastore/src/Reference/Referencer.php`
- [ ] T045 [US2] Update dereference behavior to use declaration reference definitions in `../dkan/modules/dkan_metastore/src/Reference/Dereferencer.php`
- [ ] T046 [US2] Update reference cache dependency traversal for declaration-backed references in `../dkan/modules/dkan_metastore/src/MetastoreApiResponse.php`
- [ ] T047 [US2] Add behavior definition lookup service for datastore/metastore consumers in `../dkan/modules/dkan_metastore/src/Schema/SchemaBehaviorProvider.php`
- [ ] T048 [US2] Implement `DataDictionaryResolver` service resolving the effective data dictionary from the property declaring `data_dictionary` in the active schema in `../dkan/modules/dkan_metastore/src/Schema/DataDictionaryResolver.php`
- [ ] T049 [US2] Register `DataDictionaryResolver` service in `../dkan/modules/dkan_metastore/dkan_metastore.services.yml`
- [ ] T050 [US2] Update datastore pre-reference `import_trigger` checks to use declaration behavior definitions in `../dkan/modules/dkan_datastore/src/EventSubscriber/DatastoreSubscriber.php`
- [ ] T051 [US2] Remove, disable, or supersede datastore triggering property checkboxes in `../dkan/modules/dkan_datastore/src/Form/DatastoreSettingsForm.php`
- [ ] T052 [US2] Remove, disable, or supersede the metastore reference-field-selection admin form/route in `../dkan/modules/dkan_metastore/src/Form/MetastoreSettingsForm.php`
- [ ] T053 [US2] Update datastore settings config schema for deprecated trigger config handling in `../dkan/modules/dkan_datastore/config/schema/dkan_datastore.schema.yml`

**Checkpoint**: User Story 2 is independently functional on top of US1: declared references and behaviors (`import_trigger`, `resource_mapper`, `data_dictionary`) drive metastore/datastore behavior and data-dictionary resolution, and conflicting admin configuration is not authoritative.

---

## Phase 5: User Story 3 - Separate Validation Schemas From UI Schemas (Priority: P3)

**Goal**: Validation schema lookup and UI schema lookup are separate declaration-backed concerns, so forms can use optional UI schema content without changing API validation.

**Independent Test**: Declare both validation and UI schema for `dataset`; verify metadata validation uses only validation schema, form-building uses UI schema, and omitted UI schema falls back gracefully.

### Tests for User Story 3

- [ ] T054 [P] [US3] Add unit tests for UI schema source parsing and missing UI schema fallback in `../dkan/modules/dkan_metastore/tests/src/Unit/Schema/UiSchemaDeclarationTest.php`
- [ ] T055 [P] [US3] Add unit tests for validation schema lookup remaining separate from UI schema lookup in `../dkan/modules/dkan_metastore/tests/src/Unit/SchemaRetrieverTest.php`
- [ ] T056 [P] [US3] Add kernel tests for `SchemaPropertiesHelper` with declaration-backed validation schema content in `../dkan/modules/dkan_metastore/tests/src/Kernel/SchemaPropertiesHelperDeclarationTest.php`
- [ ] T057 [P] [US3] Add functional widget tests for declared UI schema usage in `../dkan/modules/dkan_metastore/tests/src/Functional/Plugin/Field/FieldWidget/DeclarationUiSchemaTest.php`

### Implementation for User Story 3

- [ ] T058 [US3] Add UI schema retrieval methods to schema registry in `../dkan/modules/dkan_metastore/src/Schema/SchemaRegistry.php`
- [ ] T059 [US3] Add explicit UI schema retrieval API to schema retriever in `../dkan/modules/dkan_metastore/src/SchemaRetriever.php`
- [ ] T060 [US3] Update schema properties helper to avoid UI schema assumptions in `../dkan/modules/dkan_metastore/src/SchemaPropertiesHelper.php`
- [ ] T061 [US3] Update JSON form option source to request declared UI schema content in `../dkan/modules/dkan_metastore/src/Plugin/JsonFormOptionSource/MetastoreSchema.php`
- [ ] T062 [US3] Update DKAN JSON form widget integration for missing UI schema fallback in `../dkan/modules/dkan_metastore/src/Plugin/Field/FieldWidget/DkanJsonFormWidget.php`

**Checkpoint**: User Story 3 is independently functional on top of US1: validation schemas and UI schemas are retrieved separately, and omitted UI schemas do not block validation or form rendering.

---

## Phase 6: User Story 4 - Migrate Default and Custom Schemas Safely (Priority: P4)

**Goal**: Default schemas are packaged as module declarations, custom schema migration is documented, invalid declarations produce actionable messages, and existing legacy filesystem deployments remain safe.

**Independent Test**: Upgrade a default-schema site and a custom-schema site; verify default schemas remain available, legacy `docroot/schema/collections` wins while present, custom modules can replace defaults after migration, and invalid declarations are reported clearly.

### Tests for User Story 4

- [ ] T063 [P] [US4] Add kernel tests for default schema module declaration availability in `../dkan/modules/dkan_metastore/tests/src/Kernel/Schema/DefaultSchemaModuleTest.php`
- [ ] T064 [P] [US4] Add kernel tests asserting the bare-minimum `dataset` schema in `dkan_metastore.schemas.yml` exposes only `description`, `keyword`, `modified`, `publisher.name`, `theme`, `title`, and `distribution` in `../dkan/modules/dkan_metastore/tests/src/Kernel/Schema/DefaultDatasetSchemaMinimalFieldsTest.php`
- [ ] T065 [P] [US4] Add kernel tests asserting the bare-minimum `distribution` items expose only `downloadURL`, `title`, `description`, `describedBy`, `describedByType` in `../dkan/modules/dkan_metastore/tests/src/Kernel/Schema/DefaultDistributionSchemaMinimalFieldsTest.php`
- [ ] T066 [P] [US4] Add functional migration tests for custom module schema replacement by weight in `../dkan/modules/dkan_metastore/tests/src/Functional/Schema/CustomSchemaMigrationTest.php`
- [ ] T067 [P] [US4] Add functional report tests for ignored declarations under legacy override in `../dkan/modules/dkan_metastore/tests/src/Functional/Schema/SchemaSelectionReportTest.php`
- [ ] T068 [P] [US4] Add documentation example validation test for declaration snippets in `../dkan/modules/dkan_metastore/tests/src/Kernel/Schema/SchemaDeclarationDocumentationTest.php`
- [ ] T069 [P] [US4] Add kernel tests for DCAT-US 1.1 module full `dataset`/`distribution`/`data-dictionary` schema availability in `../dkan/modules/dkan_dcat_us_1_1/tests/src/Kernel/Schema/DcatUs11ModuleSchemaTest.php`
- [ ] T070 [P] [US4] Add kernel tests proving DCAT-US 1.1 module declarations supersede `dkan_metastore` bare-minimum defaults via weight/module-order selection (FR-019) in `../dkan/modules/dkan_dcat_us_1_1/tests/src/Kernel/Schema/DcatUs11SupersedesDefaultTest.php`

### Implementation for User Story 4

- [ ] T071 [US4] Add default schema declaration file in `../dkan/modules/dkan_metastore/dkan_metastore.schemas.yml`
- [ ] T072 [US4] Author bare-minimum `dataset` validation schema JSON (only `description`, `keyword`, `modified`, `publisher.name`, `theme`, `title`, `distribution`) in `../dkan/modules/dkan_metastore/schema/collections/dataset.json`
- [ ] T073 [US4] Author bare-minimum `distribution` item properties (only `downloadURL`, `title`, `description`, `describedBy`, `describedByType`) within the `dataset.json` `distribution` property
- [ ] T074 [US4] Copy the current `data-dictionary` schema unchanged into `../dkan/modules/dkan_metastore/schema/collections/data-dictionary.json`
- [ ] T075 [US4] Map default validation schema paths in `../dkan/modules/dkan_metastore/dkan_metastore.schemas.yml`
- [ ] T076 [US4] Map default UI schema paths in `../dkan/modules/dkan_metastore/dkan_metastore.schemas.yml`
- [ ] T077 [US4] Add default reference definitions on the `dataset` schema's `distribution[]` items (`describedBy` → `item_url` targeting `data-dictionary`, `downloadURL` → `resource_url`) in `../dkan/modules/dkan_metastore/dkan_metastore.schemas.yml`
- [ ] T078 [US4] Add default `import_trigger`, `resource_mapper`, and `data_dictionary` behavior definitions on the `dataset` schema's relevant properties in `../dkan/modules/dkan_metastore/dkan_metastore.schemas.yml`
- [ ] T079 [US4] Add update or install handling for default schema availability in `../dkan/modules/dkan_metastore/dkan_metastore.install`
- [ ] T080 [US4] Update custom schema guide for module declarations and legacy override behavior in `../dkan/docs/source/user-guide/guide_custom_schemas.rst`
- [ ] T081 [US4] Add developer guide declaration format documentation in `../dkan/docs/source/developer-guide/schema_declarations.rst`
- [ ] T082 [US4] Update metastore component documentation for declaration-backed discovery in `../dkan/docs/source/components/dkan_metastore.rst`
- [ ] T083 [US4] Update upgrade notes for `docroot/schema/collections` override and migration path in `../dkan/docs/source/upgrade.rst`
- [ ] T084 [US4] Scaffold the optional DCAT-US 1.1 module info file in `../dkan/modules/dkan_dcat_us_1_1/dkan_dcat_us_1_1.info.yml`
- [ ] T085 [US4] Add full DCAT-US validation and UI schema JSON for `dataset`, `distribution`, and `data-dictionary` in `../dkan/modules/dkan_dcat_us_1_1/schema/collections/*.json`
- [ ] T086 [US4] Author `dkan_dcat_us_1_1.schemas.yml` declaring full `dataset`, `distribution`, and `data-dictionary` schemas with references and behavior definitions in `../dkan/modules/dkan_dcat_us_1_1/dkan_dcat_us_1_1.schemas.yml`
- [ ] T087 [US4] Set declaration `weight` (or rely on module order) so `dkan_dcat_us_1_1` declarations supersede `dkan_metastore` bare-minimum defaults when enabled, per FR-019/FR-026–FR-028
- [ ] T088 [US4] Document enabling the DCAT-US 1.1 module and its supersession behavior in `../dkan/docs/source/user-guide/guide_custom_schemas.rst`

**Checkpoint**: User Story 4 is independently functional on top of US1: default schemas remain available, custom schema migration is documented and testable, invalid declarations are actionable, and legacy deployments remain protected.

---

## Phase 6a: Stretch Goal - JSON Schema `$ref`/`$id` Cross-Schema Dependencies (FR-038–FR-041)

**Goal**: `validation_schema` supports an optional `dependencies` list (embedded `path`/`inline` content or other schema machine names) that is registered by `$id` with the schema resolver before validation, so standard JSON Schema `$ref` values resolve across schema documents.

**Status**: Stretch goal. Not required for this feature's completion; may be deferred to a follow-up phase, and may require upstream changes in `GetDKAN/RootedJsonData`.

**Independent Test**: Declare a schema whose `validation_schema` uses `$ref` to another registered schema's `$id`, list that schema in `dependencies`, and verify validation succeeds; verify an undeclared `$ref` target produces an invalid-declaration warning per FR-041.

### Tests for Stretch Goal

- [ ] T089 [P] [STRETCH] Add unit tests for `dependencies` list parsing (embedded `path`/`inline` object entries and string schema-machine-name entries) in `../dkan/modules/dkan_metastore/tests/src/Unit/Schema/SchemaDependencyTest.php`
- [ ] T090 [P] [STRETCH] Add kernel tests verifying declared dependency schemas are registered by `$id` before validation and `$ref` resolves in `../dkan/modules/dkan_metastore/tests/src/Kernel/Schema/SchemaDependencyRegistrationKernelTest.php`
- [ ] T091 [P] [STRETCH] Add kernel tests for an unresolved `$ref` (undeclared dependency) producing an invalid-declaration warning per FR-041 in `../dkan/modules/dkan_metastore/tests/src/Kernel/Schema/UnresolvedSchemaRefTest.php`

### Implementation for Stretch Goal

- [ ] T092 [STRETCH] Confirm `GetDKAN/RootedJsonData` (opis/json-schema 2.x) support for registering schema documents by `$id` before validation; file an upstream issue/PR if changes are needed
- [ ] T093 [STRETCH] Implement `dependencies` parsing (object and string entry forms) on the schema source value object in `../dkan/modules/dkan_metastore/src/Schema/SchemaSource.php`
- [ ] T094 [STRETCH] Implement recursive dependency registration by `$id` with the schema resolver before validation in `../dkan/modules/dkan_metastore/src/SchemaRetriever.php`
- [ ] T095 [STRETCH] Implement unresolved-`$ref` detection and invalid-declaration warning handling per FR-041 in `../dkan/modules/dkan_metastore/src/Schema/SchemaDeclarationValidator.php`
- [ ] T096 [STRETCH] Document the `dependencies`/`$ref` stretch-goal capability, including its optional status, in `../dkan/docs/source/developer-guide/schema_declarations.rst`

**Checkpoint**: If completed, cross-schema `$ref` resolution works for declared dependencies and unresolved references are reported per FR-041; if deferred, no other phase is blocked.

---

## Phase 7: Polish & Cross-Cutting Concerns

**Purpose**: Final consistency, quality, and release-readiness tasks across all stories.

- [ ] T097 [P] Run metastore PHPUnit coverage for schema declaration changes and record results in `specs/005-module-yaml-schemas/quickstart.md`
- [ ] T098 [P] Run datastore PHPUnit coverage for declaration-backed behavior changes and record results in `specs/005-module-yaml-schemas/quickstart.md`
- [ ] T099 [P] Run PHPCS for touched DKAN modules and record results in `specs/005-module-yaml-schemas/quickstart.md`
- [ ] T100 Review service names, constructor injection, and backward compatibility notes in `../dkan/modules/dkan_metastore/dkan_metastore.services.yml`
- [ ] T101 Review quickstart validation steps against final implementation behavior in `specs/005-module-yaml-schemas/quickstart.md`
- [ ] T102 Update task completion evidence and residual risks in `specs/005-module-yaml-schemas/tasks.md`

---

## Dependencies & Execution Order

### Phase Dependencies

- **Phase 1 Setup**: No dependencies; can start immediately.
- **Phase 2 Foundational**: Depends on Phase 1; blocks all user stories.
- **Phase 3 US1**: Depends on Phase 2 and is the MVP.
- **Phase 4 US2**: Depends on Phase 2 and integrates most cleanly after US1 registry result APIs exist.
- **Phase 5 US3**: Depends on Phase 2 and uses US1 registry result APIs for lookup.
- **Phase 6 US4**: Depends on US1 for discovery/override behavior and benefits from US2/US3 for complete default declarations.
- **Phase 6a Stretch Goal**: Depends on US1 registry/`SchemaRetriever` APIs; optional and does not block Phase 7 if deferred.
- **Phase 7 Polish**: Depends on the desired story set being complete; does not require Phase 6a.

### User Story Dependencies

- **US1 (P1)**: Core MVP; no dependency on other stories after Foundational.
- **US2 (P2)**: Can start after Foundational but requires US1 registry APIs before final integration.
- **US3 (P3)**: Can start after Foundational but requires US1 registry APIs before final integration.
- **US4 (P4)**: Requires US1; default schema packaging should include US2 references/triggers and US3 UI schema declarations for full completion.

### Within Each User Story

- Tests are listed first and should be written to fail before implementation.
- Value objects and registry result APIs precede consumers.
- Metastore service updates precede datastore integration.
- Documentation follows validated behavior.
- Each checkpoint should be validated before moving to the next priority story.

## Parallel Opportunities

- Setup fixture tasks T002-T007 can run in parallel.
- Foundational value object tasks T009-T011 can run in parallel.
- Test tasks within each user story can run in parallel because they target different files.
- US2 reference integration and datastore trigger integration can split between metastore and datastore developers after shared trigger/reference provider contracts are stable.
- US3 UI schema work can proceed alongside US2 after US1 registry APIs are stable.
- Documentation tasks T080-T083 can run in parallel once implementation behavior is settled.
- DCAT-US 1.1 module tasks T084-T088 can proceed in parallel with documentation once the bare-minimum default (T071-T079) is stable.

## Parallel Example: User Story 1

```bash
Task: "T016 [US1] Add unit tests for declaration parsing in ../dkan/modules/dkan_metastore/tests/src/Unit/Schema/SchemaDeclarationDiscoveryTest.php"
Task: "T017 [US1] Add unit tests for declaration validation in ../dkan/modules/dkan_metastore/tests/src/Unit/Schema/SchemaDeclarationValidatorTest.php"
Task: "T018 [US1] Add unit tests for duplicate selection in ../dkan/modules/dkan_metastore/tests/src/Unit/Schema/SchemaDeclarationResolverTest.php"
Task: "T020 [US1] Add functional API tests in ../dkan/modules/dkan_metastore/tests/src/Functional/Api1/SchemaDeclarationApiTest.php"
```

## Parallel Example: User Story 2

```bash
Task: "T034 [US2] Add kernel tests for declaration-backed references in ../dkan/modules/dkan_metastore/tests/src/Kernel/Reference/DeclarationReferencerTest.php"
Task: "T037 [US2] Add unit tests for declaration-backed import_trigger datastore update triggering in ../dkan/modules/dkan_datastore/tests/src/Unit/EventSubscriber/DeclarationDatastoreSubscriberTest.php"
Task: "T043 [US2] Update reference processing in ../dkan/modules/dkan_metastore/src/Reference/Referencer.php"
Task: "T050 [US2] Update datastore import_trigger checks in ../dkan/modules/dkan_datastore/src/EventSubscriber/DatastoreSubscriber.php"
```

## Parallel Example: User Story 3

```bash
Task: "T054 [US3] Add UI schema source tests in ../dkan/modules/dkan_metastore/tests/src/Unit/Schema/UiSchemaDeclarationTest.php"
Task: "T057 [US3] Add functional widget tests in ../dkan/modules/dkan_metastore/tests/src/Functional/Plugin/Field/FieldWidget/DeclarationUiSchemaTest.php"
Task: "T061 [US3] Update JSON form option source in ../dkan/modules/dkan_metastore/src/Plugin/JsonFormOptionSource/MetastoreSchema.php"
```

## Parallel Example: User Story 4

```bash
Task: "T063 [US4] Add default schema module tests in ../dkan/modules/dkan_metastore/tests/src/Kernel/Schema/DefaultSchemaModuleTest.php"
Task: "T080 [US4] Update custom schema guide in ../dkan/docs/source/user-guide/guide_custom_schemas.rst"
Task: "T081 [US4] Add developer guide in ../dkan/docs/source/developer-guide/schema_declarations.rst"
Task: "T083 [US4] Update upgrade notes in ../dkan/docs/source/upgrade.rst"
```

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1 Setup.
2. Complete Phase 2 Foundational services and value objects.
3. Complete Phase 3 US1 discovery, validation, duplicate selection, legacy override, and schema API compatibility.
4. Stop and validate US1 independently with fixture modules and schema API checks.
5. Use US1 as the base for reference, trigger, UI schema, and migration work.

### Incremental Delivery

1. Deliver US1 so schema discovery is declaration-backed when legacy override is absent.
2. Deliver US2 so behavior moves from property-name/admin config assumptions into schema metadata.
3. Deliver US3 so validation and UI schemas are separated cleanly.
4. Deliver US4 so default schemas, custom migration, and documentation are release-ready.
5. (Optional) Deliver Phase 6a's `dependencies`/`$ref` stretch goal if time allows; otherwise defer without blocking release.

### Manual Ticket Strategy

Each numbered task should be small enough to become a manual ticket or subtask. If a task grows beyond one developer-week, split it by test fixture, implementation class, and consumer integration file before coding.

## Notes

- `[P]` marks tasks that can run in parallel because they touch different files and do not depend on incomplete implementation tasks.
- `[US1]`, `[US2]`, `[US3]`, and `[US4]` labels map tasks to the feature specification user stories; `[STRETCH]` marks Phase 6a's optional `dependencies`/`$ref` tasks (FR-038–FR-041), which are not required for feature completion.
- All user story tasks include exact target paths for implementation or tests.
- Tests are included because the specification's success criteria require automated coverage.
- Keep `docroot/schema/collections` override behavior visible in every story that touches discovery or migration.