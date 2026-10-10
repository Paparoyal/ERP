# AGENTS.md — Panduan AI Agent untuk Repo ERP (Odoo 19)

Panduan ini wajib diikuti oleh AI agent (Claude, Codex, Copilot, dll) yang bekerja di repo ini.
Konteks proyek dan perintah umum ada di `CLAUDE.md`.

## 1. Struktur Repo

```
ERP/
├── addons/                     # Semua modul custom perusahaan
│   └── <prefix>_<nama_modul>/
│       ├── __init__.py
│       ├── __manifest__.py
│       ├── models/
│       │   ├── __init__.py
│       │   └── <nama_model>.py
│       ├── views/
│       │   ├── <nama_model>_views.xml
│       │   └── menus.xml
│       ├── security/
│       │   ├── security.xml        # groups, record rules
│       │   └── ir.model.access.csv
│       ├── data/                   # data master, sequence, cron
│       ├── demo/                   # data demo (hanya untuk dev/test)
│       ├── report/                 # QWeb report
│       ├── wizard/                 # TransientModel
│       ├── controllers/
│       ├── static/
│       │   ├── description/icon.png
│       │   └── src/{js,xml,scss}/  # komponen OWL
│       ├── tests/
│       │   ├── __init__.py
│       │   └── test_<fitur>.py
│       └── i18n/id.po
├── third_party/                # Modul OCA / vendor (jangan diedit, lihat §5)
├── config/odoo.conf.example    # Contoh config, TANPA kredensial asli
├── requirements.txt
├── AGENTS.md
└── CLAUDE.md
```

Satu modul = satu tanggung jawab. Lebih baik beberapa modul kecil daripada satu modul raksasa.

## 2. Konvensi Penamaan

| Objek | Format | Contoh |
|---|---|---|
| Modul | `<prefix>_<area>_<fitur>` (snake_case) | `xyz_sale_approval` |
| Model baru | `<prefix>.<nama>` (titik, singular) | `xyz.sale.approval` |
| Class Python | PascalCase | `SaleApproval` |
| Inherit model | tetap `_inherit = 'sale.order'`, file `sale_order.py` | — |
| Field Many2one | akhiran `_id` | `partner_id` |
| Field One2many/Many2many | akhiran `_ids` | `line_ids`, `tag_ids` |
| Field custom pada model core | awali prefix | `x_` hindari; pakai `xyz_approval_state` |
| Method compute/inverse/search | `_compute_<field>`, `_inverse_<field>`, `_search_<field>` | `_compute_amount_total` |
| Method onchange / constraint | `_onchange_<field>`, `_check_<aturan>` | `_check_date_range` |
| Action tombol | `action_<verb>` | `action_approve` |
| XML ID view | `<model_underscore>_view_<tipe>` | `xyz_sale_approval_view_form` |
| XML ID action | `<model_underscore>_action` | `xyz_sale_approval_action` |
| XML ID menu | `<modul>_menu_<nama>` | `xyz_sale_approval_menu_root` |
| XML ID group | `group_<peran>` | `group_xyz_approval_manager` |
| XML ID record rule | `<model_underscore>_rule_<scope>` | `xyz_sale_approval_rule_company` |
| Access CSV id | `access_<model_underscore>_<group>` | `access_xyz_sale_approval_user` |

Prefix perusahaan: **TODO: tetapkan prefix (mis. `xyz_`)** — gunakan konsisten di seluruh modul.

## 3. Aturan Khusus Odoo 19

- **`<list>` bukan `<tree>`** untuk list view; `view_mode` pakai `list,form` (bukan `tree,form`).
- **Jangan pakai `attrs=` dan `states=`** (sudah dihapus sejak 17). Gunakan ekspresi Python langsung:
  `invisible="state != 'draft'"`, `readonly="..."`, `required="..."`, `column_invisible="..."`.
- Tidak perlu `<odoo><data>` kecuali butuh `noupdate="1"`; cukup `<odoo>`.
- **`@api.model_create_multi`** untuk override `create` (terima list of dict).
- Hindari `name_get()` (dihapus); override `_compute_display_name` dan `_rec_names_search`.
- `_sql_constraints` sudah deprecated di Odoo 19 → pakai `models.Constraint(...)` / `models.Index(...)` sebagai atribut class.
- `read_group` lama → gunakan `_read_group` / `formatted_read_group`.
- Pakai `self.env._("...")` untuk teks yang diterjemahkan (atau `_` dari `odoo` bila konteks env tidak ada).
- `group_operator=` → `aggregator=` pada field.
- Gunakan `Command.create/update/link/unlink/set/clear` untuk x2many, bukan tuple `(0, 0, {...})`.
- Frontend: **OWL 2** + ES module (`/** @odoo-module **/` tidak lagi wajib), daftarkan aset via `assets` di `__manifest__.py` (mis. `web.assets_backend`). Jangan pakai widget legacy `web.Widget`.
- `chatter`: cukup `<chatter/>` di form view (bukan `div.oe_chatter` manual).
- `version` manifest: `19.0.x.y.z`; `license` wajib (`LGPL-3` atau `OPL-1`).
- Selalu cek source Odoo 19 yang terpasang jika ragu — API bisa berubah antar versi; jangan mengandalkan contoh dari versi ≤16.

## 4. Testing

- Setiap fitur/fix wajib punya test di `tests/`, import di `tests/__init__.py`.
- Base class: `odoo.tests.TransactionCase` (logika), `HttpCase` (controller/tour), `Form` untuk simulasi UI.
- Tandai test: `@tagged('post_install', '-at_install')` untuk test yang butuh semua modul terpasang.
- Jalankan di database test terpisah (lihat perintah di `CLAUDE.md`), **jangan pernah di DB produksi**.
- Test harus lolos sebelum membuka PR; sertakan output ringkas di deskripsi PR.
- Lint: `ruff`/`flake8` untuk Python, `pylint-odoo` bila tersedia, `prettier`/`eslint` untuk JS.

## 5. Boleh & Tidak Boleh

**Boleh / wajib:**
- Kustomisasi lewat modul sendiri: `_inherit`, view inheritance (`xpath`), override method dengan `super()`.
- Tambahkan hak akses (`ir.model.access.csv`) untuk setiap model baru.
- Tulis migration script (`migrations/<versi>/pre-|post-migrate.py`) bila mengubah struktur data yang sudah ada.
- Gunakan ORM; SQL mentah hanya jika perlu performa, dengan parameter (`%s`), lalu `flush`/`invalidate` cache.
- Tanyakan ke user bila requirement bisnis tidak jelas.

**Tidak boleh:**
- ❌ **Mengubah core Odoo** (`odoo/`, `addons/` bawaan) maupun modul `third_party/` — selalu inherit di modul custom.
- ❌ **Commit kredensial**: password DB, `admin_passwd`, API key, token, file `.env`, `odoo.conf` asli, dump database.
- ❌ **Push langsung ke `main`** — selalu lewat branch + Pull Request.
- ❌ Force push ke branch bersama, rewrite history yang sudah di-review.
- ❌ `sudo()` tanpa alasan jelas (tulis komentar mengapa), `cr.commit()` di dalam method bisnis.
- ❌ Menjalankan `-u`/`-i`, menghapus data, atau drop DB pada server produksi/staging tanpa persetujuan eksplisit.
- ❌ Menghapus field/model tanpa migration plan.
- ❌ Hardcode ID record (`browse(5)`) — pakai `self.env.ref('modul.xml_id')`.
