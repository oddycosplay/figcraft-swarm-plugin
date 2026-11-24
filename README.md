# FigCraft Swarm Plugin

Plugin **multi-agent swarm** untuk **Antigravity** dengan arsitektur **Product Manager Coordinator + 5 Specialist Sub-Agents** (*Hybrid Pola B*) untuk membangun UI secara paralel di **FigCraft Live Canvas**.

> **Catatan Pemisahan Plugin:** Repositori ini khusus berisi definisi **multi-agent swarm** (tanpa `mcp_config.json`). Untuk koneksi MCP ke aplikasi desktop FigCraft, pasang plugin pendamping [`antigravity-figcraft-plugin`](https://github.com/oddycosplay/antigravity-figcraft-plugin) dan jalankan FigCraft dengan `figcraft --control 7979`.

## Komponen

- **Skill (`figcraft-swarm-build`)**: SOP orkestrasi multi-agent end-to-end dengan **Product Manager** sebagai Coordinator dan 5 sub-agent spesialis di Swarm Board:
  - `Coordinator (Product Manager)` → `figcraft-pm`: menyusun *Product Brief*, mempartisi frame (`parentId`), mengorkestrasi Swarm Board, mengeksekusi payload `batch` ke `figcraft_figcraft-live`, dan melakukan *sign-off*.
  - `@ba` (`figcraft-ba`): Business Analyst (*read-only*; merumuskan tujuan bisnis, aktor, aturan bisnis, dan alur proses/data).
  - `@ux-designer` (`figcraft-ux-designer`): UX Designer (*read-only*; menyusun arsitektur informasi, *user flow*, dan hierarki *auto-layout* per *section*).
  - `@copywriter` (`figcraft-copywriter`): Copywriter & UX Writer (*read-only*; menyusun teks final per slot UI).
  - `@ui-designer` (`figcraft-ui-designer`): UI Designer (*design tokens* `#RRGGBB`, tipografi, spesifikasi visual, dan menyusun payload JSON `batch` per `parentId` untuk dieksekusi Coordinator).
  - `@qa` (`figcraft-qa`): Visual & Design QA Reviewer (*read-only*; mengaudit layer tree & *screenshot* terhadap seluruh spesifikasi).
- **Rules (`rules/AGENTS.md`)**: Aturan wajib anti-konflik untuk Live Canvas FigCraft (`parent` ID eksklusif, `select: false`, `ids` eksplisit, larangan `undo` global saat paralel, dan audit `@qa`).

## Prasyarat

1. Plugin MCP FigCraft ([`antigravity-figcraft-plugin`](https://github.com/oddycosplay/antigravity-figcraft-plugin)) sudah terpasang dan aktif (`figcraft_figcraft-live`).
2. Aplikasi desktop FigCraft berjalan dengan control port aktif:

```powershell
figcraft --control 7979
```

## Pemasangan

Clone repositori ini ke direktori plugin Antigravity sebagai `figcraft-swarm`:

```powershell
git clone https://github.com/oddycosplay/figcraft-swarm-plugin.git "$env:USERPROFILE\.gemini\config\plugins\figcraft-swarm"
```

## Struktur Direktori

```text
figcraft-swarm-plugin/
├── plugin.json
├── README.md
├── rules/
│   └── AGENTS.md
├── skills/
│   └── figcraft-swarm-build/
│       └── SKILL.md
└── agents/
    ├── figcraft-pm.md
    ├── figcraft-ba.md
    ├── figcraft-ux-designer.md
    ├── figcraft-copywriter.md
    ├── figcraft-ui-designer.md
    └── figcraft-qa.md
```
