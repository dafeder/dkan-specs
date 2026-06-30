# Research: Minimal Datastore Without Distribution ID Requirement

## Decision: Keep ResourceMapper/DataResource as the scoped resource registry

Rationale: Current metastore reference handling registers distribution `downloadURL` values through `ResourceMapper`, and datastore import/localize flows already consume `DataResource` identifiers, versions, perspectives, file paths, MIME types, and checksums. Keeping this registry preserves the existing ETL mechanics while removing distribution UUIDs as required operational keys.

Alternatives considered: Replacing ResourceMapper with a new canonical datastore resource model was rejected for this scoped feature because it belongs to the broader decoupled datastore architecture. Continuing to use distribution UUIDs as required keys was rejected because it fails non-referenced distribution workflows.

## Decision: Initiate datastore processing from dataset-save discovery

Rationale: The most affected behavior is resource discovery when datasets are saved. Both referenced and non-referenced distribution structures expose effective `distribution[].downloadURL` values after dataset handling, except non-referenced distributions do not have their own distribution UUIDs. A single recursive discovery pass avoids separate logic branches and supports both shapes.

Alternatives considered: Distribution-save hook behavior was rejected because non-referenced distributions may not produce standalone distribution entities. Separate referenced/non-referenced discovery paths were rejected because the effective dataset structure is equivalent for discovery.

## Decision: Decouple resource discovery/registration from the metadata referencing workflow

Rationale: Resource registration currently happens as a side effect of referencing (`Referencer::distributionHandling()`), and the referencer only processes properties present in `property_list`. When distribution referencing is disabled (`property_list['distribution'] === '0'`), the `distribution` property is filtered out, so registration and datastore import triggering silently never run. Performing discovery and registration in `LifeCycle::datasetPresave()` (before `referenceMetadata()`), independent of referencing, ensures datastore initiation always occurs for valid `downloadURL` values regardless of referencing configuration.

Alternatives considered: Keeping discovery/registration on the `EVENT_PRE_REFERENCE` path was rejected because it couples datastore initiation to optional metadata referencing. Triggering from a distribution-save hook was rejected for the same reason as above (non-referenced distributions may not produce distribution entities).

## Decision: Preserve existing dispatch order and non-deduplication semantics

Rationale: The scoped feature aims to remove the distribution ID requirement without changing runtime behavior more than necessary. Dispatching each discovered valid `downloadURL` as encountered preserves current behavior and keeps implementation ticket scope smaller.

Alternatives considered: URL deduplication was rejected because it would introduce new semantics and potentially hide existing repeated dispatch behavior. Selecting only the first URL was rejected because the clarified requirement is to process all valid distribution entries.

## Decision: Use best-effort per-URL dispatch with structured logs and status summary

Rationale: Dataset distribution arrays may contain mixed-quality entries. Best-effort dispatch keeps valid resources moving even when individual entries are invalid or fail processing. Structured logs and a machine-readable summary provide administrator visibility and testable outcomes.

Alternatives considered: Fail-fast dispatch was rejected because it would make one bad URL block unrelated valid entries. Summary-only reporting was rejected because administrators also need traceable per-entry diagnostics.

## Decision: Accept distribution references as compatibility metadata but never require them for initiation

Rationale: Existing DKAN installations and referenced distribution flows remain valid. The feature changes the operational requirement, not the metadata capability: distribution UUIDs may exist and may help cache/reporting context, but discovery and datastore initiation are driven by `downloadURL` discovery.

Alternatives considered: Removing distribution referencing was rejected as unnecessary and too disruptive. Preferring distribution-ID paths when present was rejected because it keeps the old dependency on the critical path.

## Decision: Move secondary distribution-ID dependencies into explicit implementation tickets

Rationale: Code review found additional distribution-ID assumptions in cache invalidation/dependencies, Drush reverse lookup, SQL/API/admin reporting, post-import result creation, and cleanup/orphan behavior. These surfaces should be handled explicitly because they can keep the runtime dependency alive even after initiation is fixed.

Alternatives considered: Treating these as incidental cleanup was rejected because they affect acceptance testing and migration guidance. Deferring all secondary surfaces was rejected because the spec requires import/query workflows to avoid runtime distribution dereference in normal operation.

## Decision: Keep `describedBy` data-dictionary handling independent from distribution referencing mode

Rationale: Functional testing shows a non-referenced variant of `DistributionHandlingTest::testDescribedByDataDictionary` can fail when distribution referencing is disabled via `property_list['distribution'] = '0'`. This indicates `describedBy` URI validation/normalization behavior is still coupled to optional distribution referencing paths. For this feature, referenced/non-referenced distribution structures must retain equivalent `describedBy` behavior because distribution references are compatibility metadata, not operational gates.

Alternatives considered: Deferring `describedBy` behavior to a later cleanup was rejected because it is a direct runtime regression in non-referenced mode. Restricting `describedBy` support to referenced distributions was rejected because it contradicts the feature's non-referenced compatibility objective.

## Decision: Keep ticket scope below one manual week by slicing by surface area

Rationale: The implementation can be split into independent sub-week tickets: dataset discovery/summary model, ResourceMapper registration, dispatch integration, observability, cache dependency updates, reporting/post-import updates, cleanup/orphan updates, and migration/test documentation.

Alternatives considered: A single broad implementation ticket was rejected because it would mix metastore lifecycle changes, datastore service changes, reporting, and migration work. A full decoupled-datastore implementation was rejected as outside this scoped feature.

## Decision: Prefer the smallest abstraction surface that preserves the required contracts

Rationale: This scoped feature needs clear discovery and dispatch contracts, but it does not benefit from introducing extra classes that only repackage straightforward control flow. New classes should exist only when they protect a real boundary, a reusable contract, or a meaningful test seam.

Alternatives considered: Splitting each small responsibility into its own class was rejected because it increases implementation churn and migration surface without adding equivalent value for this scoped change. Keeping all new logic inline in existing subscribers/services was also rejected because the feature still needs a small number of explicit contracts for discovery results and downstream dispatch behavior.
