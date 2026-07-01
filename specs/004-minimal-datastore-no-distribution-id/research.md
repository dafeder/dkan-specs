# Research: Minimal Datastore Without Distribution ID Requirement

## Decision: Keep ResourceMapper/DataResource as the canonical runtime registry

Rationale: The corrected flow still needs stable datastore resource identity, version, and perspective handling. Existing `ResourceMapper` + `DataResource` behavior already defines those rules and remains the least disruptive path while removing mandatory distribution-ID runtime keys.

Alternatives considered: Introducing a new datastore-owned registry was rejected as out-of-scope architecture churn. Reverting to distribution UUID as the operational key was rejected because it fails non-referenced distribution workflows.

## Decision: Restore reference mapping from `downloadURL` to compound datastore identifier

Rationale: The prior workflow relied on both registration and reference mapping, not registration alone. The new implementation direction must restore the missing step that links each discovered `downloadURL` to the compound datastore identifier used by downstream status/reporting/query flows.

Alternatives considered: Registration-only discovery was rejected because it loses identity semantics needed by compatibility surfaces. Late reconstruction of references from mapping tables was rejected because it creates ambiguous lookups and duplicate churn risk.

## Decision: Move discovery ownership to the Reference layer while keeping dataset-presave invocation

Rationale: Discovery and reference mapping are metadata-reference responsibilities and belong under metastore Reference concerns. However, invocation still must occur from dataset presave orchestration so workflow initiation is not gated by optional distribution referencing toggles.

Alternatives considered: LifeCycle-only discovery service ownership was rejected because it separates discovery from the reference contract it must produce. Running only inside configurable property-list reference handlers was rejected because disabling distribution reference processing can suppress required datastore initiation.

## Decision: Keep top-level `$.distribution[]` as the discovery scope and preserve encounter-order semantics

Rationale: The current spec and tests target top-level distribution entries. Processing valid URLs in encounter order without deduplication preserves established behavior and limits migration risk.

Alternatives considered: Recursive deep metadata scans were rejected because they exceed this feature scope and can trigger unintended resources. Deduplication was rejected because it changes semantics and may hide intended repeated processing.

## Decision: Make discovered-resource references single-owner, and let downstream components consume discovered resources

Rationale: Reference ownership must be clear to prevent double-registration and inconsistent identifiers. A single reference-owned discovered resource output should drive registration, initiation, and reporting inputs.

Alternatives considered: Re-performing discovery/reference mapping in multiple layers was rejected due to drift and duplicate side effects. Letting referencer and lifecycle each register independently was rejected because it causes version churn and inconsistent status linkage.

## Decision: Preserve best-effort processing with machine-readable skip/failure visibility

Rationale: Mixed-validity distributions are expected. One bad entry cannot block other valid URLs. Structured skip/failure outcomes and aggregate counts remain required for administrator visibility and testability.

Alternatives considered: Fail-fast behavior was rejected because it reduces ingestion reliability. Free-form logging only was rejected because acceptance criteria require machine-readable outcome reporting.

## Decision: Keep `describedBy` validation/normalization independent of reference mode

Rationale: `describedBy` data-dictionary behavior is part of metadata correctness and must remain equivalent in referenced and non-referenced distribution workflows even as ownership moves under Reference.

Alternatives considered: Tying `describedBy` handling to distribution-reference mode was rejected as a regression vector. Deferring this to later cleanup was rejected because it violates current feature requirements.

## Decision: Sequence compatibility work after corrected reference ownership is in place

Rationale: Cache, status, dashboard, SQL, and Drush compatibility surfaces depend on stable discovered-resource reference semantics. Correcting ownership first reduces rework and test instability.

Alternatives considered: Updating compatibility surfaces before restoring reference mapping was rejected because contracts would still be in flux.
