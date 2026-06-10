# Research: Module-Based Schemas With YAML Declarations

## Decision: Use enabled module-root `MODULE.schemas.yml` files for declaration discovery

**Rationale**: Drupal modules are DKAN's existing extension boundary. A module-root declaration file makes schema ownership explicit, keeps schemas installable/removable with their providing module, and avoids recursively inferring schemas from unrelated JSON files.

**Alternatives considered**: Continue scanning filesystem JSON files, but that preserves the masking and filename-convention problems this feature is meant to fix. Use Drupal config entities for every schema, but that adds UI/config lifecycle scope that is not required for the first phase.

## Decision: Preserve `docroot/schema/collections` as an authoritative legacy override

**Rationale**: Existing DKAN sites customize schemas by copying `schema/collections` to `docroot/schema/collections`. Treating that directory as authoritative avoids surprising upgrades and lets maintainers migrate deliberately.

**Alternatives considered**: Merge filesystem and module declarations, but that creates ambiguous precedence. Treat filesystem schemas as fallback only, but that breaks the latest backward-compatibility requirement. Ignore filesystem schemas entirely, but that would be a breaking migration.

## Decision: Replace `SchemaRetriever` internals with a declaration-aware registry

**Rationale**: Current `SchemaRetriever` hard-codes schema IDs and retrieves `collections/{id}.json` from either `docroot/schema` or DKAN's default schema directory. Existing consumers already depend on `retrieve()` and `getAllIds()`, so a registry-backed retriever preserves the main service boundary while changing discovery internals.

**Alternatives considered**: Add a parallel schema service and leave `SchemaRetriever` unchanged, but existing consumers would keep bypassing declarations. Rewrite all consumers directly, but that creates unnecessary churn.

## Decision: Resolve duplicate schema machine names with declaration weight, then Drupal module order

**Rationale**: Optional declaration `weight` gives custom modules an explicit replacement mechanism. Standard Drupal module weight/order is a familiar deterministic fallback when weights are equal or absent.

**Alternatives considered**: Fail on duplicates, but the spec requires one schema to be selected. Last-discovered-wins is deterministic only by accident and hard for operators to reason about.

## Decision: Exclude invalid declarations individually and continue discovery

**Rationale**: A single invalid declaration should not make unrelated schemas unavailable. Per-declaration validation warnings preserve operator visibility while keeping valid schemas usable.

**Alternatives considered**: Fail all discovery on the first invalid declaration, but that is too fragile for multi-module sites. Silently skip invalid declarations, but that makes remediation difficult.

## Decision: Keep fixed core schema machine names for business behavior in this phase

**Rationale**: Existing DKAN behavior assumes names such as `dataset`, `distribution`, and `data-dictionary`. Keeping these fixed lets declaration discovery ship without introducing a schema role/alias model.

**Alternatives considered**: Introduce roles or aliases now, but that expands scope and would require more API, migration, and administrative design.

## Decision: Move references and datastore triggers into declaration metadata

**Rationale**: Current reference behavior uses configured property lists and code paths such as `Referencer::getPropertyList()`, while datastore triggers are configured in `dkan_datastore.settings`. Declaration metadata aligns behavior with schema ownership and removes property-name-only assumptions.

**Alternatives considered**: Keep admin-selected trigger fields and metastore reference config, but that leaves schema behavior split across config and declarations. Hard-code new properties in PHP, but that repeats the current customization problem.

## Decision: Provide operator-visible reports for source selection, conflicts, and invalid declarations

**Rationale**: Duplicate selection and legacy override behavior must be inspectable. A report can show active source mode, candidate declarations, winning declarations, weights/order, and skipped invalid declarations.

**Alternatives considered**: Log only, but logs are hard to inspect during site-building. API-only reporting may be useful but should not be the only operator surface.