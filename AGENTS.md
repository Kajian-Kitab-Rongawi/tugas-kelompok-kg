# AGENTS.md

Enterprise HR Knowledge Graph (tugas kelompok KG). Dokumen berbahasa Indonesia; identifier ontologi, kode, dan query berbahasa Inggris.

## Sumber kebenaran

| File | Isi |
|---|---|
| `specs/CAPABILITY-MAP.md` | Peta modul (disetujui). Indeks spec. |
| `specs/SPEC-ontology.md` | Skema RDF. Disetujui 2026-11-18. |
| `tasks/plan.md` | Rencana implementasi modul `ontology`. |
| `tasks/todo.md` | Checklist eksekusi modul `ontology`. Lokal, tidak di-commit. |

`specs/SPEC-data-pipeline.md` dan `specs/SPEC-queries.md` belum ada. Jangan menulis kode kedua modul itu sebelum spec-nya ditulis dan disetujui. Spec baru ditulis sebagai `specs/SPEC-<module-id>.md`.

## Build order

`ontology` → `data-pipeline` → `queries`. Selesaikan satu modul sampai seluruh success criteria-nya terbukti sebelum mulai modul berikutnya.

## Commands

```sh
python -m venv .venv && source .venv/bin/activate && pip install -r requirements.txt
python -c "import rdflib; rdflib.Graph().parse('ontology/hr.ttl', format='turtle'); print('ok')"
pytest tests/ontology -v
```

Tidak ada build, linter, atau server untuk modul `ontology`.

## Aturan proyek

- Istilah baru wajib bisa ditelusuri ke minimal satu CQ1-CQ5; perbarui tabel cakupan di `docs/ontology.md`.
- Tanpa reasoner, tanpa SHACL, tanpa jaringan saat test.
- Jangan commit data HR asli, PII, atau dataset Kaggle.
- Jangan menambah dependensi tanpa persetujuan (lihat `Boundaries` di spec).
- Ubah spec lebih dulu bila keputusan berubah, baru kode.

## Alur kerja (agent-skills)

| Fase | Skill |
|---|---|
| Ubah requirement atau spec | `spec-driven-development` |
| Pecah pekerjaan | `planning-and-task-breakdown` |
| Implementasi | `incremental-implementation`, `test-driven-development` |
| Ada yang gagal | `debugging-and-error-recovery` |
| Sebelum commit | `code-review-and-quality`, `code-simplification` |
| Commit | `git-workflow-and-versioning` |
| Catat keputusan | `documentation-and-adrs` |

Aturan yang berlaku di semua fase: sebutkan asumsi sebelum menulis, jangan menebak saat ragu, buktikan dengan test atau parse, jangan menyentuh file di luar scope.

## Mengubah file ini

- File ini kontrak, bukan catatan. Ubah hanya bila sebuah keputusan berubah (spec atau ADR lebih dulu) atau kesalahan yang sama sudah terjadi dua kali.
- Fakta yang bisa dibaca dari file lain ditautkan, bukan disalin.
- Setiap penambahan menyertakan satu baris yang dihapus; batas panjangnya satu layar.
