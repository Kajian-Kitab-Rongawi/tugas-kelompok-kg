# Capability Map: Enterprise HR Knowledge Graph

Status: **Disetujui** (review manusia selesai sebelum spec modul mana pun ditulis).

| Module id | Responsibility | Depends on |
|---|---|---|
| `ontology` | Skema RDFS/OWL ringan dalam Turtle: Employee, OrgUnit, Company, Employment, Position, Project, Skill + relasi. CQ1-CQ5 sebagai acceptance criteria. | — |
| `data-pipeline` | Loader CSV Kaggle (struktur organisasi), generator sintetis (Employment, Project, Skill), `docker compose` untuk Jena Fuseki, skrip load. | ontology |
| `queries` | Query CQ1-CQ5 sebagai file `.rq` bernama, test pytest terhadap Fuseki, saved query untuk UI. | data-pipeline |

Build order: ontology → data-pipeline → queries

Module id bersifat stabil dan tidak diganti selama inisiatif berjalan. Spec per modul disimpan di root proyek dengan nama `SPEC-<module-id>.md`. Peta ini adalah indeks dari spec yang ada.

## Keputusan lintas modul

- RDF + SPARQL 1.1, tanpa reasoner. Triple store: Apache Jena Fuseki (Docker, single node).
- Data hybrid: dataset Kaggle untuk struktur organisasi, generator sintetis untuk Employment, Project, dan Skill. Dataset Kaggle diunduh manual dan tidak di-commit.
- "Interactive explorer" dipenuhi oleh UI bawaan Fuseki atau YASGUI dengan saved query.
- Dokumen berbahasa Indonesia. Identifier ontology, kode, dan query berbahasa Inggris.
- Default ke komponen open-source tanpa lisensi.

## Ditunda (YAGNI) dan pemicunya

| Item | Pemicu untuk dibangun |
|---|---|
| `cv-extraction` | Ada kebutuhan nyata memasukkan CV tak terstruktur |
| Explorer UI custom | Pengguna HR tidak bisa memakai saved query |
| Modul `relationship-discovery` (shortest path generik) | Perlu path lintas banyak jenis relasi sekaligus |
| Modul `succession-planning` terpisah | Bobot skor perlu diubah oleh non-developer |
| Reasoner / inference | Performa property path bermasalah |
| SHACL | Data dari sumber nyata mulai masuk |
| Entity resolution lintas sumber | Ada lebih dari satu sumber data nyata |
| Access control / masking atribut sensitif | Data asli (gaji, penilaian) masuk |

## Pertanyaan terbuka lintas modul

1. Dataset Kaggle belum dipilih dan lisensinya belum dicek. Menghambat `SPEC-data-pipeline.md`.
2. Definisi "posisi kunci" dan "pengalaman manajerial" untuk CQ5 belum final. Placeholder dipakai di `SPEC-queries.md`.
