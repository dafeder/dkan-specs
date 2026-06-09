# DKAN Architecture Overview

This document provides a high-level view of DKAN's architecture for feature specification and development planning. For detailed implementation guidance, refer to the main [DKAN repository documentation](../dkan/docs).

## System Overview

DKAN is an open data catalog implemented in Drupal. It manages metadata about datasets and makes data discoverable and accessible. The system consists of three core, independently-deployable components that communicate through REST/JSON APIs.

```
┌─────────────────────────────────────────────────────────┐
│                    DKAN System                           │
├─────────────────┬──────────────────┬────────────────────┤
│   Metastore     │    Datastore     │     Harvest        │
│   (metadata)    │   (data + API)   │  (data ingestion)  │
└─────────────────┴──────────────────┴────────────────────┘
         │                  │                  │
         └──────────────────┴──────────────────┘
           (REST/JSON APIs + Drupal Entity Layer)
```

---

## Component 1: Metastore

**Purpose**: Store and manage structured metadata about datasets and their distributions.

**How It Works**:
- Metadata is stored as JSON in Drupal entities, conforming to a customizable schema (commonly DCAT-US, but schema-agnostic)
- Each dataset is a Drupal entity with a JSON metadata field
- Supports CRUD operations on datasets and distributions

**Key Interfaces**:
- **REST API** (`/api/1/metastore/...`)
  - `GET /api/1/metastore/schemas/{schema_id}/items` – List items for a schema
  - `POST /api/1/metastore/schemas/dataset/items` – Create dataset
  - `GET /api/1/metastore/schemas/dataset/items/{identifier}` – Retrieve dataset
  - `PUT /api/1/metastore/schemas/dataset/items/{identifier}` – Replace dataset
  - `PATCH /api/1/metastore/schemas/dataset/items/{identifier}` – Modify dataset
  - `DELETE /api/1/metastore/schemas/dataset/items/{identifier}` – Delete dataset
- Similar operations exist for other schemas, including distributions

**Consumers**:
- Harvest system (reads remote metadata to decide what to import)
- Datastore (reads metadata to understand dataset schema)
- UI/frontend (displays dataset metadata)
- External systems via API

**Key Responsibilities**:
- Schema validation (against configured schema)
- Entity versioning and audit trails
- Access control and permissions
- Metadata search/filtering

---

## Component 2: Datastore

**Purpose**: Import, store, and provide query access to tabular data described in the metastore.

**How It Works**:
- Reads dataset/distribution metadata from metastore
- Imports tabular data (CSV, etc.) into database tables
- Creates a queryable REST API for data access
- Supports filtering, sorting, pagination

**Key Interfaces**:
- **Ingest API** (triggered from metastore or scheduled)
  - `GET /api/1/datastore/imports` – List datastore imports
  - `POST /api/1/datastore/imports` – Trigger import of a distribution
  - `GET /api/1/datastore/imports/{identifier}` – View datastore statistics
  - `DELETE /api/1/datastore/imports/{identifier}` – Delete a datastore
  - More commonly invoked from Drush commands than network API calls
- **Query API** (`/api/1/datastore/query`)
  - `POST /api/1/datastore/query` – Query one or more datastore resources using JSON
  - `GET /api/1/datastore/query` – GET equivalent of the JSON query shape for simple cases
  - `POST /api/1/datastore/query/{distributionId}` – Query a single datastore resource
  - `GET /api/1/datastore/query/{distributionId}` – GET equivalent for a single resource
  - `POST /api/1/datastore/query/{distributionId}/download` – Download query results as CSV
  - `GET /api/1/datastore/query/{distributionId}/download` – Download query results as CSV
  - `POST /api/1/datastore/query/{datasetId}/{index}` – Query a resource by dataset and distribution index
  - `GET /api/1/datastore/query/{datasetId}/{index}` – GET equivalent for dataset/index queries
  - `GET /api/1/datastore/sql` – SQL-like query interface for datastore resources

**Consumers**:
- End users querying data
- UI/frontend (data exploration)
- External applications

**Key Responsibilities**:
- Determining when data files need to be localized and imported from metadata changes
- Local caching (localization) of files
- Providing endpoints for all ingested files
- Error handling
- Keeping track of versions and "perspectives" for different resources
- Applying types and indexes when provided by data dictionaries

---

## Component 3: Harvest

**Purpose**: Automatically ingest metadata from remote sources and populate the metastore.

**How It Works**:
- Connects to external data catalogs (e.g., other DKAN instances, CKAN, data.json endpoints)
- Harvests remote metadata periodically or on-demand
- Maps remote metadata to local metastore schema
- Validates and stores in metastore via direct Drupal service calls
- Basic unit of information is the "harvest plan", which consists of a source URL and references to ETL handler classes

**Key Interfaces**:
- **Harvest Processor API**
  - Custom harvest source types (plugins) implement remote connection logic
  - Examples: CKAN harvester, DKAN-to-DKAN harvester, data.json harvester
- **Harvest Management API**
  - Drush commands like `drush dkan:harvest:register`, `drush dkan:harvest:run`, and `drush dkan:harvest:status`
  - `POST /api/1/harvest/plans` – Create harvest plan
  - `POST /api/1/harvest/runs` – Trigger harvest
  - `GET /api/1/harvest/plans/{plan_id}` – Get full plan info

**Consumers**:
- Metastore (writes harvested metadata)
- Administrators (manage harvest schedules)

**Key Responsibilities**:
- Remote source connectivity and error handling
- Metadata transformation to local schema
- Duplicate detection (same dataset harvested multiple times)
- Harvest logging and reporting
---

## Communication Interfaces

The primary interface for inter-component and external communication is the Drupal service container.

Components rarely talk to each other via the REST API.

**REST API Contract**:
- **Protocol**: HTTP REST (with JSON payloads)
- **Format**: JSON for all data exchange
- **Authentication**: Drupal permissions/token-based (component to component)
- **Error Handling**: HTTP status codes + JSON error body with actionable messages
- **Versioning**: APIs are versioned (e.g., `/api/1/`); breaking changes increment version

**Administrative & Operational Access**:
- Drupal CLI (Drush) commands provide administrative access for operations like harvest management, data import, and configuration
- Drush is not intended as a primary data interface but is valid for operational and maintenance tasks
- Note: Not all REST endpoints have Drush equivalents, and vice versa
- Common documented commands include `drush dkan:dataset-info`, `drush dkan:datastore:import`, `drush dkan:datastore:localize`, `drush dkan:datastore:drop`, `drush dkan:datastore:reimport`, `drush dkan:datastore:degraded-mode`, `drush dkan:harvest:list`, `drush dkan:harvest:info`, `drush dkan:harvest:run`, `drush dkan:harvest:revert`, `drush dkan:harvest:deregister`, `drush dkan:harvest:publish`, `drush dkan:metastore:publish`, and `drush dkan:metadata-form:sync`

**Example Flow**:
1. Create new dataset (via REST): `POST /api/1/metastore/schemas/dataset/items`
2. Metastore validates and stores; returns 201 Created with the dataset identifier
3. Datastore (via REST or Drush): imports data and makes it queryable via `GET /api/1/datastore/query/{distributionId}` or `GET /api/1/datastore/sql`
4. UI/end-users consume Metastore and Datastore REST APIs
5. Administrators may use Drush commands for operational tasks (e.g., `drush dkan:datastore:drop`)

---

## Drupal Foundation

DKAN runs on Drupal and leverages:
- **Entity System**: Dataset and distribution are Drupal entities
- **Permissions**: Access control via Drupal roles and permissions
- **Module System**: Features can be extended via Drupal modules
- **Field Storage**: Metastore metadata is stored in a single JSON field (not separate fields per property)

---

## Key Architectural Principles

(Derived from [DKAN Constitution](../constitution.md))

1. **Metadata-First**: All data operations are driven by metadata schema
2. **Modular Components**: Metastore, datastore, harvest can be deployed/upgraded independently
3. **API-Driven**: REST APIs are the primary interface for data access and inter-component communication; operational tasks may use Drupal CLI
4. **Schema Customization**: Metastore schema is configurable per deployment
5. **Standards Compatible**: Designed to work with DCAT-US and similar metadata standards

---

## For Specification Writers

When specifying features, consider:
- **Which component(s) does this affect?** (Metastore, Datastore, Harvest, or combinations?)
- **Does it introduce new API endpoints or change existing ones?**
- **What is the schema impact?** (Metastore adds/changes fields?)
- **Does it require inter-component communication changes?**
- **What data quality or validation rules apply?**

Use the component descriptions above to scope your feature and identify dependencies.
