# AGENTS.md

Enterprise HR Knowledge Graph (group assignment for a Knowledge Graph course). All documents, code, comments, and identifiers are in English.

## Sources of truth

| File | Contents |
|---|---|
| `specs/CAPABILITY-MAP.md` | Module map (approved). Index of specs. |
| `specs/SPEC-ontology.md` | RDF schema. Approved 2026-11-18. |
| `tasks/plan.md` | Implementation plan for the `ontology` module. |
| `tasks/todo.md` | Execution checklist for the `ontology` module. Local, not committed. |
| `docs/ai-usage-policy.md` | Rules for using generative AI on this repo. |

`specs/SPEC-data-pipeline.md` and `specs/SPEC-queries.md` do not exist yet. Do not write code for either module before its spec is written and approved. A new spec is written as `specs/SPEC-<module-id>.md`.

## Build order

`ontology` → `data-pipeline` → `queries`. Prove every success criterion of a module before starting the next.

## Commands

```sh
python -m venv .venv && source .venv/bin/activate && pip install -r requirements.txt
python -c "import rdflib; rdflib.Graph().parse('ontology/hr.ttl', format='turtle'); print('ok')"
pytest tests/ontology -v
```

There are no build, lint, or server steps in the `ontology` module.

## Project rules

- Trace every new term to at least one CQ1-CQ5; update the coverage table in `docs/ontology.md`.
- No reasoner, no SHACL, no network at test time.
- Never commit real HR data, PII, or the Kaggle dataset.
- Do not add a dependency without approval (see `Boundaries` in the spec).
- Change the spec first when a decision changes, then the code.

## Workflow (agent-skills)

| Phase | Skill |
|---|---|
| Change a requirement or spec | `spec-driven-development` |
| Break work down | `planning-and-task-breakdown` |
| Implement | `incremental-implementation`, `test-driven-development` |
| Something fails | `debugging-and-error-recovery` |
| Before committing | `code-review-and-quality`, `code-simplification` |
| Commit | `git-workflow-and-versioning` |
| Record a decision | `documentation-and-adrs` |

Rules that apply in every phase: state assumptions before writing, do not guess when unsure, prove with a test or a parse, never touch files outside the task's scope.

## Changing this file

- This file is a contract, not a notebook. Change it only when a decision changes (spec or ADR first) or the same mistake has happened twice.
- Facts that can be read from another file are linked, not copied.
- Every addition comes with one line deleted; the ceiling is one screen.
