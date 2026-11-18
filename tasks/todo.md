# Todo: ontology

- [ ] T1 Setup
  - Acceptance: venv aktif, `rdflib` dan `pytest` terpasang, versi tercetak; blok Commands di `AGENTS.md` diganti penunjuk ke spec bila isinya sama
  - Verify: `python -c "import rdflib, pytest; print(rdflib.__version__, pytest.__version__)"`
  - Files: `requirements.txt`, `AGENTS.md`

- [ ] T2 Skema
  - Acceptance: 7 kelas, 9 object property, 5 datatype property; tiap istilah punya label dan comment; tiap properti punya domain dan range; `reportsTo` functional
  - Verify: `python -c "import rdflib; rdflib.Graph().parse('ontology/hr.ttl', format='turtle'); print('ok')"`
  - Files: `ontology/hr.ttl`

- [ ] T3 Fixture
  - Acceptance: memakai seluruh 7 kelas dan 14 properti; memuat pola CQ1-CQ5 (rantai 3 tingkat, employment eksternal bertanggal, proyek bersama, tanggal beririsan plus satu employment tanpa `endDate`, posisi kunci dengan `requiresSkill`, kandidat dengan `hasSkill` dan `isManagerial`, proyek dengan `hasLead`)
  - Verify: parse check pada `ontology/examples/sample.ttl`
  - Files: `ontology/examples/sample.ttl`

- [ ] T4 Test
  - Acceptance: tiga file test lulus; setiap kegagalan menyebut istilah atau triple yang salah
  - Verify: `pytest tests/ontology -v`
  - Files: `tests/ontology/conftest.py`, `test_ontology_valid.py`, `test_sample_conforms.py`, `test_cq_coverage.py`

- [ ] T5 Dokumentasi
  - Acceptance: `docs/ontology.md` memuat tabel CQ -> istilah dan 7 konvensi pemodelan, konsisten dengan `hr.ttl`
  - Verify: baca ulang berdampingan dengan `hr.ttl`
  - Files: `docs/ontology.md`

- [ ] T6 Gate
  - Acceptance: success criteria 1-7 di spec terbukti satu per satu
  - Verify: `pytest tests/ontology -v` hijau tanpa Fuseki, tanpa jaringan, tanpa reasoner
  - Files: -
