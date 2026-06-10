# Feature Specification: Module-Based Schemas With YAML Declarations

**Feature Branch**: `[005-add-module-yaml-schemas]`  
**Created**: 2026-06-10  
**Status**: Draft  
**Input**: User description: "new feature for \"Module-based schemas with yaml declarations\" based on body and discussion in https://github.com/GetDKAN/dkan/issues/3761"

## Clarifications

### Session 2026-06-10

- Q: How should duplicate schema machine names be resolved? -> A: Schema declarations may include an optional `weight`; highest declaration weight wins, equal weights fall back to standard Drupal module weight/order, a schema is always selected, and operators can view available schemas plus why a winner was chosen.
- Q: How should legacy filesystem schema discovery behave during migration? -> A: If `docroot/schema/collections` exists, use legacy filesystem discovery as the authoritative source and ignore module YAML declarations.
- Q: What schema declaration filename and location should modules use? -> A: Enabled modules declare schemas in a module-root `MODULE.schemas.yml` file.
- Q: How should invalid schema declarations affect discovery? -> A: Ignore only the invalid declaration, warn operators, and continue discovering valid declarations.
- Q: Should business behavior use explicit schema roles or fixed core schema names in this phase? -> A: Use fixed core schema machine names for business behavior in this phase and defer role/alias support.
- Q: What validation schema source formats are allowed in declarations? -> A: File path only; inline schema content is not supported.
- Q: What path format should schema declarations use for schema files? -> A: Module-relative file paths only (relative to module root).
- Q: Should this phase keep a separate SchemaSource abstraction? -> A: No; store normalized validation/UI schema paths directly on schema declarations.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Discover Schemas From Enabled Modules (Priority: P1)

As a DKAN site builder, I want schemas to be registered by enabled modules through explicit YAML declarations so schema availability is predictable and no longer depends on inferred filesystem locations or filename conventions.

**Why this priority**: Explicit module-owned schema registration is the core value of the feature and removes the fragility that currently blocks reliable customization.

**Independent Test**: Enable a module with a schema declaration file, verify its declared schemas are available to metastore operations, then disable the module and verify those schemas are no longer available.

**Acceptance Scenarios**:

1. **Given** an enabled module declares one or more schemas in `MODULE.schemas.yml`, **When** schema discovery runs, **Then** each declared schema is available by its declared machine name.
2. **Given** a module does not declare schemas, **When** schema discovery runs, **Then** no schemas are inferred from unrelated JSON files in that module.
3. **Given** `docroot/schema/collections` is absent and a site has both default and custom schema modules enabled, **When** schema discovery runs, **Then** schemas from all enabled declaration files are discovered without one module hiding the other module's schemas.

---

### User Story 2 - Configure References and Triggers Declaratively (Priority: P2)

As a DKAN developer, I want schema declarations to define references and datastore-triggering properties so metastore behavior is driven by schema metadata rather than brittle property-name conditionals or admin-only settings.

**Why this priority**: References and triggers are where current filename/property assumptions most directly affect business logic and customization.

**Independent Test**: Declare dataset references, distribution resource references, and datastore import triggers in YAML, then verify metastore reference handling and datastore trigger behavior follow those declarations.

**Acceptance Scenarios**:

1. **Given** a schema declaration identifies a property as a schema reference, **When** metadata is saved, **Then** that property is processed using the declared referenced schema.
2. **Given** a schema declaration identifies a property as a resource reference, **When** metadata is saved, **Then** that property is processed using the declared resource-reference behavior.
3. **Given** a schema declaration identifies a property as a datastore import trigger, **When** that property changes on update, **Then** the datastore import trigger behavior runs for affected resources.

---

### User Story 3 - Separate Validation Schemas From UI Schemas (Priority: P3)

As a DKAN form builder, I want metastore validation schemas and form-specific UI schemas to be declared separately so validation, API behavior, and form rendering can evolve without filename-based conditionals.

**Why this priority**: UI schema separation improves developer experience and removes another fragile inference point, but it can be delivered after core discovery/reference behavior.

**Independent Test**: Declare both a validation schema and UI schema for one metastore item type, then verify validation uses the validation schema while form generation uses the UI schema.

**Acceptance Scenarios**:

1. **Given** a schema declaration includes a validation schema and a UI schema, **When** metadata is validated, **Then** only the validation schema controls data validity.
2. **Given** a schema declaration includes a UI schema, **When** a metadata form is built, **Then** the form-specific behavior uses the declared UI schema.
3. **Given** a schema declaration omits a UI schema, **When** a metadata form is built, **Then** the system falls back gracefully without blocking API validation.

---

### User Story 4 - Migrate Default and Custom Schemas Safely (Priority: P4)

As a DKAN maintainer, I want a clear migration path for default and custom schemas so existing sites can adopt module-based declarations without losing schema availability or breaking metadata operations.

**Why this priority**: The feature changes how schemas are discovered, so adoption depends on safe migration and documentation.

**Independent Test**: Upgrade a site using default schemas and a site using custom schemas, then verify schema discovery, reference behavior, and trigger behavior match the pre-upgrade intent.

**Acceptance Scenarios**:

1. **Given** a site uses DKAN's default schema set, **When** the upgrade path runs, **Then** the default schema module is enabled or otherwise made available as required for continued operation.
2. **Given** a site uses custom schemas, **When** maintainers follow migration guidance, **Then** custom schemas can be moved into a custom module declaration without being masked by default schemas.
3. **Given** a schema declaration is invalid or references a missing schema file, **When** schema discovery runs, **Then** operators receive actionable validation errors.

### Edge Cases

- Multiple enabled modules declare the same schema machine name.
- Multiple enabled modules declare the same schema machine name with missing or equal declaration weights.
- A declaration references a schema file or UI schema file that does not exist.
- A declaration defines a reference to an unknown schema.
- A declaration defines an unsupported reference type or trigger name.
- An invalid declaration is skipped with warnings while other valid declarations continue to be discovered.
- A custom module intentionally replaces a default dataset or distribution schema.
- A site upgrades with legacy schemas in a filesystem schema directory but no module-based declaration.
- If `docroot/schema/collections` exists, legacy filesystem schema discovery wins and module YAML declarations are ignored.
- A declaration omits optional UI schema information.
- A schema declaration changes while existing metadata records already use earlier behavior.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: When `docroot/schema/collections` is absent, the system MUST discover metastore schemas from explicit module-root `MODULE.schemas.yml` declaration files provided by enabled modules.
- **FR-002**: The system MUST NOT infer schema availability solely from the presence of JSON files in filesystem directories.
- **FR-003**: Each schema declaration MUST have a stable schema machine name used for lookup and metastore operations.
- **FR-004**: A schema declaration MUST support a validation schema defined by a path to a schema document.
- **FR-005**: A schema declaration MUST support an optional UI schema defined separately from the validation schema.
- **FR-005a**: Validation schema and UI schema file paths in declarations MUST be module-relative to the declaring module root.
- **FR-006**: The system MUST report a validation error when a declaration references a missing schema document or missing UI schema document, exclude the invalid declaration from active discovery, and continue discovering other valid declarations.
- **FR-007**: When more than one enabled declaration provides the same schema machine name, the system MUST select a winning declaration deterministically instead of failing discovery.
- **FR-008**: Schema declarations MUST support reference definitions that identify the source property and reference behavior for schema references, identifier references, and resource references.
- **FR-009**: Reference processing MUST use declaration-defined reference rules instead of property-name-only conditionals.
- **FR-010**: Schema declarations MUST support trigger definitions that identify the source property and named trigger behavior.
- **FR-011**: Datastore import trigger behavior MUST be driven by schema declaration trigger definitions rather than datastore trigger fields stored only in administrative configuration.
- **FR-012**: DKAN business behavior in this phase MUST use fixed core schema machine names, including at minimum `dataset`, `distribution`, and `data-dictionary`, rather than a separate role or alias system.
- **FR-013**: The initial feature MUST support the default `dataset` and `distribution` schema names and MUST NOT require arbitrary multiple schemas for the same core behavior in this phase.
- **FR-014**: Literal-string schema handling is out of scope for this phase.
- **FR-015**: The system MUST keep validation schema behavior separate from UI schema behavior.
- **FR-016**: Existing schema lookup consumers MUST be able to retrieve validation schemas by declared machine name.
- **FR-017**: Existing form-building consumers MUST be able to retrieve UI schemas when declared.
- **FR-018**: Existing reference handling MUST continue to support dataset-to-distribution references, distribution-to-resource references, and data-dictionary identifier references when declared.
- **FR-019**: The default DCAT-US-style schema collection MUST be packaged so it can be enabled as a module-provided schema set.
- **FR-020**: Sites using custom schemas MUST have a documented path to provide those schemas through a custom module declaration.
- **FR-021**: When `docroot/schema/collections` exists, the system MUST use legacy filesystem schema discovery as the authoritative schema source and ignore module YAML declarations.
- **FR-022**: The system MUST expose operator-visible reporting that identifies whether active schemas are sourced from `docroot/schema/collections` or module YAML declarations.
- **FR-023**: The current admin pages for reference and trigger field selection MUST be removed, disabled, or clearly superseded once declaration-based references and triggers are authoritative.
- **FR-024**: Operators MUST receive actionable messages when schema declarations are invalid, incomplete, or conflict with one another.
- **FR-025**: Documentation MUST include the declaration format, supported fields, default schema module behavior, custom schema migration steps, and examples for references and triggers.
- **FR-026**: Schema declarations MAY include an optional numeric `weight` property used to resolve duplicate schema machine names.
- **FR-027**: For duplicate schema machine names, the declaration with the highest explicit `weight` MUST be selected as the active schema.
- **FR-028**: If duplicate schema declarations have equal or missing `weight` values, the system MUST fall back to standard Drupal module weight/order practices to select the active schema.
- **FR-029**: The system MUST provide an operator-visible view or report showing all available declarations for each schema machine name, the selected active declaration, and the reason it won.
- **FR-030**: Operator-visible declaration validation messages MUST identify the invalid declaration, the reason it was excluded, and any remaining active schema selected for the affected machine name.
- **FR-031**: This phase MUST model validation and UI schema locations as normalized module-relative paths directly on schema declarations and MUST NOT require a separate source-type abstraction.

### Key Entities *(include if feature involves data)*

- **Schema Declaration File**: A module-root `MODULE.schemas.yml` file that registers one or more schemas and their metadata.
- **Schema Definition**: A declared schema entry with machine name, validation schema file path, optional UI schema file path, optional weight, references, triggers, and endpoint metadata where applicable.
- **Validation Schema**: The JSON-compatible schema document loaded from a module-relative declaration path and used to validate metastore metadata and API behavior.
- **UI Schema**: Optional form-specific schema data loaded from a module-relative declaration path and used for metadata form rendering and editing experience.
- **Reference Definition**: Declarative rule that maps a property to schema, identifier, resource, or future reference behavior.
- **Trigger Definition**: Declarative rule that maps a property change to named system behavior such as datastore import.
- **Default Schema Module**: Module-provided package of DKAN's default DCAT-US-style schemas and declarations.
- **Custom Schema Module**: Site-owned module that declares custom schemas and can replace or extend default schema behavior according to documented rules.
- **Schema Selection Report**: Operator-visible output that lists discovered declarations, active selections, weights, fallback ordering, and conflict-resolution reasons.
- **Legacy Schema Override**: Backward-compatible behavior where the presence of `docroot/schema/collections` makes filesystem schemas authoritative and prevents module YAML declarations from participating in active schema selection.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: When `docroot/schema/collections` is absent, 100% of schemas used by metastore operations in supported modules are discoverable through enabled module declarations.
- **SC-002**: When `docroot/schema/collections` is absent, schema discovery returns consistent results regardless of unrelated JSON files present outside module declarations.
- **SC-003**: 100% of declaration validation failures produce actionable messages identifying the schema entry and failing field.
- **SC-004**: 100% of declared reference rules for dataset, distribution, resource, and identifier references are exercised by automated tests.
- **SC-005**: 100% of declared datastore import triggers in scope are exercised by automated tests.
- **SC-006**: Default schema behavior remains available after upgrade for sites using the default schema set.
- **SC-007**: A custom schema module can be enabled and used without disabling discovery of unrelated default schemas.
- **SC-008**: UI schema retrieval works for 100% of declarations that include UI schema information and falls back without fatal errors when UI schema information is omitted.
- **SC-009**: Reference and trigger administration paths no longer allow conflicting runtime configuration once declaration-based behavior is authoritative.
- **SC-010**: Migration documentation enables maintainers to convert a legacy custom schema directory into a module declaration in under 30 minutes for a simple dataset/distribution schema pair.
- **SC-011**: For 100% of duplicate schema machine-name scenarios in scope, schema discovery selects one active schema and exposes the selected declaration and selection reason in operator-visible reporting.
- **SC-012**: For 100% of environments where `docroot/schema/collections` exists, schema discovery uses filesystem schemas instead of module YAML declarations and reports that source selection to operators.
- **SC-013**: In 100% of invalid-declaration scenarios in scope, valid declarations remain discoverable and the invalid declaration is excluded with an operator-visible warning.

## Assumptions

- The first implementation focuses on module-owned schema declarations, not a user-interface for creating schemas.
- Enabled modules are the source of truth for declared schema availability.
- Schema declarations live in module-root `MODULE.schemas.yml` files.
- Default DCAT-US-style schemas remain supported, but are packaged through a module-provided declaration instead of root-level schema discovery.
- New custom schemas are expected to live in custom modules when `docroot/schema/collections` is absent; existing `docroot/schema/collections` deployments remain authoritative while present.
- Legacy filesystem schema discovery remains a backward-compatible override: when `docroot/schema/collections` exists, it wins over module YAML declarations.
- The initial phase supports one active schema for each core role such as dataset and distribution.
- The initial phase uses fixed core schema machine names for business behavior; role aliases and multiple schemas for the same core behavior are deferred.
- Duplicate schema machine names are expected during customization and are resolved deterministically rather than treated as fatal conflicts.
- Validation schemas are declared as file paths; inline schema content is out of scope for this phase.
- A separate SchemaSource-style source-type abstraction is out of scope; declarations carry normalized module-relative file paths directly.
- Additional schema role aliasing and multiple dataset/distribution schema variants may be explored later.
- Existing public API behavior should remain stable where practical while schema discovery internals change.
