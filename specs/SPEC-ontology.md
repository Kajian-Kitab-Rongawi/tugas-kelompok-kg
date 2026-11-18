# Spec: ontology

Module id: `ontology` (lihat `specs/CAPABILITY-MAP.md`). Tidak bergantung pada modul lain.

## Objective

Mendefinisikan skema Enterprise HR Knowledge Graph (kelas, properti, konvensi pemodelan) sebagai satu file Turtle yang menjadi kontrak bagi `data-pipeline` dan `queries`.

**Pengguna langsung:** developer `data-pipeline` (menghasilkan RDF yang sesuai skema) dan `queries` (menulis SPARQL di atas skema). **Pengguna akhir tidak langsung:** tim HR dan manager yang mengajukan CQ1-CQ5.

**Prinsip desain (YAGNI):** setiap kelas dan properti harus dapat ditelusuri ke minimal satu competency question. Istilah yang tidak dibutuhkan CQ tidak masuk.

**Keberhasilan:** skema cukup untuk *menyatakan* data yang dibutuhkan CQ1-CQ5, dibuktikan oleh fixture contoh dan test. Menjawab CQ (query SPARQL-nya) adalah tugas modul `queries`.

### Competency questions (ringkas)

| ID | Pertanyaan |
|---|---|
| CQ1 | Siapa yang melapor langsung/tidak langsung kepada manager tertentu? |
| CQ2 | Siapa yang pernah bekerja di perusahaan Y dengan akumulasi pengalaman ≥ N tahun atau pada peran tertentu? |
| CQ3 | Bagaimana jalur relasi profesional antara employee X dan Y (jalur pelaporan, proyek yang sama, irisan riwayat kerja)? |
| CQ4 | Siapa yang riwayat kerjanya beririsan waktu dengan employee X di perusahaan yang sama? |
| CQ5 | Siapa kandidat internal untuk succession pada posisi kunci (pengalaman manajerial/proyek, skill, posisi dalam rantai pelaporan)? |

### Inventaris istilah

**Kelas (7)**

| Kelas | Keterangan | CQ |
|---|---|---|
| `hr:Employee` | Karyawan | 1-5 |
| `hr:OrgUnit` | Departemen/unit organisasi | — (lihat Open Question 3) |
| `hr:Company` | Perusahaan. Organisasi sendiri adalah satu instance `Company` | 2-4 |
| `hr:Employment` | Satu periode kerja seorang Employee di satu Company | 2-5 |
| `hr:Position` | Posisi/jabatan yang dapat ditandai kunci dan mensyaratkan skill | 5 |
| `hr:Project` | Proyek | 3, 5 |
| `hr:Skill` | Keahlian | 5 |

**Object property (9)**

| Properti | Domain → Range | Padanan di brief | CQ |
|---|---|---|---|
| `hr:reportsTo` | Employee → Employee (functional) | REPORTS_TO | 1, 3, 5 |
| `hr:belongsTo` | Employee → OrgUnit | BELONGS_TO | — |
| `hr:hasEmployment` | Employee → Employment | (bagian dari WORKED_AT) | 2-5 |
| `hr:atCompany` | Employment → Company | (bagian dari WORKED_AT) | 2-4 |
| `hr:hasPosition` | Employment → Position | — | 5 |
| `hr:collaboratedOn` | Employee → Project | COLLABORATED_ON | 3, 5 |
| `hr:hasLead` | Project → Employee | — | 5 |
| `hr:hasSkill` | Employee → Skill | HAS_SKILL | 5 |
| `hr:requiresSkill` | Position → Skill | — | 5 |

**Datatype property (5)**

| Properti | Domain → Range | CQ |
|---|---|---|
| `hr:jobTitle` | Employment → xsd:string | 2 |
| `hr:startDate` | Employment → xsd:date | 2, 3, 4 |
| `hr:endDate` | Employment → xsd:date (opsional) | 2, 3, 4 |
| `hr:isManagerial` | Employment → xsd:boolean | 5 |
| `hr:isKeyPosition` | Position → xsd:boolean | 5 |

Nama (label) semua entitas memakai `rdfs:label`.

### Konvensi pemodelan

1. **Riwayat kerja direifikasi.** `WORKED_AT` di brief tidak diwujudkan sebagai edge langsung. Jalurnya adalah `?emp hr:hasEmployment/hr:atCompany ?company`, sehingga jabatan dan tanggal bisa melekat pada periode kerja.
2. **`endDate` tidak ada = masih bekerja.** Query harus menangani ketiadaan `endDate` (misalnya `COALESCE(?end, NOW())`).
3. **Lama pengalaman diturunkan, bukan disimpan.** "Lama pengalaman" dihitung dari `startDate` dan `endDate` di query. Tidak ada triple `duration`.
4. **Hierarki lewat property path.** `hr:reportsTo` hanya menyimpan atasan langsung (maksimal satu per Employee). Pelaporan tidak langsung memakai `hr:reportsTo+`. Tidak ada inverse yang dimaterialisasi.
5. **Posisi saat ini** adalah `Employment` di Company organisasi sendiri tanpa `endDate`, dengan `hr:hasPosition`.
6. **Project lead** dinyatakan dengan `hr:hasLead`, dan lead juga harus memiliki `hr:collaboratedOn` pada proyek yang sama (invarian data yang diuji di fixture).
7. **Skema harus bisa dipakai tanpa reasoner.** `domain`/`range` hanya dokumentasi dan acuan test, bukan sumber inferensi.

### Di luar scope (YAGNI)

Hierarki `OrgUnit` (`parentUnit`), tingkat penguasaan skill, tanggal proyek, atribut gaji/kinerja, kelas `Person` terpisah dari `Employee`, SHACL, dan reasoner.

## Tech Stack

- Turtle (RDF 1.1), kosakata RDFS + OWL minimal (`owl:Class`, `owl:ObjectProperty`, `owl:DatatypeProperty`, `owl:FunctionalProperty`)
- Python 3.11+, `rdflib>=7.0`, `pytest>=8.0` (versi dipatok di `requirements.txt` saat implementasi)
- Namespace placeholder: `https://example.org/hr#` (prefix `hr:`)

## Commands

```
Setup:          python -m venv .venv && source .venv/bin/activate && pip install -r requirements.txt
Parse check:    python -c "import rdflib; rdflib.Graph().parse('ontology/hr.ttl', format='turtle'); print('ok')"
Test:           pytest tests/ontology -v
```

Tidak ada langkah build, linter, atau server di modul ini.

## Project Structure

Hanya baris bertanda ★ yang dibuat oleh modul ini.

```
specs/
  CAPABILITY-MAP.md        → Peta kapabilitas (indeks spec)
  SPEC-ontology.md      ★  → Dokumen ini
requirements.txt        ★  → rdflib, pytest
ontology/
  hr.ttl                ★  → Skema (satu-satunya sumber kebenaran skema)
  examples/
    sample.ttl          ★  → Fixture tulis tangan, fiktif, ±40-60 triple
docs/
  ontology.md           ★  → Tabel cakupan CQ → istilah, konvensi pemodelan
tests/
  ontology/             ★  → Test modul ini
# Dibuat modul lain, bukan di sini:
# pipeline/, docker-compose.yml   → data-pipeline
# queries/*.rq, tests/queries/    → queries
```

## Code Style

Konvensi: kelas `PascalCase`, properti `camelCase` berawalan kata kerja/relasi, boolean berawalan `is`. Setiap istilah wajib punya `rdfs:label`, `rdfs:comment` (bahasa Inggris), `rdfs:domain`, dan `rdfs:range` (kecuali kelas). Padanan nama di brief ditulis di `rdfs:comment`.

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

Gaya test:

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

Framework: `pytest` + `rdflib`, tanpa Fuseki dan tanpa jaringan. Tidak ada target code coverage. Ukuran kelengkapan adalah kriteria di bawah.

| Level | Lokasi | Yang diuji |
|---|---|---|
| Sintaks | `tests/ontology/test_ontology_valid.py` | `hr.ttl` dan `sample.ttl` bisa di-parse |
| Kelengkapan skema | `test_ontology_valid.py` | 7 kelas dan 14 properti ada persis seperti inventaris. Semua punya label dan comment. Semua properti punya domain dan range |
| Kesesuaian fixture | `tests/ontology/test_sample_conforms.py` | Semua predikat dan tipe di `sample.ttl` dideklarasikan di `hr.ttl`. Domain/range terpenuhi. `reportsTo` maksimal satu per Employee. Tidak ada siklus `reportsTo`. Setiap lead proyek juga `collaboratedOn` |
| Cakupan CQ | `tests/ontology/test_cq_coverage.py` | Untuk setiap CQ1-CQ5, semua istilah yang dibutuhkan (tabel inventaris) ada di `hr.ttl`. Setiap istilah dipakai minimal satu kali di `sample.ttl` (mencegah istilah yatim) |
| Pola struktural | `test_cq_coverage.py` | `sample.ttl` memuat satu contoh tiap pola CQ (cek dengan `ASK` sederhana, bukan query CQ lengkap) |

Pola yang wajib ada di `sample.ttl`:

- CQ1: rantai pelaporan minimal 3 tingkat
- CQ2: employee dengan `Employment` di perusahaan eksternal berikut tanggal dan jabatan
- CQ3: dua employee di proyek yang sama
- CQ4: dua employee dengan `Employment` beririsan waktu di Company yang sama, plus satu `Employment` tanpa `endDate`
- CQ5: satu `Position` kunci dengan `requiresSkill`, satu kandidat dengan `hasSkill` dan `Employment` `isManagerial`, serta satu proyek dengan `hasLead`

## Boundaries

- **Always:**
  - Telusuri setiap istilah baru ke minimal satu CQ dan perbarui tabel cakupan di `docs/ontology.md`
  - Jalankan `pytest tests/ontology` sebelum commit
  - Beri `rdfs:label`, `rdfs:comment`, domain, dan range pada setiap istilah
  - Gunakan identifier berbahasa Inggris
- **Ask first:**
  - Menambah kelas atau properti yang tidak terkait CQ1-CQ5
  - Mengubah IRI namespace `hr:`
  - Mengubah konvensi pemodelan (reifikasi `Employment`, `endDate` opsional, lama pengalaman diturunkan)
  - Menambah dependensi (misalnya `pyshacl`, `owlrl`)
- **Never:**
  - Commit data HR asli, PII, atau data Kaggle ke `ontology/examples/`
  - Menyimpan nilai turunan (durasi, atasan tidak langsung) sebagai triple
  - Mengubah nama atau menghapus istilah yang sudah dipakai modul lain tanpa memperbarui modul tersebut
  - Menghapus test yang gagal tanpa persetujuan

## Success Criteria

1. `ontology/hr.ttl` ter-parse tanpa error oleh rdflib.
2. `hr.ttl` mendefinisikan tepat 7 kelas dan 14 properti sesuai inventaris, masing-masing dengan label dan comment. Setiap properti memiliki domain dan range.
3. `ontology/examples/sample.ttl` ter-parse, hanya memakai istilah yang dideklarasikan, dan memenuhi domain, range, serta invarian (functional `reportsTo`, tanpa siklus, lead juga collaborator).
4. Untuk setiap CQ1-CQ5, seluruh istilah yang dibutuhkan ada di `hr.ttl` dan seluruh pola struktural ada di `sample.ttl`.
5. Setiap istilah di `hr.ttl` dipakai minimal sekali di `sample.ttl`.
6. `pytest tests/ontology -v` lulus seluruhnya tanpa Fuseki, tanpa jaringan, dan tanpa reasoner.
7. `docs/ontology.md` memuat tabel CQ → istilah dan konvensi pemodelan yang konsisten dengan `hr.ttl`.

## Keputusan (disetujui 2026-11-18)

| # | Pertanyaan | Keputusan |
|---|---|---|
| 1 | Namespace | Placeholder `https://example.org/hr#` dipertahankan. Bila organisasi menetapkan namespace, ganti sekali dengan search-replace. |
| 2 | Kelas `Position` | Disetujui, berstatus sementara karena bergantung pada definisi CQ5. |
| 3 | `OrgUnit` / `belongsTo` | Dipertahankan walau tidak dipakai CQ mana pun: peta sudah disetujui dan dataset sumber hampir pasti memuat departemen. Kolom CQ-nya tetap "—". |
| 4 | `WORKED_AT` | Jalur `hr:hasEmployment/hr:atCompany` saja, tanpa edge pintasan. |
| 5 | Lama pengalaman | Diturunkan di query. Tidak ada triple durasi. |
| 6 | Definisi CQ5 | Tidak menghambat modul ini. Finalisasi ditunda ke `SPEC-queries.md`, dampaknya hanya pada `Position`, `isManagerial`, dan `hasLead`. |
