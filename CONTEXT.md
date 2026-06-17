# Brotec Blacktop — Job Checklist App

A clean, mobile-first, dark-themed job checklist for the Brotec Blacktop crew. It runs as a
single self-contained HTML file (no server, no build step), saves progress locally, and is
hosted free on GitHub Pages so crew can open it on any phone.

- **Live URL:** https://maverickmchugh1.github.io/brotec-job-checklist/
- **Repo:** https://github.com/maverickmchugh1/brotec-job-checklist

---

## Files
- `index.html` — the app served by GitHub Pages (this is what crew open).
- `checklist.html` — the working copy; **byte-identical** to `index.html`. Edit this, then copy it over `index.html`. There is no build step.
- `manifest.json`, `icon-180.png`, `icon-512.png` — PWA / "Add to Home Screen" support (BB tile icon, standalone fullscreen).
- `Brotec Logo no BG.png` — header logo.
- `CONTEXT.md` — this file.

> Keep `index.html` and `checklist.html` in sync on every change (`cp checklist.html index.html`).

---

## What the app does

### Live checklist (read-only structurally)
- **Dashboard header**: BB logo on the left + a "JOB CHECKLIST" title, with a live overall
  progress bar (`checked / total` across **every** item in the active checklist). The bar fills
  as items get checked. By default the home screen shows the full checklist; using a template
  swaps in that template's subset (and the bar's total updates to match).
- 13 sections of equipment/material items (Fuel & Fluids, Equipment Prep, Crack Sealing,
  Sealcoating, Materials, General Supplies, Safety, Cleaning, Post-Sealing, Binder, Tools,
  Backup, Pre-Departure).
- Tap a row or its box to check it off; tap a section header to collapse/expand; per-section
  **Check All**; free-text notes per section; per-section progress badges.
- **Press & hold** a section to drag-reorder it; press & hold an item to reorder it within its
  section. Order persists locally (`S.sectionOrder`, `S.sections[id].order`).
- Job Name + Date header. Progress is saved to `localStorage` (key `brotec-v1`) and survives
  closing the browser.
- The day-of checklist is **only for checking off** — there is no structural editing here.
  All customization happens in the Template Builder.

### + Floating Action Button (bottom-right)
Tapping the **+** expands a speed-dial of labeled actions (and a tap-anywhere scrim closes it):
- 🔗 **Share Checklist** — copies a link to a fresh blank copy of the app.
- 📑 **Templates** — opens the template library / builder.
- 🗑 **Reset Checklist** — clears all checkboxes (keeps the structure).
- ❓ **How to Use** — a short crew-friendly guide.

### Job Templates
A template is a reusable **selection** of which sections/items belong to a job type
(e.g. "Large Commercial" vs "Small Residential"), plus any custom items/sections and renames.
- **Template Builder** (full-screen): scroll through all 13 sections, tap checkboxes to include
  items, **All/None** per section, **add custom items**, **add custom sections**, **rename any
  item or section**, **change a section's emoji** (tap the icon → picker), and **add/edit a
  subnote** under any item (e.g. "check blades / replace"), and **press & hold to drag-reorder**
  sections and items. Save as new or update an existing one.
- Section expand/collapse animates a JS pixel-height transition (smoother than CSS grid-fr on
  mobile); swipe `will-change` is applied only during an active drag to avoid layer bloat.
- Drag-to-reorder is a shared `makeSortable()` engine (pointer events, ~250ms long-press, FLIP
  sibling animation, edge auto-scroll, post-drop click-swallow), used on both surfaces.
- **Swipe-left to delete** (Apple-style) any item or section in the builder. Deleting a built-in
  (master) item/section is **permanent across the whole app** (the default checklist and all
  future templates) — a "↩ Restore N deleted defaults" button appears in the builder to undo all
  master deletions. Deleting a custom item/section just removes it.
- **Use** a template by tapping its row in the Templates list (loads that exact checklist for
  today's job; warns before clearing in-progress checks). Each row also has Edit / Share / Delete
  buttons.
- **Share** a template as a self-contained `#tpl=…` link — a crew member opens it on their own
  phone, confirms, and gets that exact checklist (including renames **and subnotes**). The link
  encodes the full structure, so renaming/deleting the source template later doesn't affect links
  already sent. Corrupted links fail gracefully with a toast; the hash is scrubbed after load so a
  reload never re-prompts.

### Data model (localStorage `brotec-v1`)
`{ jobInfo, sections{ [id]:{collapsed,notes,checked,customItems,renamed,deleted,noteOverrides,order} },
customSections, deletedSections, renamedSections, sectionIcons, sectionOrder, templates[],
masterDeletedItems[], masterDeletedSections[] }`.
- `sectionOrder` (section ids) and per-section `order` (item ids) drive display order via
  `applyOrder()`; templates carry `sectionOrder` + `itemOrder`, shared in links as keys `so`/`io`.
- `sectionIcons` / template `sectionIcons` — per-section emoji overrides (baseId → emoji), applied
  by `applyTemplate` and rendered via `sectionIcon(sec)`; custom sections store their icon on the
  section object.
- `masterDeletedItems` / `masterDeletedSections` — permanent catalog prunes; `visibleSections()`
  / `visibleItems()` filter them out everywhere. Cleared by the builder's "Restore defaults".
- Each template: `{ id, name, createdAt, selections, customItems, customSections, renames,
  sectionRenames, sectionIcons, notes }`. Applying a template rebuilds the live
  `sections`/`customSections`/etc. and restores renames + icons + note overrides. Sharing
  base64url-encodes the template (unicode-safe, keys `n/sel/ci/cs/rn/sn/nt/si`) into the URL hash.
  Note overrides per item live in
  `sections[id].noteOverrides` (override the baked-in `item.note`; empty string clears a note).

---

## Build history
1. Initial mobile web app + Brotec branding (gold `#FFCC00` on near-black, BB logo).
2. Mobile hardening: iOS safe-area insets, no zoom-on-tap (16px inputs), whole-row tap targets;
   published to GitHub Pages + PWA home-screen install.
3. Removed address/crew fields and the top progress bar; added section add/rename/delete.
4. Job Templates: checkbox builder, Use/Edit/Share, incoming-link handling.
5. Removed edit mode from the live checklist; moved full editing into the builder; replaced the
   footer with the + FAB speed-dial; deleted Print; fixed the builder scroll bug. (3-agent build.)
6. Dashboard home (logo + "JOB CHECKLIST" title + live progress bar); swipe-left-to-delete in the
   builder with permanent master-list prune + "Restore defaults"; add/edit item subnotes in the
   builder (carried through Use and share links). Verified in headless Chrome — found & fixed two
   re-render bugs (restore button after item delete; live-checklist propagation on builder close).
7. Changeable section emojis (builder icon picker, carried through Use/share); smoother section
   drop-down (JS pixel-height animation replacing grid-fr) and a builder-lag fix (transient swipe
   `will-change` instead of ~90 permanent compositor layers). Verified in headless Chrome.
8. Press-and-hold drag-to-reorder for sections and items on both the live checklist and the
   builder (shared `makeSortable()` with FLIP + auto-scroll); order persists locally and in
   templates/share links. Templates list: removed the ▶ play button — tapping a row now Uses the
   template (Edit/Share/Delete buttons kept). Verified in headless Chrome (simulated drags).

---

## The 3-Agent Build Process (used for the latest pass)
The latest round of changes (remove edit mode from the live checklist, move full editing into
the builder, replace the footer with the + FAB speed-dial, delete Print, and fix the builder
scroll bug) was executed with a directed three-role agent workflow, looping until green:

1. **Foreman (manager)** — read the live code and turned the approved plan into a precise,
   ordered work order: exact CSS/HTML/JS to remove and add, the FAB spec, the builder-row
   restructure, the data-model additions, and an explicit list of done-criteria.
2. **Builder** — implemented the work order in `checklist.html`, mirrored to `index.html`,
   and self-checked (JavaScriptCore syntax check + grep that removed ids/classes were gone +
   `diff -q` for byte-identical files).
3. **Checker** — drove the real app in headless Chrome via the DevTools Protocol (a pure-Python
   stdlib WebSocket client — no Node/Playwright available), exercising every function and
   capturing screenshots, then reported PASS/FAIL per done-criterion.

**The loop mattered:** the Checker caught that the builder still didn't scroll — `min-height:0`
alone wasn't enough because `.builder-body` is a flex column, so its section children shrank to
~18px slivers instead of overflowing. The Foreman directed the fix
(`.builder-body > * { flex-shrink: 0; }`), the Builder applied it, and the Checker re-verified
(scrollHeight 1037 > clientHeight 419, scrollTop 618, sections at natural 66px height, last
section reachable). Final: 9/9 criteria pass, zero console errors.

### How to verify locally (headless Chrome via CDP)
There's no Node/Playwright on this machine, so the Checker drives Chrome directly:
`/Applications/Google Chrome.app/Contents/MacOS/Google Chrome --headless=new --remote-debugging-port=9333 --user-data-dir=<tmp>`,
then a small pure-Python (`socket`/`base64`/`struct`/`json`) WebSocket client speaks the
DevTools Protocol (`Runtime.evaluate` with `replMode:true`, `Page.navigate`,
`Page.captureScreenshot`). Notes: stub `window.confirm`/`window.prompt` via
`Page.addScriptToEvaluateOnNewDocument` before navigating to test dialog paths without hanging
the JS thread; navigate `about:blank` → target URL to force a true fresh load (a hash-only
change won't re-run scripts). For a quick visual check, just `open index.html`.

### Deploy
Commit, then `git push origin main`; GitHub Pages rebuilds automatically (~30–60s). Verify the
live URL returns 200.
