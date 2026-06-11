<!--
Sync Impact Report
Version change: 1.2.0 -> 1.2.1
Modified principles:
- IV. Standards Compliance & Data Quality: terminology update from "operators" to "administrators"
Added sections:
- None
Removed sections:
- None
Templates requiring updates:
- ✅ .specify/templates/plan-template.md (reviewed, no update required)
- ✅ .specify/templates/spec-template.md (reviewed, no update required)
- ✅ .specify/templates/tasks-template.md (reviewed, no update required)
Follow-up TODOs:
- None
-->

# DKAN Constitution

## Core Principles

### I. Metadata-First Design
Every data resource in DKAN is defined through structured, standardized metadata stored as JSON in Drupal entities. Metadata structure is governed by customizable JSON schemas; deployments may use DCAT-US (Data Catalog Vocabulary for US Federal Data) or other domain-specific vocabularies. New features that expose, transform, or validate data MUST address their metadata implications first and be compatible with the configured schema.

### II. Modular Component Architecture
DKAN's three core components—metastore, datastore, and harvest—MUST remain architecturally independent with clearly defined interfaces. Components communicate via stable APIs and contracts. New features that span components require explicit interface documentation. No component may directly access another component's internal data structures; public communication flows through documented APIs and internal orchestration flows through documented Drupal services.

### III. API-Driven Integration
All external data access, transformation, and ingestion MUST flow through well-documented APIs (REST, JSON). The datastore API, metastore API, and harvest integrations MUST support both human-readable and machine-parseable output formats (JSON preferred for machine consumption). CLI tools and UI features are consumers of these APIs, not alternative data paths; internal component coordination may use documented Drupal services when it does not bypass the public API contract.

### IV. Standards Compliance & Data Quality
Metadata must validate against the configured metastore schema; tabular data imported into the datastore undergoes schema validation and type checking; harvest integrations validate remote metadata before import. Data quality issues MUST be surfaced to administrators with actionable remediation guidance. Breaking compliance with the active schema or quality gates requires explicit governance review.

### V. Extensibility Through Drupal
DKAN leverages Drupal's module system for extensibility. Custom vocabularies, field definitions, and harvest source types extend DKAN's behavior through standard Drupal patterns. Extensions MUST be independently installable/removable and documented. Core DKAN functionality remains independent of non-essential extensions.

### VI. Test Coverage & Documentation Excellence
Test coverage and documentation are non-negotiable. All code additions MUST include unit tests for business logic and integration tests for component interactions. Diff coverage on pull requests MUST exceed 50%; overall project coverage target is 90%. API endpoints, configuration schemas, and extension points MUST be documented with examples and use cases. Breaking changes MUST include migration guides. Undocumented or untested features are considered incomplete.

## Technology & Architecture Standards

DKAN is a Drupal-based system with three integrated components:
- **Metastore**: Drupal entities storing JSON metadata conforming to customizable schemas (commonly DCAT-US, but schema-agnostic)
- **Datastore**: Tabular data import, storage, and REST API
- **Harvest**: Metadata ingestion from external sources

All public data access uses documented REST/JSON APIs. Drupal provides the entity system, permissions, service container, and extension framework. Internal inter-component orchestration may use Drupal services, but it MUST preserve the public API contract. Code written in PHP (Drupal modules) and other languages (datastore API, harvest collectors) MUST conform to their respective ecosystem standards.

## Development Workflow & Quality Gates

1. **Code Review**: All changes reviewed against Constitution principles; compliance is a merge prerequisite.
2. **Testing**: Unit tests for business logic; integration tests for component interactions; API contract tests for datastore/harvest/metastore.
3. **Documentation**: API changes require updated API documentation; schema changes require migration guides; new extensions require README and usage examples.
4. **Backwards Compatibility**: DCAT-US metadata structure is a public contract; breaking changes require major version bump and migration period.

## Governance

This Constitution is the authoritative source for DKAN architecture and development practices. All pull requests, feature proposals, and architectural changes MUST verify compliance with these core principles before approval. When conflicts arise between principles, decisions are escalated to the maintainer team with documented rationale.

Terminology note: in constitutional and derivative project artifacts, "administrator" is the preferred term for users previously described as "operator".

**Version**: 1.2.1 | **Ratified**: 2026-05-26 | **Last Amended**: 2026-06-10
