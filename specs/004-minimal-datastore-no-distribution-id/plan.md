# Implementation Plan: Minimal Datastore Without Distribution ID Requirement

**Branch**: `004-minimal-datastore-no-distribution-id` | **Date**: 2026-06-30 | **Spec**: `specs/004-minimal-datastore-no-distribution-id/spec.md`
**Input**: Feature specification from `specs/004-minimal-datastore-no-distribution-id/spec.md`

## Summary

Remove distribution IDs as required runtime keys while preserving current datastore behavior wherever practical. Resource discovery/registration must run from dataset presave (not referencing-gated paths), and compatibility surfaces (cache, API/Drush/SQL/admin, post-import status, cleanup, migration docs) must continue to work for both referenced and non-referenced distribution structures. This plan now explicitly includes data-dictionary `describedBy` handling as a required non-regression when distribution referencing is disabled.

## Technical Context

**Language/Version**: PHP 8.x (Drupal 10 module code)  
**Primary Dependencies**: Drupal core services/events, DKAN metastore/datastore modules, ResourceMapper/DataResource flow  
**Storage**: Drupal entity + DKAN resource mapping storage (existing)  
**Testing**: PHPUnit (unit, kernel, functional), PHPCS  
**Target Platform**: DKAN on Drupal (server-side PHP)  
**Project Type**: Backend modules + docs/spec artifacts  
**Performance Goals**: Preserve current queue/immediate behavior and dispatch throughput semantics; no new dedup or batching  
**Constraints**: Keep architectural churn low; preserve existing datastore initiation/status surfaces in this phase; distribution references optional but not required operational keys  
**Scale/Scope**: Scoped 004 feature only; no full datastore ownership redesign

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- Principle I (Metadata-first): PASS. Dataset/distribution metadata handling and `describedBy` data-dictionary behavior are explicitly specified and tested.
- Principle II (Modular architecture): PASS. Changes preserve metastore/datastore boundaries and use documented internal service/event contracts.
- Principle III (API-driven integration): PASS. External surfaces remain API/Drush/UI consumers; operational key migration is documented.
- Principle IV (Standards and quality): PASS. Non-referenced distribution and data-dictionary handling are tested; failures are reported with administrator-facing diagnostics.
- Principle V (Extensibility): PASS. Existing extension points remain; migration notes cover payload changes.
- Principle VI (Tests and docs): PASS with enforcement. Tasks require unit/kernel/functional coverage and migration documentation before completion.

## Phase 0: Research

Research decisions captured in `research.md` and resolved for this planning pass:

- Keep `ResourceMapper` as scoped canonical registry.
- Move discovery/registration to dataset presave and decouple from referencing.
- Preserve non-deduplicated URL processing semantics.
- Use best-effort per-URL trigger behavior with structured logs + machine-readable summary.
- Accept distribution references as compatibility metadata only.
- Add explicit non-regression decision: `describedBy` data-dictionary validation/normalization remains correct when `property_list['distribution']` disables distribution referencing.

## Phase 1: Design and Contracts

Design artifacts for this feature:

- `data-model.md`: dataset/distribution/resource entities and validation constraints.
- `contracts/dataset-save-dispatch.md`: dataset-save discovery + initiation contract.
- `contracts/compatibility-surfaces.md`: compatibility-sensitive API/CLI/admin surfaces.
- `quickstart.md`: ticket slicing, acceptance checks, and implementation sequencing.

Additional design focus for this pass:

- Explicitly cover `distribution[].describedBy` data-dictionary behavior for referenced and non-referenced distributions.
- Ensure data-dictionary URI normalization and validation are not gated by distribution referencing settings.

## Project Structure

### Documentation (this feature)

```text
specs/004-minimal-datastore-no-distribution-id/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   ├── dataset-save-dispatch.md
│   └── compatibility-surfaces.md
└── tasks.md
```

### Source Code (DKAN repository root)

```text
modules/dkan_metastore/
├── src/LifeCycle/
├── src/Reference/
└── tests/src/
    ├── Unit/LifeCycle/ResourceDiscovery/
    ├── Functional/Api1/
    └── Functional/

modules/dkan_datastore/
├── src/EventSubscriber/
├── src/Controller/
├── src/Drush/Commands/
├── src/Service/
└── tests/src/
    ├── Unit/
    ├── Kernel/
    └── Functional/

modules/dkan_common/
├── src/
└── tests/src/Kernel/

docs/source/
```

**Structure Decision**: Keep existing DKAN module structure and service/event flow. Add only scoped lifecycle/contract updates needed for dataset-save discovery, dispatch triggering, compatibility surfaces, and explicit `describedBy` non-regression behavior.

## Delivery Phases

1. Shared Foundation complete and validated.
2. Ticket 1-2 (MVP): dataset-save discovery/registration + dispatch triggering.
3. Ticket 3-5: cache, compatibility surfaces, dataset info, status/admin continuity.
4. Ticket 6-8: importer regression, cleanup/orphan behavior, migration guidance.
5. Release validation: PHPUnit + PHPCS evidence captured in quickstart.

## Post-Design Constitution Re-Check

- Re-check status: PASS.
- No constitutional violations introduced by adding explicit data-dictionary/non-referenced coverage.

## Complexity Tracking

No constitution violations requiring justification in this planning pass.
