# Spec: ontology

Module id: `ontology` (see `specs/CAPABILITY-MAP.md`). Depends on no other module.

## Objective

Define the Enterprise HR Knowledge Graph schema (classes, properties, modelling conventions) as a single Turtle file that serves as the contract for `data-pipeline` and `queries`.

**Direct users:** developers of `data-pipeline` (producing RDF that matches the schema) and of `queries` (writing SPARQL on top of the schema). **Indirect end users:** the HR team and managers who ask CQ1-CQ5.

**Design principle (YAGNI):** every class and property must trace to at least one competency question. Terms no CQ needs are left out.

**Success:** the schema is sufficient to *state* the data CQ1-CQ5 require, proven by an example fixture and tests. Answering the CQs (writing their SPARQL) is the job of the `queries` module.

### Competency questions (summary)

| ID | Question |
|---|---|
| CQ1 | Who reports directly or indirectly to a given manager? |
| CQ2 | Who has worked at company Y with cumulative experience ≥ N years, or in a given role? |
| CQ3 | What is the professional relation path between employee X and Y (reporting path, shared projects, overlapping work history)? |
| CQ4 | Whose work history overlaps in time with employee X at the same company? |
| CQ5 | Who are internal candidates for succession in key positions (managerial/project experience, skills, position in the reporting chain)? |

### Term inventory

**Classes (7)**

| Class | Description | CQ |
|---|---|---|
| `hr:Employee` | Employee | 1-5 |
| `hr:OrgUnit` | Department/organisational unit | — (see Decision 3) |
| `hr:Company` | Company. The organisation itself is one `Company` instance | 2-4 |
| `hr:Employment` | One period of work by an Employee at one Company | 2-5 |
| `hr:Position` | Position/title that can be marked key and requires skills | 5 |
| `hr:Project` | Project | 3, 5 |
| `hr:Skill` | Skill | 5 |

**Object properties (9)**

| Property | Domain → Range | Brief equivalent | CQ |
|---|---|---|---|
| `hr:reportsTo` | Employee → Employee (functional) | REPORTS_TO | 1, 3, 5 |
| `hr:belongsTo` | Employee → OrgUnit | BELONGS_TO | — |
| `hr:hasEmployment` | Employee → Employment | (part of WORKED_AT) | 2-5 |
| `hr:atCompany` | Employment → Company | (part of WORKED_AT) | 2-4 |
| `hr:hasPosition` | Employment → Position | — | 5 |
| `hr:collaboratedOn` | Employee → Project | COLLABORATED_ON | 3, 5 |
| `hr:hasLead` | Project → Employee | — | 5 |
| `hr:hasSkill` | Employee → Skill | HAS_SKILL | 5 |
| `hr:requiresSkill` | Position → Skill | — | 5 |

**Datatype properties (5)**

| Property | Domain → Range | CQ |
|---|---|---|
| `hr:jobTitle` | Employment → xsd:string | 2 |
| `hr:startDate` | Employment → xsd:date | 2, 3, 4 |
| `hr:endDate` | Employment → xsd:date (optional) | 2, 3, 4 |
| `hr:isManagerial` | Employment → xsd:boolean | 5 |
| `hr:isKeyPosition` | Position → xsd:boolean | 5 |

Every entity's name is recorded with `rdfs:label`.

### Modelling conventions

1. **Work history is reified.** The brief's `WORKED_AT` is not materialised as a direct edge. The path is `?emp hr:hasEmployment/hr:atCompany ?company`, so job title and dates can attach to the employment period.
2. **No `endDate` means still employed.** Queries must handle a missing `endDate` (for example `COALESCE(?end, NOW())`).
3. **Experience duration is derived, not stored.** "Years of experience" is computed from `startDate` and `endDate` in the query. There is no `duration` triple.
4. **Hierarchy goes through a property path.** `hr:reportsTo` stores only the direct manager (at most one per Employee). Indirect reporting uses `hr:reportsTo+`. No inverse is materialised.
5. **Current position** is the `Employment` at the organisation's own Company with no `endDate`, carrying an `hr:hasPosition`.
6. **Project lead** is stated with `hr:hasLead`, and the lead must also have `hr:collaboratedOn` on the same project (a data invariant tested in the fixture).
7. **The schema must work without a reasoner.** `domain`/`range` are documentation and test reference only, not a source of inference.

### Out of scope (YAGNI)

`OrgUnit` hierarchy (`parentUnit`), skill proficiency levels, project dates, salary/performance attributes, a `Person` class separate from `Employee`, SHACL, and a reasoner.

## Tech Stack

- Turtle (RDF 1.1), RDFS + minimal OWL vocabulary (`owl:Class`, `owl:ObjectProperty`, `owl:DatatypeProperty`, `owl:FunctionalProperty`)
- Python 3.11+, `rdflib>=7.0`, `pytest>=8.0` (versions pinned in `requirements.txt` at implementation time)
- Placeholder namespace: `https://example.org/hr#` (prefix `hr:`)

## Commands

```
Setup:          python -m venv .venv && source .venv/bin/activate && pip install -r requirements.txt
Parse check:    python -c "import rdflib; rdflib.Graph().parse('ontology/hr.ttl', format='turtle'); print('ok')"
Test:           pytest tests/ontology -v
```

There are no build, lint, or server steps in this module.

## Project Structure

Only ★ lines are created by this module.

```
specs/
  CAPABILITY-MAP.md        → Capability map (index of specs)
  SPEC-ontology.md      ★  → This document
requirements.txt        ★  → rdflib, pytest
ontology/
  hr.ttl                ★  → Schema (single source of truth for the schema)
  examples/
    sample.ttl          ★  → Hand-written, fictional fixture, ±40-60 triples
docs/
  ontology.md           ★  → CQ → term coverage table, modelling conventions
tests/
  ontology/             ★  → Tests for this module
# Created by other modules, not here:
# pipeline/, docker-compose.yml   → data-pipeline
# queries/*.rq, tests/queries/    → queries
```

## Code Style

Conventions: classes `PascalCase`; properties `camelCase` prefixed with a verb/relation word; booleans prefixed `is`. Every term requires `rdfs:label`, `rdfs:comment` (English), `rdfs:domain`, and `rdfs:range` (classes excepted). The brief's name for each term is recorded in `rdfs:comment`.

```turtle
@prefix hr:   <https://example.org/hr#> .
@prefix owl:  <http://www.w3.org/2002/07/owl#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix xsd:  <http://www.w3.org/2001/XMLSchema#> .

hr:Employment a owl:Class ;
    rdfs:label "Employment" ;
    rdfs:comment "One period of work by an Employee at a Company. Duration is derived from startDate/endDate and never stored." .

hr:reportsTo a owl:ObjectProperty, owl:FunctionalProperty ;
    rdfs:label "reports to" ;
    rdfs:comment "Direct manager of an Employee. Indirect reporting uses the path hr:reportsTo+. (Brief: REPORTS_TO)" ;
    rdfs:domain hr:Employee ;
    rdfs:range  hr:Employee .
```

Test style:

```python
def test_every_property_has_domain_and_range(ontology):
    props = set(ontology.subjects(RDF.type, OWL.ObjectProperty)) | set(
        ontology.subjects(RDF.type, OWL.DatatypeProperty)
    )
    missing = [p for p in props if (p, RDFS.domain, None) not in ontology
               or (p, RDFS.range, None) not in ontology]
    assert not missing, f"Missing domain/range: {missing}"
```

## Testing Strategy

Framework: `pytest` + `rdflib`, without Fuseki and without network. No code coverage target. Completeness is measured by the criteria below.

| Level | Location | What is tested |
|---|---|---|
| Syntax | `tests/ontology/test_ontology_valid.py` | `hr.ttl` and `sample.ttl` can be parsed |
| Schema completeness | `test_ontology_valid.py` | The 7 classes and 14 properties exist exactly as inventoried. Every term has a label and comment. Every property has domain and range |
| Fixture conformance | `tests/ontology/test_sample_conforms.py` | Every predicate and type in `sample.ttl` is declared in `hr.ttl`. The domain and range constraints are satisfied. At most one `reportsTo` per Employee. No `reportsTo` cycle. Every project lead is also a `collaboratedOn` participant on that project |
| CQ coverage | `tests/ontology/test_cq_coverage.py` | For each CQ1-CQ5, every term the inventory requires exists in `hr.ttl`. Every term is used at least once in `sample.ttl` (prevents orphan terms) |
| Structural patterns | `test_cq_coverage.py` | `sample.ttl` holds one example of every CQ pattern (checked with a simple `ASK`, not the full CQ query) |

Patterns that must exist in `sample.ttl`:

- CQ1: a reporting chain of at least 3 levels
- CQ2: an employee with an `Employment` at an external company, including dates and job title
- CQ3: two employees on the same project
- CQ4: two employees with time-overlapping `Employment` at the same Company, plus one `Employment` without `endDate`
- CQ5: one key `Position` with `requiresSkill`, one candidate with `hasSkill` and a managerial `Employment`, and one project with `hasLead`

## Boundaries

- **Always:**
  - Trace every new term to at least one CQ and update the coverage table in `docs/ontology.md`
  - Run `pytest tests/ontology` before committing
  - Give every term an `rdfs:label`, `rdfs:comment`, domain, and range
  - Use English identifiers
- **Ask first:**
  - Adding a class or property unrelated to CQ1-CQ5
  - Changing the `hr:` namespace IRI
  - Changing a modelling convention (reified `Employment`, optional `endDate`, derived experience)
  - Adding a dependency (for example `pyshacl`, `owlrl`)
- **Never:**
  - Commit real HR data, PII, or the Kaggle dataset to `ontology/examples/`
  - Store derived values (duration, indirect managers) as triples
  - Rename or remove a term already used by another module without updating that module
  - Delete a failing test without approval

## Success Criteria

1. `ontology/hr.ttl` parses without error under rdflib.
2. `hr.ttl` defines exactly 7 classes and 14 properties as inventoried, each with a label and comment. Every property has domain and range.
3. `ontology/examples/sample.ttl` parses, uses only declared terms, and satisfies the domain and range constraints and the invariants (functional `reportsTo`, no cycles, leads are also collaborators).
4. For each CQ1-CQ5, every required term exists in `hr.ttl` and every structural pattern exists in `sample.ttl`.
5. Every term in `hr.ttl` is used at least once in `sample.ttl`.
6. `pytest tests/ontology -v` passes entirely without Fuseki, without network, and without a reasoner.
7. `docs/ontology.md` holds the CQ → term table and modelling conventions consistent with `hr.ttl`.

## Decisions (approved 2026-11-18)

| # | Question | Decision |
|---|---|---|
| 1 | Namespace | The placeholder `https://example.org/hr#` is kept. If the organisation assigns a namespace, replace it once with a search-replace. |
| 2 | `Position` class | Approved, but provisional because it depends on the CQ5 definition. |
| 3 | `OrgUnit` / `belongsTo` | Kept although no CQ uses them: the map is approved and the source dataset almost certainly carries departments. Their CQ column stays "—". |
| 4 | `WORKED_AT` | Path `hr:hasEmployment/hr:atCompany` only, no shortcut edge. |
| 5 | Experience duration | Derived in the query. No duration triple. |
| 6 | CQ5 definition | Does not block this module. Finalisation is deferred to `SPEC-queries.md`; it only affects `Position`, `isManagerial`, and `hasLead`. |
