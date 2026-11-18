# Capability Map: Enterprise HR Knowledge Graph

Status: **Approved** (human review completed before any module spec was written).

| Module id | Responsibility | Depends on |
|---|---|---|
| `ontology` | Lightweight RDFS/OWL schema in Turtle: Employee, OrgUnit, Company, Employment, Position, Project, Skill + relations. CQ1-CQ5 as acceptance criteria. | — |
| `data-pipeline` | Kaggle CSV loader (organisational structure), synthetic generator (Employment, Project, Skill), `docker compose` for Jena Fuseki, load scripts. | ontology |
| `queries` | CQ1-CQ5 as named `.rq` files, pytest tests against Fuseki, saved queries for the UI. | data-pipeline |

Build order: ontology → data-pipeline → queries

Module ids are stable and are not renamed while the initiative runs. Per-module specs live in `specs/` named `SPEC-<module-id>.md`. This map is the index of the specs that exist.

## Cross-module decisions

- RDF + SPARQL 1.1, no reasoner. Triple store: Apache Jena Fuseki (Docker, single node).
- Hybrid data: a Kaggle dataset for organisational structure, a synthetic generator for Employment, Project, and Skill. The Kaggle dataset is downloaded manually and is not committed.
- "Interactive explorer" is satisfied by Fuseki's built-in UI or YASGUI with saved queries.
- All repository documents, code, comments, and identifiers are in English.
- Default to open-source components that carry no licence requirement.

## Deferred (YAGNI) and their triggers

| Item | Trigger to build it |
|---|---|
| `cv-extraction` | A real need to ingest unstructured CVs |
| Custom explorer UI | HR users cannot use the saved queries |
| `relationship-discovery` module (generic shortest path) | Paths across many relation types are needed at once |
| Separate `succession-planning` module | Scoring weights need to be changed by a non-developer |
| Reasoner / inference | Property path performance becomes a problem |
| SHACL | Real-source data starts arriving |
| Cross-source entity resolution | There is more than one real data source |
| Access control / sensitive-attribute masking | Real data (salary, ratings) arrives |

## Cross-module open questions

1. The Kaggle dataset is not chosen yet and its licence is unchecked. Blocks `SPEC-data-pipeline.md`.
2. The definitions of "key position" and "managerial experience" for CQ5 are not final. Placeholders are used in `SPEC-queries.md`.
