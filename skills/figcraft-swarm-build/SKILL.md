---
name: figcraft-swarm-build
description: Orchestrate a Product Manager Coordinator with 5 specialist swarm sub-agents (ba, ux-designer, copywriter, ui-designer, qa) on a live FigCraft desktop canvas using Hybrid Pattern B over figcraft_figcraft-live. Use when building multi-section pages, dashboards, or multi-screen flows in FigCraft Live with a multi-agent team.
---

# FigCraft Swarm Build (Product Manager Coordinator + 5 Specialist Sub-Agents)

SOP orkestrasi multi-agent end-to-end di atas **Live Server FigCraft** (`figcraft --control 7979`) melalui MCP `figcraft_figcraft-live`:
- **Coordinator = Product Manager (`figcraft-pm`)**: memegang penuh kendali *Product Brief*, partisi kanvas (`parentId`), orkestrasi Swarm Board, eksekusi atomik MCP `figcraft_figcraft-live`, dan *final sign-off*.
- **5 Sub-Agent Spesialis (Roster Eksak di Swarm Board)**:
  1. `@ba` (`figcraft-ba`) — Business Analyst
  2. `@ux-designer` (`figcraft-ux-designer`) — UX Designer
  3. `@copywriter` (`figcraft-copywriter`) — Copywriter & UX Writer
  4. `@ui-designer` (`figcraft-ui-designer`) — UI Designer & penyusun payload JSON `batch`
  5. `@qa` (`figcraft-qa`) — Visual & Design QA Reviewer

> **Arsitektur Hybrid Swarm Antigravity:** Pada Swarm Board, akses eksekusi `call_mcp_tool` untuk `figcraft_figcraft-live` berada pada **Coordinator (Product Manager)**. Worker spesialis berjalan paralel di Swarm Board dengan nama roster eksak (`ba`, `ux-designer`, `copywriter`, `ui-designer`, `qa`), lalu `@ui-designer` menyusun payload JSON `batch` yang valid terhadap schema MCP untuk dieksekusi secara atomik oleh Product Manager Coordinator.

## Prasyarat

1. Aplikasi desktop FigCraft berjalan dengan control port aktif:
   ```powershell
   figcraft --control 7979
   ```
2. MCP server `figcraft_figcraft-live` aktif dan terhubung.
3. Sebelum memanggil tool MCP `figcraft_figcraft-live`, Coordinator membaca schema tool terkait bila belum ada di konteks (`inspect_document`, `create_node`, `batch`, `set_properties`, `get_node`, `screenshot`).

---

## Fase 0: Setup Swarm & Verifikasi Kanvas oleh Product Manager (Coordinator)

1. Tetapkan judul Swarm Board agar identitas PM tampil jelas:
   ```text
   swarm(action="title", title="Product Manager — FigCraft Swarm")
   ```
2. Panggil `inspect_document` (`{ "outline": true, "depth": 2 }`) pada `figcraft_figcraft-live`.
3. Catat `document` ID, `currentPage`, dan struktur layer yang sudah ada agar desain baru konsisten dan tidak menimpa karya sebelumnya.

---

## Fase 1: Analisis Bisnis (`@ba`) & Product Brief (Coordinator-PM)

1. Spawn worker **`ba`** (`swarm(action="add", name="ba", workspace="inherit", brief=...)`) untuk menyusun *Business Requirements Summary* (tujuan bisnis, aktor/peran, aturan bisnis, alur proses/data, dan gap tanpa asumsi).
2. Berdasarkan laporan `@ba` dan permintaan pengguna, **Product Manager (Coordinator)** memfinalisasi **Product Brief**: daftar *section* terurut, kebutuhan fungsional per *section*, dan *acceptance criteria*.

---

## Fase 2: UX, Copywriting & Partisi Kanvas (Paralel)

1. Spawn **`ux-designer`** dan **`copywriter`** secara paralel di Swarm Board (`workspace: "inherit"`):
   - **`@ux-designer`**: merancang arsitektur informasi, *user flow*, dan hierarki *auto-layout* per *section* yang siap dipartisi.
   - **`@copywriter`**: menyusun teks final (*headline*, *subheadline*, CTA, label navigasi, *microcopy*) per slot UI.
2. Di saat bersamaan (atau tepat setelah struktur *section* disepakati), **Product Manager (Coordinator)** mempartisi kanvas live dalam satu `batch`:
   - Buat root container frame dan frame kosong untuk tiap *section* (`"select": false`).
   - Catat **numeric `id` eksak** setiap frame partisi (`parentId`).

---

## Fase 3: Desain Visual & Payload `batch` (`@ui-designer`) → Eksekusi Atomik (Coordinator-PM)

1. Spawn **`ui-designer`** (`swarm(action="add", name="ui-designer", workspace="inherit", brief=...)`) — atau beberapa instance sharded per *section* bila halaman sangat besar — dengan membekali:
   - `document` ID dan daftar `parentId` eksklusif hasil partisi.
   - Output `@ux-designer` dan `@copywriter`.
   - Instruksi menyusun **Shared Design Token Sheet** (`#RRGGBB`, skala tipografi, *spacing*, *radius*, *stroke*) **serta** payload JSON `batch` lengkap (`document`, `label`, `onError: "abort"`, `commands` dengan `"select": false` dan `$ref` internal) berdasarkan schema `batch.json` & `create_node.json`.
2. Begitu `@ui-designer` memposting payload JSON ke `@coordinator`, **Product Manager (Coordinator)** mengeksekusinya secara atomik ke `figcraft_figcraft-live` (tanpa konflik *selection* karena setiap `node.create` memakai `select: false` dan `parentId` terisolasi).

---

## Fase 4: Audit QA (`@qa`) & Sign-Off (Coordinator-PM)

1. **Product Manager (Coordinator)** memanggil `inspect_document` / `get_node` dan `screenshot` (root container dan/atau per *section*).
2. Spawn **`qa`** (`swarm(action="add", name="qa", workspace="inherit", brief=...)`) dengan menyertakan ringkasan requirement, *acceptance criteria*, hasil `get_node`, dan path file PNG `screenshot` untuk diaudit (*read-only*).
3. Jika `@qa` melaporkan isu `Critical` / `Major`, Coordinator meminta `@ui-designer` memperbaiki spesifikasi/`batch` atau langsung mengeksekusi perbaikan terarah dengan `set_properties` menggunakan `ids` eksplisit, lalu memverifikasi ulang dengan `screenshot`.
4. Setelah status `@qa` **`PASS`**, **Product Manager (Coordinator)** melakukan *kill* seluruh worker (`ba`, `ux-designer`, `copywriter`, `ui-designer`, `qa`), menyimpan dokumen (`save_file`) bila diminta pengguna, dan menyampaikan laporan akhir.
