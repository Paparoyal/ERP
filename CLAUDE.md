# CLAUDE.md — Konteks Proyek ERP

Aturan kerja detail untuk AI agent (struktur, penamaan, aturan Odoo 19, boleh/tidak boleh) ada di **`AGENTS.md`** — baca dan ikuti.

## Deskripsi Proyek

Repo ini berisi modul custom Odoo 19 untuk ERP **TODO: Nama Perusahaan**.
Tujuan: **TODO: ringkasan singkat proses bisnis yang didukung (mis. penjualan, inventori, akuntansi, HR)**.

Edisi Odoo: **TODO: Community / Enterprise**
Prefix modul: **TODO: mis. `xyz_`**

### Daftar Modul

| Modul | Deskripsi | Depends | Status |
|---|---|---|---|
| TODO: `xyz_base` | TODO: konfigurasi dasar perusahaan | `base` | TODO |
| TODO: `xyz_sale_...` | TODO | `sale` | TODO |
| TODO: `xyz_stock_...` | TODO | `stock` | TODO |

## Tech Stack

- **Odoo 19** (framework & ORM)
- **Python** 3.10+ (sesuai requirement Odoo 19 — cek versi di server)
- **XML** — view, action, menu, data, QWeb report
- **OWL 2 / JavaScript (ES module)** — komponen frontend backend & portal
- **PostgreSQL** 13+ — database
- Opsional: Docker / docker-compose untuk lingkungan dev — **TODO: isi jika dipakai**

## Perintah Umum

Asumsi: source Odoo di `~/odoo`, config di `config/odoo.conf` (salin dari `odoo.conf.example`, jangan di-commit).

```bash
# Jalankan server
./odoo-bin -c config/odoo.conf --addons-path=~/odoo/addons,./addons,./third_party -d erp_dev

# Install modul baru
./odoo-bin -c config/odoo.conf -d erp_dev -i xyz_nama_modul --stop-after-init

# Update modul setelah ubah Python/XML/field
./odoo-bin -c config/odoo.conf -d erp_dev -u xyz_nama_modul --stop-after-init

# Mode developer (auto-reload Python & XML, debug QWeb/asset)
./odoo-bin -c config/odoo.conf -d erp_dev --dev=all     # atau --dev=reload,xml,qweb

# Jalankan test satu modul (DB test terpisah, buang setelahnya)
./odoo-bin -c config/odoo.conf -d erp_test -i xyz_nama_modul --test-enable --stop-after-init --log-level=test

# Jalankan test tertentu via tag
./odoo-bin -c config/odoo.conf -d erp_test -u xyz_nama_modul --test-tags /xyz_nama_modul:TestClassName.test_method --stop-after-init

# Shell interaktif ORM
./odoo-bin shell -c config/odoo.conf -d erp_dev
```

Catatan:
- Ubah **Python** (field/model baru) → restart + `-u`. Ubah **XML** → `-u` (atau `--dev=xml`). Ubah **JS/SCSS** → refresh dengan `?debug=assets`.
- Jangan jalankan perintah di atas terhadap DB produksi.

## Alur Git

1. **Jangan commit/push langsung ke `main`.** Buat branch dari `main` terbaru:
   - `feat/<modul>-<deskripsi>` — fitur baru, mis. `feat/xyz_sale_approval-multi-level`
   - `fix/<modul>-<deskripsi>` — perbaikan bug, mis. `fix/xyz_stock-qty-rounding`
   - (opsional) `refactor/...`, `docs/...`, `chore/...`
2. **Format commit** (gaya Odoo):
   ```
   [TAG] nama_modul: ringkasan singkat (≤ 50 karakter)

   Penjelasan kenapa perubahan dibuat (bukan hanya apa).
   Ref: TODO-tiket
   ```
   Tag umum:
   - `[ADD]` modul/fitur baru
   - `[FIX]` perbaikan bug
   - `[IMP]` peningkatan fitur yang ada
   - `[REF]` refactor tanpa ubah perilaku
   - `[REM]` hapus kode/fitur
   - `[MIG]` migrasi versi
   - `[I18N]` terjemahan
3. **Pull Request** ke `main`:
   - Deskripsi: tujuan, perubahan utama, cara uji, screenshot (untuk perubahan UI), hasil test.
   - Minimal **1 approval** reviewer sebelum merge; CI/test harus hijau.
   - Pakai *squash merge* atau *rebase merge* — **TODO: tetapkan kebijakan tim**.
   - Hapus branch setelah merge.

## Kontak / Referensi

- Dokumentasi Odoo 19 developer: https://www.odoo.com/documentation/19.0/developer.html
- PIC teknis: **TODO**
- Server dev/staging/produksi: **TODO** (jangan tulis kredensial di sini)
