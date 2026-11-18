# Plan: ontology

Spec: `SPEC-ontology.md` (disetujui 2026-11-18). Peta: `CAPABILITY-MAP.md`.
Scope: 8 file baru di modul `ontology`. Tanpa Fuseki dan tanpa jaringan saat test.

## Kondisi awal

- `python3` 3.15.2, memenuhi `>=3.11`.
- `rdflib` dan `pytest` belum terpasang, jadi T1 butuh jaringan satu kali. Ini satu-satunya langkah yang bisa gagal karena sebab di luar repo.
- Belum ada file implementasi; baru spec dan peta.

## Urutan

| # | Task | File | Verifikasi |
|---|---|---|---|
| T1 | Setup | `requirements.txt` | `python -c "import rdflib; print(rdflib.__version__)"` |
| T2 | Skema | `ontology/hr.ttl` | perintah parse check di spec |
| T3 | Fixture | `ontology/examples/sample.ttl` | parse check |
| T4 | Test | `tests/ontology/conftest.py`, `test_ontology_valid.py`, `test_sample_conforms.py`, `test_cq_coverage.py` | `pytest tests/ontology -v` |
| T5 | Dokumentasi | `docs/ontology.md` | tabel CQ sama dengan inventaris spec |
| T6 | Gate | — | pytest hijau, success criteria 1-7 dicentang satu per satu |

T2 tidak bergantung pada T1 untuk penulisan, tetapi parse check-nya butuh T1.

## Keputusan rencana

1. **Fixture ~70-80 triple, bukan 40-60.** Types (17) + label setiap resource (17) + edge (27) + literal (11) sudah sekitar 72. Angka 40-60 di spec adalah perkiraan, bukan success criterion, jadi teks spec tidak diubah.
2. **Tes disusun artifact-first, bukan red-green-first.** Ini asersi conformance atas data deklaratif, tidak ada perilaku yang didorong keluar. Pengecualian: daftar istilah per CQ di `test_cq_coverage.py` ditulis sebelum fixture, karena daftar itulah yang mencegah istilah yatim.
3. **Domain/range diperiksa manual** dengan menelusuri triple, karena konvensi 7 melarang reasoner.
4. **Fixture adalah satu-satunya contoh data.** Tidak ada data sintetis kedua; pola CQ5 yang belum final cukup diwakili sekali.

## Di luar scope

Kaggle, Fuseki, `.rq`, SHACL, reasoner, hierarki `OrgUnit`, tingkat penguasaan skill. Semua punya pemicu di `CAPABILITY-MAP.md`.

## Risiko

- `pip install` butuh jaringan; jika gagal, T2-T6 tetap bisa diverifikasi dengan parser lain tetapi bukan bukti yang diminta spec.
- Success criterion 5 (setiap istilah dipakai di fixture) paling rawan pada `belongsTo`/`OrgUnit` dan `hasLead`. `test_cq_coverage.py` menangkapnya.
