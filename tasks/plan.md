# Plan: ontology

Spec: `specs/SPEC-ontology.md` (approved 2026-11-18). Map: `specs/CAPABILITY-MAP.md`.
Scope: 8 new files in the `ontology` module. No Fuseki and no network at test time.

## Starting point

- `python3` 3.15.2, meeting `>=3.11`.
- `rdflib` and `pytest` are not installed, so T1 needs the network once. This is the only step that can fail for reasons outside the repo.
- No implementation files yet; only specs and the map.

## Order

| # | Task | Files | Verification |
|---|---|---|---|
| T1 | Setup | `requirements.txt` | `python -c "import rdflib; print(rdflib.__version__)"` |
| T2 | Schema | `ontology/hr.ttl` | the parse check command in the spec |
| T3 | Fixture | `ontology/examples/sample.ttl` | parse check |
| T4 | Tests | `tests/ontology/conftest.py`, `test_ontology_valid.py`, `test_sample_conforms.py`, `test_cq_coverage.py` | `pytest tests/ontology -v` |
| T5 | Documentation | `docs/ontology.md` | CQ table matches the spec inventory |
| T6 | Gate | — | pytest green, success criteria 1-7 ticked one by one |

T2 can be written before T1, but its parse check cannot run until T1 is done.

## Plan decisions

1. **Fixture of ~70-80 triples, not 40-60.** Types (17) + a label on every resource (17) + edges (27) + literals (11) already comes to about 72. The 40-60 figure in the spec is an estimate, not a success criterion, so the spec text is unchanged.
2. **Tests are written artifact-first, not red-green-first.** These are conformance assertions over declarative data; there is no behaviour to drive out. The exception: the per-CQ term list in `test_cq_coverage.py` is written before the fixture, because that list is what prevents orphan terms.
3. **Domain/range are checked manually** by walking triples, because convention 7 rules out a reasoner.
4. **The fixture is the only sample data.** There is no second synthetic dataset; the still-provisional CQ5 pattern is represented once.

## Out of scope

Kaggle, Fuseki, `.rq`, SHACL, reasoner, the `OrgUnit` hierarchy, skill proficiency levels. All have triggers in `specs/CAPABILITY-MAP.md`.

## Risks

- `pip install` needs the network; if it fails, T2-T6 can still be checked with another parser but that is not the evidence the spec asks for.
- Success criterion 5 (every term used in the fixture) is riskiest for `belongsTo`/`OrgUnit` and `hasLead`. `test_cq_coverage.py` catches it.
